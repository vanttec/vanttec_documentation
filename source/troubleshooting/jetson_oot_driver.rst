Compiling and Loading an Out-of-Tree Kernel Driver on Jetson
=============================================================

Author:
 * Eduardo Hernández Valdez

Introduction
************
Jetson's L4T (Linux for Tegra) ships with a fixed set of kernel modules. If your hardware requires a driver not included (such as a USB CAN adapter using ``gs_usb``), you need to compile it yourself from the matching kernel source and load it manually. This guide walks through that process end to end.

The ``gs_usb`` driver is used as a running example. If you are targeting a different driver, substitute the relevant ``CONFIG_`` key, source path, and ``.ko`` filename where appropriate.

1 - Verify your L4T Version
****************************

Confirm the exact L4T release running on your Jetson:

.. code-block:: shell

    cat /etc/nv_tegra_release

Expected output:

.. code-block:: text

    # R36 (release), REVISION: 4.3, GCID: 38968081, BOARD: generic, EABI: aarch64, DATE: Wed Jan  8 01:49:37 UTC 2025
    # KERNEL_VARIANT: oot
    TARGET_USERSPACE_LIB_DIR=nvidia
    TARGET_USERSPACE_LIB_DIR_PATH=usr/lib/aarch64-linux-gnu/nvidia

Note the release (``R36``) and revision (``4.3``). You need these to download the correct kernel sources.

2 - Install Build Dependencies
*******************************

.. code-block:: shell

    sudo apt-get install build-essential bc kmod libssl-dev

3 - Download the Kernel Sources
*********************************

Navigate to the Jetson Linux archive and find the entry matching your L4T version:

.. code-block:: text

    https://developer.nvidia.com/embedded/jetson-linux-archive

Download the **Driver Package (BSP) Sources** (``public_sources.tbz2``) for your exact release.

4 - Extract the Sources
************************

Place the downloaded archive in a working folder, then extract:

.. code-block:: shell

    tar -xjf public_sources.tbz2
    cd Linux_for_Tegra/source
    tar -xjf kernel_src.tbz2
    cd kernel
    cd kernel-jammy-src

.. note::
    The inner folder name (e.g. ``kernel-jammy-src``) may differ depending on your Ubuntu and L4T version. Enter whichever single folder is present at that level.
..

5 - Copy Your Current Kernel Configuration
*******************************************

Use the running kernel's configuration as the build base so your compiled module is compatible:

.. code-block:: shell

    zcat /proc/config.gz > .config

6 - Enable the Driver in .config
**********************************

Open ``.config`` with your preferred text editor and search for the driver's config key. For ``gs_usb`` it looks like:

.. code-block:: text

    # CONFIG_CAN_GS_USB is not set

Change the line to:

.. code-block:: text

    CONFIG_CAN_GS_USB=m

The ``=m`` value tells the build system to compile it as a loadable module instead of building it into the kernel image. Save and close the file.

7 - Prepare the Build Environment
***********************************

.. code-block:: shell

    make ARCH=arm64 LOCALVERSION=-tegra olddefconfig
    make ARCH=arm64 LOCALVERSION=-tegra modules_prepare

8 - Compile the Driver
***********************

.. code-block:: shell

    make ARCH=arm64 LOCALVERSION=-tegra M=drivers/net/can/usb modules

.. note::
    The ``M=`` path points to the folder containing the driver's source. To find the right folder for a different driver:

    .. code-block:: shell

        find . -name "*.c" | xargs grep -rl "MODULE_ALIAS\|module_init" | grep -i <driver_name>

    Set ``M=`` to the containing directory of the result.
..

9 - Install the Module
***********************

Create the destination directory mirroring the kernel's module tree:

.. code-block:: shell

    sudo mkdir -p /lib/modules/$(uname -r)/kernel/drivers/net/can/usb/

Copy the compiled module:

.. code-block:: shell

    sudo cp drivers/net/can/usb/gs_usb.ko /lib/modules/$(uname -r)/kernel/drivers/net/can/usb/

Update the module dependency database:

.. code-block:: shell

    sudo depmod -a

10 - Load the Driver
*********************

.. code-block:: shell

    sudo modprobe gs_usb

Verify it loaded:

.. code-block:: shell

    lsmod | grep gs_usb

If no errors appear and the module shows up in ``lsmod``, your driver is active. Happy linuxing!

.. note::
    To load the module automatically at boot, add its name to ``/etc/modules``:

    .. code-block:: shell

        echo "gs_usb" | sudo tee -a /etc/modules
..

----

Troubleshooting
***************

**"No rule to make target" or module not found after** ``make modules``
    Confirm ``CONFIG_CAN_GS_USB=m`` is present in ``.config`` and that you ran both ``olddefconfig`` and ``modules_prepare`` before the module compile step.

**Kernel version mismatch: module refuses to load**
    The compiled ``.ko`` must match the running kernel exactly. Check:

    .. code-block:: shell

        uname -r
        modinfo drivers/net/can/usb/gs_usb.ko | grep vermagic

    Both strings must match. If they differ, you downloaded sources for the wrong L4T revision. Re-download the correct one.

**"Required key not available" on** ``modprobe``
    Kernel has module signature enforcement enabled. Either disable it (not recommended on production systems) or sign the module with the kernel's signing key. Refer to Nvidia's BSP documentation for the key location.

**``depmod`` reports missing symbols**
    The module depends on another module that is not loaded or not present. Check dependencies:

    .. code-block:: shell

        modinfo gs_usb.ko | grep depends

    Ensure all listed modules are available under ``/lib/modules/$(uname -r)/``.

**Device not recognized after** ``modprobe``
    Confirm hardware is connected, then inspect kernel messages for binding errors:

    .. code-block:: shell

        dmesg | tail -30
