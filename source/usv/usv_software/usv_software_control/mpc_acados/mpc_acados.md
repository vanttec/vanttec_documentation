# Installing acados for the MPC

> Author: Pedro Librado

This section tells you how to install acados and compile the `mpc_node` node of the `usv_control` package. The package is in the [vanttec_usv](https://github.com/vanttec/vanttec_usv) repository. Select the guide for your computer from the table in "Which guide to use".

```{contents}
:local:
:depth: 2
```

---

## Parts of the system

The MPC (Model Predictive Control) is a controller that solves an optimization problem in each cycle. It finds the motor commands that follow the path best during the next few seconds.

acados is the library that solves this optimization problem fast. It has two parts:

1. A C library: `libacados.so`, together with `hpipm`, `blasfeo`, and `qpOASES`. The C++ node links to this library.
2. A Python package: `acados_template`. It reads the boat model, which is written in Python, and generates C code for that model.

The full procedure has three steps on all computers:

1. Install acados (the C library and the Python package).
2. Run `asv_mpc.py`. This script generates the C code of the solver in the `c_generated_code_asv_ocp` folder.
3. Compile `usv_control` with `colcon`. CMake links `mpc_node.cpp`, the generated code, and the acados library into one executable.

## Which guide to use

All three computers use Ubuntu 22.04 and ROS 2 Humble. The Docker image of the laptop is also Ubuntu 22.04. The guides differ in where ROS runs and in the processor type.

| Computer | Where ROS runs | Processor | Guide |
|---|---|---|---|
| Laptop with the `vanttec-dev` container | Inside Docker | x86_64 | [Laptop with Docker](mpc_acados_docker.md) |
| Laptop with Ubuntu 22.04 and ROS installed directly | Directly on the laptop | x86_64 | [Ubuntu 22.04 and Jetson, without Docker](mpc_acados_native.md) |
| Jetson Orin Nano of the boat (JetPack 6) | Directly on the Jetson | ARM (`aarch64`) | [Ubuntu 22.04 and Jetson, without Docker](mpc_acados_native.md) |

The laptop with Ubuntu 22.04 and the Jetson use the same guide, because the commands are almost the same. The guide shows the few steps that are different on the Jetson.

## Differences between the computers

| Topic | Laptop with Docker | Laptop with Ubuntu 22.04 | Jetson |
|---|---|---|---|
| How you type the commands | `docker exec vanttec-dev ...` from the laptop | Directly in a terminal | Directly in a terminal on the Jetson |
| Permission to write in `/opt` | The container uses root | You use `sudo` | You use `sudo` |
| NumPy in the system Python | NumPy 2 (from the ZED base image) | NumPy 1 from apt, unless somebody installed NumPy 2 with pip | Unknown, depends on the Jetson setup |
| BLASFEO target | `X64_AUTOMATIC` | `X64_AUTOMATIC` | `ARMV8A_ARM_CORTEX_A57` |
| `t_renderer` build | `linux-amd64` | `linux-amd64` | `linux-arm64` |
| Parallel build jobs | All cores | All cores | 4 or fewer, because of the small memory |

BLASFEO is the linear algebra library that acados uses. It contains a separate optimized code path for each processor type. acados selects the BLASFEO target from the processor type. The `get_tera` function of acados also selects the correct build of `t_renderer`. You do not select these values manually.

On all three computers, the Python part of acados goes in a virtual environment in `/opt/acados/venv`. A virtual environment is a folder with its own Python packages. With this environment, the acados packages cannot change the NumPy of the system Python.

## Common rules

These rules apply to all three guides:

1. Use acados version v0.5.5. The generated code and `mpc_node.cpp` depend on this version.
2. Generate the C code on the computer that runs the node. The `c_generated_code_asv_ocp` folder contains compiled `.o` and `.so` files for one processor type only. Do not copy this folder from the laptop to the Jetson.
3. Run the acados scripts with `/opt/acados/venv/bin/python`, not with `python3`.
4. Do not install `acados_template` with `pip3` into the system Python. pip can change the system NumPy to version 2. With NumPy 2, the apt packages `cv_bridge`, SciPy, and matplotlib stop working, and other Python nodes can fail.

## Test status

Read this section before you start. Not all the guides were tested.

| Guide | Status |
|---|---|
| Laptop with Docker, option B (install in the running container) | Tested on 11 November 2026 on an x86_64 laptop. acados installed, the C code generated, `usv_control` compiled, and `mpc_node` started and created the solver. |
| Laptop with Docker, option A (rebuild the image) | Not tested. The `Dockerfile` layer has the same commands as option B, but nobody built the image with it. |
| Laptop with Ubuntu 22.04, without Docker | Not tested. The commands are the same as in the tested Docker procedure, but the system packages on your laptop can be different. |
| Jetson Orin Nano | Not tested on the Jetson. The ARM builds of `t_renderer` and CasADi exist for download. acados selects the ARM target of BLASFEO by itself. The build times and the memory advice are estimates. |

These limits apply to all the guides:

1. The test covered only the start of the node. Nobody tested the MPC with odometry, a path, and real motors after this installation.
2. Only acados v0.5.5 was tested. Newer versions, for example v0.6.0, can change the generated code and break `mpc_node.cpp`.
3. The guides use `apt-get`, `sudo`, and Ubuntu 22.04 package names. On a different Ubuntu version or a different Linux distribution, the package names and the Python version can be different.

If a guide fails on your computer, add the error and the repair to the troubleshooting table of that guide.

## Code changes that the MPC needs

The `vanttec_usv` repository already has these changes. If your copy of the repository is old, update it before you compile.

| File | Change |
|---|---|
| `Dockerfile` (`VantTec` root folder, laptop only) | A new layer installs acados in `/opt/acados`, together with the Python virtual environment. |
| `usv_control/CMakeLists.txt` | Enables `mpc_node`. Finds acados in `ACADOS_SOURCE_DIR`. If that variable is not set, it uses `/opt/acados`. Adds an RPATH to `/opt/acados/lib`. |
| `usv_control/src/mpc_node.cpp` | The call to `asv_dynamics_acados_reset` has five arguments, as acados 0.5 requires. |

An RPATH is a library path that the linker writes into the executable. Without an RPATH, the node cannot find `libacados.so` at startup. The linker flag `-Wl,--disable-new-dtags` makes the RPATH also apply to the libraries that acados uses (`hpipm`, `blasfeo`, `qpOASES`). With this flag, the node runs without `/opt/acados/lib` in `LD_LIBRARY_PATH`.

In acados 0.5, the generated reset function changed from `reset(capsule, qp_mem)` to `reset(capsule, qp_mem, numerical_values, solver_options, x_to_x0_bar)`. The node calls it with `(ocp_capsule, 1, 0, 0, 0)`. This call does the same work as the old call: it resets only the QP memory.

```{toctree}
:maxdepth: 1
:hidden:

mpc_acados_docker
mpc_acados_native
```
