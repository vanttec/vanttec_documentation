Geeral overview
================

The USV vehicle operates using a Jetson TX2 developer board loaded with a JetPack SDK provided by the manufacturer, NVDIA.

This JetPack SDK provides a Linux Kernel 5.10, UEFI based bootloader, Ubuntu 18.04 based root file system, NVIDIA drivers, necessary firmware, toolchain and more.

The latest available JetPack for the TX2 is the 4.6.2, newer versions don't support the board.

The USV's TX2 is loaded with the latest production version of the vanttec_usv repository available in the teams github.

The package implements several ros_drivers necessary for communication with the vehicle's sensors.

The main, and probably most important sensor for USV localization, is the VN-300 (Soon-to-be-changed by the SBG-D Ellipse sensor)

Thanks to major contributions from teammembers, the USV has a very robust dynamic and pathplanning model. Presented and explained in the following publications. The implementation is established in the usv_control software package.

A more specific overview of the usv_control implementation can be found in this page. [TODO]

The usv_control scripts use the data position of the USV to control the motors and reach desired positions. This is done autonomously, when feeding coordinates to the LOS? node.




