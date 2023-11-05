Embedded Modules
================

Seven embedded modules provide autonomous capabilities to the self-driving vehicle.
* TODO:
* Assign each module an ID and describe them here


.. figure:: /images/electronics_embedded/sdv_systems.png
   :align: center
   :alt: sdv_systems
   :figclass: align-center
   :width: 800px
   
   SDV Modules, Sensors and Computing Devices
CAN Communication
========================

-----
Jetson Xavier AGX

----
Install NVIDIA SDK Manager
----


   1. Download and run SDK Manager on your host machine.

.. figure:: /images/Jetson-Xavier/SDK_Manager.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 200px
   
   SDK Manager from NVIDIA

   https://docs.nvidia.com/drive/archive/5.1.0.2L/sdkm/download-run-sdkm/index.html


   2. Launch SDK Manager from terminal, using the parameters below to run an installation from the command line. In our case we used the sdk manager from cli for the Jetson Xavier AGX

   Example:: 

   ./sdkmanager --cli install --user john.doe@example.com --logintype devzone --product Jetson --version 5.2 --targetos Linux --host  --target P2888 --flash all  

   https://docs.nvidia.com/sdk-manager/sdkm-command-line-install/index.html

   3. After downloding Linux for tegra, use source_sync.sh script to sync kernel and u-boot source code.
   4. Enter to Linux_for_Tegra/bootloader/t186ref/
   5. Decompile dtb to dts for editing: 
   
   ::

      dtc -I dtb -O dts -o tegra194-a02-bpmp-p2888-a04.dts tegra194-a02-bpmp-p2888-a04.dtb
   
   6. Change the clock to PLLC or PLLAON, the correct register is in Linux_for_Tegra/source/public/hardware/nvidia/soc/t19x/kernel-include/dt-bindings/clock/tegra194-clock.h
      In our case, the options were:
   ::

      #define TEGRA194_CLK_CLK_32K 289U //0x121, 32K input clock provided by PMIC
      #define TEGRA194_CLK_OSC 91U //0x5b, input from Tegra's XTAL_IN
      #define TEGRA194_CLK_PLLC 314U //0x13a, PLL controlled by CLK_RST_CONTROLLER_PLLC_BASE
      #define TEGRA194_CLK_PLLAON 94U //0x5e, PLL controlled by CLK_RST_CONTROLLER_PLLAON_BASE for use by IP blocks in the AON domain
   7. We used the PPLC so in the dts file, we modified it to:
   
   ::

      clock@can1 {
    
	   allow_fractional_divider = <0x01>;
	   allowed-parents = <0x121 0x5b 0x13a 0x5e>;
	   clk-id = <0x09>;
      };
      clock@can2 {
    
	   allow_fractional_divider = <0x01>;
	   allowed-parents = <0x121 0x5b 0x13a 0x5e>;
	   clk-id = <0x0b>;
      };

   8.  Compile back dts to dtb:
   ::

      dtc -I dts -O dtb -o tegra194-a02-bpmp-p2888-a04.dtb tegra194-a02-bpmp-p2888-a04.dts
   9.  Finally, you need to display  *** The [bpmp-fw-dtb] has been updated successfully. ***
   10.  Enter to Linux_for_Tegra
   11.  Copy the .cfg to the bootloader that we previously installed with the source_sync.sh::
   
   ::

      cp bootloader/t186ref/BCT/tegra194-mb1-bct-ratchet-p2888-0000-p2822-0000.cfg bootloader
   12.  Whether it is in Linux_for_Tegra folder or inside the jetson after flashing the device, You have to change the device tree node  mttcan@c310000, mttcan@c320000 and attached pllaon or pplc  and The clock.
   13.  Enter to Linux_for_Tegra/source/public/hardware/nvidia/platform/t19x/galen/kernel-dts/common/tegra194-p2888-0001-p2822-0000-common.dtsi
   14.  Modify the clock-init ::
   
   ::

      clocks-init {
    
		   compatible = "nvidia,clocks-config";
		   status = "okay";
		   disable {
    
			   // clocks = <&aon_clks TEGRA194_CLK_PLLAON>,
			   // 	<&bpmp_clks TEGRA194_CLK_CAN1>,
			   // 	<&bpmp_clks TEGRA194_CLK_CAN2>;
		   };
	   };
   15. Enter to Linux_for_Tegra/source/public/hardware/nvidia/platform/t19x/galen/kernel-dts/common/tegra194-p2888-0001-p2822-0000-common.dtsi
   16. Modify the device tree for pplaon or pplc ::
   ::

      mttcan@c310000 {
    
         status = "okay";
         pll_source = "pllaon";
         clocks = <&bpmp_clks TEGRA194_CLK_CAN1_CORE>,
            <&bpmp_clks TEGRA194_CLK_CAN1_HOST>,
            <&bpmp_clks TEGRA194_CLK_CAN1>,
            <&bpmp_clks TEGRA194_CLK_PLLAON>;
         clock-names = "can_core", "can_host","can","pllaon";
      };

      mttcan@c320000 {
      
         status = "okay";
         pll_source = "pllaon";
         clocks = <&bpmp_clks TEGRA194_CLK_CAN2_CORE>,
            <&bpmp_clks TEGRA194_CLK_CAN2_HOST>,
            <&bpmp_clks TEGRA194_CLK_CAN2>,
            <&bpmp_clks TEGRA194_CLK_PLLAON>;
         clock-names = "can_core", "can_host","can","pllaon";
      }; 

   17. Inside the Linux_for_tegra that has the bootloader, use the flash.sh. In our case it was the Jetson-Xavier AGX:
   ::

     sudo ./flash.sh -k bpmp-fw-dtb jetson-agx-xavier-devkit mmcblk0p1

   18. For more information about how to flash your board, read the /txt posted in the forum: https://forums.developer.nvidia.com/t/jetson-xavier-agx-error-flash-sh/174572/6 
   19. Check CAN Is your master clock  pll_aon,  If it is  pll_c  or  osc. If it is osc , it was not successful your tests.
   ::

     sudo cat /sys/kernel/debug/bpmp/debug/clk/can1/parent
     pll_aon
   20.  If the clock is changed to pllc or pll_aon continue to step 24 or check https://forums.developer.nvidia.com/t/how-to-use-vector-can-analyzer-in-jetson-tx2/63300/2.
   21.  Insert CAN BUS subsystem support module.
   
   ::

      modprobe can

   22. Insert Raw CAN protocol module (CAN-ID filtering)
   
   ::

      modprobe can_raw

   23. Real CAN interface support (for our case, it is: mttcan)
   ::

     modprobe mttcan (dependent module is can_dev: can driver with netlink support)
     
   24.   Disable can0 or can1 to change bitrate.
   
   ::

      sudo ifconfig can0 down


   25.  CAN interface settings for both the controllers, change the bitrate depending on the can device you are using.
  
   ::

      ip link set can0 type can bitrate 500000 dbitrate 2000000 berr-reporting on fd on
      ip link set up can0

      ip link set can1 type can bitrate 500000 dbitrate 2000000 berr-reporting on fd on
      ip link set up can1
   26.  CAN interfaces are up now. Use ifconfig to list all the interfaces which are up. Installation of user app to check CAN communication
  
   ::

      sudo apt-get install can-utils
   27. Commands to run to check CAN packet send/receive. Broadcasting a can data packet:
   
   ::

      cansend <can_interface> <can_frame>
      e.g. cansend can0 123#abcdabcd
      Receiving a can data packet:
      candump can_interface
      e.g. candump can1


   28. Different tools (i.e. cangen, cangw etc) can be used for various filtering options. To check the interface statistics
  
   ::

      ip -details -statistics link show can0
      ip -details -statistics link show can1
   29.  To read info from the Can Bus use candump.
   ::

      candump -ta -x can0
   30.  Important Info: To manually download and expand the kernel sources, in a browser, navigate to https://developer.nvidia.com/embedded/downloads, to locate and download the L4T source files for your release.


CAN STM32 
========================

The following table displays the CAN frame we use to interact with the Jetson-Xavier and SMT32 modules, including the header, data, and description to assist programmers understand the car's functions. 

**JETSON Xavier TX Messages**


   +-----------------+----------------+------+----------------------------------------------+
   | Module          | Jetson ID-TX   | Data | Description                                  |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 16   | Inferior Left Front  - Turn Off              |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 15   | Inferior Right Front - Turn Off              |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 14   | Upper Left Front - Turn Off                  |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 13   | Upper Right Front - Turn Off                 |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 12   | Horn - Turn Off                              |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 11   | Wiper - Turn Off                             |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 6    | Inferior Left Front - Turn On                |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 5    | Inferior Right Front - Turn On               |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 4    | Upper Left Front - Turn On                   |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 3    | Upper Right Front - Turn On                  |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 2    | Horn - Turn On                               |
   +-----------------+----------------+------+----------------------------------------------+
   |  Panel          | 333            | 1    | Wiper - Turn On                              |
   +-----------------+----------------+------+----------------------------------------------+
   |  Throttle       | 334            | X    | Wiper position change, data=Velocity steps   |
   +-----------------+----------------+------+----------------------------------------------+
   |  Throttle       | 335            | X    | Set Max Speed, data=Max speed                |
   +-----------------+----------------+------+----------------------------------------------+
   |  Steering wheel | 338            | X    | Set Steps, data=steps                        |
   +-----------------+----------------+------+----------------------------------------------+
   |  Steering wheel | 339            | 1    | Change of direction, right                   |
   +-----------------+----------------+------+----------------------------------------------+
   |  Steering wheel | 339            | 2    | Change of direction, left                    |
   +-----------------+----------------+------+----------------------------------------------+
   |  Steering wheel | 340            | 1    | Enable driver                                |
   +-----------------+----------------+------+----------------------------------------------+
   |  Brake          | 341            | X    | Set Steps, data=steps                        |
   +-----------------+----------------+------+----------------------------------------------+
   |  Brake          | 342            | 1    | Change of direction, right                   |
   +-----------------+----------------+------+----------------------------------------------+
   |  Brake          | 342            | 2    | Change of direction, left                    |
   +-----------------+----------------+------+----------------------------------------------+
   |  Brake          | 343            | 1    | Enable driver                                |
   +-----------------+----------------+------+----------------------------------------------+
   |  Transmission   | 344            | X    | Set Steps, data=steps                        |
   +-----------------+----------------+------+----------------------------------------------+
   |  Transmission   | 345            | 1    | Change of direction, right                   |
   +-----------------+----------------+------+----------------------------------------------+
   |  Transmission   | 345            | 2    | Change of direction, left                    |
   +-----------------+----------------+------+----------------------------------------------+
   |  Transmission   | 346            | 1    | Enable driver                                |
   +-----------------+----------------+------+----------------------------------------------+

**STM32 TX Messages**

   +-----------------+----------------+------+--------------+
   | Module          | STM32 ID-TX    | Data | Description  |
   +-----------------+----------------+------+--------------+
   |  Panel          | 222            | 1    | Success      |
   +-----------------+----------------+------+--------------+
   |  Panel          | 222            | 2    | Error        |
   +-----------------+----------------+------+--------------+
   |  Throttle       | 222            | 1    | Success      |
   +-----------------+----------------+------+--------------+
   |  Throttle       | 222            | 2    | Error        |
   +-----------------+----------------+------+--------------+
   |  Brake          | 222            | 1    | Success      |
   +-----------------+----------------+------+--------------+
   |  Brake          | 222            | 2    | Error        |
   +-----------------+----------------+------+--------------+
   |  Transmission   | 222            | 1    | Success      |
   +-----------------+----------------+------+--------------+
   |  Transmission   | 222            | 2    | Error        |
   +-----------------+----------------+------+--------------+  

The CAN Rx Task analyzes CAN frames to activate flags and variables.

.. figure:: /images/electronics_embedded/stm32-programming/CAN_rx.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 800px

The CAN Header analysis function is shown below.

.. figure:: /images/electronics_embedded/stm32-programming/CAN_parse_Msg.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 800px
   
The CAN Tx Task transmits success or error signals based on previous implementations. 

.. figure:: /images/electronics_embedded/stm32-programming/CAN_tx1.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 800px

.. figure:: /images/electronics_embedded/stm32-programming/CAN_tx2.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 800px

The Task Panel toggles GPIO based on CAN message data.

.. figure:: /images/electronics_embedded/stm32-programming/TaskPanel.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 800px

The TaskPot can modulate a digital potentiometer to regulate automobile velocity and set its maximum value depending on CAN message data.

.. figure:: /images/electronics_embedded/stm32-programming/TaskPot.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 800px

.. figure:: /images/electronics_embedded/stm32-programming/TaskPot2.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 800px

Common Module Subsystems
========================
SDV modules are similar (with slight variations) in certain subsystems, such as in IO, and power supply.

-----
STM32
-----

The `STM32L431RCT6 <https://www.st.com/en/microcontrollers-microprocessors/stm32l431rc.html` is chosen as the main microcontroller unit for all modules
to follow an standard started by the development of the USV and UUV PCBs, where a similiar version was used, which in turn eases the electronics and embedded development.

.. figure:: /images/electronics_embedded/stm32_base.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   STM32 Base Configuration

High-Speed External Clock
-------------------------

The `oscillator design guide for mcus <https://www.st.com/resource/en/application_note/an2867-oscillator-design-guide-for-stm8afals-stm32-mcus-and-mpus-stmicroelectronics.pdf>` was used to select the oscillator circuit components.

The high-speed external clock is supplied with a 16MHz as the crystal oscillator.

The selected `crystal <https://www.lcsc.com/product-detail/Crystals_Yangxing-Tech-X322516MLB4SI_C13738.html>` has a load capacitance (CL) of 9pF and, assuming a stray PCB capacitance (Cs) of 3pF, two 12pF capacitors (CL1, CL2) were chosen following the next equation.

.. figure:: /images/electronics_embedded/crystal_eq.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 200px
   
   Crystal Load Equation


.. figure:: /images/electronics_embedded/hsec.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 200px
   
   Oscillator Circuit

STM32 Programming, Supply and Communications
--------------------------------------------
An ST-Link is used to program the STM32 via a 10 pin header 1.27mm pitch.


A 5-pin M12 connector is used to supply +12V, and to connect the module to the CAN Bus. The 5th pin is left unused.

.. figure:: /images/electronics_embedded/prog.png
   :align: center
   :alt: hsec
   :figclass: align-center
   :width: 400px
   
   Programming, Supply and Comms

An ST-Link is used to program the STM32 via a 10 pin 1.27mm-pitch header.

A 5-pin M12 connector is used to supply +12V and to connect the module to the CAN Bus. The 4th pin is left unused, connected to GND.

.. figure:: /images/electronics_embedded/m12_pinout.png
   :align: center
   :alt: m12_pinout
   :figclass: align-center
   :width: 200px
   
   M12 Connector Pinout

CAN
---
The `TCAN330GD <https://www.ti.com/lit/ds/symlink/tcan332g.pdf?HQS=dis-mous-null-mousermode-dsf-pf-null-wwe&ts=1665117642045&ref_url=https%253A%252F%252Fwww.mouser.mx%252F>` is the CAN transceiver used to provide
an interface to the CAN bus.

.. figure:: /images/electronics_embedded/can.png
   :align: center
   :alt: can_transceiver
   :figclass: align-center
   :width: 500px
   
   CAN Transceiver Circuit

Module ID
---------
Each module can be mannually assigned an ID, which can be used for the CAN messages identification. A maximum of 16 IDs can be assigned.

.. figure:: /images/electronics_embedded/module_id.png
   :align: center
   :alt: module id
   :figclass: align-center
   :width: 200px
   
   Module ID selection


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

The power supply subsystem provides power to the whole module which we plan to use 12V, it contains:
   * A 1A fuse for overcurrent protection.
   * Reverse polarity voltage protection
   * 12V to 3.3V DC Converter
   * 12V to 5V DC Converter (optional)
   * LEDs as power indicators
   * NTC thermistor for the regulators temperature

Reverse Polarity Protection
---------------------------
For reverse polarity voltage protection with mosfets, a couple of things should be considered.

* The mosfet drain-source current (Ids) must withstand the module current requirements.
* The drain-source voltage (Vds) must be larger than BATT+.
   * A `NDS352AP <https://www.mouser.mx/ProductDetail/onsemi-Fairchild/NDS352AP?qs=mdiO5HdF0KhbUArAR6yyEg%3D%3D>` mosfet is chosen, with a Vds = 30V and the maximum value of Drain Current (ID) of -1.3 A.
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
   
   Base Reverse Polarity Voltage Protection Circuit

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
   
   IO Base Configuration

Stepper-Based Modules
=====================
The steering and pedal brake modules share essentially the same purpose: control stepper motors and read encoder and brake signals (optional).
For this reason, the PCB for both modules is exactly the same.

---
I/O
---
The brake module reads a signal proceeding from the pedal brake that indicates that the pedal is being pressed.

The steering module does not required external inputs.

Encoder
-------
The steering module considers an absolute encoder to provide feedback on the steering angle. The sensor used is the `RM8004 <https://www.ifm.com/es/es/product/RM8004>`.
This encoder is connected directly to the CAN bus.

.. figure:: /images/electronics_embedded/encoder_m12.png
   :align: center
   :alt: encoder_m12
   :figclass: align-center
   :width: 200px
   
   Encoder M12 Connector

   +-----+------------------------+
   | Pin |   Meaning              |
   +-----+------------------------+
   | 1   | CAN_GND                |
   +-----+------------------------+
   | 2   | VBBc                   | 
   +-----+------------------------+
   | 3   | GND (PE)               |
   +-----+------------------------+ 
   | 4   | CAN_HIGH               |
   +-----+------------------------+ 
   | 5   | CAN_LOW                |
   +-----+------------------------+

Stepper Driver Connection
-------------------------
--


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
Curtis speed controller
-----
Curtis speed controller is a device used for high precission motor speed control applications in electric vehicles.
This controller modulates speed based on the low current entries J1, J2 and J3.

.. figure:: /images/electronics_embedded/throttle_module/curtis_speed_controller.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   Curtis speed controller pins
|

   +----------+-------------------------------------------------+
   | Pins     | Description                                     |
   +----------+-------------------------------------------------+
   | J1       | Keyswitch, not used in this module              | 
   +----------+-------------------------------------------------+   
   | J2       | Wire 1 of 2-wire throttle. Must be connected to |
   |          | Potentiometer High or ITS Throttle terminal.    |                                         
   +----------+-------------------------------------------------+  
   | J3       | Wire 2 of 2-wire throttle. Must be connected to |
   |          | Potentiometer wiper, ITS Low terminal.          |
   +----------+-------------------------------------------------+  

-----
Digital Potentiometer
-----
This circuit can enable manual o automatic regulation of velocity with a digital potentiometer of 10k. We are powering the potentiometer with 12 Volts, to modulate velocity we used the inputs A,B,W , in the terminal A or B will be  entering 10V and the result will be going to the Wiper W and eventually will go to J3 from curtis controller.

DS3502U+
--------
This was the original potentiometer used for autonomous mode control. However, this is not the main one anymore. Although the main potentiometer is :ref:`MCP45HVX1 <MCP45HVX1>`, the circuit and logic behind works basically the same way.

* Pin description of DS3502U+ potentiometer:
  
.. figure:: /images/electronics_embedded/throttle_module/14_pot_pins.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   DS3502U+ potentiometer pin description

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

* DS3502U+ Potentiometer datasheet:
  
.. figure:: /images/electronics_embedded/throttle_module/15_pot_max_values.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   DDS3502U+ potentiometer maximum ratings

|

.. _MCP45HVX1:
MCP45HVX1
--------
It is the main digital potentiometer of the Throttle module. It works almost the same way than the DS3502U, but it includes some extra pins and another feautres.

This potentiometer also comunicates with the STM32 via I2C to modify or read the registers.

  
.. figure:: /images/electronics_embedded/throttle_module/IC_pot_view.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 700px
   
   MCP45HVX1 potentiometer pin view

|

  
.. figure:: /images/electronics_embedded/throttle_module/IC_pot_circuitView.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   MCP45HVX1 circuit view

|

* Pin description of MCP45HVX1:
  
.. figure:: /images/electronics_embedded/throttle_module/MCP45HVX1_pin_description.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px
   
   MCP45HVX1 potentiometer pin view

|

.. _MCP45HVX1_Programming:
Programming the MCP45HVX1
--------
The development of the functions related to the control of this device is performed inside **mcp45hvx1.c**
inside the throttle module STM32 project.

.. figure:: /images/electronics_embedded/throttle_module/Throttle_mcp45hvx1_init.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 530px
   
   mcp45HVX1.c device initialization

|

At first, there is the definition of I2C address of potentiometer (**pot_address**), the TCON register (Terminal Control)
that consists of 4 bits, and allows the developer which terminals (PA0, PW0, PB0) are going to be enabled and used for the
circuit. Also, it is defined the **enable_potentiometer** and finally the name of the I2C handler (**hi2c1**) configured in the project.

In **initialize_devices** it determines if the connection with the potentiometer has been established. If this is performed
successfully, then the potentiometer is enabled.

.. figure:: /images/electronics_embedded/throttle_module/Throttle_mcp45hvx1_beginPot.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 580px
   
   mcp45HVX1.c begin potentiometer function

|

**begin_pot** is the function where the initialization is completed. If the potentiometer is enabled, it starts setting the
wiper register with the initial value with the direction **0b00**

Then, the pin **WLAT** (wiper latch) is setted to LOW. As shown in the pin description, this allows the direct update of the 
wiper value. When it is HIGH, the new value received by the I2C message is first stored in the shift register. This feature 
allows the user to connect the wiper update using external events. For this implementation, this pin is only set once on LOW
and used as a reset.

After that, the **SHDN** pin is set to HIGH. Its functionality is basically to be an enable for the potentiometer terminals.

Finally, it is called **defaultTCON** that configures the Terminal Control register.

.. figure:: /images/electronics_embedded/throttle_module/Throttle_mcp45hvx1_writeWiper.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 700px
   
   mcp45HVX1.c write wiper function

|

The **writeWiper** function is used to update the value of the wiper calling **write_register** function on the 00 address.

.. figure:: /images/electronics_embedded/throttle_module/Throttle_mcp45hvx1_writeRegister.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 700px
   
   mcp45HVX1.c write register and send command functions

|

Two of the most important functios are **send_command** and **write_register**.

**write_register** is in charge of write the register of a given address calling send_command with the command parameter 
of **0b00** which is used to indicate a writing operation.

**send_command** receive and address, a command which is a int, the data to be sent and the data size. Basically, this
function executes all the logic needed to create the I2C message that will be sent to the potentiometer.

The **writeWiper** function is used to update the value of the wiper calling **write_register** function on the 00 address.

.. figure:: /images/electronics_embedded/throttle_module/Throttle_mcp45hvx1_defaultTCON.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 230px
   
   mcp45HVX1.c default terminal control register initialization

|

As mentioned before, the **defaultTCON** function is used to set the bits of the TCON register, and calls the function 
that writes it as write_wiper function does. This register is not being modified because the
right terminal is directly conected to the relay circuit using a solder jumper.

.. _digiPot:
Digital potentiometer pin description.
--------

   +----------+-------------------------------------------------------------------------------------------+
   | Pins     | Meaning                                                                                   |
   +----------+-------------------------------------------------------------------------------------------+
   | Pot      | Connected to GPIO of STM32 to switch motor.                                               |
   +----------+-------------------------------------------------------------------------------------------+
   | J2E      | This signal is connected to Curtis controller J2 , it has 10 Volts. When is manual mode,  |
   |          |                                                                                           |
   |          | it's connected to WiperPot, and to Digital pot terminals in autonomous mode               |
   +----------+-------------------------------------------------------------------------------------------+
   | WiperPot | Used for manual control, this signal corresponds to the HIGH terminal of manual pedal     |
   |          |                                                                                           |
   |          | potentiometer acoording to Curtis controller documentation.                               |
   +----------+-------------------------------------------------------------------------------------------+ 
   | J3       | The modulated signal will come from the wiper (W) and be connected straight to J3         |
   |          |                                                                                           |
   |          | CurtisController. This is also connected to the wiper terminal of the manual pedal        | 
   +----------+-------------------------------------------------------------------------------------------+ 


.. figure:: /images/electronics_embedded/throttle_module/digital_potentiometer_circuit_oct_2023.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 800px
   
   Digital Potentiometer schematic 

|
.. figure:: /images/electronics_embedded/throttle_module/13_pot_general_view.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 600px

   Throttle module in general diagram 

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

-----------------
Throttle module STM32 Programming
-----------------
On this section we will describe the most important pieces of the code running on the STM32 inside the Throttle module.

Pin definition
--------------

.. figure:: /images/electronics_embedded/throttle_module/STM_Throttle_PINs.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 650px
   
   STM32 Throttle module pin definition.
|

* Pin description:

+---------+------------+------------------------------------------------------------------------------+
| Pins    | Name       |   Description                                                                |
+---------+------------+------------------------------------------------------------------------------+
| PA6     | WLAT       | Wiper Latch of the potentiometer. Used as a reset. Set during initialization |
+---------+------------+------------------------------------------------------------------------------+
| PA7     | SHDN       | Shutdown pin of potentiometer. Used as enable. Set during initialization     |
+---------+------------+------------------------------------------------------------------------------+
| PA9     | Pot        | Pin used for changing the input of the Curtis Speed Controller: manual mode  |
|         |            |                                                                              |
|         |            | with 5k pedal or autonomous mode with digital potentiometer                  |
+---------+------------+------------------------------------------------------------------------------+
| PA10    | RelayMotor | Pin used to change the state of the motor relay (:ref:`see more <digiPot>`)  |
+---------+------------+------------------------------------------------------------------------------+
| PA11    | CAN1_RX    | Pin assigned to CAN RX channel                                               |
+---------+------------+------------------------------------------------------------------------------+
| PA12    | CAN1_TX    | Pin assigned to CAN TX channel                                               |
+---------+------------+------------------------------------------------------------------------------+
| PB6     | I2C1_SCL   | Clock of I2C interface for the digital potentiometer                         |
+---------+------------+------------------------------------------------------------------------------+
| PB7     | I2C1_SDA   | Data channel of I2C interface for the digital potentiometer                  |
+---------+------------+------------------------------------------------------------------------------+
| PA3     | hand_brake | Input pin for the hand_brake signal                                          |
+---------+------------+------------------------------------------------------------------------------+

main.c
------
The main file is in charge of calling all the initialization of ports, devices, interfaces and tasks to
be excecuted.

.. figure:: /images/electronics_embedded/throttle_module/Throttle_main_initializations.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 440px
   
   main.c initialization
|

It starts with the initializaton of canlib with ID **VANTTEC_CAN_ID_THROTTLE_TX** to declare the 
message transceiver id, after that, it initializes the functionality of transmission and reception of 
CAN messages, and the basic tasks of receive, send, reed and hanlde the message queue.

As show before, **initialize_devices** and **begin_pot** do the initial setup for the digital potentiometer.

Then, it initializes the proper tasks of the throttle as **init_requirements_task** where there are the 
functions that hanldes the basic requirement signals as emergency stop, reverse, etc.; manipulating the values
of the RX messages. On the other hand, **init_throttle_tasks** there are tasks that update the values of the pins,
read the hand brake input and control the digital potentiometer.

Throttle tasks
--------------

.. figure:: /images/electronics_embedded/throttle_module/Throttle_throttleTask_init.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 680px
   
   Throttle tasks initialization

|

This function creates all the threads for each task to perfom the control of Throttle module:

* Manage and update the potentiometer.
* Enable the motor.
* Switch of drive mode.
* Reading of hand brake input.

.. figure:: /images/electronics_embedded/throttle_module/Throttle_throttleTask_pot.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 800px
   
   Potentiometer task

|

**pot task** is in charge of receive the messages of the CAN device id of **VANTTEC_CAN_ID_THROTTLE_RX**, 
and message id **0x05** (this is the one assigned to messages of digital pot value). The data is stored on
**pot_data** variable.

In the excecution loop, it performs the logic to update the digital potentiometer. If it is a different value 
from the last one, checks if it is above the maximum permitted value, in case of true, then it sets the value 
to be the maximun. Also, it checks if it is 0, and then corrects it to 01. Finally, it updates the wiper calling
**writeWiper** and updates the last data variable.

.. figure:: /images/electronics_embedded/throttle_module/Throttle_throttleTask_motor.png
   :align: center
   :alt: stm32 schematic
   :figclass: align-center
   :width: 800px
   
   Motor enable task

|

In the **motor_task**, it will subscribe the messages of the CAN device id of **VANTTEC_CAN_ID_THROTTLE_RX**, 
and message id **0x06** (this is the one assigned to messages of motor signal). The data is stored on
**motor_data** variable.

In the excecution loop the task will check if motor_data is 1, in this case it will write a HIGH on the 
RelayMotor pin to activate the relay. In any other case it will write a LOW and return the Relay to default.

When the relay is activated, the motor is energized directly (this is used for autonomous mode).
If it is on default/common-closed state, the motor will only be powered when the driver press the manual pedal
and activates the pedal swtich (this is used for manual mode).

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