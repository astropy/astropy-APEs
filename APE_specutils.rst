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

Spectroscopy lies at the core of much of astronomy, and it is used to classify different objects, measure their composition, determine their velocity, and study the underlying phenomena leading to the emission of light over a range of energies.  Spectroscopic observations are obtained across the electromagnetic spectrum from radio wavelengths to Gamma-rays.  While the methodology for the measurement of spectra can be different, the analysis and visualization needs for spectra share many common requirements.  This includes in the measurement of spectral features, atomic spectroscopy, modeling of physical processes, and interacting with the data. 

At the most basic level, a spectrum is a representation of flux as a function of energy.  The flux could be in terms of counts, number of photons, flux, or temperature; the energy is in usually in terms of wavelength, frequency, or energy.    When  analyzing spectra, there can be a number of other important considerations as well including the error associated with the measurement, the provenance of the data, the units and frame of the observations, and the data quality. 

With the importance of spectroscopy to astronomy, it has long been the goal of the astropy project to provide a common framework to help build tools and allow for cross package collaboration.   The goal of this APE is to provide a framework for spectroscopic tools in python.    In the process of developing this APE, we hope to bring the community together to collaborate on software for astronomical spectroscopy and provide a path forward to provide a common set of tools for the community.

Background
++++++++++

A spectroscopic package has always part of the initial discussions of astropy.  This was born out by the large number of different spectroscopic packages that were in development in 2011.  Many of these projects had similar, but not quite compatible interfaces (i.e. differences in formats, arguments, and style) with overlapping functionality.   At the same time, some commonly used functionality was not implemented at all or only implemented as scripts (ie. heliocentric corrections).   At the same time, different users had slightly different priorities and requirements while some basic functionality did not exist in the python universe.   

The proposed first step of this was to develop a common spectral object and a collection of tasks that has resulted in the [specutils](https://github.com/astropy/specutils) package.  This package contains a `Spectrum1D` object and has the functionality to read and write in a number of common formats for spectroscopic data.  *NICK* Some statement about `Spectrum1DRef` and why it was added? 

Spectrum 1D in specutils
problems: already has been forked into Spectrum1DRef, not widely adopted, no longer maintained
issues: order of arguments, reversible wcs/dispersion, using astropy.io


In the meantime, there has been a further growth of spectroscopic packages.  New packages continue to appear regularly with much the same functionality as previous packages or slightly different versions.   While there are some benefits to the development of multiple versions of different software systems, this represents a substantial amount of redundant work as well as the lack of a standard set of well tested and maintained software.  

A wide range of different spectroscopic packages have been developed including the following packages:
* Add your package here

Path forward
++++++++++++

We propose the development of a suite of tools for spectroscopic packages.  These packages should be developed independently but around a common object that describes spectrum1d object, and all import this object from a common package (* astropy or specutils?*).  The packages should fall primarily under three areas:  reduction of spectroscopic data, analysis of spectroscopic data, and visualization of spectroscopic data.   Functionality that is developed in one of these packages that might be useful to other packages should be migrated into the specutils package.   Furthermore, functionality developed by these packages should be easily callable by the visualization tools.  

Packages developed should rely on the common Spectrum1D object.   They should aim for the same level of testing and documentation found in the astropy package.   Packages should be advertised on the affiliated package pages and encourage contributions from the community.  Prior to new packages being accepted,  overlap with existing packages should be considered and merged when appropriate.   Developers should be encouraged to build or update existing tools to meet their needs rather than creating a new package.  

Specifications
++++++++++++++

A spectroscopy package should provide a representation of the data, tools for reducing spectroscopic observations, analysis tools, and methods for interacting and visualization of the spectra.    In this section, we describe some of the functionality that will be required for a common spectroscopic package.   While equally important, we leave the specification of multi-dimensional spectral objects to future work.  At the same time, we should consider how these tools will be compatible with multi-dimensional data in the future. 

A spectrum describes how the flux changes as a function of the energy.   There are a number of other parameters that can be used to describe a spectrum, and a 1D Spectrum object should have the following properties:
* Dispersion
* Flux 
* Errors
* Units
* System (ie. heliocentric)
* meta information about the provenance of the spectra
To ease development, the Spectrum1D object could have energy, wavelength, and frequency properties that covert the dispersion to the appropriate dimensions.  In addition to this information, software should provide a method for reading and writing formats regularly used for spectroscopy.   Common formats include iraf, SDSS, *please list others*, midas, FITS tables, and ascii.   Reading and writing into a 1D spectrum object should use the astropy io register.   In addition, methods should exist for arithmetic on spectra1d objects including how to handle objects with different dispersion sampling.  

Because of the diverse set of analysis requirements and reduction techniques for astronomical spectroscopy, we do not focus on specific requirements for the reduction or analysis of spectra, but the aspects that are similar regardless of the type of spectroscopy.   Some of the common functionality that is required for spectroscopic packages include:
* identification of features in the spectra
* Measuring features in the spectra including fitting models, calculating line centroids and shapes, and fluxes
* Transforming the spectra to different systems 
* Measuring velocities for the spectra including cross-correlation
* removal or debelnding of different spectra

Common visualization requirements:
* Be able to display a spectra
* Being able to explore the spectra by zooming in on features or moving to different areas. 
* Being able to examine the details of the spectra
* Integrate tools into visualization to interactively perform analysis




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

1. Despite the existence of an astropy specutils package with a Spectrum1D object, it has not been widely adopted.
2. Do nothing.  Pro:  This requires no additional work or collaboration.   Con:   We will continue to redevelop the same set of existing tools and waste limited resources in the community.  No established way of teaching how to develop spectral objects in python. 
3. Having the Spectrum1D object be a base class and the development of an object for each discipline (ie.  RadioSpectrum1D, OpticalSpectrum1D, XRaySpectrum1D).   Pro:  More descriptive and then more familiar for users of a field, Neg:  Likely the redevelopment of certain tools, possibility of conflicts between tools developed for different domains. 
4. Developing functional code with no Spectrum1D object. Pro:  Faster development with less overhead.  Con:  Increase difficulty in maintaining common namespace (wave vs. wavelength vs. something else) that can lead to conflicts in collaborating. 


Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
