==========================
Github for version control
==========================

Git tutorials for begginers
===========================

For new git users, here are some useful tutorials for reference.

An intro to Git and GitHub for begginers:

https://product.hubspot.com/blog/git-and-github-tutorial-for-beginners

Git Tutorial for Beginners 2022:

.. raw:: html

    <iframe width="560" height="315" src="https://www.youtube.com/embed/eeuNAIZoWRU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


Github standards and recommendations for VantTec
================================================

Repository documentation
------------------------

* All repositories must have a detailed description of their purpose, both in a README.md file and in Read The Docs

Repository naming
-----------------

* Repositories must have short descriptive names, separated by underscores. Vehicle projects must be named as vanttec_(VEHICLE-ACRONYM)
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
  


Branch naming (based on GitFlow)
--------------------------------

* main/master - the main branch for the project, and tracks release code only.
* release_v_#_# - used to develop software for extended periods of time (one semester/year).
    * Considerable large changes such as the adoption of a new ROS version, large package changes, or a completely new or majorly improved phisically tested vehicle feature, should change the first digit (as in version 1.0 to version 2.0)
    * Small changes, such as untested in real life improvements or features, change the second digit (such as in version 1.1 to version 1.2).
    * Once development is completed, stable, with code complying with standards, and tested, it is merged to the main branch.
    * When the release is merged, a tag must be created for the specified release version of each vehicle repository (check the USV repository for examples).
        * This should be performed by project captains.
* feature - used for new developments.
    * E.g: feature/control/pid_controller
* hotfix - used for emergency fixes.
    * E.g: hotfix/some_problem
* refactor - used when restructuring code is required, without altering its behaviour, and by making it easier to understand. Expected not to be used frequently, as features should be made properly. Documenting code could count as a refactor.
    * E.g: refactor/perception/documentation

Notes:
------

* If possible, avoid modifying code directly on the release_candidate and the main/master branches. Instead create a new branch and open a pull request after the desired changes are completed.
* If many features are being developed for a single technical area, name the branches as follows:
    * feature/perception/yolo_detection
    * feature/perception/yolact_detection
    * feature/perception/opencv_shape_filter

Forking
-------

* If using a third party repository to develop an application for a project, do not upload it as a new repository in VantTec's organization!!! Instead, fork the desired repository in this organization.

Contributions
-------------

Contributions to the release_candidate or main branches must be made through pull requests. Developers must open a pull request to merge changes from branches other than the previously mentioned ones.

To start a pull request, click the corresponding tab at the top of the repository in Github.

.. figure:: /images/git_1.png
   :align: center
   :alt: vanttec_documentation
   :figclass: align-center
   :target: vanttec_documentation
   :height: 150px
   :width: 600px


Click on the New pull request button, and choose the base branch on which the changes will be merged and Create pull request.

.. figure:: /images/git_2.png
   :align: center
   :alt: vanttec_documentation
   :figclass: align-center
   :target: vanttec_documentation
   :height: 150px
   :width: 550px


Once open, ALWAYS request a review from the code owner or the project leader before merging the pull request. To do so, click the cog symbol and chose the appropriate reviewer.


.. figure:: /images/git_3.png
   :align: center
   :alt: vanttec_documentation
   :figclass: align-center
   :target: vanttec_documentation
   :height: 150px
   :width: 550px

For reviewers
--------------

* Check that the contribution at least complies with the VantTec software development standards.
* Additional requests may be made by reviewers.
* When reviewing the files, comments can be left at any desired line of code by clicking in the + blue button. It is also possible to select multiple lines.


.. figure:: /images/git_4.png
   :align: center
   :alt: vanttec_documentation
   :figclass: align-center
   :target: vanttec_documentation
   :height: 200px
   :width: 600px


For Developers
--------------

* If required, perform proper modifications to the code on request of the reviewers, and reply to comments with a done once completed.
* Once all reviews are passed, and no merge conflicts are present, the pull request can be merged.

.. figure:: /images/git_5.png
   :align: center
   :alt: vanttec_documentation
   :figclass: align-center
   :target: vanttec_documentation
   :height: 400px
   :width: 600px

* After the pull request is closed, delete the branch if no longer necessary.