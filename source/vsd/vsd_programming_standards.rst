=====================
Programming Standards
=====================

We have documented the programming standards used in this project in the following segments:

* Standard on :ref:`python <python>`
* Standard on :ref:`c++ <cpp>`
* Standard on :ref:`ROS <ros>`


.. _python:
VantTec Python Standard
=======================

The VantTec Python standard is based on the “PEP 8 -- Style Guide for Python Code” and the “The Hitchhiker’s guide to Python”
Also, look at this script for references on how to write good Python code: https://github.com/hblanks/zen-of-python-by-example/blob/master/pep20_by_example.py.

For recommendations and modifications, please refer to Guillermo Cepeda or Sebastian Martinez.

Programming recommendations
---------------------------

Use is not operator rather than not ... is. While both expressions are functionally identical, the former is more readable and preferred::

        # Correct:
    if foo is not None:
        # Wrong:
    if not foo is None:

Always use a def statement instead of an assignment statement that binds a lambda expression directly to an identifier::
    
    # Correct:
    def f(x): return 2*x

    # Wrong:
    f = lambda x: 2*x

When catching exceptions, mention specific exceptions whenever possible instead of using a bare except: clause::

    try:
        import platform_specific_module
    except ImportError:
        platform_specific_module = None

Correct::

    def foo(x):
        if x >= 0:
            return math.sqrt(x)
        else:
            return None

    def bar(x):
        if x < 0:
            return None
        return math.sqrt(x)

CODE LAYOUT
-----------

INDENTATION
-----------

Do not use tab, use four spaces instead per indentation level.

As you can see::

   # Correct:

    # Aligned with opening delimiter.
    foo = long_function_name(var_one, var_two,
                             var_three, var_four)

    # Add 4 spaces (an extra level of indentation) to distinguish arguments from the rest.
    def long_function_name(
            var_one, var_two, var_three,
            var_four):
        print(var_one)

    # Hanging indents should add a level.
    foo = long_function_name(
        var_one, var_two,
        var_three, var_four)
    # Wrong:

    # Arguments on first line forbidden when not using vertical alignment.
    foo = long_function_name(var_one, var_two,
        var_three, var_four)

    # Further indentation required as indentation is not distinguishable.
    def long_function_name(
        var_one, var_two, var_three,
        var_four):
        print(var_one)


The closing brace/bracket/parenthesis on multiline constructs should be like this::
    
    my_list = [1, 2, 3,
    	       4, 5, 6]
    result = some_function_that_takes_arguments('a', 'b', 'c', 
    						'd', 'e', 'f')


MAXIMUM LINE LENGTH
--------------------

Limit all lines to a maximum of 80 characters.
If the length of a line is larger than 80 characters, try to use a “space + backslash”. With this, the editor will detect it is a line continuation marker
Example::
    with open('/path/to/some/file/you/want/to/read') as file_1, \
         open('/path/to/some/file/being/written', 'w') as file_2:
        file_2.write(file_1.read())

PRO TIP
-------
If you use Visual Studio Code as your code editor, you can add a vertical line into your screen, as an 80 characters visual reference.
Just go to File >> Preferences >> Settings >> search for Editor:Rulers and in the json file just paste this::
    "editor.rulers": [120]

.. figure:: /images/vsd_vscode_protip.png
   :align: center
   :alt: vsc
   :figclass: align-center
   :target: vsc
   :height: 200px
   :width: 300px

LINE BREAK BEFORE BINARY OPERATIONS
-----------------------------------
Using line breaks before binary operations helps readability::

    # easy to match operators with operands
        income = (gross_wages
          + taxable_interest
          + (dividends - qualified_dividends)
          - ira_deduction
          - student_loan_interest)
  
BLANK LINES
-----------

* Surround top-level function and class definitions with two blank lines. (IMPORTANT!)
* Method definitions inside a class are surrounded by a single blank line.
* Extra blank lines may be used (sparingly) to separate groups of related functions. Blank lines may be omitted between a bunch of related one-liners (e.g. a set of dummy implementations).
* Use blank lines in functions, to indicate logical sections.

SOURCE FILE ENCODING AND INTERPRETER
-----------------------------------

At the beginning of every script you should add these lines::

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-


* First line:

    * The program loader takes the presence of “#!” as an indication that the file is a script, and tries to execute that script using the interpreter specified by the rest of the line.

* Second line:

    *Code in the core Python distribution should always use UTF-8 (or ASCII in Python 2).
    *Files using ASCII (in Python 2) or UTF-8 (in Python 3) should not have an encoding declaration.

IMPORTS
-------

Imports should usually be on separate lines::
    
    # Correct:
    import os
    import sys

This is also okay::

    from subprocess import Popen, PIPE

* Imports are always put at the top of the file, just after any module comments and docstrings, and before module globals and constants.
* Never use: from <library> import *
* Imports should be grouped in the following order 
    1. Standard imports
    2. Related third party imports
    3. Local application / library specific imports
    4. When importing a class from a class
    	::from Class import MyClass

STRING QUOTES
-------------

In Python, double-quoted strings and single-quoted strings are the same, however, double quotes will only be used when dealing with paths and topics (ROS).

WHITESPACE IN EXPRESSIONS AND STATEMENTS
----------------------------------------

Avoid extraneous whitespace in the following situations:

* Immediately inside parentheses, brackets, and braces::

    yes: spam(ham[1], {eggs: 2})
    no:  spam( ham[1], {eggs: 2} )

* Between a trailing comma anda a following close parenthesis::

    yes: foo = (0,)
    no:  foo = (0, )

* Immediately before a comma, semicolon, or colon ::

    yes: if x == 4: print x, y; x, y = y, x
    no:  if x == 4: print x, y ; x, y = y, x

Always surround these binary operators with a single space on either side: assignment (=), augmented assignment (+=, -= etc.), comparisons (==, <, >, !=, <>, <=, >=, in, not in, is, is not), Booleans (and, or, not)::

    # Correct:
    i = i + 1
    submitted += 1
    x = x*2 - 1
    hypot2 = x*x + y*y
    c = (a+b) * (a-b)
    
    # Wrong:
    i=i+1
    submitted +=1
    x = x * 2 - 1
    hypot2 = x * x + y * y
    c = (a + b) * (a - b)

Don't use spaces around the = sign when used to indicate a keyword argument, or when used to indicate a default value for an unannotated function parameter::

   # Correct:
    def complex(real, imag=0.0):
        return magic(r=real, i=imag)
    
    # Wrong:
    def complex(real, imag = 0.0):
        return magic(r = real, i = imag)

NAMING CONVENTIONS

As a general rule, use short and descriptive names!

Classes
-------

With CapWords::

	class MyClass

Objects
-------

With camelCase::

	autoNav = AutoNav()


Global Variables
----------------

Let's try to avoid them

Functions and Variable Names
---------------------------

* For functions and variables: with lowercase_and_underscore
	* **Variable names follow the same convention as function names. Never use names such as I (i), l (L), O or o, as some can't be differentiated from one another**
* Use one leading underscore only for non-public methods and instance variables of a class.
* camelCase is allowed only in contexts where that's already the prevailing style (e.g. threading.py), to retain backwards compatibility.

Function and Method Arguments
-----------------------------

* Always use self for the first argument to instance methods.
* Always use cls for the first argument to class methods.
* Use one leading underscore only for non-public methods and instance variables.
	* More info about this here: https://realpython.com/instance-class-and-static-methods-demystified/
* When writing class attributes or composition, do it like this: myClass.myObject_, myClass.my_attribute_

Method Names and Instance Variables
-----------------------------------

* Use lowercase_and_underscores
* Use one leading underscore only for non-public methods and instance variables.
* To avoid name clashes with subclasses, use two leading underscores to invoke Python's name mangling rules. Python mangles these names with the class name: if class Foo has an attribute named __a, it cannot be accessed by Foo.__a. (An insistent user could still gain access by calling Foo._Foo__a.) Generally, double leading underscores should be used only to avoid name conflicts with attributes in classes designed to be subclassed.

Constants
---------
Use CAPITAL_LETTERS_AND_UNDERSCORES

COMMENTS
--------
Comments at the beginning of files
----------------------------------
"""
@file :        file.py
@date:         Thu Dec 26, 2019
@date_modif:   Thu Dec 26, 2019
@author:       name
@e-mail:		
@author:    (If multiple co-authors, write the name and e-mail of each one)
@e-mail:
@brief:
@version:
"""

Class Comments
--------------

Comment before class only if it's not descriptive

Functions Comments
------------------

"""
@name:
@brief:
@param     a[in]:  describe 
	   b[out]: describe
@return
"""

MORE TIPS
---------

One statement per line
----------------------

It is bad practice to have two disjointed statements on the same line of code.
Wrong::

    print 'one'; print 'two'
    if x == 1: print 'one'
    if <complex comparasion > and <other complex comparasion>:
        # do something

Correct::

    print 'one'
    print 'two'
    if x == 1: 
        print 'one'
    cond1 = <complex comparasion>
    cond2 = <other complex comparasion>
    if cond1 and cond2:
        # do something

Use sets or dictionaries instead of lists in cases where:
---------------------------------------------------------

* The collection will contain a large number of items
* You will be repeatedly searching for items in the collection
* You do not have duplicate items.

For small collections, or collections which you will not frequently be searching through, the additional time and memory required to set up the hashtable will often be greater than the time saved by the improved search speed.

Access a Dictionary Element
---------------------------

Dont use the dict.has_key() method. Instead, use x in d syntax, or pass a default argument to dict.get().

Bad::

    d = {'hello':'world'}
    if d.has_key('hello'):
        print d['hello']
    else:
        print 'default value'

Good::

    d = {'hello':'world'}
    print d.get('hello', 'default value') # prints 'world'
    print d.get('foo', 'default value') # prints 'default value'

    #or
    if 'hello' in d:
        print d['hello']
    else:
        print 'default value'


.. _cpp:

VantTec C++ Standard
=====================

To create this code standard, we took in consideration the Google C++ style guide. 
For recommendations and modifications, please refer to Pedro Sánchez, Roberto Mendivil or Sebastian Martinez

https://google.github.io/styleguide/cppguide.html

PROJECT STRUCTURE
-----------------

Every cpp project must have the next file structure:

* include
* src
* test
* CMakeList.txt
* README

The #define Guard
-----------------

All header files should have #define guards to prevent multiple inclusion. Always use the next format: <PROJECT>_<PATH>_<FILE>_H_.

For Example, the file foo/src/bar/baz.h in project foo should have the following guard::

    #ifndef FOO_BAR_BAZ_H_
    #define FOO_BAR_BAZ_H_
        ...
    #endif  // FOO_BAR_BAZ_H_

Names and order of includes
---------------------------

Include headers in the following order: 

1. C System headers (std)
2. C++ Standard Library headers
3. Other libraries headers (third-party)
4. Your project's headers.

Separate each non-empty group with one blank line and sort them in alphabetical order.

Namespaces
----------

Do not use namespaces 
(Can be used for global functions)

VARIABLES
---------

Local Variables

Always initialize variables before using it::
    
    Example:

    int i = 0 		std::vector<int> v={1,2,3}

    Declare variable close to its use
    Example:

    const char *p= temp_
    *p = foo();


If and only if variables are only used on loops (whiles), then Variables can be initialize on loops statements. 

Example::

    while(char *p = foo() < other condition...)

Otherwise on nested loops variables must be declare before the loop

Example::

    int temp_1=0;
    int temp_2 =0;

	    	while (temp_1 < range ){
		        while(temp_2 < range2){
		            temp_2++;
		            }
		        temp_1++;
		    }

Initialize objects as variables, always before and close to is use.

NAMING
------

Names must always describe the main purpose

Example::  

    int speed_challenge_state = .. 
    usv_perception.cpp

Avoid the use of abbreviations and incomplete words

Example::

    Right:        int speed_challenge_counter= ..-
    Wrong :     int speedch_Cnt = ...

File Naming
-----------

* Lower Case 
* Separate names with underscore ( _ ) ** *or dashes (-)*
* Descriptive naming 

Example:: 

    sliding_mode_controller.cpp


Typedef naming 
--------------

* CapWord
* start with UPPERCASE

Example::

    typedef hash_map<referenceFrames*, std::string> ReferenceFrame;


Class and Struct Naming
-----------------------

* CapWord
* start with UPPERCASE

Example::

    class SpeedChallenge {}; 


Function naming
---------------

* camelCase
* start with lowercase

Example::

    void decodificarXbee();


Variable Naming
---------------

* lowercase
* separate word with underscore ( _ )

Example::

    int bouy_red 

For variables in classes, end with an underscore

Example::
    
    Int bouy_

Constant naming
---------------

* Use ALL_CAPITALS

Example:: 
    
    const int STATES_NUMBER= 9;


MACROS
------

Do not use MACROS !

Use instead:
* constants
* inline functions
* enum 


COMMENTS
--------

Comments at the beginning of files
/*
@file :               file.cpp
@date:              Thu Dec 26, 2019
@date_modif:   Thu Dec 26, 2019
@author:           name
@e-mail:		
@co-author:      (If multiple co-authors, write the name and e-mail of each one)
@e-mail:
@brief:
@version:
Copyright 
All right Reserved       or     Open Source (it will depend on the project)
*/

Class Comments
--------------

* Comment before class only if it not descriptive

Functions Comments
------------------

/*
@name:
@brief:
@param     a[in]:  describe 
	         b[out]: describe
@return
*/

Other conveniences and notes 
----------------------------

Number of characters per line : 80


Suggestions

If you use Visual Studio Code as your code editor, you can add a vertical line into your screen, so you can see where your line should end.
Just go to File >> Preferences >> Settings >> search for Editor:Rulers and in the json file just paste this:

.. figure:: /images/vsd_cpp_vsc.png
   :align: center
   :alt: vanttec_documentation
   :figclass: align-center
   :target: vanttec_documentation
   :height: 200px
   :width: 300px

Now you have a nice vertical line

Class vs Structs:
-----------------

Use a struct only for passive objects that carry data; everything else is a class.

.. _ROS:
VantTec ROS Standard
====================

The information and standard that VantTec uses is gatered from the official ROS wiki.
https://wiki.ros.org/ROS/Patterns/Conventions#Naming_ROS_Resources

Standard Units of Measure and Coordinate Systems
------------------------------------------------

Standard units and coordinate conventions for use in ROS have been formalized in:
http://www.ros.org/reps/rep-0103.html

Naming ROS Resources
--------------------

Names play an important role in ROS and following naming conventions simplifies the process of learning and understanding large systems. This page documents conventions for common ROS resources, though you should familiarize yourself with the ROS name specification before proceeding.

Packages
--------

* The ROS packages occupy a flat namespace, so naming should be done carefully and consistently. There is a standard for package naming in REP-144

* Package names should follow common C variable naming conventions: lower case, start with a letter, use underscore separators, e.g. laser_viewer

* Package names should be specific enough to identify what the package does. For example, a motion planner is not called planner. If it implements the wavefront propagation algorithm, it might be called wavefront_planner. There's obviously tension between making a name specific and keeping it from becoming overly verbose.

    * Using catchall names such as utils should be avoided as they do not scope what goes into the package or what should be outside the package. 

Topics / services
-----------------

* Topic and service names live in a hierarchical namespace, and client libraries provide mechanisms for remapping them at runtime, so there is more flexibility than with packages. However, it's best to minimize the need for namespacing and name remapping.

* Topic and service names should follow common C variable naming conventions: lower case, with underscore separators, e.g. laser_scan

* Topic and service names should be reasonably descriptive. If a planner node publishes a message containing its current state, the associated topic should be called planner_state, not just state.

Messages
--------

* Message files are used to determine the class name of the autogenerated code. As such, they must be CamelCased. e.g. LaserScan.msg

* NOTE: This is an exception to the convention that all filenames are lower case and underscore separated. Using CamelCase message names will prevent issues from arising due to inconsistent support for filename case sensitivity across various operating systems.

* Message fields should be lowercase with underscore separation. e.g. range_min

Nodes
-----

* Nodes have both a type and name. The type is the name of the executable to launch the node. The name is what is passed to other ROS nodes when it starts up. We separate these two concepts because names must be unique, whereas you may have multiple nodes of the same type.

* When possible, the default name of a node should follow from the name of the executable used to launch the node. This default name can be remapped at startup to something unique

    Node type names:

In general, we encourage the node type names to be short because they are scoped by the package name. For example, if your laser_scan package has a viewer for laser scans, simply call it view (instead of laser_scan_viewer). Thus, when you run it with rosrun, you would type::
    rosrun laser_scan view

TF frame_ids
See https://wiki.ros.org/geometry/CoordinateFrameConventions#Naming

Global Executables
------------------

Executables that go into the global $PATH may have one of two prefixes:

* ros (e.g. rostopic, roscd)

  * Command-line tools that display information to stdout.
* rqt_ (e.g. rqt_console)
    * Tools that use a QT-based graphical user interface (GUI). Before ROS Hydro, these were prefixed with rx and used WxWindows-based interfaces.

The prefix naming enables easy tab completion for finding ROS tools and also creates a natural mapping between GUI and GUI-less versions of tools (e.g. rosconsole vs. rqt_console).

Informational Distance Measurements
-----------------------------------

Representation of special conditions within distance measurements like "too close" or "too far" in ROS have been formalized in:
http://www.ros.org/reps/rep-0117.html







