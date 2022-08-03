==========================
Github for version control
==========================

Git tutorials for begginers
===========================

For new git users, here are some useful tutorials for reference.

An intro to Git and GitHub for begginers:
| https://product.hubspot.com/blog/git-and-github-tutorial-for-beginners

Git Tutorial for Beginners 2022:

.. raw:: html

    <iframe width="560" height="315" src="https://www.youtube.com/embed/eeuNAIZoWRU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


Git standards and recommendations for VantTec
=============================================

Repository naming
-----------------

* Repositories must have short descriptive names, separated by underscores. Vehicle projects should be named as vanttec_(VEHICLE-ACRONYM)
    * vanttec_uuv
    * vanttec_sdv
    * vanttec_drone/uav
    * vanttec_usv

* Submodules should be named after the vehicle project repository they are part of, for example (VEHICLE-ACRONYM)_(TECHNICAL-AREA)
    * uuv_control
    * uuv_perception
    * uuv_comms
  
* Repositories intended to be used in multiple projects must be named as follows vanttec_(TECHNICAL-AREA)
    * vanttec_perception
    * vanttec_control
    * vanttec_planning
  
Branch naming
-------------

* Please use descriptive names about the project you are working on and the purpose of the branch.

Recommended branch names:

* main - the main branch for the project, this is the branch that is used for new releases.
* feature - this is the branch that is used for the implementation of a new development. 
* hotfix - bug solution branch.
* release_candidate_v_#_# - used to develop software for extended periods of time (one semester/year), which is to be merged to main once development is completed.
    * Considerable large changes such as the adoption of a ROS new version, large package changes, or a considerably new phisically tested vehicle ability, should change the first digit (as in version 1.0 to version 2.0), little changes such as untested in real life improvements change the second digit (such as in version 1.1 to version 1.2).
  

Pull requests
-------------



