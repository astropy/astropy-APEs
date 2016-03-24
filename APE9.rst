A General Timeseries Class for Astropy
-------------------------------------

author: Stuart Mumford

date-created: 2016 March 24

date-last-revised: 2016 March 24

type: Standard Track

status: Discussion


Abstract
--------

The goal of a timeseries object in astropy is to provide a core set of
functionality that can be shared between astropy, affiliated packages and the
wider community.

The scope of this timeseries object is designed to remain focused on the core
functionality needed, and not to try and replace or re-implement the
functionality in the pandas library. The main motivation for this class over
using pandas is primarily to support the functionality of other parts of the
astropy package such as Time, Units and Coordinates packages.

This document proposes an implementation based on Table, with extra constraints
and specific functionality dedicated to timeseries type data.

Detailed description
--------------------

Many different areas of astrophysics have to deal with 1D timeseries data, i.e.
either sampling a continuous variable at fixed times or counting some events
binned into time windows. These types of applications require some basic
functionality:

#. Extending / concatenating timeseries'
#. Sorting
#. Slicing / selecting time ranges
#. Re-binning timeseries
#. Interpolating to different time stamps.
#. Masking

While this functionality is found in non-domain specific packages such as
pandas, a timeseries object in Astropy would also provide some functionality
which is more related to Astronomy:

#. Converting between time systems
#. Astropy unit support
#. Propagation of uncertainties
#. Support for background subtraction



Branches and pull requests
--------------------------

**TODO:**
Any pull requests or development branches containing work on this APE should be
linked to from here.  (An APE does not need to be implemented in a single pull
request if it makes sense to implement it in discrete phases). If no code is yet
implemented, just put "N/A"


Implementation
--------------

**TODO:**
This section lists the major steps required to implement the APE.  Where
possible, it should be noted where one step is dependent on another, and which
steps may be optionally omitted.  Where it makes sense, each  step should
include a link related pull requests as the implementation progresses.


Backward compatibility
----------------------

This would be new functionality.


Alternatives
------------

Forego the functionality provided by Time and Units and recommend everyone use pandas.


Decision rationale
------------------

