A General Timeseries Class for Astropy
-------------------------------------

author: Stuart Mumford

date-created: 2016 March 24

date-last-revised: 2023 Oct 20

type: Standard Track

status: Accepted


Abstract
--------

The goal of a timeseries object in astropy is to provide a core set of
functionality for representing data where time is the physical event list. This
includes both continuous values binned in time or event list type data.

The motivation for creation of this class is to support representation and
analysis of astrophysical timeseries data, including support for the Units, Time
and Coordinates packages in Astropy. This object should also be compatible with
other packages in the wider scientific Python ecosystem, and try to minimise
overlap in functionality with these packages.

This document proposes an implementation based on Table, with extra constraints
and specific functionality dedicated to timeseries type data.

Detailed description
--------------------

Many different areas of astrophysics have to deal with 1D timeseries data, i.e.
either sampling a continuous variable at fixed times or counting some events
binned into time windows. These types of applications require some basic
functionality:

#. Extending timeseries with extra rows
#. Concatenating multiple timeseries objects (e.g. adding new columns)
#. Sorting
#. Slicing / selecting time ranges (indexing)
#. Re-binning and re-sampling timeseries
#. Interpolating to different time stamps.
#. Masking and support for missing values (NaN)
#. Support for subtraction and addition (e.g. background).
#. Plotting and visualisation.
#. Have high performance into \~millions of rows.

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
handling of methods such as re-binning or re-sampling, as well as for binned
data the width of the bins must be stored.


To distinguish between binned and sampled timseries we propose having two ways
of constructing a timeseries, ``BinnedTimeSeries`` and ``SampledTimeSeries``.
This distinction in the name of the classes will allow a easier to understand
API. Some examples of constructing these objects is given below.


Initialize a ``SampledTimeSeries`` with a time and a data column::

  ts = SampledTimeSeries(time=['2016-03-22T12:30:31', '2016-03-22T12:30:32', '2016-03-22T12:30:40'],
                         data=[1, 4, 3])

Initialize a ``SampledTimeSeries`` with a time and a fixed delta::

  ts = SampledTimeSeries(time='2016-03-22T12:30:31', time_delta=3 * u.s, n_samples=10)

Initialize a ``SampledTimeSeries`` with a time and an array of deltas (also no data)::

  ts = SampledTimeSeries(time="2011-01-01T00:00:00", time_delta=np.random.random((10,))*u.s)

Initialize a ``BinnedTimeSeries`` with even contiguous bins by specifying the bin width::

  ts = BinnedTimeSeries(start_time='2016-03-22T12:30:31', bin_size=3 * u.s, data=[1, 4, 3])

Initialize a ``BinnedTimeSeries`` with uneven non-contiguous bins with lists of start times, bin sizes and data::

  ts = BinnedTimeSeries(start_time=['2016-03-22T12:30:31', '2016-03-22T12:30:38', '2016-03-22T12:34:40'], 
                        bin_sizes=[5, 100, 2]*u.s,
                        data=[1, 4, 3])

Initialize a ``BinnedTimeSeries`` with uneven contiguous bins by giving an end time::

  ts = BinnedTimeSeries(start_time=['2016-03-22T12:30:31', '2016-03-22T12:30:32', '2016-03-22T12:30:40'], 
                        end_time='2016-03-22T12:30:55',
                        data=[1, 4, 3])

Initialize a ``BinnedTimeSeries`` with uneven non-contiguous bins by specifying the start and end times for the bins::

  ts = BinnedTimeSeries(start_time=['2016-03-22T12:30:31', '2016-03-22T12:30:33', '2016-03-22T12:30:40'], 
                        end_time=['2016-03-22T12:30:32', '2016-03-22T12:30:35', '2016-03-22T12:30:41'],
                        data=[1, 4, 3])


Once a binned timeseries was constructed it would present an API that had a
'time' and a 'timedelta' column, which represents the start time of the bin and
the width of the bin. The timedelta column would not have to be stored in memory
for all of the examples above and could be optimised to allow computation
on-the-fly based on if the bins are contiguous or not.


Branches and pull requests
--------------------------

N/A


Implementation
--------------



Backward compatibility
----------------------

This would be new functionality.


Alternatives
------------

Forego the functionality provided by Time and Units and recommend everyone use pandas.


Decision rationale
------------------
This APE has long been implemented in https://github.com/astropy/astropy/pull/8540 and somehow the Coco just never merged it.
