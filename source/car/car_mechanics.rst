SDV Mechanics
=============

Technical report from FJ 2022 Semester (Mechanics)
--------------------------------------------------

Authors:
 
* Orlando Yair Barajas Perez
* Carlos Daniel Estrada Guerra
* Oscar Enrique Aillón De La Cruz
* Diego Antonio Ramírez Rivera
* Carlos Iván Tamayo Palacios

Introduction
------------

This document describes the milestones achieved by the mechanical team from February to June 2022. During this term, the team’s objectives were to get to know and present the project to the new team members, analyze the car’s status and repair and modify it to make it autonomous. In addition to this, the team's efforts were also concentrated on creating actuation systems such as the steering wheel system, braking system, and sensor mounts.
The main difficulty was the purchase of batteries, but fortunately, VantTec and ZF created a collaborative relationship, where the latter offered to sponsor the team with cash, easing this purchase.

Batteries
---------

As for any electric vehicle, batteries are essential for the car’s operation. During the initial inspection, it was noticed that the installed batteries were no longer functional due to the car’s long-term abandonment and lack of maintenance. Thus, testing most of the systems and components was not possible, resulting in great uncertainty about the vehicle’s condition and operability, therefore, limiting the automation process. Replacing the batteries was set as one of the team’s main priorities.
The electrical team decided that 4 12v batteries were going to be implemented in this car adding up to 48v in a series configuration, which is what the car requires. Due to school purchase protocols and lack of cash this was a major backup in the project progress. With ZF support we will be able to make this purchase and continue with the project.
To this moment the most likely company to serve as our battery provider is LTH with batteries of 12V and 115 Ah

Breaking system
---------------

The braking system is one of the most important components in autonomous vehicles, it provides security and control over the motion generated without the necessity for human monitoring. 
Electrification and the introduction of brake-by-wire technologies are options taken in consideration for the present project. However, less sophisticated systems were part of initial design proposals to decrease cost production and time-consuming activities. 

Rack and pinion mechanism
-------------------------

A rack and pinion mechanism consist in a circular gear and a linear pattern which converts a revolving motion into a linear motion. This system typically is powered by hand or by a motor that converts the energy applied into a linear motion.
Most of the cases, the rack is fixed, and the pinion is carried through the cross section (rack). For this project, the pinion will rotate around its axis and the rack will rotate around its axis. The rack will follow a linear path to put pressure over the brake pedal. This linear motion simulates the human's feet function while breaking. 

.. figure:: /images/car_break_system.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px

Electro-hydraulic system
------------------------

Hydraulic systems perform its primary function using a fluid that is pressurized. In the automobile industry, this principle is applied for hydraulic brakes systems, a mechanism that uses brake fluid to transmit the force applied by the pilot's foot to the brakes and finally, to the disc/drum brakes. 
Similarly, electro-hydraulic devices use an electric motor that rotates clockwise, turning a set of components applying a constant force that pressurizes the brake fluid. For this, a reservoir is added to inject and extract brake fluid when the valves let the fluid pass across the brake lines and when the electrical motor starts to run counterclockwise, returning the fluid. The electro-hydraulic system is intended to be part of the already existing hydraulic brake system in this vehicle. 
In other words, this component will be connected directly to the brake lines via an additional. Thus, both of the braking systems, electro-hydraulic and hydraulic, could work without compromising any of them. 

.. figure:: /images/car_electro_hydraulic_system.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px

Gearbox pedal dragging
----------------------

This mechanism consists of a NEMA 24 with a gearbox that will give us a ratio 15:1, this ratio will allow us to create a 15x greater torque than what our stepper motor is capable of delivering. One of the major problems with this mechanism is the part of encoders, we need to install encoders in one of the gears in order to sense how many turns it has given and to create a control loop. This will be our first physical prototype as it a[[ers to be the safest and more reliable of them all. This will be 3d printed and fortunately be installed during summer term.

.. figure:: /images/gearbox_2.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px


Brake-by-wire system 
--------------------

Currently, ZF gave us a challenging idea which consists of creating a mechanism with a motor and a linear actuator that can actuate as brake pad, this mechanism is intended to be used in every wheel to maximize braking control and effectiveness.
The first stage of this project will be delivered in October 2023, so this brake idea will be implemented for the second stage of the project that consists of driving throughout campus autonomously.

Steering Wheel
--------------

To automate the steering wheel, a system consisting of a spur gear attached directly to the steering column was proposed. This gear would be connected to a stepper motor, forming a gear train and increasing the torque provided by the motor. 

However, when attempting to implement this system, several issues arised. Firstly, we had to find an appropriate space to place the stepper motor. A suitable space was found, but we then realized that the steering column was not parallel to the motor, so the gears would not align and not work properly. In order to solve this problem, the stepper motor mount was created with a slight inclination that would enable the gears to be aligned at all times without the turn of the steering wheel affecting the incline angle.

.. figure:: /images/car_stearing_wheel.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px

Here we can see another angle

.. figure:: /images/stearing_wheel_2.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px
