# Ubuntu 22.04 and Jetson, without Docker

> Author: Pedro Librado

This guide tells you how to install acados and compile `mpc_node` on a computer that runs ROS 2 Humble directly, without Docker. It applies to two computers: a laptop with Ubuntu 22.04, and the Jetson Orin Nano of the boat. Read [Installing acados for the MPC](mpc_acados.md) first. That page explains the parts of the system and the test status of this guide.

Nobody tested this guide yet, on the laptop or on the Jetson. The commands come from the tested Docker procedure. If a step fails, add the error and the repair to the troubleshooting table.

```{contents}
:local:
:depth: 2
```

---

## Requirements

| Item | Laptop | Jetson |
|---|---|---|
| System | Ubuntu 22.04, x86_64 | JetPack 6 (L4T R36), Ubuntu 22.04, `aarch64` |
| ROS | ROS 2 Humble, installed in `/opt/ros/humble` | ROS 2 Humble, installed in `/opt/ros/humble` |
| Workspace | `~/ros2_ws`, with `vanttec_usv` in `src/` | `~/ros2_ws`, with `vanttec_usv` in `src/` |
| Internet | Necessary to download acados and the Python packages | Necessary to download acados and the Python packages |
| Free disk space | Approximately 2 GB | Approximately 2 GB |

If your workspace is in a different folder, change `~/ros2_ws` in the commands.

## Installed locations

| Item | Path |
|---|---|
| acados source and build | `/opt/acados` |
| `.so` libraries | `/opt/acados/lib` |
| Python virtual environment with `acados_template` | `/opt/acados/venv` |
| `t_renderer` (changes the acados templates into C code) | `/opt/acados/bin/t_renderer` |
| Generated C code of the solver | `~/ros2_ws/src/vanttec_usv/src/usv_control/scripts/acados/c_generated_code_asv_ocp` |

## Step 1: make sure that the system is correct

Do these steps in a terminal on the computer that will run the node.

1. Show the Ubuntu version.

   ```bash
   lsb_release -rs
   ```

2. Make sure that the output is `22.04`.
3. Show the processor architecture.

   ```bash
   uname -m
   ```

4. Make sure that the output is `x86_64` on the laptop, or `aarch64` on the Jetson.
5. On the Jetson only, show the L4T version.

   ```bash
   cat /etc/nv_tegra_release
   ```

6. On the Jetson only, make sure that the output starts with `# R36`.

## Step 2: install the tools

1. Update the package lists.

   ```bash
   sudo apt-get update
   ```

2. Install the build tools and the Python `venv` module.

   ```bash
   sudo apt-get install -y git cmake build-essential python3.10-venv
   ```

The `python3.10-venv` package lets you create Python virtual environments.

## Step 3: download acados

1. Download acados v0.5.5 and its submodules (`hpipm`, `blasfeo`, `qpOASES`) into `/opt/acados`.

   ```bash
   sudo git clone --depth 1 --branch v0.5.5 --recurse-submodules --shallow-submodules https://github.com/acados/acados.git /opt/acados
   ```

2. Make your user the owner of the folder. The next steps then do not need `sudo`.

   ```bash
   sudo chown -R $USER:$USER /opt/acados
   ```

## Step 4: compile the C library

1. Prepare the build with the qpOASES solver.

   ```bash
   cmake -S /opt/acados -B /opt/acados/build -DACADOS_WITH_QPOASES=ON
   ```

2. Find the line that starts with `Target: BLASFEO is` in the output. Make sure that it shows `X64_AUTOMATIC` on the laptop, or `ARMV8A_ARM_CORTEX_A57` on the Jetson.
3. On the laptop, compile and install the library with all the cores.

   ```bash
   cmake --build /opt/acados/build -j"$(nproc)" --target install
   ```

4. On the Jetson, compile and install the library with 4 cores.

   ```bash
   cmake --build /opt/acados/build -j4 --target install
   ```

5. If the Jetson stops responding, or the compiler stops with `Killed`, do item 4 again with `-j2`.
6. Show the installed libraries.

   ```bash
   ls /opt/acados/lib
   ```

7. Make sure that the list has `libacados.so`, `libblasfeo.so`, `libhpipm.so`, and `libqpOASES_e.so`.

The Orin Nano has 6 cores and a small quantity of memory. On the Jetson, the build takes approximately 5 to 15 minutes. This time is an estimate.

## Step 5: install the Python package in a virtual environment

Do not install `acados_template` with `pip3` into the system Python. pip can change the system NumPy to version 2, and the apt packages `cv_bridge`, SciPy, and matplotlib then stop working.

1. Create the virtual environment.

   ```bash
   python3 -m venv /opt/acados/venv
   ```

2. Install `acados_template` in the virtual environment, with a NumPy version lower than 2.

   ```bash
   /opt/acados/venv/bin/pip install "numpy<2" -e /opt/acados/interfaces/acados_template
   ```

Step 2 also installs CasADi, SciPy, and matplotlib. PyPI has x86_64 and `aarch64` builds of these packages, so pip does not compile them.

## Step 6: download t_renderer

`t_renderer` is the program that changes the acados templates into C code. When `t_renderer` is missing, acados stops and asks a download question in the terminal. This step downloads it first.

1. Download `t_renderer` with the `get_tera` function of acados. The function selects the build for your processor.

   ```bash
   ACADOS_SOURCE_DIR=/opt/acados /opt/acados/venv/bin/python -c "from acados_template.utils import get_tera; print(get_tera(force_download=True))"
   ```

2. Make sure that the command shows `/opt/acados/bin/t_renderer`.

## Step 7: make sure that the installation is correct

1. Import the Python packages and show their versions.

   ```bash
   /opt/acados/venv/bin/python -c "import numpy, scipy, casadi, acados_template; print(numpy.__version__, scipy.__version__, casadi.__version__)"
   ```

2. Make sure that the command shows three versions, and that the NumPy version starts with `1.`.

## Step 8: set the environment variables

1. Add these two lines at the end of `~/.bashrc`:

   ```bash
   export ACADOS_SOURCE_DIR=/opt/acados
   export LD_LIBRARY_PATH=/opt/acados/lib:$LD_LIBRARY_PATH
   ```

2. Open a new terminal, or run `source ~/.bashrc`.

`ACADOS_SOURCE_DIR` tells `acados_template` and CMake where acados is. `LD_LIBRARY_PATH` tells the system where to find the `.so` libraries. The `mpc_node` executable already contains the path to `/opt/acados/lib`, so the second line is only a backup.

## Step 9: generate the C code of the solver

Do this step the first time. Do it again each time that you change `asv_mpc.py` or `asv_dynamics.py`.

Do not copy the `c_generated_code_asv_ocp` folder from a different computer. The folder contains compiled files for one processor type only.

1. Go to the `scripts/acados` folder. The script generates the code in the current folder.

   ```bash
   cd ~/ros2_ws/src/vanttec_usv/src/usv_control/scripts/acados
   ```

2. Run the script with the Python of the virtual environment.

   ```bash
   MPLBACKEND=Agg /opt/acados/venv/bin/python asv_mpc.py
   ```

3. Make sure that the end of the output shows `Build completed`.
4. Make sure that the folder `c_generated_code_asv_ocp` and the file `asv_ocp.json` exist.

`MPLBACKEND=Agg` stops matplotlib from opening a window. If you connect to the Jetson with SSH, this variable is necessary. The output can show a warning about `Gauss-Newton Hessian approximation with EXTERNAL cost type`. This warning is normal with the current solver configuration.

## Step 10: compile the package

1. Go to the workspace.

   ```bash
   cd ~/ros2_ws
   ```

2. Load the ROS 2 environment.

   ```bash
   source /opt/ros/humble/setup.bash
   ```

3. Compile the `usv_control` package.

   ```bash
   colcon build --packages-select usv_control
   ```

4. Make sure that the output ends with `1 package finished`. Lines of type `warning` do not stop the build.
5. Show the acados libraries that the executable uses.

   ```bash
   ldd install/usv_control/lib/usv_control/mpc_node | grep -E "acados|blasfeo|hpipm|qpOASES|not found"
   ```

6. Make sure that the four libraries point to `/opt/acados/lib`, and that no line shows `not found`.

## Step 11: test the node

Do these steps in the `~/ros2_ws` folder, after step 10.

1. Load the workspace environment.

   ```bash
   source install/setup.bash
   ```

2. Run the node for 4 seconds.

   ```bash
   timeout 4 ros2 run usv_control mpc_node
   ```

3. Make sure that the output shows `OCP solver created successfully`. This line shows that the node loaded acados.

If no odometry and no path are published, the node shows `MPC IS DISABLED` and `Preparation phase returned status 4`. This is the expected result, because the MPC has no data. To test the controller with data, start the localization and the path of the boat.

## Troubleshooting

| Error | Cause | Repair |
|---|---|---|
| `c++: fatal error: Killed signal terminated program` | The computer had no free memory during the build. This is more probable on the Jetson. | Compile again with `-j2` or `-j1`. With `colcon`, add `--parallel-workers 1`. |
| `E: Unable to locate package python3.10-venv` | The system is not Ubuntu 22.04, or the Python version is different. | Install the `venv` package for your Python version, for example `python3-venv`. |
| `ModuleNotFoundError: No module named 'acados_template'` | You used `python3` instead of the Python of the virtual environment. | Run the script with `/opt/acados/venv/bin/python`. |
| `numpy.core.multiarray failed to import` or `_ARRAY_API not found` | NumPy 2 is used together with apt packages. | Use the virtual environment in `/opt/acados/venv`. Do not install with `pip3` into the system Python. |
| acados asks `Do you wish to set up Tera renderer automatically?` | `t_renderer` is missing. | Do step 6 again. |
| `cannot execute binary file: Exec format error` | `t_renderer` or the generated code came from a computer with a different processor. | Delete `/opt/acados/bin/t_renderer` and the `c_generated_code_asv_ocp` folder. Do steps 6 and 9 again. |
| `acados_solver_asv_dynamics.h: No such file or directory` | The C code was not generated. | Do step 9. |
| `too few arguments to function 'asv_dynamics_acados_reset'` | Your copy of `mpc_node.cpp` is for an old acados version. | Update `vanttec_usv`. |
| `error while loading shared libraries: libacados.so` | The node cannot find the libraries. | Make sure that `/opt/acados/lib` exists. Do step 8 again, then compile again. |
