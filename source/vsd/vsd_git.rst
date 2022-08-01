===
Git 
===

Git tutorials for begginers
===========================

For new git users, here are some tutorials that helped the VantTec team to create the VantTec software.
We hope you find them usefull.

An intro to Git and GitHub for begginers:
https://product.hubspot.com/blog/git-and-github-tutorial-for-beginners

Git Tutorial for Beginners 2022:

.. raw:: html

    <iframe width="560" height="315" src="https://www.youtube.com/embed/eeuNAIZoWRU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


Git standards and recommendations for VantTec
=============================================

Repository naming
-----------------

* Al files should be names with short descriptive names and separated by underscores.
* for example, al vehicle projects should be named as vanttec_(VEHICLE ACRONYM)
    * vanttec_uuv
    * vanttec_sdv
    * vanttec_drone/uav
    * vanttec_usv

* for all submodules they should be named after the vehicle project repo that you are working in, for example (VEHICLE ACRONYM)_(THECNIC_AREA)
    * uuv_control
    * uuv_perception
    * uuv_comms
  
* for multipourpose Repository software that are used in multiple vehicles, the naming should be as follows vanttec_(THECNIC_AREA)
    * vanttec_perception
    * vanttec_control
    * vanttec_planning
  

Branch naming
-------------

* Please use descriptive names about the project you are working on and the purpose of the branch.
  
Here are some of the best practices for branch naming:
* main - the main branch for the project, this is the branch that is used for the release.
* feature - this is the branch that is used for the implementation of a new feature development. 
* hotfix - Bug solution branch.

For example, if you are working on the VantTec software, you can use the following branch names:
* realease_candidate_v_#_# -> on this branch you will work for the year period development.
* Big changes as the change of ROS version, packages changes, or a new proven code, should change the first number as version 1.0 to version 2.0, little changes as hotfiz should change as version 1.1 to version 1.1.
  


