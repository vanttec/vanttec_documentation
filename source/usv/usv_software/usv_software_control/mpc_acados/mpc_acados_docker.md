# Laptop with Docker

> Author: Pedro Librado

This guide tells you how to install acados and compile `mpc_node` on a laptop that runs ROS in the `vanttec-dev` Docker container. You do all the steps in the container, because ROS 2 Humble is installed in the container and not on the laptop. Read [Installing acados for the MPC](mpc_acados.md) first. That page explains the parts of the system and the test status of this guide.

```{contents}
:local:
:depth: 2
```

---

## Requirements

| Item | Expected value |
|---|---|
| Container | `vanttec-dev`, started with `docker compose` from the `VantTec` root folder |
| Workspace | The laptop folder `VantTec/ros2_ws`, mounted in the container at `/ws` |
| Internet | The laptop must have internet access to download acados and the Python packages. |

## Installed locations

All paths in this table are inside the container.

| Item | Path |
|---|---|
| acados source and build | `/opt/acados` |
| `.so` libraries | `/opt/acados/lib` |
| Python virtual environment with `acados_template` | `/opt/acados/venv` |
| `t_renderer` (changes the acados templates into C code) | `/opt/acados/bin/t_renderer` |
| Generated C code of the solver | `/ws/src/vanttec_usv/src/usv_control/scripts/acados/c_generated_code_asv_ocp` |

## Why Python uses a separate environment in Docker

The ZED base image of the container contains NumPy 2. The Ubuntu 22.04 apt packages, for example SciPy and matplotlib, were compiled for NumPy 1. They fail with NumPy 2, and `acados_template` needs both of them. The virtual environment in `/opt/acados/venv` uses NumPy 1 and does not change the system Python of the container.

## Step 1: install acados

Two options are available. If you can recreate the container, use option A. If you must keep the current container, use option B. Option B is the tested option.

### Option A: rebuild the image

The `Dockerfile` in the `VantTec` root folder has a layer that installs acados. Docker builds only this layer and the layers after it again.

Do not use option A while the container contains work that is not saved in `ros2_ws`. Option A deletes the current container, and all the software that you installed manually in it is lost. The `ros2_ws` folder is not lost, because the container mounts it from the laptop.

1. Go to the folder that contains `compose.yaml`.

   ```bash
   cd ~/VantTec
   ```

2. Build the image.

   ```bash
   docker compose build
   ```

3. Recreate the container with the new image.

   ```bash
   docker compose up -d
   ```

The new image also sets `ACADOS_SOURCE_DIR=/opt/acados`, and adds `/opt/acados/lib` to `LD_LIBRARY_PATH`.

### Option B: install in the running container

These commands do the same work as the layer in the `Dockerfile`, but in the current container. Run them in a terminal on the laptop, not inside the container. The build takes a few minutes.

The installation from option B exists only in the current container. If you delete or recreate `vanttec-dev`, do option B again or use option A.

```bash
docker exec -u root vanttec-dev bash -c '
set -e
apt-get update && apt-get install -y python3.10-venv
git clone --depth 1 --branch v0.5.5 --recurse-submodules --shallow-submodules https://github.com/acados/acados.git /opt/acados
cmake -S /opt/acados -B /opt/acados/build -DACADOS_WITH_QPOASES=ON
cmake --build /opt/acados/build -j"$(nproc)" --target install
python3 -m venv /opt/acados/venv
/opt/acados/venv/bin/pip install "numpy<2" -e /opt/acados/interfaces/acados_template
ACADOS_SOURCE_DIR=/opt/acados /opt/acados/venv/bin/python -c "from acados_template.utils import get_tera; get_tera(force_download=True)"
chmod -R a+rwX /opt/acados
'
```

Each line of the script does one task:

1. `apt-get install python3.10-venv` installs the module that creates virtual environments.
2. `git clone` downloads acados v0.5.5 and its submodules (`hpipm`, `blasfeo`, `qpOASES`).
3. `cmake -S ... -B ...` prepares the build with the qpOASES solver.
4. `cmake --build ... --target install` compiles the libraries and copies them to `/opt/acados/lib`.
5. `python3 -m venv` creates the virtual environment.
6. `pip install` installs `acados_template` and CasADi in the virtual environment, with a NumPy version lower than 2.
7. `get_tera(force_download=True)` downloads `t_renderer`. Without this line, acados stops and asks a download question in the terminal.
8. `chmod` lets the `dev` user read and write in `/opt/acados`.

### Make sure that the installation is correct

1. Run this command in a terminal on the laptop.

   ```bash
   docker exec vanttec-dev /opt/acados/venv/bin/python -c "import numpy, scipy, casadi, acados_template; print(numpy.__version__, scipy.__version__, casadi.__version__)"
   ```

2. Make sure that the command shows three versions, and that the NumPy version starts with `1.`.

## Step 2: generate the C code of the solver

Do this step the first time. Do it again each time that you change `asv_mpc.py` or `asv_dynamics.py`.

1. Run this command in a terminal on the laptop.

   ```bash
   docker exec -w /ws/src/vanttec_usv/src/usv_control/scripts/acados -e ACADOS_SOURCE_DIR=/opt/acados -e MPLBACKEND=Agg vanttec-dev /opt/acados/venv/bin/python asv_mpc.py
   ```

2. Make sure that the end of the output shows `Build completed`.
3. Make sure that the `c_generated_code_asv_ocp` folder and the `asv_ocp.json` file exist in `scripts/acados`.

The command has three options:

1. `-w` runs the script in the `scripts/acados` folder. The script generates the code in the current folder.
2. `-e ACADOS_SOURCE_DIR` tells `acados_template` where acados is.
3. `-e MPLBACKEND=Agg` stops matplotlib from opening a window.

The output can show a warning about `Gauss-Newton Hessian approximation with EXTERNAL cost type`. This warning is normal with the current solver configuration.

## Step 3: compile the package

1. Run this command in a terminal on the laptop, after step 2.

   ```bash
   docker exec -w /ws vanttec-dev bash -c 'source /opt/ros/humble/setup.bash && colcon build --packages-select usv_control'
   ```

2. Make sure that the output ends with `1 package finished`. Lines of type `warning` do not stop the build.

## Step 4: test the node

1. Run this command in a terminal on the laptop, after step 3.

   ```bash
   docker exec -w /ws vanttec-dev bash -c 'source install/setup.bash && timeout 4 ros2 run usv_control mpc_node'
   ```

2. Make sure that the output shows `OCP solver created successfully`. This line shows that the node loaded acados.

If no odometry and no path are published, the node shows `MPC IS DISABLED` and `Preparation phase returned status 4`. This is the expected result, because the MPC has no data. To test the controller with data, run the simulation or the boat.

## Troubleshooting

| Error | Cause | Repair |
|---|---|---|
| `ModuleNotFoundError: No module named 'acados_template'` | You used `python3` instead of the Python of the virtual environment. | Run the script with `/opt/acados/venv/bin/python`. |
| `numpy.core.multiarray failed to import` or `_ARRAY_API not found` | NumPy 2 is used together with the apt SciPy or matplotlib. | Use the virtual environment in `/opt/acados/venv`. Do not install with `pip3` into the system Python. |
| acados asks `Do you wish to set up Tera renderer automatically?` | `t_renderer` is missing. | Do the `get_tera(force_download=True)` line of step 1 again. |
| `acados_solver_asv_dynamics.h: No such file or directory` | The C code was not generated. | Do step 2. |
| `too few arguments to function 'asv_dynamics_acados_reset'` | The node code is for an old acados version. | Update `vanttec_usv`. The current `mpc_node.cpp` uses five arguments. |
| `error while loading shared libraries: libacados.so` | The node cannot find the libraries. | Make sure that `/opt/acados/lib` exists in the container. Compile again with the current `CMakeLists.txt`. |
| `Error response from daemon: No such container: vanttec-dev` | The container does not run. | Start it with `docker compose up -d` in the `VantTec` root folder. If you used option B, do option B again. |
