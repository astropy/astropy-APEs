A shared Python interface for World Coordinate Systems
------------------------------------------------------

author: Thomas Robitaille, Erik Tollerud, Stuart Mumford, Adam Ginsburg

date-created: 2017 November 7

date-last-revised: 2018 July 28

type: Standard Track

status: Accepted

*Note: this APE14 was origially accepted on 2018-Feb-28, but due to issues with
implementation was significanly revised on 2018-May-23 and is under new review*

Abstract
--------

Astronomical data are often provided with information about the “real-world”
coordinates that correspond to pixel coordinates. This mapping is the essence of
the “World Coordinate System (WCS)” concept. In addition to FITS WCS (the origin
of the “WCS” term), other WCS standards/representations are becoming necessary
for new missions and observatories (e.g., the James Webb Space Telescope or the
Large Synoptic Survey Telescope). In order to build functionality in Astropy and
affiliated packages that may require use of WCS, having to deal with different
kinds of WCS objects with different APIs will be essentially impossible. Thus,
the purpose of this APE is to define a standardized interface to WCS objects
based on simple objects (strings, scalars, and arrays). Packages implementing
WCS objects will then be able to either modify their classes to conform to the
API recommended here or build thin wrappers that conform to the API. The APE
also provides a recommendation for a high-level object to be implemented in
Astropy that understands familiar Python objects such as ``SkyCoord``, ``Time``,
and other Astropy classes.

Detailed description
--------------------

Background/Terminology
^^^^^^^^^^^^^^^^^^^^^^

One of the key basic operations necessary for any astronomical data analysis is
the association of “pixels” with their “real-world” coordinates (and vice
versa). While this concept has existed essentially for as long as computers
have been used on astronomical data, the FITS WCS standard (`Calabretta &
Greisen, 2002, A&A 395, 1077 <http://dx.doi.org/10.1051/0004-6361:20021327>`_)
gave this concept a name: “World Coordinate System” (WCS). Because of this,
“WCS” and “FITS-WCS” are often treated synonymously, but the concept is of
broader applicability, as evidenced by the existence of serialization formats
and Python packages that perform this task (e.g., gWCS, pyast). Hence, we use
the term “WCS” to refer to the broad concept of WCS, rather than the specific
FITS-WCS representation and implementation.

For the purposes of this APE, we define WCS transformations as mapping pixels to
some world coordinate (or the inverse). In our terminology, *pixel* is a
generalized concept of some form of ordered in-memory (or on-disk) storage of
data. While we use pixel as a shorthand, this need not be only two-dimensional
(the typical usage of *pixel*) - it includes one-dimensional (e.g., spectral
axes), or more-than-2-dimensional (e.g., data cubes). Similarly *world* in this
context is a generalized concept of any sort of physical coordinate used in the
data. The precise physical interpretation is not specified in the
concept of WCS. For the purposes of this APE, *world* could even be an
intermediate state of knowledge. E.g., for the unity transformation the world
coordinate *is* the pixel coordinate, for example, if no calibration information
has yet been determined. But some sense of direction and a language for the
physical type and representation of a world coordinate needs to be conveyed in
any practical implementation of WCS. A common understanding of how to convey
that information and represent it in Python is the primary subject of this
APE.

Scope
^^^^^

This APE is not meant to limit further development on more generalized
coordinate transform APIs. A future APE may define the API for accessing
“intermediate frames” as used in gWCS or the LSST transform systems. This APE
is only intended to define a clear API for converting from pixel coordinates to
a specific end set of “world” coordinates (as defined above). This APE doesn't
focus on how to get intermediate frames if they are present, or how to combine
together multiple steps of a transformation - while such functionality is useful
and should be *compatible* with this APE's interface, an API for these
operations is out-of-scope for this APE. This APE also does not address how to
*create* such WCSs. Rather it is aimed specifically at an API for the simpler
problem of actually performing the “end-to-end” transformation from pixel to
some world space.

Overview of the proposed WCS Interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Our proposal in this APE is the following:

* To develop a common API that WCS objects can expose, either directly or
  through thin wrappers, and which will allow functionality accessing the WCS
  objects to be agnostic of the actual WCS representation. We want this API to
  be as simple as possible and mainly return simple Python objects such as
  strings, lists and arrays. We refer to this as the 'low-level' API.

* To develop a high-level Astropy WCS object that can wrap any WCS object
  providing the low-level API and be smarter about the kinds of objects to
  accept or return to the user (including for example SkyCoord or Time objects).

The idea of this two-tiered approach as opposed to a single-tier approach is
that if we asked different WCS objects to provide a high-level API, this would
(1) cause a lot of duplication of logic of constructing the appropriate Astropy
objects, and (2) force those objects to return specifically Astropy objects,
whereas having a package-independent API would be better because it allows other
packages, even non-Python packages, to follow a well-defined API for the sake of
interoperability.


Pixel Conventions
"""""""""""""""""

The exact choice of which value represents the "edge" of a pixel is arbitrary
and could potentially vary across WCS implementations. Indeed, the
``astropy.wcs`` API explicitly supports multiple interpretations due to the
conflicting expectations of Python and other older tools.  However, the API this
APE  proposes opts to use a single choice to simplify user code. Because these
definitions are simply pixel-level offsets, implementations can
straightforwardly have their own internal conventions and simply offset them to
match this definition.  Hence, the specific convention used for this APE
reflects the "pixel value is center of pixel" philosophy in the FITS-WCS standard
(Section 2.1.4 of `Greisen et al., 2002, A&A 446, 747 <https://doi.org/10.1051/0004-6361:20053818>`_),
while at the same time matching the Python 0-index philosophy.  That is, the first pixel is considered
pixel ``0``, but pixel value ``(0, 0)`` is that *center* of that pixel.  Hence
the first pixel spans pixel values ``-0.5`` to ``0.5``.

Explicitly *not* covered in this specification is a definition of which
dimension in a spatial WCS is "x" vs "y".  There are too many conflicting
conventions (as well as well-founded technical reasons for various choices), and
hence that information should be embedded in the attributes that describe the
dimensions.


Low-level API
^^^^^^^^^^^^^

The proposed low-level API could be implemented as an abstract base class but we
would not require all implementations to inherit from it (if they do not inherit
from it, then they should still register with the ABC to allow code to check
``isinstance(object, LowLevelWCSAPI)``. Existing WCS classes are then free to
either implement that interface or provide thin wrappers providing this
interface. A guiding philosophy of this interface is to use relatively primitive
python objects (scalars, arrays, and strings), which can be “promoted” to more
complex and useful Python objects in the high-level interface. This allows
straightforward implementation of this interface even in C-implemented Python
objects, without tying that into the high-level interface.

Note that in this APE, when referring to arrays, we mean any Python object that
follows the buffer protocol described in `PEP3118
<https://www.python.org/dev/peps/pep-3118/>`_ rather than only specifically
Numpy arrays (which do follow the buffer protocol). This allows for other array
types, such as the `built-in Python array
<https://docs.python.org/3/library/array.html>`_ objects or future advanced data
structures.

The following class shows the required properties and methods the uniform
low-level API recommends:

.. code-block:: python

    class LowLevelWCSAPI(metaclass=abc.ABCMeta):

        @property
        def pixel_n_dim(self):
            """
            The number of axes in the pixel coordinate system
            """

        @property
        def world_n_dim(self):
            """
            The number of axes in the world coordinate system
            """

        @property
        def pixel_shape(self):
            """
            The shape of the data that the WCS applies to as a tuple of
            length ``pixel_n_dim`` (optional).

            If the WCS is valid in the context of a dataset with a particular
            shape, then this property can be used to store the shape of the
            data. This can be used for example if implementing slicing of WCS
            objects. This is an optional property, and it should return `None`
            if a shape is not known or relevant.
            """

        @property
        def pixel_bounds(self):
            """
            The bounds (in pixel coordinates) inside which the WCS is defined,
            as a list with ``pixel_n_dim`` ``(min, max)`` tuples (optional).

            WCS solutions are sometimes only guaranteed to be accurate within a
            certain range of pixel values, for example when definining a WCS
            that includes fitted distortions. This is an optional property, and
            it should return `None` if a shape is not known or relevant.
            """

        @property
        def world_axis_physical_types(self):
            """
            Returns an iterable of strings describing the physical type for each
            world axis. They should be names from the VO UCD1+ controlled
            Vocabulary (http://www.ivoa.net/documents/latest/UCDlist.html).
            If no matching UCD type exists, this can instead be "custom:xxx",
            where xxx is an arbitrary string.  Alternatively, if the physical
            type is unknown/undefined, an element can be `None`.
            """

        @property
        def world_axis_units(self):
            """
            Returns an iterable of strings given the units of the world
            coordinates for each axis. The strings should follow the recommended
            VOUnit standard (though as noted in the VOUnit specification
            document, units that do not follow this standard are still allowed,
            but just not recommended).
            """

        @property
        def axis_correlation_matrix(self):
            """
            Returns an ``(world_n_dim, pixel_n_dim)`` matrix that indicates
            using booleans whether a given world coordinate depends on a given
            pixel coordinate. This should default to a matrix where all elements
            are True in the absence of any further information. For completely
            independent axes, the diagonal would be True and all other entries
            False. The pixel axes should be ordered in the ``(x, y)`` order,
            where for an image, ``x`` is the horizontal coordinate and ``y`` is
            the vertical coordinate.
            """

        def pixel_to_world_values(self, *pixel_arrays):
            """
            Convert pixel coordinates to world coordinates. This method takes
            n_pixel scalars or arrays as input, and pixel coordinates should be
            zero-based. Returns n_world scalars or arrays in units given by
            ``world_axis_units``. Note that pixel coordinates are assumed
            to be 0 at the center of the first pixel in each dimension. If a
            pixel is in a region where the WCS is not defined, NaN can be
            returned. The coordinates should be specified in the ``(x, y)``
            order, where for an image, ``x`` is the horizontal coordinate and
            ``y`` is the vertical coordinate.
            """

        def numpy_index_to_world_values(self, *index_arrays):
            """
            Convert array indices to world coordinates. This is the same as
            ``pixel_to_world_values`` except that the indices should be given
            in ``(i, j)`` order, where for an image ``i`` is the row and ``j``
            is the column (i.e. the opposite order to ``pixel_to_world_values``).
            """

        def world_to_pixel_values(self, *world_arrays):
            """
            Convert world coordinates to pixel coordinates. This method takes
            n_world scalars or arrays as input in units given by ``world_axis_units``.
            Returns n_pixel scalars or arrays. Note that pixel coordinates are
            assumed to be 0 at the center of the first pixel in each dimension.
            to be 0 at the center of the first pixel in each dimension. If a
            world coordinate does not have a matching pixel coordinate, NaN can
            be returned.  The coordinates should be returned in the ``(x, y)``
            order, where for an image, ``x`` is the horizontal coordinate and
            ``y`` is the vertical coordinate.
            """

        def world_to_numpy_index_values(self, *world_arrays):
            """
            Convert world coordinates to array indices. This is the same as
            ``world_to_pixel_values`` except that the indices should be returned
            in ``(i, j)`` order, where for an image ``i`` is the row and ``j``
            is the column (i.e. the opposite order to ``pixel_to_world_values``).
            The indices should be returned as rounded integers.
            """

        @property
        def world_axis_object_components(self):
            """
            A list with n_dim_world elements, where each element is a tuple with
            three items:

            * The first is a name for the world object this world array
              corresponds to, which *must* match the string names used in
              ``world_axis_object_classes``. Note that names might appear twice
              because two world arrays might correspond to a single world object
              (e.g. a celestial coordinate might have both “ra” and “dec”
              arrays, which correspond to a single sky coordinate object).

            * The second element is either a string keyword argument name or a
              positional index for the corresponding class from
              ``world_axis_object_classes``

            * The third argument is a string giving the name of the property
              to access on the corresponding class from
              ``world_axis_object_classes`` in order to get numerical values.

            See below for an example of this property.
            """

        @property
        def world_axis_object_classes(self):
            """
            A dictionary with each key being a string key from
            ``world_axis_object_components``, and each value being a tuple with
            three elements:

            * The first element of the tuple must be a string specifying the
              fully-qualified name of a class, which will specify the actual
              Python object to be created.

            * The second element, should be a tuple specifying the positional
              arguments required to initialize the class. If
              ``world_axis_object_components`` specifies that the world
              coordinates should be passed as a positional argument, this this
              tuple should include ``None`` placeholders for the world
              coordinates.

            * The last tuple element must be a dictionary with the keyword
              arguments required to initialize the class.

            See below for an example of this property. Note that we don't
            require the classes to be Astropy classes since there is no
            guarantee that Astropy will have all the classes to represent all
            kinds of world coordinates. Furthermore, we recommend that the
            output be kept as human-readable as possible.

            The classes used here should have the ability to do conversions by
            passing an instance as the first argument to the same class with
            different arguments (e.g. ``Time(Time(...), scale='tai')``). This is
            a requirement for the implementation of the high-level interface.

            The second tuple element for each value of this dictionary can in
            turn contain either instances of classes, or if necessary can contain
            serialized versions that should take the same form as the main
            classes described above (a tuple with three elements with the
            fully qualified name of the class, then the positional arguments
            and the keyword arguments). For low-level API objects implemented
            in Python, we recommend simply returning the actual objects (not
            the serialized form) for optimal performance.
            """

We now take a look at an example of use of ``world_axis_object_components`` with
``world_axis_object_classes``. An example output from both methods on the same
WCS object is:

.. code-block:: python

    >>> wcs.world_axis_object_components
    [('skycoord', 'ra', 'ra.degree'),
     ('time', 0, 'tai.value'),
     ('skycoord', 'dec', 'dec.degree')]
    >>> wcs.world_axis_object_classes
    {'skycoord': ('astropy.coordinates.SkyCoord', (),
                  {'frame': 'fk5', 'equinox':'J2005'}),
     'time': ('astropy.time.Time', (None,), {'scale': 'tai', 'format': 'unix'})}

This indicates that the first and third world axis can be used to instantiate an
Astropy ``SkyCoord`` object with ``ra=`` set to the first world axis, and
``dec=`` set to the third axis, and the ``frame=fk5`` and ``equinox=J2005``
arguments, while the second world axis can be used to instantiate an Astropy
``Time`` object as the first positional argument, and with the ``scale=tai``
keyword argument. Note that the coordinate frame classes could be custom
sub-classes if needed.

Low-level API examples
^^^^^^^^^^^^^^^^^^^^^^

**Simple 1D spectrum** - a 1D mapping from pixel to wavelength:

.. code-block:: python

    wcs.axis_correlation_matrix = [[True]]
    wcs.world_axis_units = ['angstrom']
    wcs.world_axis_physical_types = ['em.wl']
    wcs.world_axis_object_components = [('spec', 0, 'value')]
    wcs.world_axis_object_classes  = {'spec':('astropy.units.Wavelength', (None,),
                                              {'airorvacwl': 'air'})}

**Simple 2D image mapping** where the pixel axes are lined up with RA and Dec
(in FITS-WCS this would be CAR)

.. code-block:: python

    wcs.axis_correlation_matrix = [[True, False], [False, True]]
    wcs.world_axis_units = ['deg', 'deg']
    wcs.world_axis_physical_types = ['pos.eq.ra', 'pos.eq.dec']
    wcs.world_axis_object_components = [(('sc', 'ra', 'ra.degree'),
                                         ('sc', 'dec', 'dec.degree')]
    wcs.world_axis_object_classes  = {'sc':('astropy.coordinates.SkyCoord', (),
                                            {'frame': 'icrs'})}

**Extremely complex spectral data cube** with 3 *pixel* dimensions and 4 *world*
dimensions. The first two *pixel* dimensions encode a mixed set of spatial
dimensions and a third dimension which is completely spectral (i.e., the output of
an IFU detector), and the third *pixel* dimension is a separable fourth world
dimension encoding time-of-observation.

.. code-block:: python

    wcs.axis_correlation_matrix = [[True, True, False],
                                   [True, True, False],
                                   [True, True, False],
                                   [False, False, True]]
    wcs.world_axis_units = ['deg', 'deg', 'angstrom', 'day']
    wcs.world_axis_physical_types = ['pos.galactic.lon', 'pos.galactic.lat', 'em.wl', 'time']
    wcs.world_axis_object_components = [('spat', 'ra', 'ra.degree'),
                                        ('spat', 'dec', 'dec.degree'),
                                        ('spec', 0, 'value'),
                                        ('time', 0, 'utc.value')]
    wcs.world_axis_object_classes  = {'spat': ('astropy.coordinates.SkyCoord', (),
                                               {'frame': 'icrs'}),
                                      'spec': ('astropy.units.Wavelength`, (None,), {}),
                                      'time': ('astropy.time.Time', (None,),
                                               {'format':'mjd', 'scale':'utc'})}

**The identity transform** for a 1D array (i.e., pixel -> pixel):

.. code-block:: python

    wcs.axis_correlation_matrix = [[True]]
    wcs.world_axis_units = ['pixel']
    wcs.world_axis_physical_types = ['instr.pixel']
    wcs.world_axis_object_components = [('spec', 0, 'value')]
    wcs.world_axis_object_classes  = {'spec':('astropy.units.pixel', (None,), {})}

Pixel and world coordinate ordering
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The API above provides a way to distinguish between pixel coordinates defined
using the standard Cartesian ordering (x, y) and array indices defined using the
row-major ordering (i.e. row, column). For example, values returned from
``world_to_pixel_values`` would be in the correct order to use for plotting
using e.g. Matplotlib, while values returned from ``world_to_numpy_index_values``
would be in the correct order to use for indexing a Numpy array. Both are valid
in different contexts and we therefore provide two methods for each transformation.

We do not mandate a specific order for the world coordinates. While it might be
tempting to assume the 'same' order as for pixel coordinates, this only makes
sense for simple cases (for example an image of the sky where ra/dec are roughly
lined up with x/y). In a generalized WCS system, such a correspondance does not
exist. As an example, consider an equatorial coordinate system rotated 45
degrees from the pixel coordinates. Such a system could be represented by the
following::

    wcs.axis_correlation_matrix = [[True, True], [True, True]]
    wcs.world_axis_physical_types = ['pos.eq.ra', 'pos.eq.dec']

or by::

    wcs.axis_correlation_matrix = [[True, True], [True, True]]
    wcs.world_axis_physical_types = ['pos.eq.dec', 'pos.eq.ra']

Neither of these is more correct than the other since ra/dec are not
preferentially lined up with x/y, so we need to allow both.

It is also possible to have a different number of world coordinates compared to
pixel coordinates. For example, we could imagine having a 1D array of values
determined by tracing a non-linear path through a spectral cube. The WCS would
look like::

    wcs.axis_correlation_matrix = [[True], [True], [True]]
    wcs.world_axis_physical_types = ['pos.eq.ra', 'pos.eq.dec', 'spect.dopplerVeloc.radio']

but the order of the coordinates is of course arbitrary, and one cannot simply
refer to the order of the pixel coordinates since there is only one pixel
coordinate that is correlated with all three world coordinates. Thus, one could
equally represent the WCS as:

    wcs.axis_correlation_matrix = [[True], [True], [True]]
    wcs.world_axis_physical_types = ['spect.dopplerVeloc.radio', 'pos.eq.ra', 'pos.eq.dec']

and there is no 'right' order.

We note that the API we present here makes it easy to create a WCS with
reorederd world coordinates - this would involve changing the order of
``world_axis_physical_types``, ``world_axis_units``, and
``world_axis_object_components``, changing the order of
``axis_correlation_matrix`` along the first dimension, and changing the order
of the inputs of the ``world_to_pixel/numpy_index_values`` methods and the
order of the outputs of the ``pixel/numpy_index_to_world_values`` methods. Thus,
implementations of the low or high-level API could provide convenience methods
to reorder or sort the world axes.

This flexibility does not however extend to pixel coordinates. For example for a
given array with an associated WCS, the output of ``world_to_numpy_index_values`` has
to consistently return the values in the order that can be used to index the
array, so the indices/pixel coordinates of the WCS cannot be re-ordered if the
data is left unchanged.

For consistency with existing WCS libraries, we recommend that implementations
based on FITS-WCS choose to order ``world_axis_physical_types`` in the same
order as the ``CTYPE`` values, but we do not require this.

Common UCD1+ names for physical types
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As outlined above, the ``world_axis_physical_types`` attribute should include
strings that follow the VO UCD1+ vocabulary for defining physical types. The
`full UCD1+ vocabulary <http://www.ivoa.net/documents/latest/UCDlist.html>`_
includes a large number of options, but here we summarize some of the most
common ones that will likely be used::

    em.energy                | Energy value in the em frame
    em.freq                  | Frequency value in the em frame
    em.wavenumber            | Wavenumber value in the em frame
    em.wl                    | Wavelength value in the em frame
    instr.pixel              | Pixel (default size: angular)
    pos.az.alt               | Alt-azimutal altitude
    pos.az.azi               | Alt-azimutal azimut
    pos.bodyrc.lat           | Body related coordinate (latitude on the body)
    pos.bodyrc.long          | Body related coordinate (longitude on the body)
    pos.cartesian.x          | Cartesian coordinate along the x-axis
    pos.cartesian.y          | Cartesian coordinate along the y-axis
    pos.cartesian.z          | Cartesian coordinate along the z-axis
    pos.ecliptic.lat         | Ecliptic latitude
    pos.ecliptic.lon         | Ecliptic longitude
    pos.eq.dec               | Declination in equatorial coordinates
    pos.eq.ra                | Right ascension in equatorial coordinates
    pos.galactic.lat         | Latitude in galactic coordinates
    pos.galactic.lon         | Longitude in galactic coordinates
    pos.healpix              | Hierarchical Equal Area IsoLatitude Pixelization
    pos.heliocentric         | Heliocentric position coordinate (solar system bodies)
    spect.dopplerVeloc       | Radial velocity, derived from the shift of some spectral feature
    spect.dopplerVeloc.opt   | Radial velocity derived from a wavelength shift using the optical convention
    spect.dopplerVeloc.radio | Radial velocity derived from a frequency shift using the radio convention
    time                     | Time, generic quantity in units of time or date
    time.epoch               | Instant of time related to a generic event (epoch, date, Julian date, time stamp/tag,...)

The full UCD1+ vocabulary does not include all possible type names that would be
needed to represent **all** WCSes (for example, there are no keywords for
helioprojective coordinates). In this case, the element of
``world_axis_physical_types`` for those coordinates should be a string prefixed
with ``custom:``. This should also be taken as a call to work with the
International Virtual Observatory Alliance (IVOA) to implement
new type names, which the Astropy Project will facilitate as needed. If a
``custom:`` type name is needed, we recommend that these be coordinated and
agreed as much as possible between different packages to make sure that these
can be useful (which would not be the case if each package created their own
set of custom type names).

High-level Astropy Object
^^^^^^^^^^^^^^^^^^^^^^^^^

Unlike the low-level API, the 'high-level' interface described here will be a
single Astropy-developed class since it interfaces with various Astropy objects.
This high-level API would provide the ability for example to get ``SkyCoord``,
``Time`` etc. objects back from a pixel to world conversion, and conversely to
be able to convert ``SkyCoord``, ``Time`` etc. to pixel values.

While we encourage the use of this class, we will also define a base class that
declares the API described here, and we allow other libraries, if needed, to
implement their own high-level object.

The high-level object would not inherit from the low-level classes but instead
wrap them. The high-level object should provide at a minimum the
following four methods:

.. code-block:: python

    def pixel_to_world(self, *pixel_arrays):
        """
        Convert pixel coordinates to world coordinates (represented by Astropy
        objects). See ``pixel_to_world_values`` for pixel indexing and ordering
        conventions.
        """

    def numpy_index_to_world(self, *index_arrays):
        """
        Convert array indices to world coordinates (represented by Astropy
        objects). See ``numpy_index_to_world_values`` for array indexing and ordering
        conventions.
        """

    def world_to_pixel(self, *world_objects):
        """
        Convert world coordinates (represented by Astropy objects) to pixel
        coordinates. See ``world_to_pixel_values`` for pixel indexing and
        ordering conventions.
        """

    def world_to_numpy_index(self, *world_objects):
        """
        Convert world coordinates (represented by Astropy objects) to array
        indices. See ``world_to_numpy_index_values`` for array indexing and ordering
        conventions. The indices should be returned as rounded integers.
        """


The low-level object must be available under the attribute name ``low_level_wcs``
and the low-level methods such as ``pixel_to_world_values`` will thus be
available by doing:

.. code-block:: python

    >>> wcs.low_level_wcs.pixel_to_world_values(...)

Since a single Astropy object might correspond to two non-contiguous dimensions
in the WCS (for example the first and third world dimensions), we need to
specify the rules for the order in which Astropy objects are returned from the
high-level ``pixel_to_world`` method, and in which order they should be given to
the high-level ``world_to_pixel`` method. The standard order should be that
given by considering only the first occurrence of the coordinate alias string in
``world_axis_object_components``. For example, if
``world_axis_object_components`` is

.. code-block:: python

    [('skycoord', 'ra', 'ra.degree'),
     ('time', 0, 'tai.value'),
     ('skycoord', 'dec', 'dec.degree')]

Then the order of the Astropy objects should be ``SkyCoord`` then ``Time`` (we
essentially ignore ``('skycoord', 'dec')``). This rule will always be followed
for ``pixel_to_world``, but on the other hand provided there is no ambiguity,
``world_to_pixel`` could be more forgiving if the coordinates are specified in
the wrong order (though an error should be raised if there are any ambiguities
and the order is not the standard one).

Branches and pull requests
--------------------------

N/A

Implementation
--------------

The following pull requests provide implementations of what is described in
this APE:

* `astropy/astropy#7325 <https://github.com/astropy/astropy/pull/7325>`_
  implements the abstract base class for the low-level API.

* `astropy/astropy#7326 <https://github.com/astropy/astropy/pull/7326>`_
  implements the FITS-WCS low-level API and the high-level API class in
  Astropy that can wrap any low-level API object.

* `spacetelescope/gwcs#146 <https://github.com/spacetelescope/gwcs/pull/146>`_
  implements a GWCS high-level object.

Backward compatibility
----------------------

N/A

Alternatives
------------

A possible alternative to consider is simply leave things status quo and have no
agreed-on API. Rather instead have Astropy endorse a specific implementation
like gwcs as the API to assume for when wcs objects are needed (e.g. NDData and
spectroscopic objects). However, this would likely lead to less uptake of the
upstream objects that require wcs. E.g., while ``astropy.wcs`` is a
commonly-used case in the present, it does not support new use cases like LSST
or the distortion models for JWST, and is tied specifically to the FITS format.
Therefore tools that wish to support both FITS WCS *and* newer systems would
have to write their own complex logic for doing so, as well as potentially even
more complex logic for converting the wcs outputs into composite Python objects.
The structure outlined in this API would make that unnecessary by instead having
a single interface that user code can write against, and only needs to consider
other details when creating or modifying wcs.

On a more specific note, the primary reason for using a string as the key for
the dictionary for ``world_axis_object_classes`` (and the corresponding names in
world_axis_object_components) is because there might be multiple world axes that
need to use the same class with different initializing parameters. Otherwise a
simpler solution would have been to use the class object *itself* as the key.

Additionally, for ``world_axis_physical_types``, an alternative was considered
of adopting a much more general set of terms vs UCD1+ such as ``"celestial"``,
``"spectral"``, etc. and just coming up with the list in this APE (possibly
using terms that approxmiately align with the STC standard).  But it was decided
that adopting the VO UCD1+ would be best because it would not lead to Astropy
needing to maintain a separate "standard" of terminology where one already
exists.


Decision rationale
------------------

The content of this APE was discussed and accepted by multiple community stakeholders
who have technical knowledge, practical experience, and project-level interest in WCS.
The APE was accepted on Feb 28, 2018.
