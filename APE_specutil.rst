Vision for Astropy Spectroscopic Tools
======================================

author: S. M. Crawford, N. Earl, A. Ginsburg, E. Tollerud

date-created: 2017 April 21 <replace with the date you submit the APE>

date-last-revised: 2017 Jul 19 <keep this up to date anytime something changes>

type:  Process, Standard Track

status: Discussion


Abstract
--------

Spectroscopy lies at the core of much of astronomy.  The goal of this APE is to
coordinate and plan the development of spectroscopic software within the
astropy project.  The requirements for spectroscopic tools include handling and
reducing observations, different types of analysis, and visualization of
spectra.  The goal of this APE is to provide a framework that different
astronomy packages can use to coordinate the development of spectroscopic
tools.  The idea is a set of foundational packages that can then be extended to
build other tools. As part of this coordination effort, this APE provides
specifications for a `Spectrum1D` class, a standard interface for 1D spectra
that will provide the basis for interoperability.

Detailed description
--------------------

Spectroscopy lies at the core of much of astronomy.  It is used to classify
different objects, measure their composition, determine their velocity, and
study the underlying phenomena leading to the emission of light over a range of
energies.  Spectroscopic observations are obtained across the electromagnetic
spectrum from radio wavelengths to gamma-rays.  While the methodology for the
measurement of spectra can be different, the analysis and visualization needs
for spectra share many common requirements.  These requirements include
measurement of spectral features, modeling of physical processes, and
interaction with the data.

At the most basic level, a spectrum is a representation of flux as a function
of energy.  The flux can be expressed in terms of counts, number of photons,
energy flux, or temperature; the energy is usually expressed in units of
wavelength, frequency, or energy.  When analyzing spectra, there are often
other important considerations, such as the error associated with the
measurement, the provenance of the data, the units and frame of the
observations, and the data quality.

With the importance of spectroscopy to astronomy, it has long been the goal of
the Astropy Project to provide a common framework to help build tools that
allow for cross-package collaboration.  The goal of this APE is to describe how
these tools can be coordinated.  In the process of developing this APE, we hope
to bring the community together to collaborate on software for astronomical
spectroscopy and provide a path forward for creating a common set of tools for
the community.

Background
-------------------

A need for a spectroscopic package has been recognized from astropy's
inception. This was borne out by the large number of different spectroscopic
packages that were in development in 2011 (when Astropy was started).  Many of
these projects had similar, but not quite compatible interfaces (e.g.,
differences in formats, arguments, and style) with overlapping functionality.
At the same time, some commonly used functionality was not implemented at all
or only implemented as scripts (e.g., heliocentric corrections).

Nearly from the beginning this problem was recognized, and a package called
``specutils`` was created as one of the first affiliated packages.  This
package contained a `Spectrum1D` object and had the functionality to read and
write some common formats for spectroscopic data. However, this package was not
widely adopted, primarily due to limited coordination/communication with other
efforts and slower-than-expected progress in developing a generalized world
coordinate system (WCS), which was viewed as a key requirement for many desired
features. As a result, this package has been renamed `legacy specutils
<https://github.com/astropy/specutils/tree/legacy-specutils>`_ and has been
retired. However, the lessons learned in the development of this package
motivate this APE and provide a way to set a clearer path forward for the
astropy spectroscopy effort.

With previous abandoned efforts in mind, this proposal aims to introduce a new,
more robust and extensible framework on top of which new spectroscopic tools
can be built with minimal duplication of effort.

There has been a further growth of spectroscopic packages.  New packages
continue to appear regularly with much the same functionality as previous
packages or slightly different versions.  While there are some benefits of
having multiple independent development threads, there are also major
drawbacks, such as a lack of standardization and significant redundant effort.
We hope that this APE will focus the community’s efforts into a set of
intercompatible tools with minimal duplicated effort.


Spectroscopic Coordinating Committee
------------------------------------

This APE formally names a committee for coordinating this effort (although this
committee has been operating in an informal manner prior to this APE).
Patterned after the Astropy Coordinating Committee, the Spectroscopic
Coordination Committee will be charged with overseeing the effort described
below, and it will be a central point of communication for Astropy spectroscopy
efforts. One member of the committee will be a liaison member who is also a
member of the Astropy Coordination Committee.

The path forward
++++++++++++++++

This APE proposes a coordinated suite of packages for doing astronomical
spectroscopy in Python.  It suggests a division of functionality into three
packages, then specifies in detail the structure and API of the core spectral
object, Spectrum1D.

The packages will be developed separately but are connected by a common
interface object: `Spectrum1D`. This object will be in a common package - for
the near-term, the common package will be `specutils`, but once the interface
has stabilized, these interface classes may be moved to the `astropy` core
package. Packages should use this common `Spectrum1D` object (described in more
detail below) for representing spectra, although they are welcome to subclass
it to add package-specific functionality.  In the future, more interface
objects might be called for - e.g., `Spectrum2D`, `SpectralCube`, or similar.
In previous discussions at Astropy coordination meetings, there has also been
suggestions to eventually create domain-specific subclasses like
`RadioSpectrum1D` or `XRaySpectrum1D` (with, e.g., different default units
appropriate for that field or particular domain-specific methods).  These
domain-specific classes are not defined in this APE; instead, the intent is for
`Spectrum1D` to provide a starting point for such efforts, which will then
develop as they are called for by the community.

The development outlined in this APE will consist of three packages: 

* `specutils <https://github.com/astropy/specutils)>`_.  This will provide the
  basic interface classes (including the `Spectrum1D` object), as well as
  *basic* analysis of spectroscopic data (which are enumerated in `Analysis
  tools to be included in specutils`_ below).  This package is the primary
  focus of this APE.

* `specreduce`.  This will provide a standard toolbox for reducing spectra,
  roughly akin to the IRAF “specred” functionality (although not necessarily
  *all* of those tasks, nor in a one-to-one manner).  Note that this should
  *not* contain any instrument-specific code.  Observatories or instruments are
  encouraged to use and contribute back functionality to `specreduce`, but
  should provide local functionality in their own dedicated packages or guides.
  This package will provide the tools to quickly be able to build specific
  instrument pipelines.

* `specviz`. This package will provide 1D spectrum visualization capabilities,
  but is scoped to be focused primarily on that, and not arbitrarily complex
  spectrum visualization (e.g., data cubes). Note that, unlike the above two,
  this package should *not* be considered a toolbox that other tools should be
  derived from.  While other packages are welcome to use it, it is meant to
  provide a critical piece of functionality, while acknowledging that
  visualization, GUI code, and even personal taste are variable enough that
  there is room for a few other visualization packages in the ecosystem. It
  will, however, have a plugin architecture that may allow users/developers to
  add to it.

In addition to the packages described above, there are a wide range of more
domain-specific, experimental, or taste-specific packages that are likely to be
desired in the field of astronomical spectroscopy. These packages can be
developed as independent affiliated packages, and this APE encourages such
development.  However, the intent is for such packages to not *duplicate*
behavior in the foundational three packages described above. Because of the
wide range of scientific use cases for spectra, it is understood that some
level of duplication between affiliated packages is expected.  E.g., multiple
line-fitting packages may be reasonable because different science cases may be
better-suited to different line-fitting approaches.

Packages should aim for the same level of testing and documentation found in
the `astropy` package, and should follow the standard Astropy affiliated
package process to do this.  This includes, in particular, considering overlap
with existing packages - existing code should be considered and merged when
appropriate.  The Spectroscopic Coordinating Committee will assist the Astropy
Coordinating Committee with this process.  Developers are encouraged to build
or update existing tools to meet their needs rather than create new packages.

The `Spectrum1D` class 
----------------------

A spectroscopy package should provide a representation of the data, tools for
reducing spectroscopic observations, analysis tools, and methods for
interacting and visualization of the spectra. In this section, we describe some
of the functionality that will be required for a common spectroscopic package.
While equally important, we leave the specification of multi-dimensional
spectral objects with different spectral axes to future work.  At the same
time, we should consider how these tools will be compatible with
multi-dimensional data in the future.

Attributes of Spectrum1D 
```````````````````````````` 

In this context, a spectrum describes how the flux changes as a function of the
energy of the radiation from some object.  There are a number of other
parameters that can be used to describe a spectrum, and a ``Spectrum1D`` object
will have the following properties:

* ``spectral_axis``
* ``flux``
* ``uncertainty``
* ``wcs`` (more detail on this attribute in the section below)
* ``meta`` (a dict-like or maybe None)

The ``spectral_axis`` is always a quantity with the same shape as the last (or
only) dimension of ``flux``. It also must have a spectral unit (i.e., length,
frequency, energy, or velocity).  To ease use, the Spectrum1D object will have
``energy``, ``wavelength``, and ``frequency`` properties that convert the
`spectral_axis` to the appropriate units.  These are to be considered the
“center” of the spectral bins.  Bin edges will also be accessible, but the
exact way of accessing bin_edges will be determined at a later date (as it is
intertwined with the ``.wcs`` representation, discussed further below).
``flux`` will be a Quantity, so the units of the spectrum should be accessed as
``.flux.unit``.  The ``uncertainty``, when provided, will be a Quantity-like
object with the same shape as the ``flux``.  If it has an `uncertainty_type`
attribute, `arithmetic operations`_ will propagate uncertainties following
standard rules (see `the nddata APE for details about the uncertainty attribute
<https://github.com/astropy/astropy-APEs/blob/master/APE7.rst#proposal-for-an-nddatabase-abstract-base-class>`_).

Dimensionality
``````````````

``flux`` and ``uncertainty`` may be multidimensional as long as the last
dimension matches the shape of ``spectral_axis`` This is meant for fast
operations on collections of spectra that share the same spectral_axis.  While
it may seem to conflict with the “1D” in the class name, this name scheme is
meant to communicate the presence of a single common spectral axis.  For more
on this, see the section on data cubes below.

Metadata
````````

Note that the ``meta`` attribute is meant for metadata, e.g., information from
a FITS header.  A ``Spectrum1D`` object may include additional attributes
beyond the metadata. In many cases this is best implemented via *subclasses* of
``Spectrum1D``, but these attributes can also be applied directly by analysis
steps discussed in the “Analysis tools to be included in specutils” section
below.  For example, a background subtraction tool would add an attribute
``background`` to the result spectrum, which is itself a ``Spectrum1D`` with
the same shape as this one which contains the subtracted background.

Creating Spectrum1D objects
```````````````````````````

While the above properties specify the interface for accessing a `Spectrum1D`,
these are *not* all required for initialization. At the minimum, the user will
have to provide a ``flux`` (which can be a Quantity or an array) and the
``spectral_axis``. The Spectrum1D’s spectral axis can be instantiated with
either a Quantity object, array, or a pixel-to-spectral transformation function
(the `wcs` object).

In addition to this information, `specutils` will provide a method for reading
and writing formats regularly used for spectroscopy.  Common formats include
IRAF, SDSS, MIDAS, FITS tables, and ascii tables.  This I/O Reading and writing
a `Spectrum1D` object will use the astropy I/O registry.  A
`Spectrum1D.create_from_array` method should be available to directly create a
`Spectrum1D` object without the user having to interface with the ``wcs``
property.

Arithmetic Operations 
````````````````````` 

In addition, methods should exist for arithmetic on `Spectrum1D` objects.
Objects with different `spectral_axis`’s will raise an error if attempts are
made to apply arithmetic to them.  Arithmetic on non-matching spectral axes
therefore requires explicit interpolation so as not to make assumptions about
the “best” interpolation algorithm for a given use case. `Spectrum1D` will
contain interpolation methods, either onto different wavelengths (with
flux-conserving at least an option) or over bad data points.  It will include
methods for transforming the spectra to different unit systems (e.g.,
converting between different velocity frames, frequency vs wavelength, flux
transformations, air to vacuum). Note that in general these operations yield
*new* `Spectrum1D` objects rather than changing anything in place.



Examples of Spectrum1D
``````````````````````
We provide some representative examples of Spectrum1D’s proposed behavior.


Example 1: Instantiation from simple numpy arrays::

    >>> spec = Spectrum1D(spectral_axis=np.arange(50),     
                          flux=np.random.randn(50))
    >>> spec.spectral_axis
    < Quantity [ 0, 1 , 2 , … , 49 ] >
    >>> spec.spectral_axis.value
    array([ 0, 1 , 2 , … , 49 ])
    >>> spec.wcs
    < Identity Transform WCS: pixel - pixel transformation >
    >>> spec.wcs.pixel_to_world(np.arange(5,10))
    [ 5, 6, 7, 8, 9 ]
    >>> spec.frequency
    UnitsError: ‘None’ is not convertible to ‘Hz’
    >>> spec.flux
    < Quantity [ 0.1234, ... , 0.4321 ] >
    >>> spec.flux.value
    array( [ 0.1234, ... , 0.4321 ] )


Example 2: Instantiation from astropy quantity arrays::

    >>> spec = Spectrum1D(spectral_axis=np.arange(1, 50)*u.nm,     
                          flux=np.random.randn(49))
    >>> spec.spectral_axis
    < Quantity [ 1 , 2 , … , 49 ] nm >
    >>> spec.wcs
    < Identity Transform WCS: pixel - nm transformation >
    >>> spec.wcs.pixel_to_world(np.arange(5,10))
    < Quantity [ 5, 6, 7, 8, 9 ] nm >
    >>> spec.frequency
    <Quantity [  5.99584916e+16,  4.99654097e+16,  4.28274940e+16,
                       3.74740572e+16,  3.33102731e+16] Hz>
         >>> spec.velocity
         UnitsError: A velocity convention and rest value are required for    
         velocity conversion.

Example 3: Instantiation from FITS-WCS::

    >>> mywcs = wcs.WCS(header={'CDELT1': 1, 'CRVAL1': 6562.8, 'CUNIT1':'Angstrom', 'CTYPE1': 'WAVE', 'RESTFRQ':1400000000, 'CRPIX1': 25})
    >>> spec = Spectrum1D(flux=[5,6,7]*u.Jy, wcs=mywcs)
    >>> spec.spectral_axis
        <Quantity [  6.53880000e-07,  6.53980000e-07,  6.54080000e-07] Angstrom>
         >>> spec.wcs.pixel_to_world(np.arange(3))
         array([  6.53880000e-07,   6.53980000e-07,   6.54080000e-07])
         >>> spec.wavelength
         <Quantity [  6.53880000e-07,  6.53980000e-07,  6.54080000e-07] Angstrom>
         >>> spec.frequency
         <Quantity [  4.58482379e+24,  4.58412273e+24,  4.58342187e+24] Hz>
         >>> spec.velocity
         <Quantity [-299792.458,-299792.458,-299792.458] km / s>

Example 4: Spectral arithmetic::

    >>> spec1 = Spectrum1D(flux=[1,2,3], spectral_axis=[0,1,2]*u.nm)
    >>> spec2 = Spectrum1D(flux=[1,1,1], spectral_axis=[1,2,3]*u.nm)
    >>> spec1 + spec1
    < New Spectrum with flux=[2,4,6] spectral_axis=[0,1,2] >
    >>> spec1 + spec2
    ValueError: Spectral axes do not match



WCS, Spectrum1D, and Data cubes
```````````````````````````````

While it will not be necessary or required to provide or interact with the
``wcs`` property, the following description should inform how the
``Spectrum1D`` object should be developed.

The wcs property will provide a conversion between pixel position and the
spectral axis.  It will be defined in detail in a pending WCS APE, but in short
it must have two methods: `world_to_pixel` and `pixel_to_world`.  These
functions map from “pixel” coordinates (i.e., indices) to “world” coordinates,
where world can be anything - even pixels. See the examples above. This
attribute will also contain metadata like the specific spectral frame (e.g.
optical versus radio velocity definitions).  Note that the details of this
``wcs`` attribute may change depending on the outcome of the WCS APE process,
and hence *this* APE does not rely heavily upon it.  However, additional
functionality making use of more advanced WCS functionality is a possibility
for future development of the `Spectrum1D` class.

With this future development of WCS in mind, this APE is *not* concerning
itself with representations of data cubes, times series of spectra, or similar
objects with multiple spectral axes in the same object.  This is the underlying
meaning of the “1D” in `Spectrum1D`.  While these are critical use-cases to
address, this APE is specifically scoped to *not* include those cases. However,
the API outlined in this APE is meant to guide future development in these
directions, with the intent that the ``wcs`` attribute eventually contain
information for a multi-dimensional spectrum. Conventions like the attribute
names used in this class (where sensible) will then carry over to such future
developments.

Analysis tools to be included in specutils
++++++++++++++++++++++++++++++++++++++++++

In contrast to the above, much of the functionality needed to work with
spectroscopy *operates on* spectra, rather than being an intrinsic part of the
data structure.  `specutils` will contain standard tools of this sort, although
the exact definition of “standard” (and therefore implemented in the
`specutils` package) will need to be determined on a case-by-case basis. Some
of the common functionality that has been identified as important and general
enough for inclusion in `specutils` includes :

* Continuum subtraction or normalization
* Identification of spectral lines or other features
* Measuring features in the spectra including fitting models, calculating line centroids/shapes, equivalent widths, and fluxes
* Deblending of lines 
* Measuring velocities for the spectra including cross-correlation

This should *not* be taken as an exhaustive list, however.  Additional
functionality that meets the above guidelines will be implemented or accepted
via PRs to `specutils` if of sufficient general interest.

The next steps: specreduce and specviz
--------------------------------------
This APE is not intended to specify the full details of the ``specviz`` and
``specreduce`` packages.  They will develop separately following the
standard process for developing astropy packages, but this section
outlines *example* functionality expected for these packages.

``specreduce``: 
* Methods for extracting a 1D spectrum from a 2D image of a dispersed spectrum, including boxcar and “optimal” extraction
* Wavelength calibration
* Removal of sky features
* Flux calibration of spectra
* Integration with (but *not* duplication of) 2D image-level reduction steps from packages like `ccdproc <https://github.com/astropy/ccdproc>`_

``specviz``:
* Display a spectrum
* Explore the spectra by zooming in on features or moving to different areas. 
* 
* Do analysis like line flux measurements (using algorithms provided in `specutils`) or line fitting
* Overlay spectral line lists (including redshift offsets if relevant)



Branches and pull requests
--------------------------
N/A


Implementation
--------------

`specutils <https://github.com/astropy/specutils>`_ already exists in its
github repository, although the changes outlined in this APE will need to be
implemented there. `specreduce <https://github.com/crawfordsm/specreduce>`_
also has an implementation, which can be adapted to use the framework described
here.  Similarly, visualization via the `specviz
<https://github.com/spacetelescope/specreduce>`_ package will do the same.  Of
course, other packages for spectroscopic analysis exist, and hopefully will
also be adapted to this framework in due time.


Backward compatibility
----------------------

The proposed changes are incompatible with the previous `specutils` package.
To alleviate this the pre-existing code was forked into the “legacy specutils”
package prior to changes proposed in this APE.

Alternatives
------------

Despite the existence of an astropy `specutils` package with a `Spectrum1D`
object, it has not been widely adopted.  This is the original motivation for
this APE.  However, there are other options to consider:

1. Do nothing.  Pro: This requires no additional work or collaboration.  Con:
   We will continue to redevelop the same set of existing tools and waste
   limited resources in the community.  There will also be in clear,
   established way of teaching how to develop spectral objects in python.  
2. Have the Spectrum1D object be a very limited base class and have development
   for different discipline/wavelenth ranges happen independently (i.e.
   RadioSpectrum1D, OpticalSpectrum1D, XRaySpectrum1D would have their own
   packages).  Pro: This is more descriptive and then more familiar for users
   of a field. Con: This will likely lead to redevelopment of some tools, and
   likely lead to less sharing of tools developed for different domains.  
3. Developing functional code with no Spectrum1D object. Pro: Faster
   development with less overhead.  Con: Increase difficulty in maintaining
   common namespace (wave vs. wavelength vs. something else) that can lead to
   conflicts in collaborating. Also is quite different from other Astropy
   efforts and Python standards (e.g., PEP8).


Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is
accepted or rejected>

