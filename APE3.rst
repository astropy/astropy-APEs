Coordinates Subpackage Plan
---------------------------

author: Erik Tollerud, Adrian Price-Whelan, Thomas Aldcroft, Thomas Robitalle

date: <date the APE was submitted>

type: Standard Track

status: Discussion

Abstract
--------

The abstract should be a short description of what the APE will achieve.


Detailed description
--------------------

most of the content goes here.


Branches and pull requests
--------------------------

* The API PR


Implementation
--------------

* low-level
* high-level
* earth locations
* ICRS <-> Alt/Az


Backward compatibility
----------------------

This will certainly break backwards-compatibility for anything that relies in
any way on the internal representations of coordinates in the current version.
The "low-level" API will resemble the current coordinates API, but likely with
some backwards-incompatible changes.   Where possible, we will attempt to keep
older interfaces and deprecate them for at least one more version.   This
breaking of backwards compatibility is acceptable, because it was in the
original plan to attempt a  few iterations of coordinates, and there is still a
big warning that it may change in the future in the current documentation.


Alternatives
------------

* completely split frames and data
* put it all in the high-level classes
* leave alone

Decision rationale
------------------

<To be filled in when the APE is accepted or rejected>
