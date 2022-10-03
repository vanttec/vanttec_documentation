UAV electronics and Embedded Systems
======================================

Electronics
-----------------------------
Following sections will explain all components that make part of the vehicle, as well as information about the ground station and usage guides.

Main Components (to be updated)
-------------------------------

.. list-table:: 

    * - .. figure:: /images/drone_electronics_faceup.png
           :height: 300px
           :width: 300px

           Drone electronics picture (face up).
      - .. figure:: /images/drone_electronics_facedown.png
           :height: 300px
           :width: 300px

           Drone electronics picture (face down).

========  ========================== =======================================================
Number    Name                                            Notes
========  ========================== =======================================================
1         Pix32 Flight Controller    Provides stabilization and controlable flight modes, as
                                     well as ROS integration.

2         FrSky X8R RC Receiver      Connects to FrSky Taranis X9D, allows for manual 
                                     control of the vehicle.
          
3         MRO 815 MHz telemetry      Provides information of the UAV via MAVlink protocol.
          (antenna and chip)            

4         Safety switch              Additional safety measure for handling the UAV.
                                      
5         Afro Opto ESC 20A          Allows for brushless motor control, arduino flashing
                                     guide provided here: `Arduino ESC flashing guide`_

6         PX4 Passive Buzzer         Provide audio cues to indicate UAV state.

7         S500 PDB                   Routes battery power to ESCs and soldered buck 
                                     converter.

8         Multistar Elite            920 Kv rating, which scales to a maximum of 10212 rpm
          2216 CW/CCW Motors         with no load, given a 11.1V battery.

9         LM2596 DC Voltage          Downscales battery voltage to 5V for the Raspberry Pi.
          Regulator

10        Raspberry Pi Camera V2     Provides downward vision to an internal ROS topic.
          Module
========  ========================== =======================================================

Additional Components
---------------------

====================================== =======================================================
Name                                            Notes
====================================== =======================================================
Series 5000 Zippy Li-Po 300 Battery    Provides power to all onbard electronics, including the
                                       onboard computer.

MRO USB Receiver                       Connects to telemetry for ground station control and 
                                       communications.

Master Airscrew 12x4.5 Propellers      Increased length to account for the weight of
                                       onboard electronics.
====================================== =======================================================

Guides
======

Manual Operation guide
----------------------

**Assembly and Setup: UAV**

With propellers unmounted:

(1) Connect battery and XT90-XT60 adapter to PDB.

(2) Check power on flight controller (blinking yellow LED on controller).

(3) Mount and tighten propellers with monkey wrench: With the controller's lower arrow pointing downwards, screw the propellers as following: 
    CCW proppeler on motors A(1) and C(2) and CW proppler on D(3) and B(4) motors:

(4) Switch Taranis controller on and check connection with receiver (solid green LED on receiver).

(5) Press and hold safety switch until ESCs beep together. **At this point motors can be armed, handle controller carefully**.

(6) Check that all failsafes have been cleared (flashing blue LED on controller).

**Assembly and Setup: Ground station**

(1) Arm and connect telemetry antenna to USB port.

(2) Open QGroundControl or Mission Planner and select automatic port search.

(3) Verify if UAV is in Stabilize mode, change if otherwise.

**Flight**


Distancing at least 2 meters from the UAV, and clearing all obstacles and equipment from the landing zone:

(1) Move RC controller left stick (throttle) all the way down and right, until the UAV emits a long beep and the flight controller LED turns solid blue. **At this point the UAV will respond to controller input, handle carefully**.

(2) Return the throttle until it is all the way down and centered, then gradually raise it towards the center of the joystick area, at this point the motors will activate and the drone will start to go up. Use both axes of the right Taranis joystick (pitch and roll) to move the drone forward and backward, and left and right, respectively.

(3) For disarming, get as close as safely possible to the ground and either change the flight mode to LAND, using the Taranis SB stick, or force disarm by pressing down on the SL and SA sticks.

(4) Verify that the LED on the flight controller starts flashing blue, returning to standby mode.



Arduino ESC flashing guide
--------------------------

(Coming soon.)
