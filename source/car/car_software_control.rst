Control
=======

Here you will find all documentation related to car controllers, control library utilities and ROS nodes for 
speed and position control.

VANTTEC Controllers
-------------------
`VANTTEC_CONTROLLERS <https://github.com/vanttec/vanttec_controllers>`__ is a library that implements the models 
and controllers for all the vehicles inside VANTTEC. Some of the included control classes are: 

* PID
* Adaptative Sliding Mode 
* Adaptive Non-Singular Terminal Sliding Mode

For this section, we will focus on car model and implemented controllers. The first part of development of the model was done 
using simulink.

Eigen
-----

.. figure:: /images/SDV/Eigenlogo.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 100px

    Eigen library

|
Eigen is a linear algebra library for C++ known by its versatility, fast development and implementation, and reliability.
It supports vector and matrix operations, including multiple classes, functions, and a linear solver. 

This library does not have any other dependency than the C++ standard library and it's the model development.