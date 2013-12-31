Coordinates Subpackage Plan
---------------------------

author: Erik Tollerud, Adrian Price-Whelan, Thomas Aldcroft, Thomas Robitalle

date: <date the APE was submitted>

type: Standard Track

status: Discussion

Abstract
--------

This APE provides a high-level overview of the planned architecture of the
`astropy.coordinates` subpackage for v0.4 and beyond.  It is a synthesis of a
feedback received from previous versions of the `coordinates` subpackage and
related discussion on astropy-dev and coordinates-related pull requests.  The
base idea is to split the current coordinate class heirarchy into three sets of
classes.  data "representation" classes, "low-level" classes that serve both as
descriptions of coordinate frames *and* containers around the data, and a "high-
level"  class that uses the low-level classes for actual functionality, but
provides a more user-friendly interface.  The transformation  framework will
remain essentially unchanged, except that it will operate specifically on the
low-level classes.


Detailed description
--------------------

Background/Terminology
^^^^^^^^^^^^^^^^^^^^^^

Discussion surrounding the current coordinates API have revealed the importance
of shared terminology, as some of the crucial concepts necessary for designing a
clean API involve language with subtle (but important) distinctions.  Chief
among these is the concept of a "coordinate system."  To some members of the
community, "coordinate system" means the *representation* of a point in space,
e.g., "Cartesian coordinate system" is different from "Spherical polar
coordinate system".  Another use of "coordinate system" is to mean a unique
reference frame with a particular set of reference points, e.g., "the ICRS
coordinate system" or the "J2000 coordinate system."  This second meaning is
further complicated by the fact that such systems use quite different ways of
defining a frame.

Because of the likelihood of confusion between theses
meanings of "coordinate system", this APE (and the associated coordinates API) will avoid this
terminology, and instead adopt the following meanings (loosely inspired by the IAU2000 resolutions on celestial coordinate systems):

* "Coordinate representations" are particular ways of describing unique points
  in a vector space. (Here, this means three-dimensional space, but future
  extensions might have different dimensionality, particularly if relativistic
  effects are desired.)  Examples include Cartesian coordinates, cylindrical
  polar, or latitude/longitude spherical polar coordinates.

* "Reference Systems" are a scheme for orienting points in a space and
  describing how they are transformed to other systems.  Examples include the
  ICRS, equatorial coordinates with mean equinox, or the WGS84 geoid for
  latitude/longitude on the Earth.

* "Coordinate frames" are a specific realization of a reference system - e.g.,
  the ICRF, or J2000 equatorial coordinates.  For some systems, there may be
  only one meaningful frame, while others may have many different frames.

* A "Coordinate" is a combination of all of the above that specifies a unique
  point.

more...


Branches and pull requests
--------------------------

`PR #xxx <>`_ to the astropy-api repository contains a file
`coordinates_api_2.py` that outlines the actual API.  The PR will be merged if
this APE is accepted.


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
