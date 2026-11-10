Perception
==========

The perception system finds the objects around the USV and gives their position.
A USB webcam (Logitech C920) gives the image and a YOLOv5 model finds the objects in it.
A Velodyne VLP-16 LiDAR gives the distance to each object.
The software is in the `visionsystemx <https://github.com/vanttec/visionsystemx>`_ repository.

.. toctree::
   :maxdepth: 2
   :caption: Perception guides

   ./usv_software_perception/usv_software_perception_yolov5_colab
   ./usv_software_perception/usv_software_perception_lidar_camera_calibration
