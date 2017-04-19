Vision for Astropy Spectroscopic Tools
=======================================

author: S. M. Crawford, N. Earl, A. Ginsburg, E. Tollerud

date-created: 2017 April 21 <replace with the date you submit the APE>

date-last-revised: 2017 April 18 <keep this up to date anytime something changes>

type:  Process, Standard Track

status: Discussion


Abstract
--------

Spectroscopy lies at the core of much of astronomy.  It is the goal of this APE to coordinate and plan the development of spectroscopic software within the astropy project.   The requirements for spectroscopic tools include handling and reducing observations, different types of analysis, and visualization of spectra.  The goal of this APE is to provide a framework that different astronomy packages can use to coordinate the development of spectroscopic tools.   The idea is a set of foundational packages that can then be extended to build other tools. As part of this coordination effort, this APE also provides specifications for `specutils` that includes a `Spectrum1D` class, a standard interface for 1D spectra, providing the basis for interoperability, and general purpose spectroscopic tools.  .

Detailed description
--------------------

Spectroscopy lies at the core of much of astronomy.  It is used to classify different objects, measure their composition, determine their velocity, and study the underlying phenomena leading to the emission of light over a range of energies.  Spectroscopic observations are obtained across the electromagnetic spectrum from radio wavelengths to Gamma-rays.  While the methodology for the measurement of spectra can be different, the analysis and visualization needs for spectra share many common requirements.  This includes in the measurement of spectral features, atomic spectroscopy, modeling of physical processes, and interacting with the data. 

At the most basic level, a spectrum is a representation of flux as a function of energy.  The flux could be in terms of counts, number of photons, flux, or temperature; the energy is usually in terms of wavelength, frequency, or energy.  When analyzing spectra, there can be a number of other important considerations as well, including the error associated with the measurement, the provenance of the data, the units and frame of the observations, and the data quality. 

With the importance of spectroscopy to astronomy, it has long been the goal of the Astropy Project to provide a common framework to help build tools and allow for cross-package collaboration.  The goal of this APE is to provide a document describing how these tools can be coordinated.  In the process of developing this APE, we hope to bring the community together to collaborate on software for astronomical spectroscopy and provide a path forward for creating a common set of tools for the community.

Background
++++++++++

A spectroscopic package has been a primary element of the Astropy Project from its inception.  This was borne out by the large number of different spectroscopic packages that were in development in 2011 (when Astropy was started).  Many of these projects had similar, but not quite compatible interfaces (i.e. differences in formats, arguments, and style) with overlapping functionality.   At the same time, some commonly used functionality was not implemented at all or only implemented as scripts (e.g., heliocentric corrections).  At the same time, different users had slightly different priorities and requirements while some basic functionality did not exist in the Python universe.   

The proposed first step of this was to develop a common spectral object and a collection of tasks that has resulted in the `specutils <https://github.com/astropy/specutils>`_ package.  This package contains a `Spectrum1D` object and has the functionality to read and write in a number of common formats for spectroscopic data. However, this package has not been widely adopted, primarily due to limited coordination/communication with other efforts and slower-than-expected progress in developing a generalized WCS that can be used in specutils.  This motivates this APE, as a way to set a clearer path forward for the specutils effort.

Currently, the Spectrum1DRef object operates ostensibly as a wrapper around specutils’s core Spectrum1D object. It is, in a basic sense, a workaround to avoid the highly specialized implementation details of earlier attempts at a broadly applicable spectroscopy package. In this case, a custom WCS framework had been developed and maintained within the specutils package. The Spectrum1DRef was a compromise between maintaining package compatibility, and re-introducing the more general Astropy-related behaviors (IO, arithmetic, WCS, etc).

Consequently, this makes it extremely difficult to develop a framework for spectroscopy tools targeted toward the broader astronomy community. With that in mind, this proposal aims to introduce a new, more robust and extensible framework on top of which new spectroscopic tools can be built with minimal duplication of effort. Leveraging the applicable Astropy ecosystem guarantees an accessible development process and a rich community investment.


There has been a further growth of spectroscopic packages.  New packages continue to appear regularly with much the same functionality as previous packages or slightly different versions.   While there are some benefits to the development of multiple versions of different software systems, this represents a substantial amount of redundant work as well as the lack of a standard set of well tested and maintained software.  


Spectroscopic Coordinating Committee
++++++++++++++++++++++++++++++++++++
This APE formally names a committee for coordinating this effort (although this committee has been operating in an informal manner well prior to this APE).  Patterned after the Astropy Coordinating Committee, the Spectroscopic Coordination Committee will be charged with overseeing the effort described below, as well as being a central point of communication for Astropy spectroscopy efforts. One member of the committee will be a liaison member who is also a member of the Astropy Coordination Committee.  

The path forward
++++++++++++++++
This APE proposes a coordinated suite of packages for doing astronomical spectroscopy in Python.  There are two tiers to this coordinated plan: this APE is concerned primarily with the first tier, which is for development of “standard” tools (i.e., those that have relatively little debate about implementation, or are historically well-used). The intent is for this tier to develop a baseline set of tools and interfaces.  This tier will be maintained by the Astropy community specifically to provide these base tools and interfaces that will allow all the packages to work with each other when possible.  

The packages will be developed separately but are connected by a common interface object: `Spectrum1D`. This object will be in a common package - for the near-term, the common package will be `specutils`, but once the interface has stabilized, these interface classes may be moved to the `astropy` core package. Packages should use this common `Spectrum1D` object (described in more detail below) for representing spectra, although they are welcome to sub-class it to add package-specific functionality.  Also note that, in the future, more interface objects might be called for - e.g. `Spectrum2D` or `SpectralCube` or similar.  In previous discussions at Astropy coordination meetings, there has also been suggestions to eventually create domain-specific subclasses like `RadioSpectrum1D` or `XRaySpectrum1D` (with e.g. different default units appropriate for that field).  The intent is for `Spectrum1D` to provide a starting point for such efforts, which will then develop in a similar manner as they are called for by the community.

The foundational tier of packages will consist of three packages: 

* `specutils <https://github.com/astropy/specutils)>`_.  This will provide the basic interface classes, as well as *basic* analysis of spectroscopic data. “Basic” here simply means highly standard pieces like measuring equivalent widths, fluxes, Gaussian/Lorentzian/Voigt fitting, RV corrections, etc.
* `specreduce`.  This will provide a standard toolbox for reducing spectra, roughly akin to the IRAF “specred” functionality.  Note that this should *not* contain any instrument-specific code.  Observatories or instruments are encouraged to use and contribute back functionality to `specreduce`, but should provide local functionality in their own dedicated packages or guides.  This, ideally, would provide the codes to quickly be able to build specific instrument pipelines. 
* `specviz`. This will provide 1D spectrum visualization capabilities, but is scoped to be focused primarily on that, and not arbitrarily complex spectrum visualization (e.g. data cubes). Note that, unlike the above two, this package should *not* be considered a toolbox that other tools should be derived from.  While other packages are welcome to use it, it is meant to provide a critical piece of functionality, while acknowledging that visualization, GUI code, and even personal taste are variable enough that there is room for a few other visualization packages in the ecosystem.

The extended packages are not detailed further here.  These are a wide range of  more domain-specific, experimental, or taste-specific packages that are likely to be desired.  Hence this tier will be more loosely structured - these will be treated as independent affiliated packages, but with a somewhat looser standard of “overlapping functionality” than in the first tier (which should have as little overlap/duplication as possible).

In both tiers, Packages should aim for the same level of testing and documentation found in the `astropy` package, and should follow the standard Astropy affiliated package process to do this.  This includes, in particular, considering overlap with existing packages - this should be considered and merged when appropriate.  The Spectroscopic Coordinating Committee will assist the Astropy Coordinating Committee with this process.  Developers are encouraged to build or update existing tools to meet their needs rather than creating a new package.  

The `Spectrum1D` class 
++++++++++++++++++++++++++++++

A spectroscopy package should provide a representation of the data, tools for reducing spectroscopic observations, analysis tools, and methods for interacting and visualization of the spectra.    In this section, we describe some of the functionality that will be required for a common spectroscopic package.   While equally important, we leave the specification of multi-dimensional spectral objects to future work.  At the same time, we should consider how these tools will be compatible with multi-dimensional data in the future. 

A spectrum describes how the flux changes as a function of the energy.   There are a number of other parameters that can be used to describe a spectrum, and a 1D Spectrum object should have the following properties:

* Dispersion
* Flux 
* Errors
* Units
* System (ie. heliocentric)
* meta information about the provenance of the spectra

To ease development, the Spectrum1D object could have energy, wavelength, and frequency properties that convert the dispersion to the appropriate dimensions.  In addition to this information, software should provide a method for reading and writing formats regularly used for spectroscopy.   Common formats include iraf, SDSS, *please list others*, midas, FITS tables, and ascii.   Reading and writing into a 1D spectrum object should use the astropy io register.   In addition, methods should exist for arithmetic on spectra1d objects including how to handle objects with different dispersion sampling.  

Spectrum1D will define a specific interpretation of the meaning of a pixel coordinate value (i.e., is it the mean? Left side? Median? In what units).  This sentence is a placeholder; we plan to have a breakout session on April 21 to identify whether there is a standard we can select and finalize this choice.


Specifications of Other Things to be included in specutils
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Because of the diverse set of analysis requirements and reduction techniques for astronomical spectroscopy, we do not focus on specific requirements for the reduction or analysis of spectra, but the aspects that are similar regardless of the type of spectroscopy. This functionality will be included in `specutils` so that it does not need to be duplicated in various domain-specific tools.   Some of the common functionality that is required for spectroscopic packages include:

* Identification of spectral lines or other features
* Measuring features in the spectra including fitting models, calculating line centroids/shapes, and fluxes
* Transforming the spectra to different unit systems (e.g., converting between different velocity frames, frequency vs wavelength, flux transformations)
* Measuring velocities for the spectra including cross-correlation
* deblending of lines
* Interpolation, either onto different wavelengths (with flux-conserving at least an option) or over bad data points
* Continuum subtraction or normalization

The next step: specreduce and specviz
++++++++++++++++++++
This APE is not intended to specify the full details of the specvis and specreduce packages, but we provide a broad-brush description of what the two packages include.  They will each have their own separate APEs in the future.

Common data reduction requirements for optical spectra:

 * Methods for extracting a 1D spectrum
 * Wavelength calibration
 * Removal of sky features
 * Flux calibration of spectra

Common visualization requirements:

* Be able to display a spectra
* Being able to explore the spectra by zooming in on features or moving to different areas. 
* Being able to examine the details of the spectra
* Integrate tools into visualization to interactively perform analysis



Branches and pull requests
--------------------------
N/A


Implementation
--------------

`specutils <https://github.com/astropy/specutils>`_ already exists in its github repository, although the changes outlined in this APE will need to be implemented there. `specreduce <https://github.com/crawfordsm/specreduce>`_ also has an implementation, which can be adapted to use the framework described here.  Similary, visualization via the `specviz <https://github.com/spacetelescope/specreduce>`_ package will do the same.  Of course, other packages for spectroscopic analysis exist, and hopefully will also be adapted to this framework in due time.


Backward compatibility
----------------------

The proposed changes will likely break backwards compatibility of the `specutils` package.  To alleviate this there will be a release of `specutils` prior to changes proposed in this APE.


Alternatives
------------

Despite the existence of an astropy `specutils` package with a `Spectrum1D` object, it has not been widely adopted.  This is the original motivation for this APE.  However, there are other options to consider:

1. Do nothing.  Pro:  This requires no additional work or collaboration.   Con:   We will continue to redevelop the same set of existing tools and waste limited resources in the community.  There will also be in clear, established way of teaching how to develop spectral objects in python. 
2. Have the Spectrum1D object be a very limited base class and have development for different  discipline/wavelenth ranges happen independently (i.e.  RadioSpectrum1D, OpticalSpectrum1D, XRaySpectrum1D would have their own packages).   Pro:  This is more descriptive and then more familiar for users of a field. Con:  This will likely lead to redevelopment of some tools, and likely lead to less sharing of  tools developed for different domains. 
3. Developing functional code with no Spectrum1D object. Pro:  Faster development with less overhead.  Con:  Increase difficulty in maintaining common namespace (wave vs. wavelength vs. something else) that can lead to conflicts in collaborating. Also is quite different from other Astropy efforts and Python standards (e.g. PEP8).


Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>

