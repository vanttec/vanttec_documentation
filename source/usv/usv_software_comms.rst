Comms
=======

Author:
**Max Pacheco Ramírez**

.. contents:: To enable remote communication from the OGS (On-Ground-Station) to the boat (Jetson Computer), the following tasks are performed:
  :depth: 2
  :local:

Set symlinks for device modularization
-----

Because of the number of USB devices connected to the Jetson and the OGS, maintaining an order in which they must be plugged in can be confusing, and when dealing with the competition and all the hurries, this can result in a source for failures. That’s what symlinks are for!

For serial usb devices
^^^^^

Udev rules are set so the custom getty service to access to the boat’s terminal points to a Holybro radio device, to make on-site testings faster. These work by binding a usb device with a specific vendor and product id unique to each device. In this case, as only one Holybro radio (at most) will be connected to either the OGS or the boat, both radios are bound to the ``/dev/ttyHOLY`` identifier. This also makes it so it doesn’t matter which radio you have, you will always refer to the device on ``/dev/ttyHOLY``. The same is done for the XBee radios (here, which radio you connect matters, because each runs a different ROS 2 node, and there are cases in which you want to connect both XBees to your computer, like when debugging for the USV’s web interface. Also the IMU is configured this way, for the same reason that plugging order shouldn’t matter.

To implement this, add this lines to the udev rule file ``/etc/udev/rules.d/99-dev.rules`` both in the Jetson and in your device:

.. code-block:: shell

  # /etc/udev/rules.d/99-dev.rules

  SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{serial}=="00000000", SYMLINK+="ttyXBEESTATION"
  SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6015", ATTRS{serial}=="D30DS1HZ", SYMLINK+="ttyXBEEBOAT"

  SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6015", ATTRS{serial}=="D30GM3SB", SYMLINK+="ttyHOLY"
  SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6015", ATTRS{serial}=="D30GJX3L", SYMLINK+="ttyHOLY"

  SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{serial}=="FT0C9STB", SYMLINK+="ttyIMU"

Next, reset your device’s udev rules with the following command:

.. code-block:: shell

  sudo udevadm control --reload-rules && sudo udevadm trigger


For network devices
^^^^^

For the usb canable adapter, a similar approach is followed. Add the following lines to the file ``/etc/systemd/network/10-can.link`` in the Jetson and your OGS, so both our Geschwister Schneider CAN type adapters are linked to the same interface in case we switch between one or another.

.. code-block:: shell

  # /etc/systemd/network/10-can.link

  [Match]
  Driver=gs_usb

  [Link]
  Name=can_vtec

Next, update the interface links:

.. code-block:: shell

  sudo update-initramfs -u
  sudo reboot


Also, you should consider adding the following script to an accessible file so that you can easily enable the can interface on your computers:

.. code-block:: shell

  #!/bin/bash
  sudo modprobe can
  sudo modprobe can_raw
  sudo modprobe mttcan

  sudo ip link set down can_vtec
  sudo ip link set can_vtec type can bitrate 125000
  sudo ip link set up can_vtec


Enable remote access to boat’s terminal
-----

In order to control global, important, or not necessarily ROS-related commands on the remote computer, Getty services are implemented.


Set RF Modules
^^^^^

Holybro SiK Telemetry Radios are used for this task, configured at 115,200 kbps on a separate channel to avoid interference with the RTK information being sent from another pair of the same radios, or the radios from other teams.


Getty
^^^^^

Based on this `blog <https://0pointer.de/blog/projects/serial-console.html>`__ detailing instructions for getty service configurations, the following lines are added to a new file ``/etc/systemd/system/custom-getty@.service`` in the Jetson:

.. code-block:: shell

  # /etc/systemd/system/custom-getty@.service
  [Unit]
  Description=Custom Serial Getty on %I
  Documentation=man:agetty(8)
  After=systemd-user-sessions.service plymouth-quit-wait.service
  After=rc-local.service

  [Service]
  ExecStart=-/sbin/agetty -L 115200 %I vt220
  Type=idle
  Restart=always
  UtmpIdentifier=%I
  TTYPath=/dev/ttyHOLY%I
  TTYReset=yes
  TTYVHangup=yes

  [Install]
  WantedBy=multi-user.target

To enable this service (every time before you’re starting the autonomy challenge), use the bash scripts on the ``.setup`` directory in vanttec_usv repository (if not in main branch, in branch develop). Also, if you’re having trouble with the service, see the ``README.md`` file on that hidden directory for debugging.

Connect!
^^^^^

Wowie! You should now be able to access the Jetson’s login prompt connecting to your XBee device using minicom on CLI:

.. code-block:: shell

  sudo minicom -D /dev/ttyHOLY -b 115200 -c on

Remote access to ROS 2 topics
-----

In order to have a visualization of the autonomous system's status, because just doing a topic echo on the transparent mode from getty services isn't as visually understandable as RViz, ROS 2 topics are sent on separate serial devices: the XBee S3B RF Modules.

XBee radios configuration
^^^^^
Using the XCTU Software, both XBee devices are set to a baud rate of 230,400 kbps and in API mode. The configuration parameters include:

- AP (API Enable) = API Mode with Escapes [2]
- BD (Interface Data Rate) = 230400 [7]
- NI (Node Identifier) = ``BOAT_XBEE`` for boat device, ``STATION_XBEE`` for ground station
- ID (Network ID) = Same value for both devices to ensure communication

ROS 2 nodes
^^^^^

In order to send the relevant variables for debugging from the remote device to the ground station, a ROS 2 interface through 2 nodes was developed. The logics for the XBee ROS 2 interface are:

- **Configuration**

   - Uses a YAML configuration file to specify:

     - Topic names and message types
     - XBee port configurations
     - Update rates
     - Enabled/disabled topics

   - Allows easy modification of transmitted data without code changes

- **Boat Node (xbee_boat_node)**

   - Subscribes to selected ROS 2 topics on the USV
   - Processes incoming messages into transmittable data
   - Packs data efficiently with message IDs and size information
   - Sends data chunks through XBee radio
   - Supports both numeric and string data types
   - Uses configurable update rate (default 10Hz)

- **Station Node (xbee_station_node)**

   - Receives packed data from boat node
   - Unpacks data based on message type
   - Republishes data to corresponding ROS 2 topics with ``/usv_comms`` prefix
   - Handles different message types appropriately:

     - Standard messages (pose, velocity, etc.)
     - Object lists with mixed data types

   - Provides error handling and logging

- **Message Protocol**

   - Efficient binary protocol for data transmission
   - Message structure:

     - Message ID (1 byte)
     - Message Size (2 bytes)
     - Packed data (variable length)

   - Type-aware packing for both numbers and strings
   - Error detection and validation

- **Key Features**

   - Modular design with base class for common functionality
   - Configuration-driven topic selection
   - Robust error handling and logging
   - Automatic network discovery
   - Bandwidth-efficient message packing
   - Support for complex message types (like object lists)

- **Usage**

   - Start nodes:

     .. code-block:: shell

       # On the boat:
       ros2 run usv_comms xbee_boat_node
       
       # On the ground station:
       ros2 run usv_comms xbee_station_node
     
   - Monitor topics:

     .. code-block:: shell

       # View republished topics
       ros2 topic list | grep "/usv_comms"

This implementation provides a reliable and configurable way to monitor the autonomous system's state remotely, enabling better debugging and visualization through tools like RViz while maintaining efficient bandwidth usage through careful message packing and topic selection.

