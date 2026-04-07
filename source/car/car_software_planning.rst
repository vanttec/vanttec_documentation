Planning
========

Source Code: `Repository <https://github.com/vanttec/sdv_planner_simulation>`__
===============================================================================

Packages info
=============

-  ackermann_drive_controller - the source code for the kinematics
   controller for Ackermann vehicles
-  beetle_description - contains the vehicle simulation model
-  beetle_gazebo - kinematics simulator and controller
-  simulation - Contains a Gazebo World for planning testing
-  beetle_navigation2 - vehicle software system and parameters

+-----------------------------------------------------------------------+
| # Installation                                                        |
+-----------------------------------------------------------------------+
| Docker Image Available. Follow                                        |
| `Readme.md <https                                                     |
| ://github.com/vanttec/sdv_planner_simulation/blob/main/README.md>`__. |
+-----------------------------------------------------------------------+

How to Run?
===========

[Separate for Debugging]
------------------------

-  .. rubric:: Simulation & RVIZ
      :name: simulation-rviz

   ros2 launch simulation simulation.launch.py

-  .. rubric:: Navigation
      :name: navigation

   ros2 launch beetle_navigation2 nav.launch.py

[Whole System]
--------------

-  .. rubric:: Simulation & RVIZ & Navigation
      :name: simulation-rviz-navigation

   ros2 launch beetle_navigation2 bringup.launch.py

[Utils]
-------

-  .. rubric:: Build Packages:
      :name: build-packages

   colcon build

-  .. rubric:: See TF Tree:
      :name: see-tf-tree

   .. rubric:: ros2 run rqt_tf_tree rqt_tf_tree
      :name: ros2-run-rqt_tf_tree-rqt_tf_tree

Planning Information:
=====================

All params related to navigation are defined in:
`beetle_navigation2/config/navigation_params.yaml <https://github.com/vanttec/sdv_planner_simulation_beetle/blob/6a9631b8b11734b241b051f2f9c27626d6ab5927/beetle_navigation2/config/navigation_params.yaml>`__

-  bt_navigator - General NavStack Configuration
-  controller_server - Server for handling the controller requests
-  local_costmap - Dynamic Obstacles Avoidance using PointCloud input.
-  global_costmap - Static Obstacle Avoidance using a pre-mapped
   environment.
-  map_server - Load a pre-mapped environment.
-  map_saver - Save a mapped environment.
-  planner_server - Motion Planning Algorithm

   -  “DUBIN” motion model doesn’t allow reverse.
   -  “REEDS_SHEPP” motion model allows forward and reverse.

-  smoother_server - Handling smooth path requests
-  behavior_server - Handling recovery behavior requests
-  robot_state_publisher - SDV TF publisher.
-  waypoint_follower - Waypoint following using the NavigateToPose
   action server
-  velocity_smoother - Velocity, acceleration, and deadband smoothing
   from Nav2 to reduce wear-and-tear on robot motors and hardware
   controllers.
-  ekf_filter_node_odom - Odom Extended Kalman Filter
-  ekf_filter_node_map - Map Extended Kalman Filter
-  navsat_transform - Odom Generation using Imu and GPS.
-  controller_manager - Gazebo Simulated Physical controllers
-  ackermann_drive_base_controller - ackermann_drive_controller package
   configurations.
