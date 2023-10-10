Braking ROS Node (encoder_briter.py)
#####################################

This is the braking system encoder node, it is pretty simple, there is no configuration mode, you do the configuration on the fly and it can save the previous config thanks to its internal memory.
This encoder is a Briter CANbus Multi-turn Absolute Rotary Encoder with 24 revolutions, 12-bit resolution.

Datasheet/user manual here: https://briterencoder.com/wp-content/uploads/2021/09/CAN-Multi-Turns-User-manual-V2.2.pdf

File Overview
^^^^^^^^^^^^^
The file overview will explain the important parts of the node and furnish essential information about its overall function and key features

Main
^^^^^
The main function initializes the ros node and creates an instance of the Encoder which has the setup of the encoder and a publisher of the Encoder message (Encoder.msg custom ROS message)

EncoderPublisher Class
^^^^^^^^^^^^^^^^^^^^^^

This class represents an interface for the Briter encoder in ROS2, it inherits the Node class from ROS2 and initializes the following attributes:

#. id: Encoder id in hex
#. steps: steps per revolution
#. degrees: degrees
#. timer_period: period between the timer_callback executions (more about timer_callback below)

Publishers:

.. figure:: /images/ros_braking_node/braking_publisher.png
    :align: center
    :alt: publishers
    :figclass: align-center
    :width: 600px

This publisher publishes encoder data to "encoder_freno" topic and it publishes at a 10Hz rate.


CAN Interface:

.. figure:: /images/ros_braking_node/can_interface.png
    :align: center
    :alt: can_interface
    :figclass: align-center
    :width: 600px

For aditional information about the can.interface.Bus() function visit the official documentation of the used CAN library: https://python-can.readthedocs.io/en/stable/api.html

An instance of NewPrinter() class (more about this class below) is created as self.listener, the object is then passed to the can.Notifier() function in the instantiation of the self.notifier variable. The can.Notifier function is used as a message distributor for a bus. Notifier creates a thread to read messages from the bus and distributes them to listeners.

.. figure:: /images/ros_braking_node/listener_creation.png
    :align: center
    :alt: listener
    :figclass: align-center
    :width: 600px

.. figure:: /images/ros_braking_node/can_notifier.png
    :align: center
    :alt: notifier
    :figclass: align-center
    :width: 600px


After this, we have several functions for different purposes in the encoder.

==================== =========================================================== ===============================================================================================================================================================================================================
Name                 Arguments                                                      Functionality
==================== =========================================================== ===============================================================================================================================================================================================================
shutdown             None.                                                          Calls the CAN notifier stop function.
timer_callback       None.                                                          Calls the ask_position function with the self.id as the argument.
ask_position         id (Encoder CAN ID)                                            Defines a CAN message variable with the data array that gets the position from the Encoder (check the datasheet's CAN commands data table at the start of this page for more information) and then it sends the message through the CAN bus.
query_mode           id (Encoder CAN ID)                                            Sets the encoder mode to query through a can.Message variable as the function above and also sends it through the CAN bus.
cambiar_id           id, new (Encoder CAN ID, desired ID)                           Creates the CAN message and it sends it through the bus.
cambiar_baudrate     id, baud (Encoder CAN ID, desired baudrate)                    If the baudrate argument matches one of the baudrate options it creates the CAN message and then it sends it throught the bus, if it doesn't match any option it raises an Exception.
set_return_time      id, microsegundos (Encoder CAN ID, microseconds value)         Sets the automatic return time of the encoder. Not working at the moment of writing this, more info about this functionality in the datasheet.
cambiar_posicion     id, pos (Encoder CAN ID, desired position)                     Sets the current position value of the encoder. Not working at the moment of writing this, more info about this functionality in the datasheet.
position_reset       id (Encoder CAN ID)                                            Sets the current position to zero. Creates the CAN message and then sends it through the CAN bus.
==================== =========================================================== ===============================================================================================================================================================================================================

NewPrinter Class
^^^^^^^^^^^^^^^^^

The NewPrinter() class is used to create a listener that can be called directly to handle the CAN messages sent. The NewPrinter() class implements the on_message_received function that is executed when theres a new message on the bus (visit https://python-can.readthedocs.io/en/stable/listeners.html#can.Listener for more info)

When on_message_received is executed, the data is interpreted and processed so it can be usable, the data is decoded, the absolute position is adjusted and a step count is calculated based on the number of steps. Then it initializes the message to be sent as send_msg using the Encoder() custom message. After that it calculates the angle, the absolute angle and the turn, and if the angle is more than 30 degrees it calls the position_reset function from the Encoder. The data is  published to the "encoder_freno" topic mentioned previously.
