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
    :alt: car control
    :figclass: align-center
    :width: 100px

    Eigen library


Eigen is a linear algebra library for C++ known by its versatility, fast development and implementation, and reliability.
It supports vector and matrix operations, including multiple classes, functions, and a linear solver. 

This library does not have any other dependency than the C++ standard library and it's the model development.

Car Model
---------
The car model consist of 3-degrees-of-Freedom (DOF) car. On the **Ackerman model** it is described the real configuration
of the vehicle considering that each frontal wheel having an independient steering angle.

.. figure:: /images/SDV/Ackerman-CarModel.png
    :align: center
    :alt: car control
    :figclass: align-center
    :width: 600px

    Ackerman car model

|

However this model can be simplified to a bycicle mode considering that both wheels have the same steering angle, 
modeling them as a single one. The same can be applied to the two rear wheels.

.. figure:: /images/SDV/bycicle-model.png
    :align: center
    :alt: car control
    :figclass: align-center
    :width: 700px

    Bycicle model

|

The kimenatics of a bycicle model are:

.. centered::
    :math:`\begin{matrix}
    \dot{x} & = & V_{x} \cos \varphi -  V_{y} \sin \varphi\\ 
    \dot{y} & = & V_{x} \sin \varphi +  V_{y} \cos \varphi \\ 
    \dot{\varphi } & = & r
    \end{matrix}`

|

The dynamic model of the car is definend in terms of mass, bracking forces,
throttle forces, drag, rolling, gravitational force, and tire logitudinal and lateral forces, by the following
equations:

.. centered::
    :math:`\begin{matrix}
    m\dot{V_{x}} & = & F_{x} - F_{Fy} \sin \delta + mV_{y}r\\ 
    m\dot{V_{y}} & = & F_{Ry} + F_{Fy} \cos \delta - mV_{x}r\\ 
    I_{z}\dot{r} & = & F_{Fy}  I_{F} \cos \delta - F_{Ry} I_{R}
    \end{matrix}`

|

Where:

    * :math:`m` : Mass.
    * :math:`\delta` : Wheel angle.
    * :math:`r` : Yaw rate (angular velocity).
    * :math:`V_{x}` : Speed on x.
    * :math:`V_{y}` : Speed on y.
    * :math:`F_{x}` : Force in x.
    * :math:`F_{Fy}` : Front wheel force in y (frontal lateral force).
    * :math:`F_{Ry}` : Rear wheel force in y (rear lateral force).
    * :math:`I_{z}` : Yaw polar inertia.
    * :math:`I_{F}` : Front inertia.
    * :math:`I_{R}` :  inertia.

The additon of forces is the following:

.. centered::
    :math:`F_{x} = F_{brake} + F_{throttle} - F_{drag} - F_{rr} - F_{g}`

|

**NOTE:** it must be considered that car never accelerates and brakes simultaneously.

Brake force is defined as:

.. centered::
    :math:`F_{brake} = m \dot{V_{brake}}( \beta , v)`

|

.. centered::
    :math:`\dot{V_{brake}} = \min (0, 2.27 - 6.12 \beta + 0.00535 v)`   ,   :math:`\beta \in [0,1]`

|

Where:

    * :math:`\beta` : Pedal position.
    * :math:`v` : Car velocity.

**NOTE:** the values of the function were obtained  from a linear regression of the resultan acceleration
generated

The throttle force is computed using the driver command and a constant specificed by the motor model as 340N 
to reach a speed of 9.2 m/s:

.. centered::
    :math:`F_{throttle} = C_{m} D`  ,   :math:`D \in [0,1]`

|

Air drag force equation is:

.. centered::
    :math:`F_{drag} = \frac{1}{2} \rho A C_{o} v^{2}`

|

Where:

    * :math:`\rho` : air density.
    * :math:`A` : transversal are of the car.
    * :math:`C_{o}` : drag coefficient.

The rolling resistance force is given by:

.. centered::
    :math:`F_{g} = m g \sin \theta`

|

Where:

    * :math:`C_{r}` : rolling resistance coefficient.
    * :math:`\theta` : pitch angle.

The horizontal component of gravitational force is defined as:

.. centered::
    :math:`F_{g} = m g \sin \theta`

|

The front and rear tire sets are modeled to each provide a force perpendicular to the rolling direction 
of the tire and proportional to the angle.

.. centered::
    :math:`F_{Fy} = -C\alpha \cdot - \alpha_{F}`

|

.. centered::
    :math:`F_{Ry} = -C\alpha \cdot \alpha_{R}`

|

.. centered::
    :math:`\alpha_{F} = \arctan \left ( \frac{V_{y} + I_{F} r}{V_{x}} \right )`

|

.. centered::
    :math:`\alpha_{R} = \arctan \left ( \frac{V_{y} - I_{R} r}{V_{x}} \right )`

|

After the equations have been defined, then the car is modeled using simulink in the following code 
blocks that contains the functions of the system.

.. figure:: /images/SDV/simulink-dynamic-model.png
    :align: center
    :alt: car control
    :figclass: align-center
    :width: 800px

    Simulink car model

|