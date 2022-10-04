Embedded Modules
================
================

Seven embedded modules provide autonomous capabilities to the self-driving vehicle.

.. figure:: /images/electronics_embedded/sdv_systems.png
   :align: center
   :alt: sdv_systems
   :figclass: align-center
   :height: 400px
   :width: 800px


Common Module Subsystems
========================
SDV modules are similar (with slight variations) in certain subsystems, such as in IO, and power supply.

------------
Power Supply
------------

.. figure:: /images/electronics_embedded/power_supply.png
   :align: center
   :alt: power_supply
   :figclass: align-center
   :height: 400px
   :width: 500px

The power supply subsystem provides power to the whole module. It has a fuse at the very input to protect the circuit for currents over
1A, reverse polarity protection, one or two power regulators, leds for power indicators and a NTC thermistor to check the regulators temperature.

Reverse polarity protection
---------------------------
For reverse polarity voltage protection with mosfets, a couple of things should be considered.

* The drain-source voltage (Vds) must be larger than BATT+.
   * A `NDS352AP <https://www.mouser.mx/ProductDetail/onsemi-Fairchild/NDS352AP?qs=mdiO5HdF0KhbUArAR6yyEg%3D%3D>` mosfet is chosen, with a Vds = 30V.
* The gate-source voltage (Vgs) must not be surpassed by BATT+. A zener must be added to protect the mosfet in the case that BATT+ is larger than Vgs.
   * The NDS352AP Vgs = 20V, so a `MMSZ4702T1G https://www.lcsc.com/product-detail/Zener-Diodes_onsemi-MMSZ4702T1G_C242274.html` zener with a Zener voltage of 15V is chosen.

Stepper-based Modules
=====================
The steering and pedal brake modules share essentially the same purpose: control a stepper motor and read encoder signals.
For this reason, the PCB for both modules is exactly the same.