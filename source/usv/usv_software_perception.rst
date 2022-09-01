Perception
==========

The perception system implemented on the USV platform is mainly reliant on two main sensors: the VLP-16 puck and the ZED camera.

The ZED - YOLOv3 implementation uses the zed_ros_wrapper package to correctly parse the data from the sensor and publish several ROS topics. 

Information on this package can be found in: (https://github.com/stereolabs/zed-ros-interfaces)

Weights for the YOLOv3 trained for buoy detection are loaded in using the darknet_ros_zed package.

Information on this package can be found in: (https://github.com/leggedrobotics/darknet_ros)

The VLP-16 puck implementation uses the velodyne_ros package to correctly parse the data and publish several ros topics:

Information on this package can be found in: https://github.com/ros-drivers/velodyne


The current implementation works independently, and makes use of an additional script called obj_register_node.py to document each recognized obnstacle in a data structure.
