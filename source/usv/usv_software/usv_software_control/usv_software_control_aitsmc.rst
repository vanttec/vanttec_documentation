AITSMC
=========

Adaptive integral terminal sliding mode control is a control strategy that is employed in the USV project. 
It was first developed in the earlier years of VantTec and has proven to be a simple and usable strategy. 
Currently, it is employed within the repository’s usv_control package as aitsmc_new_node, which receives a setpoint, 
alongside state information, and realizes the control computation, finally publishing thrust values. 
This control strategy became a largely published topic within Vanttec, primarily by 
`Alejandro Gonzalez Garcia <https://scholar.google.com.mx/citations?user=j5zC8uMAAAAJ&hl=es>`__ and 
`Ivana Collado Gonzalez <https://scholar.google.com.mx/citations?user=5VZAzoUAAAAJ&hl=es>`__, alongside
`Dr. Herman Castañeda Cuevas <https://scholar.google.com.mx/citations?user=aAbtWIkAAAAJ&hl=es>`__.

Here is one of such works: :download:`Adaptive Integral Terminal Sliding Mode
Control for an Unmanned Surface Vehicle
Against External Disturbances <Adaptive Integral Terminal Sliding Mode
Control for an Unmanned Surface Vehicle
Against External Disturbances.pdf>`