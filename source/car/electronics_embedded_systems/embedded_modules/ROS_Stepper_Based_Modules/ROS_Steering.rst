Steering Node (enconder_rm8004.py)
##################################

The streering node starts can communication with rm8004 encoder, starts encoder in operational mode, listens to data recieved in the bus, interprets the data, publishes encoder interpreted data to topic "ifm_encoder"  and publishes the absolute steering angle to topic "sdc_state_steering" 

File Overview
=========
The file overview will explain the important parts of the node and furnish essential information about its overall function and key features

Main:
^^^^^
The main function initializes the node and creates an instance of RM8004Encoder() which itself is an encoder publisher

RM8004Encoder Class:
^^^^^^^^^^^^^^^^^^^

The RM8004Encoder Class represents an interface for the IFM RM8004 encoder in Ros2, it inherits the Node ros2 base class and initializes
the following attributes:

#. steps: steps per revolution
#. revolution: revolution per unit
#. degrees: maximum degrees
#. encoder_data: instance of Encoder(), Encoder is a custom message type that stores encoder data (see sdv_msgs/msg/Encoder.msg)
#. encoder_id: node number id, according to encoder documentation: the address of an encoder without terminal cap is set to 32 as a standard
#. bit_res: bit resolution
#. car_steering_range: steerinng angle range
#. car_steering_pos: variable that stores the steering position in encoder steps besed on the specified steering range, encoder steps in one revolution and the total angles of rotation. The result of: self.car_steering_range * self.revolutions is the amount of encoder steps that are in the entire steering angle range and that result divided by degrees results in the total amount of steps in certain degree, giving you a step "position"

Parameters:

After the initial attributes are declared, two parameters are declared: "channel" of type string and "bitrate" of type int using the declare_parameter() function and retrieving their value using the get_parameter() function. The value of "channel" and "bitrate" are declared in a yaml file (see sdv_can/config/can_params.yaml) and made accesible by putting them in the share directory of the package using get_package_share_directory() function (see sdv_can/launch/can_devices.launch.py ) 

Publishers:

Two important publishers are created:

.. figure:: /images/ROS_steering_node/publishers.png
    :align: center
    :alt: publishers
    :figclass: align-center
    :width: 600px

These two publishers publish encoder data to "ifm_enncoder" topic and a steerung angle to the "/sdc_state/steering" topic

Starting can communication:

.. figure:: /images/ROS_steering_node/can_com.png
    :align: center
    :alt: publishers
    :figclass: align-center
    :width: 600px

For aditional information about the can.interface.Bus(), can.Message() and bus.send() functions visit the official documentation of the used can library: https://python-can.readthedocs.io/en/stable/api.html

NewPrinter Class:
^^^^^^^^^^^^^^^^^

At the end of RM8004Encoder() class an instance of NewPrinter() class is created, the object is then passed to the can.Notifier() function. The can.Notifier function is used as a message distributor for a bus. Notifier creates a thread to read messages from the bus and distributes them to listeners.

.. figure:: /images/ROS_steering_node/notifier.png
    :align: center
    :alt: publishers
    :figclass: align-center
    :width: 600px

The NewPrinter() class is used to create a listener that can be called directly to handle the CAN messages sent. The NewPrinter() class implements the on_message_recieved funtion that is executed when theres a new message on the bus (visit https://python-can.readthedocs.io/en/stable/listeners.html#can.Listener for more info)

When on_message_recieved executes the data is interpreted and processed so it can be usable, the data is decoded, converted to decimal, the absolute position is adjusted and a step count is calculated based on the number of steps. If the absolute position exceeds half of the total resolution it is adjusted so it falls
in the acceptable range. The data is  published to the encoder and steering angle topics discussed previously.

.. figure:: /images/ROS_steering_node/publish_on_recieve.png
    :align: center
    :alt: publishers
    :figclass: align-center
    :width: 600px