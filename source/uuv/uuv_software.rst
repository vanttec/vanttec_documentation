============
UUV Software
============

Docker installation
--------------

Requirements:

* Docker Engine 

   https://docs.docker.com/engine/install/ubuntu/

* Nvidia-Docker2 

   https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html

* Nvidia drivers

   nvidia-driver-515

Repository::

   git clone https://github.com/vanttec/UV_dev_docker

Docker Configuration
---------------------
Inside the docker in the .bashrc file, add the following lines::

   source /ws/devel/setup.bash
   export SVGA_VGPU10=0

Compliation Tools
------------------
For compiling the simulation, use catkin tools::

   sudo apt-get install python3-catkin-tools
   catkin build

Repositories to install
-------------------------

* darknet_ros_zed::

   git clone --recursive https://github.com/vanttec/darknet_ros.git

* octomap_mapping::

   git clone https://github.com/vanttec/octomap_mapping

* vanttec_uuv::

   git clone https://github.com/vanttec/vanttec_uuv
   git checkout feature/nav_dev
   
* vanttec_sim::

   git clone https://github.com/vanttec/vanttec_sim

* vehicle_user_control::

   git clone https://github.com/vanttec/vehicle_user_control

* opencv and opencv_contrib

   https://docs.opencv.org/4.x/d7/d9f/tutorial_linux_install.html


Dependencies
-----------------
 
To run the simulation, you will need the following libraries::
 
   sudo apt-get install ros-melodic-octomap ros-melodic-octomap-mapping ros-melodic-octomap-msgs ros-melodic-octomap-ros ros-melodic-octomap-rviz-plugins ros-melodic-octomap-server
   sudo apt-get install ros-melodic-octomap-ros
   sudo apt-get install ros-melodic-tf2-geometry-msgs
   sudo apt-get install libsdl1.2-dev
   sudo apt-get install libsdl1.2debian
   sudo apt-get install ros-melodic-gazebo-plugins
   sudo apt-get install ros-melodic-gazebo-ros-pkgs
   sudo apt-get install ros-melodic-robot-state-publisher
   sudo apt-get install ros-melodic-xacro
   sudo apt-get install python3-rospkg
   sudo apt-get install python3-pandas
   sudo apt-get intall python-pip
   sudo apt-get install python3-sklearn
   pip3 install numpy==1.18
   pip3 install scipy==1.1.0
   pip3 install scikit-learn==0.21.3
   pip3 install joblib==1.0.0
   pip3 install opencv-python
   pip3 install opencv-contrib-python 
   pip install opencv-contrib-python
   pip install opencv-python
   pip install imutils


Instructions for running the simulation
------------------------------------------

To use the simulation, you need to run the following commands in separate terminals::

   roscore

   rosrun rviz rviz

   roslaunch uv_worlds lake.launch

   roslaunch vehicle_descriptions vtec_u3.launch 

   rosrun vehicle_descriptions gazebo_interface

   roslaunch vanttec_uuv uuv_simulation.launch 

   roslaunch uv_worlds task_obstacles.launch 

   roslaunch octomap_server octomap_tracking_server.launch 

   rosrun octomap_server uuv_octomap.py

   rosrun vanttec_uuv nav_v1.py 

   rosrun octomap_server oclust_aserver.py

   rosrun vanttec_uuv invert_camera.py

   roslaunch darknet_ros RoboSub2021.launch

   rosrun vanttec_uuv yolo_zed.py


Known Issues 
-------------

Error: The uuv appears in the surface instead of appearing underwater when running the simulation.
Solution: Comment line 4 of rviz.launch located in /ws/src/vanttec_uuv/launch

Error: When running RoboSub2021.launch it stays “waiting for image” and YOLO does not load.
Solution: Inside of /ws/src/darknet_ros/darknet_ros/config/ros.yaml in the line 4 substitute “/invert_image” for “/camera/rgb/image_raw” and inside  /ws/src/darknet_ros/darknet_ros/launch/RoboSub2021.launch in the line 26 change default for “false”.

Missing Files
--------------
* The images bootlegger.jpg and gmangate.jpg should be in vanttec_uv_sim/uv_worlds/models/gate1
* The file camera2.yaml should be inside octomap_mapping/octomap_server/cfg/common
* RoboSub2021.launch should be in darknet_ros/launch, meanwhile robosub_2021_tiny3.cfg must be in darknet_ros/yolo_network_config inside a folder named cfg and robosub2021_96_98.weights in darknet_ros/yolo_network_config/weights.

