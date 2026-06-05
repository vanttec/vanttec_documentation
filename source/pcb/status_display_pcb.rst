==========================
VantTec Status Display PCB
==========================

Description
===========

This is a 2-layer PCB designed as a complementary device for our team in order to get information more easily about the status of the vehicle, such as sensors or battery power percentage. 

Software Used
=============
KiCad 6.0

Components Used
===============

- STM32G431KBT6
- Adafruit 2.8" and 3.2" Color TFT Touchscreen
- ECS-160-12-33-AGN-TR - > 16 MHz Crystal
- R-78E3.3-1.0 -> Regulator
- TCAN330G -> 3.3V CAN Transceiver

Notes
=====

- The display can be connected using either I2C or SPI communication protocol.
- The microcontroller uses the CAN protocol to receive info and data from another device.

Embedded
========

Peripherals
-----------

CAN
~~~

=====  ============
Pin    Description
=====  ============
PA11   CAN_RX
PA12   CAN_TX
=====  ============

I2C
~~~

=====  ============
Pin    Description
=====  ============
PA??   SCL
PA??   SDA
=====  ============

SPI
~~~
=====  ============
Pin    Description
=====  ============
PA5    SCK
PA6    MISO
PA7    MOSI
=====  ============

Display
~~~~~~~

=====  ============
Pin    Description
=====  ============
PA0    DISP_D0     
PA1    DISP_D1     
PA2    DISP_D2     
PA3    DISP_D3     
PA4    DISP_D4     
PA9    DISP_D5     
PA10   DISP_D6     
PB0    DISP_D7     
PB3    PWM_TIM2    
PB4    DC_DISP       
=====  ============

Other Pins
~~~~~~~~~~

=====  ============
Pin    Description
=====  ============
PA8    CHIP_SELECT 
PA13   SWDIO       
PA14   SWDCLK      
PB8    BOOT0       
PG10   nRST        
PF0    OSC_IN      
PF1    OSC_OUT     
PB6    LED_STATUS  
=====  ============

Schematics
==========

.. image:: /images/stm-display.svg
   :align: center
   :alt: status display pcb Schematics

PCB 3D Model
============

Front View
~~~~~~~~~~

.. image:: /images/3d-stm-display-front.png
   :align: center
   :alt: pcb 3d front view

Back View
~~~~~~~~~

.. image:: /images/3d-stm-display-back.png
   :align: center
   :alt: Front view PCB

Known Issues
------------
- MISO and MOSI pins are switched