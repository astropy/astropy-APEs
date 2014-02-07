Coordinates Subpackage Plan
---------------------------

authors: Erik Tollerud, Adrian Price-Whelan, Thomas Aldcroft, Thomas Robitalle

date: 2014 January 22

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

Discussion surrounding the current `coordinates` API have revealed the
importance of shared terminology, as some of the crucial concepts necessary for
designing a clean API involve language with subtle (but important) distinctions.
Chief among these is the concept of a "coordinate system."  To some members of
the community, "coordinate system" means the *representation* of a point in
space, e.g., "Cartesian coordinate system" is different from "Spherical polar
coordinate system".  Another use of "coordinate system" is to mean a unique
reference frame with a particular set of reference points, e.g., "the ICRS
coordinate system" or the "J2000 coordinate system."  This second meaning is
further complicated by the fact that such systems use quite different ways of
defining a frame.

Because of the likelihood of confusion between theses meanings of "coordinate
system", this APE (and the associated coordinates API) will avoid this
terminology, and instead adopt the following meanings (loosely inspired by the
IAU2000 resolutions on celestial coordinate systems):

* A "Coordinate Representation" is a particular way of describing a unique
  point in a vector space. (Here, this means three-dimensional space, but future
  extensions might have different dimensionality, particularly if relativistic
  effects are desired.)  Examples include Cartesian coordinates, cylindrical
  polar, or latitude/longitude spherical polar coordinates.

* A "Reference System" is a scheme for orienting points in a space and
  describing how they transforms to other systems.Examples include the ICRS,
  equatorial coordinates with mean equinox, or the WGS84 geoid for
  latitude/longitude on the Earth.

* A "Coordinate Frame", "Reference Frame", or just "Frame" is a specific
  realization of a reference system - e.g., the ICRF, or J2000 equatorial
  coordinates.For some systems, there may be only one meaningful frame, while
  others may have many different frames (differentiated by something like a
  different equinox, or a different set of reference points).

* A "Coordinate" is a combination of all of the above that specifies a unique
  point.

This terminology is used by this APE and the associated API document. 


Problems with current `coordinates` subpackage
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The current (Astropy v0.2/0.3) `coordinates` subpackage effectively provides
classes to represent common celestial coordinates and transformations between
them.  It also makes all of  the machinery available for subclassing, allowing
users to define their own  coordinate frames and provide transformations to and
from the builtin frames. 

In this scheme (motivated mainly by ease of implementation), coordinate
representations, reference systems, and frames are mixed together in classes
just called e.g., `ICRS` or `FK5`. However, comments following implementation of
this system has revealed a number of shortcomings regarding this architecture:  

* Quite a bit of boilerplate copy-and-pasting is necessary to define a new
  reference systems. While some minor modifications might simplify this, it is
  difficult to ensure this won't have unintended consequences due to the lack of
  clear separation-of-concerns.

* There is no reasonable way to define new representations.Some reference
  systems may be more naturally expressed in e.g. cylindrical coordinates, but
  defining such a class would require re-implementing nearly everythin in the
  current coordinate classes (which are based on spherical polar coordinates).
  
* There has been a large amount of debate and (in some cases, wasted)
  implementation effort on parsing and formatting coordinate strings.Similarly,
  parsing of units for coordinate imputs has been subject to debate, change, and
  confusion.

* In the current system, one person is often a bottleneck for development of the
  subpackage because the single class makes it hard to keep a change from not
  cascading to other parts of the package.

The description below and the API associated with this APE present an approach
that addresses these concerns by separating `coordinates` functionality into a
few different class heirarchies, and using it to re-organize (without wasting)
the existing code base.


High-level outline of new `coordinates` classes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The `Angle`, `Latitude`, `Longitude`, and `Distance` classes will be retained,
as well as the current transformations and associated architecture.  This
represents the bulk of the work that has thus far gone into `coordinates`.  The
existing coordinates classes will be sub-divided into three pieces:

1. Classes for coordinate representations, subclasses of a new abstract class
   `CoordinateRepresentation`. These classes store the actual spatial
   information of a coordinate (or coordinates, if they are arrays) as
   `Quantity` subclasses (e.g., `Angle`, `Distance`). These classes also provide
   the methods necessary to transform points from one representation to another,
   and a single method for invoking these transformations.

2. A class heirarchy for specifying reference systems and frames (their
   realization) - the "low-level" coordinate classes.  These will be subclasses
   of `CoordinateFrame`. While it may at first make sense to separate specifying
   frames from the actual representations in those frames, doing so would result
   in a large amount of code duplication, as each system would require two very
   similar classes for specifying a frame and for the objects containing the
   representations.  Hence, here we combine them into a single class that is a
   pure frame if it does not have position data, but if it does, is actually a
   coordinate.
 
3. Classes that are containers for the low-level classes, but provide additional
   functionality to make them easier to use.  For example, if created in the FK5
   system, it would contain `equinox` information even if trasnformed into ICRS
   (which has no concept of equinox), allowing  the high-level class to round-
   trip transformations. Initially this will be a single class, `SkyCoordinate`,
   but might later be expanded to other coordinates that are  spatial, but not
   necessarily celestial (e.g., locations on the surface of the Earth).

By separating the coordinates into these three domains, the aforementioned
problems will either be addressed, or made easier to address by dividing the
code more logically. Additionally, the new API takes advantage of features that
were not available when `coordinates` was first designed to simplify passing in
units (specifically, the use of `Quantity` objects.)

Branches and pull requests
--------------------------

`PR #12 <http://github.com/astropy/astropy-api/pull/12>`_ in the astropy-api 
repository contains the file `coordinates_api_2.py` that outlines the actual
API, and is thus a crucial part of this APE.  The PR will be merged if this APE
is accepted.


Implementation
--------------

This APE can be implemented in separate parts:

* The low-level classes can be implemented by writing a representation class
  for spherical and cartesian coordinates, and then adapting the existing
  coordinate systems to the new heirarchy. Much of the algorithmic side of the
  code should be reusable, particularly the transformation functions themselves.
  This could possibly be done by two people (one working on the representations,
  another on the frame classes), but it may be more efficient for a single
  developer to do this.

* In parallel, the high-level classes can be developed, ideally by an additional
  developer. This interface with the low-level classes is inteded to separate
  concerns, and is specified in the API document (although some details will no
  doubt need to be worked out when the coding actually begins).

The above will complete the framework described in this APE.  The immediate
follow-on work that should be enabled by this will be:

* Defining locations on the Earth as part of the transform heirarchy.  Such
  location objects will be useful in `astropy.time`, as well as a planned 
  subpackage for storing observatory locations.
 
* Implementing the full ICRS <-> Alt/Az transformation stack, following the
  IAU2000 algorithms, mostly as implemented in ERFA.This will be simplified
  greatly by the re-organization described in this APE, as it will be easier to
  define new referebnce system/frames, and the IAU2000 stack requires a whole
  series  of such intermediate systems.


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

A number of different alternatives have been discussed or considered.  Below I
address a few that led to this APE due to being imperfect solutions, but with 
some valuable parts.

* Keep the current system. This is problematic due to reasons described in the
  description section above.

* Implement a similar stack, but don't store the representations in the frame
  classes, instead provide classes for frames and representations, and combine
  them only in "high-level" classes.This is a possibility, but would require
  quite a bit of code in the high-level class customized for particular low-
  level classes.This is mainly because different reference systems have
  different "preferred" representations (e.g., equatorial systems are
  traditionally represented in RA/Dec and possibly distance, not cartesian
  coordinates). without a way for the frame to convert to its preferred system,
  there's no easy way to delegate operations like generating reasonable-looking
  strings or `__repr__`.

* Similar stack as this APE, but have the frames without data be a separate
  class from those with data.This would necessitate making two copies of every
  class, which would either waste effort or require confusing "magic" with
  metaclasses. Either way, the proposed APE will probably result in easier-to-
  understand code, as there will be fewer classes to be familiar with.

* Have separate classes for each relevant representation of a reference
  system/frame.This is also possible, and simplifies writing the frame classes,
  as there is no need to  transform to/from the "preferred"
  representation.However, it makes it very difficult to switch between
  representations, a task that is crucial for transformations (in most cases
  they are defined only on cartesian representations).It also  results in more
  work for any user that wants a custom system but wants to be able to use
  multiple representations.


Decision rationale
------------------

<To be filled in when the APE is accepted or rejected>
