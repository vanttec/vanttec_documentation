========
USV Software
============

Technical report from FJ 2024 Semester (Software)
========

Authors:
* Max Pacheco Ramírez
* Christian Raúl Villarreal Treviño

Introduction
------------

The following document describes the functionality of the USV's software developed by its software team which is divided into multiple ROS2 packages that in conjunction enable simulation and navigation tasks of our unmanned surface vehicle.

usv_control
------------

The usv_control package is comprised of many elements that are listed in the following: 

CMakeLists
~~~~~~~~~~~~
The CMakeLists.txt file configures the build process of a ROS2 package named usv_control with various executables and dependencies. It creates executables for various node source files specifying dependencies and language features, specifies installation destinations for them and other resources, enables linting and testing and generates the package manifest.

config
~~~~~~~~~~~~
The config subdirectory icnludes a configuration file in YAML format used for the configuration of a ROS node that interacts with an SBG device via a UART interface. This includes node frequency, UART settings, sensor parameters, magnetometer settings, GPS configurations, output configurations, etc.

launch
~~~~~~~~~~~~
The launch subdirectory includes two launch files which are XML files for the execution and configuration of multiple ROS nodes or other launch files for the management of various components of our ROS2 system. One is for the execution of multiple ROS2 nodes for the execution of a Gazebo simulation and the other for an RViz simulation.

libs
~~~~~~~~~~~~
Includes a subdirectory named 'usv_libs' with a CMakeLists.txt file that sets the project's build system with library compilation, unit testing and Python bindings generation. These mentioned libraries help with our control and simulation tasks. It also includes controller logic files and test files for controllers and the USV's dynamic model.

package.xml
~~~~~~~~~~~~
The package.xml file provides metadata about the 'usv_control' ROS2 package such as its dependencies and build configuration.

src
~~~~~~~~~~~~
The src subdirectory includes multiple nodes listed in the following:

aitsmc_node
""""""""""""""
   Defines the logic of the AITSMC controller as a node that receives setpoint and state information, computes control outputs and publishes thruster values.

asmc_node
""""""""""""""
   Defines the logic of the ASMC node controller as a node that receives setpoint and state information, computes control outputs and publishes thruster values. It also handles transitions in autonomous mode.

dynamic_model_sim
""""""""""""""
   ROS2 node that simulates the dynamics of our unmanned surface vehicle with a dynamic model and publishes its position, velocity and odometry information.

kinematic_model_sim
""""""""""""""
   ROS2 node that simulates the kinematics of our unmanned surface vehicle with a kinematic model and publishes its position, velocity and odometry information.

twist_to_setpoint_node
""""""""""""""
   ROS2 node that converts Twist messages which describe velocity and rotational motion in ROS2 into separate velocity and heading setpoint messages published to their corresponding topics.

usv_tf2_broadcaster_node
""""""""""""""
   ROS2 node that listens to the position topic, transforms it into a TF2 transform message that describe the relationship between different coordinate frames in a system and passes it to the TF2 system. This converted value can then be used by other nodes for localization, navigation or visualization.

waypoint_handler_node
""""""""""""""
   ROS2 node that handles waypoint navigation for our USV. It enables the USV to navigate toward a desired waypoint while handling manual and autonomous modes, controlling the USV's heading, velocity and pivot based on received commands and the vehicle's current state.

.. toctree::
   :maxdepth: 3
   :caption: Software departments
   
   usv_software_control
   usv_software_perception
   usv_software_planning
