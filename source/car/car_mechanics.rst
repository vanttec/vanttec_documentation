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

Vehicle’s mechanical state 
--------------------------


* This vehicle is adapted with drum brakes in all 4 wheels, this were cleaned on 2022’s summer (Late July) in this same moment, it was noticed that the master cylinder had a minor leak that affected the performance of the braking system, this master cylinder is located in the front of the car, right behind the steering wheel.  Braking shoes appear to be in good condition. For future troubleshooting, if brake performans decays, adjusting these with a screw located either in the bottom or in the back of the system may help to expand the brake shoes even further and create more friction with the drum


* The rear differential’s oil level was Low and was refilled this summer with 80W-90 gear oil, in case of leaking, the nut located in the back of the differential should be well tightened, if this doesn't work the whole differential should be opened and the old internal gasket should be removed, cleaned and  replaced with a new one.
* The motor state and functionality is good
* The gearbox lever works good, but it is hard to change to some gears, replacing this mechanism or changing to reverse gear with electronics would be very useful
* Parking brake works well, this works mechanically and electronically, this brake has a whip that activates brakes on all 4 wheels. This parking brake sends a signal to the dashboard that tons on a light to indicate if the pariking brake is activated

FEA (Finite Element Analysis)
--------------------------
FEA - Steering wheel
--------------------------
* A finite element analysis was intended to be done in order to validate the behaviour of a direction actuator designed by the 2022-2023 vanttec's mechanical team for the SDV. This analysis was done because as this actuator was designed by vanttec's engineers, the behaiviour in certain scenario and performance should be analyzed 
* For this analysis Ansys workbench software was used and nylamid was the material chosenfor the gears while ABs was the material chosen for enclosures and supports. This analysis was an static structural analysis which allows us to evaluate results in an instant of time without dynamic elements.
* From this first run of simulation, it can be concluded that nylamid is a great option for the manufacture of this actuator, deformation font exceed 0.5 mm.
* A second run was made with a redesign that consisted in reducing the gears to 1/4' as a plate of of 1/4' of steel was purchased. This actuator is shown below:

* An static structural analysis was done with an 8 Nm moment in the pinion and a 48 Nm in the direction bar simulating the rolling resitance. This is the mayor value of rolling resistance it may exist because this was messured while the vehicle was stopped, this value was 5 Kgf.
* This model was put through a thermal load of 40 celsius degrees simulating MTY weather as well as other 40 degrees from the possible temperature produced by the motor.
* All simulations were done with ABS instead of PLA, this will result in similar behaviour but not exact ones, this decision was made because PLA was not abailable as a material in ANSYS.

* This simulation results are shown bellow:

Total Deformation (mm)
.. figure:: /images/SDV/ME/SW_FEA_TotalDef.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px

Equivalent stress (MPa)
.. figure:: /images/SDV/ME/SW_FEA_Eqstress.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px

Equivalent elastic strain 
.. figure:: /images/SDV/ME/SW_FEA_EqElastics.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px

Safety factor 
.. figure:: /images/SDV/ME/SW_FEA_SF.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px

Life
.. figure:: /images/SDV/ME/SW_FEA_Life.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px

Fatigue Safety Factor 
.. figure:: /images/SDV/ME/SW_FEA_FatSF.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px

   * Analysing this results it can be noticed that there is an existing maximum deformation of 0.99 mm which is les than 1 mm and it's in a zone in which this deformation may be caused by the ausence of the other part of the case.
   * Equivalent stress is high but the max value is concentrated only in 1 gear and in only one of the theet so when distributing this stress to the 5 gears, this will reduce and will result in stresses lower than Ultimate tesile strength

   * Modal Analysis 
      * There were 6 natural frequencies found when the motor was at 0 RPM and 6 when natural frequencies when the motor is 300 RPMs, which will be the used speeds
.. figure:: /images/SDV/ME/SW_FEA_ModalFreq.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px

      * This frequencies may represent a major problem because there is a group situated near 300Hz, this group of frequencies lay in the road noise, but the suspension and the plastic vibration reducers will aid mitigate these.
   * A frequency graphic widely used in automotive applications is shown bellow:
   .. figure:: /images/SDV/ME/SW_FEA_freqGraph.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px 

FEA - Braking Unit 
--------------------------
   * Another analysis was made for the braking unit actuator, for this analysis workbench was utilized and structural steel was the material used for simulation beacause there were not enough mechanical characteristics of AISI 1020 material in order to do this simulation. Structural steel has similar yet inferior mechanical characteristics. therfore having an adequate behaviour with structural steel we can ensure the correct performance with AISI 1020.
   * The obtained results from an structural steel with forces that will be used in the given application are shown bellow:

Equivalent stress
   * In the equivalent stress analysis, the major stress is concentrated in one of the ancle support holes, this stress generates only small deformations in that part that may not have a signifact effect in the actuator, an additional support is recommended in the corner od the motor in which there's none.
.. figure:: \images\SDV\ME\BU_FEA_SS.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px


Total deformation
   * In the total deformation analysis, the maximum deformation point is located in the corner in which ther'n no support. Dispite of that, this maximum point is only 0.12 mm, so this might not be significant as there will be another superior pice closing the actuator and acting as a cap.
.. figure:: \images\SDV\ME\BU_FEA_TD.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px


Security factor 
   * With the existing stress calculated with this simulation, a maximum security factor of 15 and a minimum security factor of 0.276 were found. This minimum point was located in the corner of the wedge located between the shaft and the biggest gear, because there is a mayor existing torque, there will be a filet in those corners due to stress but it won't affect actuators performance.
.. figure:: \images\SDV\ME\BU_FEA_SF.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px


Fatigue (Lifes)
   * The actuatror was exposed to fatigue and the results were a maximum of 380,530,000 lifes (Equivalent to 380 moths) and a minimum of 21307 (Equivalent to 102 months).
.. figure:: \images\SDV\ME\BU_FEA_L.png
   :align: center
   :alt: car_picture
   :figclass: align-center
   :height: 200px
   :width: 300px
