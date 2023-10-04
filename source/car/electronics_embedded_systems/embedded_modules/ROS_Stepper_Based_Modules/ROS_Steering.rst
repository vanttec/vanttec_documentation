Steering Node (enconder_rm8004.py)
================

The streering node starts can communication with rm8004 encoder, starts encoder in operational mode, listens to data recieved in the bus, interprets the data, publishes encoder interpreted data to topic "ifm_encoder"  and publishes the absolute steering angle to topic "sdc_state_steering" 

