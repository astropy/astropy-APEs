Specutils Specifications
------------------------

author: S. M. Crawford, N. Earl, A. Ginsburg, E. Tollerud

date-created: 2017 April 21 <replace with the date you submit the APE>

date-last-revised: 2017 March 7 <keep this up to date anytime something changes>

type:  Standard Track

status: Discussion


Abstract
--------

Spectroscopy lies at the core of much of astronomy.  It is the goal of this APE to coordinate and plan the development of spectroscopic software within the astropy project.   The requirements for spectroscopic include handling and reducing observations, different types of  analysis, and visualization of spectra.  The goal of this APE is to provide a framework that different astronomy packages can coordinate the development of spectroscopic tools.  As part of this, this APE also provides the specifications for a Spectrum1D object, which could provide the basis for coordination.  

Detailed description
--------------------

Spectroscopy lies at the core of much of astronomy, and it is used to classify different objects, measure their composition, determine their velocity, and study the underlyng phenemona leading to the emission of light over a range of energies.  Spectroscopic observations are obtained across the electromagnetic spectrum from radio wavelengths to Gamma-rays.  While the methodology for the measurement of spectra can be different, the analysis and visualization needs for spectra share many common requirements.  This includes in the measurement of spectral features, atomic spectroscopy, modeling of physical processes, and interating with the data. 

At the most basic level, a spectrum is a represenation of flux as a function of energy.  The flux could be in terms of counts, number of photons, flux, or temperature; the energy is in usually in terms of wavelength, frequency, or energy.    When  analyzing spectra, there can be a number of other important considerations as well including the error associated with the measurement, the provenance of the data, and the data quality. 

As such, it has long been a goal of the astropy project to coordinate spectroscopic development. 

Background
++++++++++

A spectroscopic package has always part of the initial discussions of astropy.  This was born out by the large number of different spectroscopic packages that were in development in 2011.  Many of these projects had similar, but not quite compatible interfaces (i.e. differences in formats, arguments, and style) with overlapping functionality.   At the same time, some commonly used functionality was not implemented at all or only implemented as scripts (ie. heliocentric corrections).   At the same time, different users had slightly different priorities and requirements while some basic funcationality did not exist in the python universe.   

The proposed first step of this was to develop a comon spectral object and a collection of tasks that has resulted in the [specutils](https://github.com/astropy/specutils) package.  This package contains a `Spectrum1D` object and has the functionality to read and write in a number of common formats for spectroscopic data.  *NICK* Some statement about `Spectrum1DRef` and why it was added? 

In the meantime, there has been a further growth of spectroscopic packages.  New packages continue to appear regularly with much the same functionality as previous packages or slightly different versions.   While there are some benefits to the development of multiple versions of diffeernt software systems, this representats a substantial amount of redundent work as well as the lack of a standard set of well tested and maintained software.  

A wide range of different spectroscopic packages have been developed including the following packages:


Goals
+++++

The overall goal of this APE is to help build coordination of the development of spectroscopic packages.     

spectroscopic software packages is to produce a manner for reducing spectroscopic observations, analyzing those observations, and visualization of the spectra.  

At the same time, new algorithms are constantly being developed for 


Spectrum 1D in specutils

problems: already has been forked into Spectrum1DRef, not widely adopted, no longer maintained

issues: order of arguments, reversible wcs/dispersion, using astropy.io 

Need for common analysis tools

Functionality we want in general spectroscopic tools (specutils, specreduce, or specviz)

- detecting lines
- measuring lines
  - fitting models to lines
  - measuring equivalent widhts
  - centroid of lines
- redshift measurementions
- heliocentric corrections





Branches and pull requests
--------------------------

Any pull requests or development branches containing work on this APE should be
linked to from here.  (An APE does not need to be implemented in a single pull
request if it makes sense to implement it in discrete phases). If no code is yet
implemented, just put "N/A"


Implementation
--------------

This section lists the major steps required to implement the APE.  Where
possible, it should be noted where one step is dependent on another, and which
steps may be optionally omitted.  Where it makes sense, each  step should
include a link related pull requests as the implementation progresses.


Backward compatibility
----------------------

This section describes the ways in which the APE breaks backward compatibility.


Alternatives
------------

If there were any alternative solutions to solving the same problem, they should
be discussed here, along with a justification for the chosen approach.


Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
