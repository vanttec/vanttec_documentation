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
  
* Repositories intended to be used in multiple projects must be named as vanttec_(TECHNICAL-AREA)
    * vanttec_perception
    * vanttec_control
    * vanttec_planning
  
Branch naming (based on GitFlow)
--------------------------------

* Please use descriptive names about the project you are working on and the purpose of the branch.

* main/master - the main branch for the project, this branch tracks release code only.
* release_v_#_# - used to develop software for extended periods of time (one semester/year), which is to be merged to main once development is completed.
    * Considerable large changes such as the adoption of a new ROS version, large package changes, or a completely new or majorly improved phisically tested vehicle feature, should change the first digit (as in version 1.0 to version 2.0)
    * Small changes, such as untested in real life improvements or features, change the second digit (such as in version 1.1 to version 1.2).
* feature - this is the branch used for new developments. 
* hotfix - used for emergency fixes.
 
* If possible, avoid modifying code directly on the release_candidate and the main/master branches. Instead create a new branch and open a pull request after the desired changes are completed.

Pull requests
-------------

