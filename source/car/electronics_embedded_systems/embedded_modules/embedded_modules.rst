Embedded Modules
================

Seven embedded modules provide autonomous capabilities to the self-driving vehicle.

.. figure:: /images/electronics_embedded/sdv_systems.png
   :align: center
   :alt: sdv_systems
   :figclass: align-center
   :width: 800px
   
   SDV Modules, Sensors and Computing Devices

|

Common Module Subsystems
========================
SDV modules are similar (with slight variations) in certain subsystems, such as in IO, and power supply.

-----
STM32
-----

The `STM32L431RCT6 <https://www.lcsc.com/product-detail/Microcontroller-Units-MCUs-MPUs-SOCs_STMicroelectronics-STM32L431RCT6_C92468.html>` is chosen as the main microcontroller unit for all modules
to follow an standard started by the development of the USV and UUV PCBs, where a similiar version was used, which in turn eases the electronics and embedded development.

.. figure:: /images/electronics_embedded/stm32_base.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Base STM32 configuration

|

High-Speed External Clock
-------------------------
.. figure:: /images/electronics_embedded/hsec.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 200px
   
   Oscillator circuit

`This <https://www.st.com/resource/en/application_note/an2867-oscillator-design-guide-for-stm8afals-stm32-mcus-and-mpus-stmicroelectronics.pdf>` design guide was used to choose the crystal circuit components.

The high-speed external clock is supplied with a 16MHz as the high-speed external crystal oscillator.

The `crystal <https://www.lcsc.com/product-detail/Crystals_Yangxing-Tech-X322516MLB4SI_C13738.html>` has a load capacitance (CL) of 9pF, and assuming a stray PCB capacitance (Cs) of 3pF, 12pF capacitors (CL1 and CL2) were chosen following the next equation.

.. figure:: /images/electronics_embedded/crystal_eq.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 200px
   
   Crystal equation

|


STM32 Programming, Supply and Communications
--------------------------------------------
An ST-Link is used to program the STM32 via a 10 pin header.

A 5-pin M12 connector is used to supply +12V, and to connect the module to the CAN Bus. The 5th pin is left unused.

.. figure:: /images/electronics_embedded/prog.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 400px
   
   Programming, Supply and Comms

|

Module ID
---------
Each module can be mannually assigned an ID, which can be used fpr the CAN messages identification. A maximum of 16 IDs can be assigned.

.. figure:: /images/electronics_embedded/module_id.png
   :align: center
   :alt: module id
   :figclass: align-center
   :width: 200px
   
   Module ID selection

|

------------
Power Supply
------------

.. figure:: /images/electronics_embedded/power_supply.png
   :align: center
   :alt: power_supply
   :figclass: align-center
   :width: 500px
   
   Base power supply configuration

|

The power supply subsystem provides power to the whole module, which contains:
   * A 1A fuse for overcurrent protection.
   * Reverse polarity voltage protection
   * DC Converters
   * LEDs as power indicators
   * NTC thermistor for the regulators temperature

Reverse Polarity Protection
---------------------------
For reverse polarity voltage protection with mosfets, a couple of things should be considered.

* The mosfet drain-source current (Ids) must withstand the module current requirements.
* The drain-source voltage (Vds) must be larger than BATT+.
   * A `NDS352AP <https://www.mouser.mx/ProductDetail/onsemi-Fairchild/NDS352AP?qs=mdiO5HdF0KhbUArAR6yyEg%3D%3D>` mosfet is chosen, with a Vds = 30V.
* The gate-source voltage (Vgs) must not be surpassed by BATT+. A zener must be added to protect the mosfet in the case that BATT+ is larger than Vgs.
   * The NDS352AP Vgs = 20V, so a `MMSZ4702T1G https://www.lcsc.com/product-detail/Zener-Diodes_onsemi-MMSZ4702T1G_C242274.html` zener with a Zener voltage of 15V is chosen.
* The drain-source resistance (Rds) must be as low as possible, for low power dissipation.

Unlike using a diode, this circuit does not step down the voltage as a mosfet is being used to open or close the circuit.

The LTSpice simulation showcases the correct operation of the circuit.

.. figure:: /images/electronics_embedded/reverse_polarity.png
   :align: center
   :alt: reverse polarity protection
   :figclass: align-center
   :width: 600px
   
   Base reverse polarity voltage protection circuit

|

DC Converters
-------------
Depending on the module, a single 12V to 3.3V plus a 12V to 5V could be used, both with a maximum output current of 1A.
The regulators employed are:

* `R-785.0-1.0 <https://www.mouser.mx/ProductDetail/RECOM-Power/R-785.0-1.0?qs=YWgezujkI1LK5NzKL%2Fc9sg%3D%3D>`
* `R-783.3-1.0 <https://www.mouser.mx/ProductDetail/RECOM-Power/R-783.3-1.0?qs=XF8hdbuHJAVK%252BT0VfuIcYQ%3D%3D>`

---
I/O
---

IO for all modules is usually based on the use of an optocoupler to isolate inputs from the module circuits.

As input voltages may range from 5V to 12V (or more), a constant current (~15mA) circuit is required to power the optocoupler LED.
More on this `here <https://www.instructables.com/Circuits-for-using-High-Power-LED-s/>` and `here <https://www.ti.com/lit/wp/slyy163/slyy163.pdf?ts=1664295229681&ref_url=https%253A%252F%252Fwww.google.com%252F>`.

The transistors used are `BC817-40,215 <https://www.lcsc.com/product-detail/Bipolar-Transistors-span-style-background-color-ff0-BJT-span_Nexperia-BC817-40-215_C52801.html>`, which withstand
a maximum collector-emiter voltage of 45V.

.. figure:: /images/electronics_embedded/io.png
   :align: center
   :alt: input-output circuit
   :figclass: align-center
   :width: 600px
   
   Base IO configuration

|


Stepper-based Modules
=====================
The steering and pedal brake modules share essentially the same purpose: control a stepper motor and read encoder signals.
For this reason, the PCB for both modules is exactly the same.

Transmission Module
===================
To be determined.