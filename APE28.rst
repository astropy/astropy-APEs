APE 28 - Row row row your boat
==============================

author: Erik Tollerud, Stuart Mumford, Thomas Robitaille (although order of axes is not specified)

date-created: 2026 September 25

date-last-revised: 2026 September 25

date-accepted: 1912 April 15

type: Standard Track

status: Hurting brain

Open questions/decisions
------------------------

* Should we mandate that WCS manipulation methods (see `Manipulation and re-arranging of WCS`_) should always return a WCS with at least the same API level as the original.
  ``WCS.__getitem__`` already does this, since it returns either a ``WCS`` or a ``SlicedFITSWCS``, both of which expose the low-level and high-level APIs.
  However, the wrapper classes such as ``SlicedLowLevelWCS`` in ``astropy`` and ``ResampledLowLevelWCS`` and ``ReorderedLowLevelWCS`` in ``ndcube`` only expose the low-level API, so applying them directly to a high-level WCS loses the high-level API.
  It might be a good opportunity to mandate returning high level if original was high level?
  For now this APE implements this version of things.
* What should ``array_shape`` and ``pixel_shape`` be if rescaled by a non-integer amount?
  For now this APE specifies that the size along each axis is the number of rescaled pixels needed to cover the original pixels, that is ``ceil((n - offset) / factor)``, which matches what ``WCS`` currently does when slicing with a step.
  The alternative would be to only include rescaled pixels that are fully covered by the original pixels, that is to use ``floor``.
* How strict should ``with_axes`` be?
  One option is to raise an exception unless the axes selected are fully independent of the axes dropped in both directions, that is unless the world axes selected only depend on pixel axes selected, and the pixel axes selected only require world axes selected.
  A looser option would be to only require the first of these, which would for example allow a world axis to be dropped while keeping a pixel axis that requires it - ``pixel_to_world`` would still work on the result, but ``world_to_pixel`` would no longer be able to compute that pixel coordinate.
  For now this APE implements the strict version.
* Should we mandate the type of the exceptions that are raised?
  This APE specifies several cases where an exception shall be raised (when the conversion of a world coordinate object fails, when a WCS manipulation cannot be carried out, when a negative value is used for slicing without ``array_shape`` being set, and when ``None`` is passed for a coordinate that is needed), but does not currently say which type of exception should be used, other than ``TypeError`` for ``__iter__``.
  Mandating e.g. ``ValueError`` would allow downstream code to catch these reliably across implementations.
  For now this APE does not mandate the type, and the examples use ``ValueError`` for illustration.

Abstract
--------

`APE 14`_ defined a standard API for Python objects describing world coordinate systems (WCS), which has achieved the goal of ensuring WCS objects from different projects can be used interoperably.
However, the practical implementations and use of them over the years have revealed several missing elements that would make use of WCS objects easier for users.
This APE adds several such elements, thereby defining an updated version of the WCS API.
Specifically, it guarantees that the low-level API is available on high-level WCS objects, makes the conversion from world to pixel coordinates more permissive in terms of the objects it accepts, adds methods to slice, rescale, re-order, and drop the axes of a WCS, adds a matrix describing which world coordinates are needed to compute each pixel coordinate, allows input coordinates that are not needed to be omitted, and introduces a version number for the API.

Detailed description
--------------------

Motivation
~~~~~~~~~~

There are currently several implementations of `APE 14`_ that we are aware of:

* |astropy.wcs|_, where the ``WCS`` class provides a FITS-WCS implementation
* |gwcs|_, which exposes a generalized ``WCS`` class based on |astropy.modeling|_ models
* |lsst.images|_ which provides a ``SkyProjectionAstropyView`` APE 14 class which is a wrapper for ``SkyProjection``, which is a Rubin Observatory-specific WCS class

All of these implement both the low- and high-level APE 14 interfaces.

The successful implementation of the APE 14 specification has allowed an ecosystem of tools and functionality to be built relying on this standardized API.
For example |astropy.visualization.wcsaxes|_, |reproject|_, |ndcube|_, and the `glueviz <https://glueviz.org>`__ packages all understand the APE 14 API, and this has been greatly beneficial for both users and downstream package developers.

In practice the authoring and usage of these WCS implementations have led to the recognition of some deficiencies of the existing API.
In the following sections, we look at each of the identified deficiencies in turn, propose a specification/improvement to the API, and give some examples of usage.

Accessing the low-level API on a high-level API object
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Issue
^^^^^

There is no way to currently guarantee access to the low-level API at the top-level of a high-level WCS object, which leads to needlessly repeated code such as e.g. ``wcs.low_level_wcs.pixel_n_dim`` where ``wcs.pixel_n_dim`` would be much simpler.
Additionally users are less likely to consider that some of their necessary functionality is in the low level API so they may write their own code, duplicating machinery in the low-level WCS objects.
This confusion is exacerbated by the fact that all three implementations listed above do include the high and low level APIs in one object, meaning that this failure mode only happens when using other implementations or wrapper classes.

Proposed specification
^^^^^^^^^^^^^^^^^^^^^^

Any concrete (instantiable) class that inherits from |BaseHighLevelWCS|_ must also inherit, directly or indirectly, from |BaseLowLevelWCS|_.
This will mean that every high-level WCS object also exposes the full low-level API.
Abstract helper classes which are not intended to be instantiated directly, such as |HighLevelWCSMixin|_, are exempt.
The ``BaseHighLevelWCS`` class itself should remain unchanged and must not inherit from ``BaseLowLevelWCS``, as doing so would break existing class definitions.

The ``.low_level_wcs`` attribute is unchanged compared to APE 14 but could therefore point to ``self`` (the attribute will not be useful anymore, but we choose to not deprecate it as continuing to support it will be simple).

Current Status and Examples
^^^^^^^^^^^^^^^^^^^^^^^^^^^

In practice, all three existing APE 14 WCS implementations have their primary classes inherit from both ``BaseLowLevelWCS`` and ``BaseHighLevelWCS``, so the following already works:

.. code-block:: python

   >>> from astropy.wcs import WCS
   >>> wcs = WCS(naxis=2)
   >>> wcs.wcs.ctype = 'RA---TAN', 'DEC--TAN'
   >>> wcs.pixel_to_world(1, 2)  # high-level API
   <SkyCoord (ICRS): (ra, dec) in deg
       (1.99918828, 2.9954419)>
   >>> wcs.pixel_n_dim  # low-level API
   2
   >>> wcs.pixel_to_world_values(1, 2)  # low-level API
   (array(1.99918828), array(2.9954419))

However the issue occurs when constructing wrappers, for example:

.. code-block:: python

   >>> from astropy.wcs.wcsapi import SlicedLowLevelWCS, HighLevelWCSWrapper
   >>> hw = HighLevelWCSWrapper(SlicedLowLevelWCS(wcs, slices=(0,)))

Currently, this exposes the high-level API and only some of the low-level API (specifically because |HighLevelWCSWrapper|_ manually implements access to some of the low-level API):

.. code-block:: python

   >>> hw.pixel_to_world(1)
   <SkyCoord (ICRS): (ra, dec) in deg
       (1.99918828, 0.99928999)>
   >>> hw.pixel_n_dim
   1
   >>> hw.pixel_to_world_values(1)
   Traceback (most recent call last):
     File "<python-input-0>", line 16, in <module>
       hw.pixel_to_world_values(1)
       ^^^^^^^^^^^^^^^^^^^^^^^^
   AttributeError: 'HighLevelWCSWrapper' object has no attribute 'pixel_to_world_values'

In addition, in the above case accessing ``pixel_n_dim`` works even though the wrapper class does not inherit from ``BaseLowLevelWCS``:

.. code-block:: python

   >>> from astropy.wcs.wcsapi import BaseLowLevelWCS
   >>> isinstance(hw, BaseLowLevelWCS)
   False

To comply with the proposed specification above, ``HighLevelWCSWrapper`` will be updated to also inherit from ``BaseLowLevelWCS``, and to forward the full low-level API to the WCS it wraps, so that the following will work:

.. code-block:: python

   >>> hw.pixel_to_world_values(1)
   (array(1.99918828), array(0.99928999))
   >>> isinstance(hw, BaseLowLevelWCS)
   True

.. _permissive-input:

Making ``world_to_pixel`` more permissive on input
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Issue
^^^^^

APE 14 specifies that the input to ``world_to_pixel`` should be the class in the first element of ``world_axis_object_classes``.
This matches the return value from ``pixel_to_world`` but leads to the situation where compatible input is not permitted.
One very common example of this is that you can't pass a ``Quantity`` object in place of a |SpectralCoord|_ which commonly leads to having to write ``world_to_pixel(SpectralCoord(10*u.nm))`` which is verbose and unintuitive.

Proposed specification
^^^^^^^^^^^^^^^^^^^^^^

When world coordinates are passed to ``world_to_pixel`` or ``world_to_array_index``, they are matched to the classes defined in ``world_axis_object_classes`` as follows:

* If every object passed in is an instance of exactly one of the classes in ``world_axis_object_classes``, the objects are matched to the classes by type, and can be given in any order.
  This is the existing APE 14 behavior and is unchanged.

* Otherwise, all objects are interpreted positionally, and must be given in the order in which the corresponding classes first appear in ``world_axis_object_components``.
  Any object that is not an instance of the class expected at its position shall be converted by passing it as the first and only argument to the initializer of that class.
  Note that the positional and keyword arguments stored alongside the class in ``world_axis_object_classes`` are not used for the conversion, since these describe how to construct the object from low-level values rather than from user input.
  Objects that are already instances of the class expected at their position are used as-is.

There is no partial matching: as soon as one of the objects requires conversion, all objects have to be given in the correct order, even those which could have been matched by type.
Once converted, the resulting objects are treated exactly as if the user had passed them in directly, so that for example any unit or frame conversions needed by the WCS are carried out as normal.
If the conversion of any object fails, an exception shall be raised, and the error message should indicate the classes that were expected and the order in which they were expected.

Note that this will only be useful in cases where the high-level object classes would accept an unambiguous single argument.
For example, for a time axis, ``1 * u.s`` will not be accepted since it requires a format, whereas ``'2026-04-03T10:12:30'`` could be accepted since it can be used to instantiate ``Time`` on its own.

Current Status and Examples
^^^^^^^^^^^^^^^^^^^^^^^^^^^

A user has a 1D WCS constructed using the ``gwcs`` package with a single pixel dimension that maps to a ``SpectralCoord`` world coordinate, and they want the pixel that corresponds to 6563 angstroms:

.. code-block:: python

   import astropy.units as u
   from astropy.coordinates import SpectralCoord
   from astropy.modeling import models
   from gwcs import coordinate_frames as cf
   from gwcs.wcs import WCS as GWCS

   pixel_frame = cf.CoordinateFrame(
       naxes=1,
       axes_type=["SPATIAL"],
       axes_order=(0,),
       unit=[u.pix],
       name="pixel"
   )
   spectral_frame = cf.SpectralFrame(
       axes_names=["wavelength"],
       unit=[u.angstrom],
       axis_physical_types=["em.wl"]
   )
   transform = models.Scale(10) | models.Shift(6000)

   wcs = GWCS([(pixel_frame, transform), (spectral_frame, None)])

Converting from pixel to world coordinates correctly gives a ``SpectralCoord`` object, and converting back from a ``SpectralCoord`` object also works:

.. code-block:: python

   >>> repr(wcs.pixel_to_world(56.3))
   '<SpectralCoord 6563. Angstrom>'
   >>> wcs.world_to_pixel(SpectralCoord(6563 * u.angstrom))
   56.300000000000004
   >>> wcs.world_to_pixel(6563 * u.angstrom)
   Traceback (most recent call last):
   ...
   ValueError: Expected the following order of world arguments: SpectralCoord

With the proposed specification here, the following should work:

.. code-block:: python

   >>> wcs.world_to_pixel(6563 * u.angstrom)
   56.300000000000004

Note that the FITS-WCS class from ``astropy`` already happens to be more permissive on input as it uses a converter function in ``world_axis_object_classes`` which is used to parse world coordinate inputs, so this specification will formalize this behavior.
As an example, defining the following WCS:

.. code-block:: python

   import astropy.units as u
   from astropy.coordinates import SpectralCoord
   from astropy.wcs import WCS

   wcs = WCS(naxis=1)
   wcs.wcs.ctype = ["WAVE"]
   wcs.wcs.cunit = ["Angstrom"]
   wcs.wcs.crpix = [1]
   wcs.wcs.crval = [6000]
   wcs.wcs.cdelt = [10]

we can do the same conversions as for the ``gwcs`` WCS above:

.. code-block:: python

   >>> repr(wcs.pixel_to_world(56.3))
   '<SpectralCoord 6.563e-07 m>'
   >>> wcs.world_to_pixel(SpectralCoord(6563 * u.angstrom))
   array(56.3)
   >>> wcs.world_to_pixel(6563 * u.angstrom)
   array(56.3)

Manipulation and re-arranging of WCS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Issue
^^^^^

There is no way within the APE 14 API to manipulate already existing WCS objects.
Simple operations that are built from the existing WCS but simplify it, such as cropping a celestial WCS to be a patch that is a subset of the existing WCS, or dropping dimensions are not exposed as methods on the WCS.
It is common to manipulate the data associated with a WCS object using operations such as slicing (cropping and dropping axes), axis reordering and changing pixel scale.
Currently these operations are implemented as "wrapper" classes, which take any APE 14 compliant object and return new WCS objects with modified inputs and outputs such as |SlicedLowLevelWCS|_ in ``astropy`` and |ResampledLowLevelWCS|_ in ``ndcube``.

These wrappers solve a real problem, and are used in many places in ``ndcube`` (such as the ``crop``, ``__getitem__``, and ``rebin`` methods), for chunking input in ``reproject``, and for plotting subsets of data in ``astropy.visualization.wcsaxes``.
The major limitation of this wrapper approach is that the type of the returned WCS is modified.
If you take a |astropy.wcs.WCS|_ object and e.g. make a resampled version using ``ResampledLowLevelWCS``, you now no longer have an ``astropy.wcs.WCS`` object - instead you have an instance of a wrapper class.
This means you cannot serialize this modified WCS back to a FITS header.
The same issue presents with ``gwcs.WCS``: while you can now serialize these wrappers to `ASDF <https://www.asdf-format.org>`__, you can no longer introspect the ``gwcs`` object state.

Proposed specification
^^^^^^^^^^^^^^^^^^^^^^

Four new methods are added to ``BaseLowLevelWCS`` to facilitate these kinds of manipulation, so that they are available on all WCS objects, including those that only implement the low-level API.
None of these will *modify* the WCS, instead returning new instances.
The following rules apply to the WCS object returned by all four methods:

* Wherever possible, the object returned should be of the same type as the original WCS.
  If the result cannot be represented by the same type of object, any other compliant WCS can be returned, for example an instance of a wrapper class.

* If the requested operation cannot be carried out, an exception shall be raised.

* The object returned shall expose at least the same level of API as the original WCS.
  In particular, if the original WCS exposes the high-level API, the object returned shall expose it too.

* If ``array_shape`` and ``pixel_bounds`` are set on the original WCS, they shall be set on the object returned, and updated to reflect the operation.

Default implementations of all four methods, which return instances of wrapper classes, shall be provided in ``BaseLowLevelWCS``.
The methods are defined as follows:

.. code-block:: python

   def with_axes(self, pixel: list[int], world: list[int]):
       """
       Return a new WCS with axes re-ordered or dropped depending on the parameters provided.

       Unlike slicing with an integer, dropping a pixel axis with this method does not fix the pixel
       coordinate along that axis to a specific value. The axes selected therefore need to be
       independent of the axes that are dropped, that is:

       * Each of the world axes selected must only depend, according to ``axis_correlation_matrix``, on
         pixel axes that are selected.

       * Each of the pixel axes selected must only require, according to
         ``reverse_axis_correlation_matrix``, world axes that are selected.

       If either of these conditions is not satisfied, an exception shall be raised.

       Parameters
       ----------
       pixel : list of int
           The indices of the pixel axes to include in the new WCS. For example, for a WCS with three
           pixel axes, this could be ``[0, 1, 2]`` to return the pixel axes unmodified, ``[0, 2]`` to
           drop the middle pixel axis, or e.g. ``[2, 0, 1]`` to re-order the axes without dropping
           them. Duplicate indices are not allowed, and neither are negative indices.
       world : list of int
           The indices of the world axes to include in the new WCS, as for ``pixel``.

       Returns
       -------
       wcs :
           A WCS object containing the selected axes.
       """


   def pixel_sliced(self, slices: tuple[slice | int, ...]):
       """
       Apply a pixel slice to the WCS to truncate or drop pixel dimensions.

       Note that the input to this method is in *pixel* order.

       Parameters
       ----------
       slices : tuple of slice or int
           A tuple ``pixel_n_dim`` long of `slice` or `int` objects. An int drops the axis, fixing the
           pixel coordinate along that axis to the value given, and a slice truncates the axis. If
           ``array_shape`` is set on the WCS, negative values for integers and for the ``start`` and
           ``stop`` attributes of `slice` objects are interpreted relative to the end of the axis, as
           for numpy arrays. If ``array_shape`` is not set, negative values cannot be interpreted and
           an exception shall be raised.

           The ``step`` attribute of a `slice` object can be `None` or 1. The behavior for other
           values of ``step`` is not defined by this API.

       Returns
       -------
       wcs :
           A WCS object modified to match the slice.
       """


   def __getitem__(self, slices):
       """
       Apply an array slice to the WCS to truncate or drop array dimensions.

       Note that the input to this method is in *array* order.

       This is identical to ``pixel_sliced`` except that the order of the inputs is reversed to array
       order, and that partial input is accepted to be consistent with the numpy slicing API: the input
       can be a single int or slice, or a tuple shorter than the number of array dimensions, in which
       case the axes that are not specified are left unmodified. A single `Ellipsis` can also be
       included, and stands for as many full slices as are needed to match the number of array
       dimensions.

       Parameters
       ----------
       slices : slice, int, Ellipsis, or tuple of these
           The slices to apply, in array order.

       Returns
       -------
       wcs :
           A WCS object modified to match the slice.
       """


   def pixel_rescaled(
       self,
       factor: int | float | tuple[int | float, ...],
       offset: int | float | tuple[int | float, ...] = 0,
   ):
       """
       Apply a scaling factor to one or more pixel axes.

       Parameters
       ----------
       factor : int, float, or tuple of int or float
           The factor by which to increase the pixel size for each pixel axis (values less than one
           decrease the pixel size). If a tuple, must be ``pixel_n_dim`` long; a scalar applies the
           same factor to all pixel axes.
       offset : int, float, or tuple of int or float, optional
           The shift of the lower edge of the 0th pixel (i.e. the pixel coordinate -0.5) of the
           resampled grid relative to the lower edge of the 0th pixel in the original underlying pixel
           grid, in units of original pixel widths. If a tuple, must be ``pixel_n_dim`` long; a scalar
           applies the same offset to all pixel axes. Defaults to zero (no offset).

       Returns
       -------
       wcs :
           A WCS object containing the rescaled WCS. If ``array_shape`` is set on the original WCS, the
           size along each axis of the rescaled WCS is the number of rescaled pixels needed to cover
           the original pixels, that is ``ceil((n - offset) / factor)`` where ``n`` is the original
           size.
       """

Only slices with a unit step, that is with a ``step`` of ``None`` or ``1``, are part of the API defined here, and all implementations shall support these.
This APE does not forbid implementations from accepting other values of ``step``, so that existing behavior does not need to be removed.
For example, ``astropy.wcs.WCS`` currently interprets a ``step`` greater than one as rescaling the pixel axis by that factor.
However, code that needs to work with any WCS should not rely on this, and should instead use ``pixel_rescaled`` to rescale pixel axes.
We recommend, but do not require, that implementations which accept other values of ``step`` consider deprecating this in favor of ``pixel_rescaled`` in cases where the values of step were treated as rescaling.

Since defining ``__getitem__`` on a class causes Python to treat instances as iterable, ``BaseLowLevelWCS`` shall also define an ``__iter__`` method that raises a ``TypeError``.

Current Status and Examples
^^^^^^^^^^^^^^^^^^^^^^^^^^^

At the moment, ``astropy.wcs.WCS`` implements ``__getitem__`` largely as specified above, as well as a ``slice`` method which is equivalent to ``pixel_sliced`` when called with ``numpy_order=False``, but other implementations do not.
In addition, ``astropy.wcs.WCS`` implements a ``sub`` method which is similar to ``with_axes``, although with an arguably more complex API.

As an example, we can define a FITS-WCS for a spectral cube:

.. code-block:: python

   from astropy.wcs import WCS

   wcs = WCS(naxis=3)
   wcs.wcs.ctype = "RA---TAN", "DEC--TAN", "WAVE"
   wcs.wcs.cunit = "deg", "deg", "nm"
   wcs.wcs.crpix = 50, 50, 1
   wcs.wcs.crval = 10, 20, 500
   wcs.wcs.cdelt = -0.01, 0.01, 0.1
   wcs.array_shape = (40, 100, 100)

Truncating axes by slicing in array order already works, and returns a ``WCS`` object:

.. code-block:: python

   >>> cutout = wcs[5:15, :, 20:60]
   >>> type(cutout)
   <class 'astropy.wcs.wcs.WCS'>
   >>> cutout.array_shape
   (10, 100, 40)

With the proposed specification, the same result can be obtained by slicing in pixel order:

.. code-block:: python

   >>> cutout = wcs.pixel_sliced((slice(20, 60), slice(None), slice(5, 15)))
   >>> cutout.array_shape
   (10, 100, 40)

At the moment, the only general way to rescale the pixel axes of a WCS, for example to describe data that has been binned by a factor of two along all axes, is to use the wrapper class from ``ndcube``, which returns an object that can no longer be written to a FITS header:

.. code-block:: python

   >>> from ndcube.wcs.wrappers import ResampledLowLevelWCS
   >>> rebinned = ResampledLowLevelWCS(wcs, 2)
   >>> type(rebinned)
   <class 'ndcube.wcs.wrappers.resampled_wcs.ResampledLowLevelWCS'>
   >>> rebinned.to_header()
   Traceback (most recent call last):
   ...
   AttributeError: 'ResampledLowLevelWCS' object has no attribute 'to_header'

With the proposed specification, the following will return a ``WCS`` object, since the rescaling can be represented in FITS-WCS:

.. code-block:: python

   >>> rebinned = wcs.pixel_rescaled(2)
   >>> type(rebinned)
   <class 'astropy.wcs.wcs.WCS'>
   >>> rebinned.wcs.cdelt
   array([-2.e-02,  2.e-02,  2.e-10])
   >>> rebinned.array_shape
   (20, 50, 50)

Note that ``astropy.wcs.WCS`` currently returns the same result for ``wcs[::2, ::2, ::2]``, but as described above, slicing with a ``step`` other than one is not part of the API defined here.

Finally, the spectral part of the WCS can be extracted with ``with_axes``, since the spectral axis is independent of the celestial axes:

.. code-block:: python

   >>> spectral = wcs.with_axes(pixel=[2], world=[2])
   >>> type(spectral)
   <class 'astropy.wcs.wcs.WCS'>
   >>> spectral.wcs.ctype[0]
   'WAVE'
   >>> spectral.array_shape
   (40,)

which is equivalent to what can currently be done with ``wcs.sub([3])``.
On the other hand, the following will raise an exception, since the first two pixel axes require the declination, which is not one of the world axes selected:

.. code-block:: python

   >>> wcs.with_axes(pixel=[0, 1, 2], world=[0, 2])
   Traceback (most recent call last):
   ...
   ValueError: pixel axes 0 and 1 require world axis 1, which is not included in the world axes selected

Reverse correlation matrix
~~~~~~~~~~~~~~~~~~~~~~~~~~

Issue
^^^^^

APE 14 defines ``axis_correlation_matrix`` as being a matrix that indicates which pixel coordinates each world coordinate depends on.
However, many real and important WCS that exist do not have the same mixing behavior between pixel and world coordinates in the reverse WCS as in the forward.
A common case of this is a rastering slit spectrograph instrument in solar physics such as `SPICE <https://spice.ias.u-psud.fr/>`__ on Solar Orbiter or `ViSP <https://nso.edu/telescopes/dkist/instruments/visp/>`__ at the DKIST.
In these instruments the two pixel dimensions of the celestial image are built up one exposure at a time, leading to each column of the image having a different time of observation.
The end result is data with 3 pixel dimensions and 4 world dimensions (for a single map scan) where the pixel dimension along the raster direction is fully correlated with both the celestial world coordinates and the temporal coordinate.
However, there is no mechanism in the APE 14 API for specifying which world coordinates are needed to derive each pixel coordinate.

Knowing which world coordinates are needed to derive each pixel coordinate is useful in itself for introspection, but it is also required to determine whether world axes can safely be dropped from a WCS (for example with the ``with_axes`` method described in `Manipulation and re-arranging of WCS`_), and to determine which world coordinates can be omitted when converting from world to pixel coordinates (see `Omitting unneeded input coordinates`_).

Proposed specification
^^^^^^^^^^^^^^^^^^^^^^

Low-level objects shall have a ``reverse_axis_correlation_matrix`` property which returns a boolean array with shape ``(pixel_n_dim, world_n_dim)``, in which element ``[i, j]`` is ``True`` if the world-to-pixel transformation requires world coordinate ``j`` in order to compute pixel coordinate ``i``, and ``False`` otherwise.
Note that the shape is the transpose of that of ``axis_correlation_matrix``, which is ``(world_n_dim, pixel_n_dim)``.

The matrix shall never understate the dependencies: if element ``[i, j]`` is ``False``, then the value of pixel coordinate ``i`` returned by ``world_to_pixel_values`` must not depend on the value given for world coordinate ``j``.
The matrix is however allowed to overstate the dependencies, that is an element can be ``True`` even if the pixel coordinate does not in practice depend on the world coordinate.

When the world coordinates carry redundant information, there can be several equally valid ways of computing a given pixel coordinate.
In the rastering slit spectrograph example above, the pixel coordinate along the raster direction could be derived either from the celestial coordinates or from the time.
Since a boolean matrix cannot express such alternatives, the matrix shall describe the world coordinates that are actually used by the implementation of ``world_to_pixel_values``.

A default implementation derived from ``axis_correlation_matrix`` shall be provided in ``BaseLowLevelWCS``, so that the property is available for all existing low-level WCS classes.
Since this default has no knowledge of the transformation, it may overstate the dependencies, and implementations should override it where they are able to determine that fewer world coordinates are needed.

Wrapper classes shall propagate the matrix of the WCS they wrap, modified as needed to reflect any changes to the pixel and world axes.

Current Status and Examples
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Prior to this APE, there existed no straightforward way to get the reverse axis correlation matrix for a given WCS.

As an example of what the implementation will look like, we can construct a simplified FITS-WCS for a rastering slit spectrograph, in which the time depends on the pixel coordinate along the raster direction.
Since FITS-WCS requires the same number of pixel and world axes, the WCS has a fourth pixel axis which we then drop by slicing:

.. code-block:: python

   from astropy.wcs import WCS

   wcs_4d = WCS(naxis=4)
   wcs_4d.wcs.ctype = "HPLN-TAN", "HPLT-TAN", "WAVE", "UTC"
   wcs_4d.wcs.cunit = "arcsec", "arcsec", "nm", "s"
   wcs_4d.wcs.cdelt = 2, 0.5, 0.01, 1
   wcs_4d.wcs.crval = 0, 0, 500, 0
   wcs_4d.wcs.pc = [[1, 0, 0, 0], [0, 1, 0, 0], [0, 0, 1, 0], [30, 0, 0, 1]]

   wcs = wcs_4d[0]

The resulting WCS has three pixel axes (raster, slit, and dispersion) and four world axes (longitude, latitude, wavelength, and time).
The existing ``axis_correlation_matrix`` shows that the time depends on the first pixel coordinate:

.. code-block:: python

   >>> wcs.axis_correlation_matrix.astype(int)
   array([[1, 1, 0],
          [1, 1, 0],
          [0, 0, 1],
          [1, 0, 0]])

With the proposed specification, the reverse matrix shows that the time is not needed to compute any of the pixel coordinates, since the first pixel coordinate is derived from the celestial coordinates:

.. code-block:: python

   >>> wcs.reverse_axis_correlation_matrix.astype(int)
   array([[1, 1, 0, 0],
          [1, 1, 0, 0],
          [0, 0, 1, 0]])

Omitting unneeded input coordinates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Issue
^^^^^

APE 14 requires all input coordinates to be passed when converting between pixel and world coordinates, even when some of them are not needed to compute the result.
In the rastering slit spectrograph example described in `Reverse correlation matrix`_, a user who wants to know which pixel corresponds to a given position on the Sun and wavelength also has to provide a time, even though the time has no impact on the result, and there is no obvious value to use.
The same can happen when converting from pixel to world coordinates, for a WCS in which one of the pixel axes does not affect any of the world coordinates, for example the pixel axis along which a set of images with a common WCS have been stacked.

Proposed specification
^^^^^^^^^^^^^^^^^^^^^^

An input coordinate is referred to as *unneeded* if none of the output coordinates of the transformation require it.
Specifically:

* A pixel coordinate is unneeded when converting from pixel to world coordinates if all the elements in the corresponding column of ``axis_correlation_matrix`` are ``False``.

* A world coordinate is unneeded when converting from world to pixel coordinates if all the elements in the corresponding column of ``reverse_axis_correlation_matrix`` are ``False``.

The low-level methods ``world_to_pixel_values``, ``world_to_array_index_values``, ``pixel_to_world_values``, and ``array_index_to_world_values`` shall accept ``None`` in place of the value for any unneeded input coordinate, and shall return the same result as if any valid value had been passed for that coordinate.
``None`` replaces the whole argument for that coordinate, and does not take part in broadcasting.
If ``None`` is passed for an input coordinate that is needed, an exception shall be raised, and the error message should indicate which coordinate is required.

The high-level methods ``pixel_to_world`` and ``array_index_to_world`` take the same inputs as their low-level equivalents, and shall accept ``None`` in the same way.

The high-level methods ``world_to_pixel`` and ``world_to_array_index`` shall accept ``None`` in place of a high-level object provided that all the world coordinates that would be extracted from that object are unneeded.
For example, ``None`` can only be passed in place of a ``SkyCoord`` if both the longitude and the latitude are unneeded.

Since ``None`` is not an instance of any of the classes in ``world_axis_object_classes``, passing ``None`` to ``world_to_pixel`` or ``world_to_array_index`` means that all objects are interpreted positionally, following the rules described in the `section on more permissive input <permissive-input_>`_.
``None`` is never passed to the initializer of the expected class.

Note that since no default implementation can be provided in ``BaseLowLevelWCS`` for this behavior, each implementation needs to handle ``None`` values itself, for example by replacing them with an arbitrary valid value before carrying out the transformation.
Wrapper classes need to either pass ``None`` through to the WCS they wrap, if the coordinate is also unneeded in that WCS, or replace it with a valid value.

Current Status and Examples
^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is not currently guaranteed to work by any of the existing implementations.
Using the rastering slit spectrograph WCS defined in the examples for `Reverse correlation matrix`_, a value has to be given for the time even though any value gives the same result:

.. code-block:: python

   >>> lon, lat, wave, time = wcs.pixel_to_world_values(10, 20, 30)
   >>> time
   array(331.)
   >>> wcs.world_to_pixel_values(lon, lat, wave, time)
   (array(10.), array(20.), array(30.))
   >>> wcs.world_to_pixel_values(lon, lat, wave, 12345)
   (array(10.), array(20.), array(30.))

With the proposed specification, the following will work:

.. code-block:: python

   >>> wcs.world_to_pixel_values(lon, lat, wave, None)
   (array(10.), array(20.), array(30.))

whereas passing ``None`` for e.g. the wavelength will raise an exception since the third pixel coordinate requires it:

.. code-block:: python

   >>> wcs.world_to_pixel_values(lon, lat, None, time)
   Traceback (most recent call last):
   ...
   ValueError: world coordinate 2 cannot be None since it is required to compute pixel coordinate 2

As an example of an unneeded pixel coordinate, we can use ``gwcs`` to construct a WCS for a stack of exposures which share the same celestial WCS, and in which the third pixel axis is the index of the exposure:

.. code-block:: python

   import astropy.units as u
   from astropy.coordinates import ICRS
   from astropy.modeling import models
   from gwcs import coordinate_frames as cf
   from gwcs.wcs import WCS as GWCS

   pixel_frame = cf.CoordinateFrame(
       naxes=3,
       axes_type=["SPATIAL", "SPATIAL", "SPATIAL"],
       axes_order=(0, 1, 2),
       unit=[u.pix, u.pix, u.pix],
       axes_names=["x", "y", "exposure"],
       name="pixel",
   )
   celestial_frame = cf.CelestialFrame(reference_frame=ICRS(), unit=(u.deg, u.deg))
   transform = (
       models.Mapping((0, 1), n_inputs=3)
       | models.Shift(-50) & models.Shift(-50)
       | models.Scale(-0.01) & models.Scale(0.01)
       | models.Pix2Sky_TAN()
       | models.RotateNative2Celestial(10, 20, 180)
   )

   wcs_stack = GWCS([(pixel_frame, transform), (celestial_frame, None)])

The ``axis_correlation_matrix`` shows that none of the world coordinates depend on the third pixel coordinate, so any value gives the same result:

.. code-block:: python

   >>> wcs_stack.axis_correlation_matrix.astype(int)
   array([[1, 1, 0],
          [1, 1, 0]])
   >>> wcs_stack.pixel_to_world_values(10, 20, 0)
   (10.424853645160226, 19.699502839519266)
   >>> wcs_stack.pixel_to_world_values(10, 20, 7)
   (10.424853645160226, 19.699502839519266)

With the proposed specification, the following will be guaranteed to work:

.. code-block:: python

   >>> wcs_stack.pixel_to_world_values(10, 20, None)
   (10.424853645160226, 19.699502839519266)

Note that this happens to already work with ``gwcs`` in this specific case, since the value is discarded before any computation is carried out, but this is not something that can currently be relied on in general.

Out of scope: introspection
~~~~~~~~~~~~~~~~~~~~~~~~~~~

There is no straightforward way of introspecting WCS objects to determine for example which axes are celestial or spectral, or which frame they are defined in.
In practice, it is possible to look at ``world_axis_object_classes`` and try to infer which high level object classes and frame definitions are used if it is known which kind of classes will be used (e.g. ``astropy.coordinates`` classes), but in the general sense, there is no robust way to determine this.
While this is a real limitation, we do not address it in this APE, but rather leave it for a future broader effort towards standard serialization of Astropy objects.

API versioning
--------------

Since user and downstream code consuming WCSes may not know if APE 14 WCSes implement the present APE, we introduce a new attribute on the API to indicate the API version.
While some of the changes here can easily be detected by consumers of WCSes, changes such as for example whether WCSes accept ``None`` (see `Omitting unneeded input coordinates`_) or being more flexible on input (see the `section on more permissive input <permissive-input_>`_) cannot be as easily inferred.
In addition, since future changes may be introduced to the API, it is safest to start versioning it.

WCSes implementing this APE shall define a ``wcsapi_version`` attribute which shall be set to the integer value ``2``.
In addition, ``BaseLowLevelWCS`` and ``BaseHighLevelWCS`` shall also include the attribute and set it to ``1`` so that existing implementations will automatically be exposed as having the original APE 14 API.

Since a wrapper class can only provide the behavior described in this APE if the WCS it wraps does too, wrapper classes that implement this APE shall not set ``wcsapi_version`` to a fixed value, but shall instead return the ``wcsapi_version`` of the WCS they wrap, treating a WCS that does not have this attribute as having a version of ``1``.

Branches and pull requests
--------------------------

* `astropy/astropy#20466 <https://github.com/astropy/astropy/pull/20466>`__ adds ``reverse_axis_correlation_matrix`` to ``BaseLowLevelWCS`` with a default implementation based on the ``axis_correlation_matrix``, along with an implementation for ``astropy.wcs.WCS`` and support in ``SlicedLowLevelWCS``

Implementation
--------------

The default implementations of the WCS manipulation methods in ``BaseLowLevelWCS`` rely on wrapper classes, so all the wrapper classes needed shall live in ``astropy.wcs.wcsapi``.
This is already the case for ``SlicedLowLevelWCS``, while wrapper classes for re-ordering and rescaling axes, equivalent to |ReorderedLowLevelWCS|_ and |ResampledLowLevelWCS|_ in ``ndcube``, will need to be added to ``astropy``.
When the original WCS exposes the high-level API, the default implementations shall make sure that the wrapper returned does too.

The changes needed in the ``astropy`` core package to implement this APE are the following:

* Adding ``reverse_axis_correlation_matrix`` to ``BaseLowLevelWCS``, ``astropy.wcs.WCS``, and the wrapper classes (this is done for ``SlicedLowLevelWCS`` in `astropy/astropy#20466 <https://github.com/astropy/astropy/pull/20466>`__).

* Updating |HighLevelWCSWrapper|_ to inherit from ``BaseLowLevelWCS`` and to forward the full low-level API to the WCS it wraps.

* Updating |HighLevelWCSMixin|_, and the |high_level_objects_to_values|_ function it relies on, to implement the more permissive handling of the objects passed as world coordinates, and to accept ``None`` in place of these objects.
  Since ``astropy.wcs.WCS``, ``gwcs.WCS``, and ``SkyProjectionAstropyView`` in ``lsst.images`` all rely on this mixin for their high-level API, this will not require any changes to the high-level API in those classes.

* Updating the low-level methods of ``astropy.wcs.WCS`` and of the wrapper classes to accept ``None`` for unneeded input coordinates.

* Adding wrapper classes for re-ordering and rescaling axes, and adding the default implementations of ``with_axes``, ``pixel_sliced``, ``__getitem__``, ``pixel_rescaled``, and ``__iter__`` to ``BaseLowLevelWCS``.

* Implementing ``with_axes``, ``pixel_sliced``, and ``pixel_rescaled`` on ``astropy.wcs.WCS`` so that a ``WCS`` object is returned whenever the result can be represented in FITS-WCS, and updating the existing ``__getitem__`` to match the specification.

* Adding ``wcsapi_version`` to ``BaseLowLevelWCS``, ``BaseHighLevelWCS``, ``astropy.wcs.WCS``, and the wrapper classes.

For other packages that provide WCS classes or wrapper classes, such as ``gwcs``, ``lsst.images``, and ``ndcube``, the only changes strictly needed before setting ``wcsapi_version`` to ``2`` are to accept ``None`` for unneeded input coordinates in the low-level methods and, for wrapper classes, to propagate ``reverse_axis_correlation_matrix`` and ``wcsapi_version``.
These packages can in addition override ``reverse_axis_correlation_matrix`` and the WCS manipulation methods where they are able to provide better results than the default implementations, and ``ndcube`` will be able to make use of the wrapper classes in ``astropy`` in place of its own ``ReorderedLowLevelWCS`` and ``ResampledLowLevelWCS``.

Backward compatibility
----------------------

All the changes in this APE either add to the APE 14 API or make it more permissive, so any existing code that uses an APE 14 compliant WCS will continue to work unchanged.
Existing WCS implementations will also continue to work without modification, and will be reported as having a ``wcsapi_version`` of ``1`` (see `API versioning`_).

Code that makes use of the features described in this APE needs to distinguish between two categories of changes:

* The new ``reverse_axis_correlation_matrix`` property (see `Reverse correlation matrix`_) and the WCS manipulation methods (see `Manipulation and re-arranging of WCS`_) have default implementations in ``BaseLowLevelWCS``.
  These can be relied on regardless of ``wcsapi_version`` for any WCS that inherits from ``BaseLowLevelWCS``, provided that the version of ``astropy`` installed implements this APE.
  For WCSes that do not override them, the default implementations may give less optimal results, namely a matrix that overstates the dependencies, and wrapper objects rather than objects of the same type as the original WCS.

* The remaining changes cannot be provided through default implementations, since they change the behavior of existing methods or the requirements on existing classes.
  These are the guarantee that the low-level API is available on high-level objects (see `Accessing the low-level API on a high-level API object`_), the more permissive handling of the objects passed as world coordinates (see the `section on more permissive input <permissive-input_>`_), and the ability to pass ``None`` for unneeded input coordinates (see `Omitting unneeded input coordinates`_).
  These should only be relied on if ``wcsapi_version`` is ``2`` or greater.
  Objects that do not have a ``wcsapi_version`` attribute should be treated as having a version of ``1``.

Alternatives
------------

The main alternative is the status quo, in which the deficiencies described above are worked around in downstream packages.
For the individual changes proposed here, we also considered the following alternatives:

* Making ``BaseHighLevelWCS`` inherit from ``BaseLowLevelWCS``, rather than requiring concrete classes to inherit from both.
  This would enforce the requirement automatically, but would break existing classes that list ``BaseLowLevelWCS`` before a high-level base class or mixin in their bases, which includes the main classes in ``astropy``, ``gwcs``, and ``lsst.images``, since Python would no longer be able to determine a consistent method resolution order.

* Continuing to rely only on wrapper classes to manipulate WCS objects, and moving these to ``astropy``.
  This does not address the main limitation of wrapper classes, which is that the object returned is not of the same type as the original WCS, even in cases where the result could be represented natively.

* Providing the WCS manipulation operations as standalone functions rather than methods.
  Functions would work with existing WCS objects without any changes to the base classes, but would have no straightforward way of returning an object of the same type as the original WCS, since only the implementation of a given WCS class knows how to do this.

* Making slices with a ``step`` other than one part of the API, as a shortcut for rescaling pixel axes.
  This is what ``astropy.wcs.WCS`` currently does, but it is ambiguous, since for a numpy array a ``step`` selects every n-th element rather than combining elements, and ``pixel_rescaled`` provides an explicit way of doing the same thing.
  We therefore only require support for unit steps, while not forbidding implementations from accepting other values.

* Deriving the dependence of pixel coordinates on world coordinates from the existing ``axis_correlation_matrix`` rather than adding ``reverse_axis_correlation_matrix``.
  As described in `Reverse correlation matrix`_, the two are not equivalent when the world coordinates carry redundant information, so this would not allow unneeded world coordinates to be identified.

* Allowing ``None`` to be passed for any input coordinate, and returning NaN for the output coordinates that require it, rather than only allowing ``None`` for unneeded coordinates.
  This would be more flexible, but would require every implementation to be able to evaluate part of its transformation with missing inputs, which is not straightforward for transformations in which the axes are coupled.

* Using separate attributes to indicate support for each of the individual changes rather than a single ``wcsapi_version`` (see `API versioning`_).
  This would allow implementations to adopt the changes one at a time, but would add an attribute for every change now and in future, and since default implementations can be provided for several of the changes, the amount of work needed to fully implement this APE is limited.

Decision rationale
------------------

<To be filled in when status changes, if applicable, as laid out in APE 1>

.. _APE 14: https://github.com/astropy/astropy-APEs/blob/main/APE14.rst

.. |astropy.wcs| replace:: ``astropy.wcs``
.. _astropy.wcs: https://docs.astropy.org/en/stable/wcs/index.html

.. |astropy.modeling| replace:: ``astropy.modeling``
.. _astropy.modeling: https://docs.astropy.org/en/stable/modeling/index.html

.. |astropy.visualization.wcsaxes| replace:: ``astropy.visualization.wcsaxes``
.. _astropy.visualization.wcsaxes: https://docs.astropy.org/en/stable/visualization/wcsaxes/index.html

.. |gwcs| replace:: ``gwcs``
.. _gwcs: https://gwcs.readthedocs.io

.. |lsst.images| replace:: ``lsst.images``
.. _lsst.images: https://pipelines.lsst.io

.. |reproject| replace:: ``reproject``
.. _reproject: https://reproject.readthedocs.io

.. |ndcube| replace:: ``ndcube``
.. _ndcube: https://docs.sunpy.org/projects/ndcube/en/stable/introduction.html

.. |astropy.wcs.WCS| replace:: ``astropy.wcs.WCS``
.. _astropy.wcs.WCS: https://docs.astropy.org/en/stable/api/astropy.wcs.WCS.html

.. |BaseLowLevelWCS| replace:: ``BaseLowLevelWCS``
.. _BaseLowLevelWCS: https://docs.astropy.org/en/stable/api/astropy.wcs.wcsapi.BaseLowLevelWCS.html

.. |BaseHighLevelWCS| replace:: ``BaseHighLevelWCS``
.. _BaseHighLevelWCS: https://docs.astropy.org/en/stable/api/astropy.wcs.wcsapi.BaseHighLevelWCS.html

.. |HighLevelWCSMixin| replace:: ``HighLevelWCSMixin``
.. _HighLevelWCSMixin: https://docs.astropy.org/en/stable/api/astropy.wcs.wcsapi.HighLevelWCSMixin.html

.. |HighLevelWCSWrapper| replace:: ``HighLevelWCSWrapper``
.. _HighLevelWCSWrapper: https://docs.astropy.org/en/stable/api/astropy.wcs.wcsapi.HighLevelWCSWrapper.html

.. |SlicedLowLevelWCS| replace:: ``SlicedLowLevelWCS``
.. _SlicedLowLevelWCS: https://docs.astropy.org/en/stable/api/astropy.wcs.wcsapi.SlicedLowLevelWCS.html

.. |high_level_objects_to_values| replace:: ``high_level_objects_to_values``
.. _high_level_objects_to_values: https://docs.astropy.org/en/stable/api/astropy.wcs.wcsapi.high_level_objects_to_values.html

.. |SpectralCoord| replace:: ``SpectralCoord``
.. _SpectralCoord: https://docs.astropy.org/en/stable/api/astropy.coordinates.SpectralCoord.html

.. |ResampledLowLevelWCS| replace:: ``ResampledLowLevelWCS``
.. _ResampledLowLevelWCS: https://docs.sunpy.org/projects/ndcube/en/stable/api/ndcube.wcs.wrappers.ResampledLowLevelWCS.html

.. |ReorderedLowLevelWCS| replace:: ``ReorderedLowLevelWCS``
.. _ReorderedLowLevelWCS: https://docs.sunpy.org/projects/ndcube/en/stable/api/ndcube.wcs.wrappers.ReorderedLowLevelWCS.html
