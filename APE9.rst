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

#. Extending timeseries with extra rows
#. Concatenating multiple timeseries objects
#. Sorting
#. Slicing / selecting time ranges (indexing)
#. Re-binning and re-sampling timeseries
#. Interpolating to different time stamps.
#. Masking
#. Support for subtraction and addition.
#. Plotting and visualisation.

While this functionality is found in non-domain specific packages such as
pandas, a timeseries object in Astropy would also provide some functionality
which is more related to Astronomy:

#. Converting between time systems
#. Astropy unit support
#. Support variable width time bins.

It is proposed that this functionality be implemented via a subclass of the
``QTable`` class, with a few requirements specific to the timeseries class:

#. A 'Time' index column exists which enforces unique indexes.
#. The table is always sorted in terms of increasing time.


Binned Data vs Sampled Data
###########################

There are two different types of data that a timeseries class or classes would
have to support, sampled data and binned data. Sampled data is taken at a
precise time stamp whereas binned data is a number of counts contained within a
some time window. These two different types of timeseries data require different
handling of methods such as re-binning or re-sampling, as well as for sampled
data the width of the bins must be stored.

This APE proposes that we place the following restrictions on binned data:

#. Contiguious bins, i.e. the start of the i+1th bin is the end of the ith bin.
#. The width of the bins is stored in a second column containing ``TimeDelta`` objects.



Branches and pull requests
--------------------------

N/A


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

