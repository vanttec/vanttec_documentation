=================
VantTec STM32 PCB
=================

.. image:: /images/vanttec_logo.png
   :align: center
   :alt: Front view PCB

Description
===========

This is the main board where different embedded systems are connected. 

Software Used
=============
KiCad 6.0

Components Used
===============

- STM32F405RGT6


Notes
=====

- There are two connectors to further expand PCB functionality.
- The boards has communication with SPI
- The board includes a connector for an hydrophone
    - 4 Analog input pins connected directly to the STM32 
    - It has four GPIO pins that can be used with SPI 

Embedded
========

Peripherals
-----------

CAN2
~~~~

FD cannot be used because of the transciever

=====  ============
Pin    Description
=====  ============
PB12   CAN_RX 
PB13   CAN_TX
=====  ============

PWM
~~~

=====  ============
Pin    Description
=====  ============
PC6    TIM3_CH1
PC7    TIM3_CH2
PB0    TIM3_CH3
PB1    TIM3_CH4
PB6    TIM4_CH1
PB7    TIM4_CH2
PB8    TIM4_CH3
PB9    TIM4_CH4
=====  ============

I2C2
~~~~

=====  ============
Pin    Description
=====  ============
PB10   SCL 
PB11   SDA
=====  ============

- PCA9685PW

Used to generate a PWM pulse for either the turret or servo

SPI1
~~~~

- PCA120IPT. Used to measure current on the thrusters

=====  ============
Pin    Description
=====  ============
PA8    CS 
PA5    SCK
PA6    MISO
PA7    MOSI
=====  ============


ADC
~~~~


=====  =================   ==========================================================
Pin    Description         Extra Notes
=====  =================   ==========================================================
PC5    BATTERY_VOLTAGE     Connected to a voltage divider with 1M and 220K resistors
PC0    BATTERY_TEMP        In order to connect to the battery thermistor
PA1    HYDRO_ANALOG_1     
PA2    HYDRO_ANALOG_2     
PA3    HYDRO_ANALOG_3     
PA4    HYDRO_ANALOG_4     
=====  =================   ==========================================================


USART1
~~~~~~

Its connected to a header, possibly to debug the device.

=====  ============
Pin    Description
=====  ============
PB10   SCL 
PB11   SDA
=====  ============

USART5
~~~~~~

The RX pin is only connected to the SBUS in order to communicate with the boat for teleop.
This pin goes trough a SN74LVC1G240DBVT 

Schematics
==========

.. image:: /images/
   :align: center
   :alt: status display pcb Schematics

PCB 3D Model
============

Front View
----------

.. image:: /images/
   :align: center
   :alt: pcb 3d front view

Back View
---------

.. image:: /images/
   :align: center
   :alt: Front view PCB

Known Issues
============