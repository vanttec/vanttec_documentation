Comms
=======

Author:
**Max Pacheco Ramírez**

.. contents:: To enable remote communication from the OGS (On-Ground-Station) to the boat (Jetson Computer), the following tasks are performed:
  :depth: 2
  :local:

XBee Radios Configuration
-----
Using the XCTU Software, both XBee devices are set to a baud rate of 230_400 kbps and in Transparent Mode

Getty service
-----
Based on this `blog <https://0pointer.de/blog/projects/serial-console.html>`__ detailing instructions for getty service configurations, the following lines are added to a new file ``/etc/systemd/system/custom-getty@.service`` in the Jetson:

.. code-block:: shell
   
   [Unit]
   Description=Custom Serial Getty on %I
   Documentation=man:agetty(8)
   After=systemd-user-sessions.service plymouth-quit-wait.service
   After=rc-local.service

   [Service]
   ExecStart=-/sbin/agetty -L 230400 %I vt220
   Type=idle
   Restart=always
   UtmpIdentifier=%I
   TTYPath=/dev/ttyXBEE%I
   TTYReset=yes
   TTYVHangup=yes
   
   [Install]
   WantedBy=multi-user.target

To enable this service (EVERY TIME), see the bash scripts on the ``.setup`` directory in vanttec_usv repository, in branch ``develop``. Also, if you're having trouble with the service, see the ``README`` file on the same directory for debugging.

Udev rules
-----
Udev rules are set so the custom getty service points to the ``/dev/ttyXBEE`` device, to make on-site testings faster. These work by binding a usb device with a specific vendor and product id unique to each device. In this case, as only one XBee device (at most) will be connected to either the OGS or the boat, both XBees are bound to the ``/dev/ttyXBEE`` identifier. This also makes it so it doesn't matter which XBee you have, you will always refer to the device on ``/dev/ttyXBEE``. Furthermore, add this lines to the udev rule file ``/etc/udev/rules.d/99-xbee.rules`` both in the Jetson and in your device, and reconnect the XBee radio:

.. code-block:: shell
  
   SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", SYMLINK+="ttyXBEE"
   SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6015", SYMLINK+="ttyXBEE"


Connect!
------------
Wowie! You should now be able to access the Jetson's login prompt connecting to your XBee device using minicom on CLI:

.. code-block:: shell

   sudo minicom -D /dev/ttyXBEE -b 230400 -c on
