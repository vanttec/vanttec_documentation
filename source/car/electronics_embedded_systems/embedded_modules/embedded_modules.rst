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

Throttle Module
===================

This circuit can activate a relay module that is connected to the motor (Highlighted in orange). It is a switch that can turn on the motor. This module can also enable manual o automatic regulation of velocity with a digital potentiometer of 10k.

.. figure:: /images/electronics_embedded/throttle_module/1_relay.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Motor Relay 

|

.. figure:: /images/electronics_embedded/throttle_module/2_curtis_controller.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   General Diagram Throttle module with selected motor relay

|
-----
Octocoupler SFH617A-1
-----
CTR is defined as the ratio of the collector to the forward current expressed in percent. The collector current (Ic) is the current in the photo transistor while forward current is the current in the diode (If). 

   +-------+--------------------------+
   | Signal| KiCad schematic          |
   +-------+--------------------------+
   | Vdd   | Second                   |
   +-------+--------------------------+
   | Vout  | RelayMotor               | 
   +-------+--------------------------+
   | Rf    | VGS_sat (Mosfet).        |
   +-------+--------------------------+ 
   | Re    | R6                       |
   +-------+--------------------------+ 
   | IF    | R7                       |
   +-------+--------------------------+ 
   | Vf    | Forward Voltage =1.35.   |
   |       | According to datasheet   |
   +-------+--------------------------+ 
   | If    | VGS_sat (Mosfet).        |
   +-------+--------------------------+ 
   | VCEO  | VGS_sat (Mosfet).        |
   +-------+--------------------------+ 
   | CTR   | 60%                      |
   +-------+--------------------------+ 


.. figure:: /images/electronics_embedded/throttle_module/3_CTR.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   CTR configuration

|

-----
Mosfet WST2N7002A
-----
Vgs(th) max = 3 ( We are saturating it with 11.7V)

.. figure:: /images/electronics_embedded/throttle_module/4_Mosfet_voltage.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Mosfet GS(th) voltage

|
Vgs and Vds are not surpassing the maximum values.

.. figure:: /images/electronics_embedded/throttle_module/5_Mosfet_voltage(Vth).png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Mosfet Graph (Vds/Id)

|
Vgs and Vds are not surpassing the maximum values.

.. figure:: /images/electronics_embedded/throttle_module/6_MaxValues.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
    Mosfet Maximum Values

|

.. figure:: /images/electronics_embedded/throttle_module/e_1.png
   :align: left
   :alt: stm32 schematic
   :figclass: align-center
   :width: 300px
|


.. figure:: /images/electronics_embedded/throttle_module/7_Octocoupler_max.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 590px
   
   Octocoupler maximum and minimum values VCE

|
.. figure:: /images/electronics_embedded/throttle_module/e_2.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 350px

|

.. raw:: html
   <br>

-----
LTspice Tests
-----
We tested these equations with octocoupler PC817C and Mosfet QS6K1.

.. figure:: /images/electronics_embedded/throttle_module/8_lts_mosfet.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Ltspice diagram octocoupler and mosfet

|

.. figure:: /images/electronics_embedded/throttle_module/e_3.png
   :align: left
   :alt: stm32 schematic
   :figclass: align-center
   :width: 200px
|

.. figure:: /images/electronics_embedded/throttle_module/9_lts_mosfet1.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   LTspice test

|

.. figure:: /images/electronics_embedded/throttle_module/e_4.png
   :align: left
   :alt: stm32 schematic
   :figclass: align-center
   :width: 320px
|
.. figure:: /images/electronics_embedded/throttle_module/info1.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Mosfet QS6K1 gate threshold voltage minimum and maximum values
|
.. figure:: /images/electronics_embedded/throttle_module/e_5.png
   :align: left
   :alt: stm32 schematic
   :figclass: align-center
   :width: 400px
|

.. figure:: /images/electronics_embedded/throttle_module/10_lts_mosfet2.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   LTspice test

|

.. figure:: /images/electronics_embedded/throttle_module/info2.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Octocoupler collector-emitter saturation voltage minimum and maximum values.
|
-----
Real-life Tests
-----
* Tests were made with mosfet and an arduino, we used a 1k pull down resistor. Link of video below.

.. figure:: /images/electronics_embedded/throttle_module/11_fisico_mosfet.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 300px
   
   Real life tests with mosfet

|

.. figure:: /images/electronics_embedded/throttle_module/kicad_mosfet.jpeg
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Kicad tests with mosfet

|
-----
Digital Potentiometer DS3502U+
-----
This circuit can enable manual o automatic regulation of velocity with a digital potentiometer of 10k. We are powering the potentiometer with 12 Volts, to modulate velocity we used the inputs A,B,W , in the terminal A or B will be  entering 10V and the result will be going to the Wiper W and eventually will go to J3 from curtis controller.

* Meaning of pins:

   +----------+-----------------------------------------------+
   | Pins     | Meaning                                       |
   +----------+-----------------------------------------------+
   | Pot      | Connected to GPIO of STM32 to switch motor.   |
   +----------+-----------------------------------------------+
   | J2E      | This signal is connected to Curtis controller |
   |          | J2 , it has 10 Volts.                         | 
   +----------+-----------------------------------------------+
   | WiperPot | If we decide to use manual control, this      |
   |          | signal will first be sent to our car's 5K     |
   |          | pedal and then be connected to our J3 Curtis  |
   |          | controller, which is how velocity is          |
   |          | manually modulated.                           |        
   +----------+-----------------------------------------------+ 
   | J3       | The modulated signal will come from the wiper |
   |          | (W) and be connected straight to J3 Curtis    |
   |          | Controller if we opt to employ                |
   |          | the autonomous mode.                          |
   +----------+-----------------------------------------------+ 


.. figure:: /images/electronics_embedded/throttle_module/12_pot.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Digital Potentiometer schematic 

|
.. figure:: /images/electronics_embedded/throttle_module/13_pot_general_view.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px

   Throttle module in general diagram 

|

* Pin description Digital potentiometer:
  
.. figure:: /images/electronics_embedded/throttle_module/14_pot_pins.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Digital potentiometer pin description


|

   +----------+----------------+
   | Pins     | MCU /Car       |                    
   +----------+----------------+
   | RH       | J2E / GND / -  | 
   +----------+----------------+   
   | RL       | J2E / GND / -  | 
   +----------+----------------+  
   | V+       | 12V            | 
   +----------+----------------+  
   | SCL      | SCL            | 
   +----------+----------------+  
   | A0       | GND            | 
   +----------+----------------+ 
   | A1       | GND            |  
   +----------+----------------+ 
   | VCC      | 3.3 V          |  
   +----------+----------------+ 
   | SDA      | SDA            |  
   +----------+----------------+ 

* Digital Potentiometer datasheet:
  
.. figure:: /images/electronics_embedded/throttle_module/15_pot_max_values.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Digital potentiometer maximum ratings

|


   +----------+-------------------------------+
   | Pins     | Digital Potentiometer | STM32 |                    
   +----------+-------------------------------+
   | SCL      | SCL                   | SPB6  |  
   +----------+-------------------------------+   
   | SDA      | SDA                   | PB7   | 
   +----------+-------------------------------+  


* STM32 datasheet:

.. figure:: /images/electronics_embedded/throttle_module/16_datasheet_stm32.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   STM32 Pin description I2C 

|
-----
Why A0 and A1 is connected to GND?
-----
The DS3502's slave address is determined by the state of the A0 and A1 address pins. These pins allow up to four devices to reside on the same I2C bus. Address pins tied to GND result in a 0 in the corresponding bit position in the slave address. Conversely, address pins tied to V CC result in a 1 in the corresponding bit positions. For example, the DS3502's slave address byte is 50h when A0 and A1 pins are grounded. I 2 C communication is described in detail in the I 2 C Serial Interface Description section.

.. figure:: /images/electronics_embedded/throttle_module/17_datasheet_i2c_frame.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   DS3502 Slave Address Byte

|
-----
Why VDD has decoupling capacitors?
-----

To assist smooth out any low-frequency variations in an input voltage, a 10uF capacitor is placed farthest from the IC. Then comes the 0.1uF capacitor that is located the closest to the IC. This one will assist in eliminating any high-frequency noise in your circuit. You can provide your IC a constant, smooth voltage to operate with by connecting these two capacitors together.
Things to consider in layout with decoupling capacitors:

  * *Placement:* Whether your power supply is 3.3V or 5V, you should always connect your decoupling capacitors to ground.
  * *Distance:* Decoupling capacitors should always be positioned as close as feasible to your integrated circuit. They will be less effective the further away they are.
  * *Ratings:* For each integrated circuit on your board, It is  generally advised adding a smaller 0.1-10uF electrolytic capacitor and a single 100nF ceramic capacitor. 

.. figure:: /images/electronics_embedded/throttle_module/18_decoupling_capacitors.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Digital Potentiometer power supply with decoupling capacitors
|
-----
Why so many 0 resistors in Terminal A and B?
-----
A zero-ohm link or zero-ohm resistor is a wire link packaged in the same physical package format as a resistor. Wiring alternatives for the experiments we conducted included connecting J2 of the Curtis controller to A and receiving the modulated voltage in W at J3 of the Curtis controller. Likewise, by connecting J2 in the other direction to B and giving W the modulated voltage. In the first experiment, we increase one vehicle's speed from 2 to 5 km/h while decreasing the other from 24 to 36 km/h. Personally, I think J2 ought to be linked to B, J3 ought to be connected to W, and A shouldn't be connected to ground. There will always be a chance for a modification with the 0 resistors, though, if I am mistaken.

.. figure:: /images/electronics_embedded/throttle_module/19_0_resistors.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Digital Potentiometer schematic showing RH,RW,RL 
|
-----
Tests with Digital Pot  X9c103s 10k and Pot5k
-----
* We carried out a number of experiments and found that the speed could be modulated, going from 35 to 24 km/h. If we switched the terminals A and B to GND and VCC, we saw that the speed increased to 2 to 4/5 km/h. We assumed that the reason for the slight fluctuation was that we are utilizing a voltage divider in J2 from Curtis Controller. It would occur in the throttle pedal since it doesn't require the voltage divider. However, X9x1903s can only accept a maximum voltage of 5 volts; in our previous experiments, we entered 3 volts into the chip. Other thing that we noticed is that because of the voltage divider J2 was always connected to Ground , that is why we couldn't achieve a proper modulation of velocity.

.. figure:: /images/electronics_embedded/throttle_module/20_adc_pot.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 400px
   
   Arduino Graph with ADC input and digital control
|

* Tests were made with a 10k potentiometer to see if J2 and J3 can modulate velocity in Curtis controller, it worked exactly as the 5k pedal in our car.
.. figure:: /images/electronics_embedded/throttle_module/21_manual_pot.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 400px
   
   Manual test with 5k potentiometer
|


-----
References
-----

* Link of tests in drive link: https://drive.google.com/drive/folders/1uXBLU69br4WVwCLYkIgSi3tL1Cs67hec?usp=sharing
* Datasheet, LSCS or mouser link:
* SFH617A-1: https://www.mouser.mx/ProductDetail/Vishay-Semiconductors/SFH617A-1?qs=xCMk%252BIHWTZPd%252B0%2F2oZ4QGA%3D%3D
* Mosfet WST2N7002A: https://www.lcsc.com/product-detail/MOSFETs_Winsok-Semicon-WST2N7002A_C2830888.html
* PC817C: https://pdf1.alldatasheet.es/datasheet-pdf/view/43376/SHARP/PC817C.html
* Mosfet QS6K1: https://www.mouser.com/datasheet/2/348/qs6k1-210565.pdf
* STM32L432xx: https://datasheet.lcsc.com/lcsc/1811081824_STMicroelectronics-STM32L431CBT6_C277951.pdf
* Link of various tests in drive link: https://drive.google.com/drive/folders/1PjMfXWRD-9Su_6FwQhY2J2OD5yJEoXE3?usp=sharing