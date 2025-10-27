NDData Plan
-----------

authors: Thomas Robitaille, Perry Greenfield, Matt Craig

date-created: 2014 October 2

date-last-revised: 2014 December 17

date-accepted: 2014 December 17

type: Standard Track

status: Accepted

Abstract
--------

This APE is intended to provide a long-term plan for the ``astropy.nddata``
sub-package. The package has been the subject of continuous debate since the
start of the astropy project, and has changed in scope several times, so this
APE is aimed at agreeing on the scope and future of the sub-package and
providing a well-structured foundation on which future development of
``astropy.nddata`` can occur.

Detailed description
--------------------

Introduction
^^^^^^^^^^^^

At the first Astropy coordination meeting in 2011, it was decided that as well
as having a generic table container, it would be useful to have a generic
container for gridded data. The ``astropy.table`` package has since then seen a
large amount of development, and the API has now stabilized. The added value of
the table package compared to using simple Numpy structured arrays is clear -
the ``Table`` class makes it very easy to do common operations on tables such
as adding or removing columns or rows, and reading/writing tables to common
file formats.

On the other hand, ``NDData`` development has stagnated and we have not been
able to converge on a stable API. Part of this is due to the fact that there is
in fact a huge variety of 'n-dimensional datasets' and that there is very
little in common for example between a spectrum and an image, in terms of what
can be done with them. This has prevented the ``NDData`` class from including
much functionality, and in fact we have been adding functionality then removing
it after we have realized that it is not general enough.

An example to illustrate this issue is that of arithmetic - what happens when
we add two n-dimensional datasets? Of course, the data values can be added, but
what happens to the mask, to the flags, or to the uncertainties? The truth is
that there is no general recipe for dealing with this, and that it may be a
mistake to try and define it in such a general way.

This APE takes the approach of re-thinking the purpose of NDData and trying to
define a scope for the future.

The 'why' of NDData
^^^^^^^^^^^^^^^^^^^

First, why do we actually need a generic data container that can be
sub-classed? What is the benefit of this versus simply defining separate base
classes for spectra, images, and other types of data? There are several
possible answers:

1. We want to provide users with a consistent experience across data objects -
   that is, the user should know that meta-data is always consistently named
   ``meta``, that a mask can always be accessed with ``mask``, and that the
   data can be accessed with ``data``.

2. By providing a common base class which can define a unified I/O interface
   which taps into the astropy I/O registry, we can seamlessly make it that
   all data objects have ``read`` and ``write`` methods that behave
   consistently.

3. We want functions and methods in Astropy to know if an object passed to them
   is an n-dimensional data object that can be expected to have specific
   attributes (such as ``wcs``, ``mask``, and so on).

In principle, none of these *require* a base class. We could simply agree on a
standard for data objects that defines what certain attributes should be
called. This is a valid solution, but at the same time, having a base class can
enforce this and factor out some boilerplate code, and as described in
`Alternatives`_, the questions raised here would apply to a base ``Image``
class that was sub-classed as ``XRayImage``, ``CCDImage``, and so on.

Proposal for an ``NDDataBase`` abstract base class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The first proposal in this APE is to separate a definition of the ``NDData``
interface from concrete realizations of ``NDData``-like objects by creating an
abstract base class called ``NDDataBase``.

The proposed ``NDDataBase`` simplifies the current ``NDData`` class to the extreme,
such that it only defines which properties are needed for ``NDData``.

The ``NDDataBase`` class should **not** define any arithmetic operations, which are
impossible to generalize.

The following properties should be included in the base class:

* ``data`` - the data itself. No restrictions are placed on the type of this
  data in the ABC. Subclasses could, for example, make it a plain Numpy array,
  masked Numpy array, an   Astropy Quantity, or an h5py data buffer. Subclasses
  could also require, for   example, that ``data`` provides a ``shape``
  attribute and/or be sliceable   in order to 'prove' that it is an
  n-dimensional data object.

* ``mask`` - the mask of the data, following the Numpy convention of `True`
  meaning masked, and `False` meaning unmasked. Sub-classes could choose to
  connect this to ``data.mask``. Masks do not need to be Numpy arrays, they
  could be for example 'lazy' masks based on functions that will be evaluated
  on-the-fly.

* ``unit`` - the unit of the data values, which will be internally
  represented as an Astropy Unit. If present, subclasses should try to ensure
  numerical operations properly take into account and propagate units.
  Sub-classes could choose to connect this to ``data.unit``, in which case data
  should be a ``Quantity`` or behave like it.

* ``wcs`` - an object that can be used to describe the relationship between
  input and world coordinates. This can (but does not
  have to) be an Astropy WCS object. Once the generalized WCS system is in
  place in Astropy, we will probably require this to be such an object.
  Subclasses are free to be more restrictive in what they permit for the
  ``wcs`` object.

* ``meta`` - a dict-like object that can be used to contain arbitrary metadata.
  This could be a plain Python dict, an ordered dict, a FITS Header object, and
  so on, provided that it offers dict-like item access and iteration.

* ``uncertainty`` -- an object that represents the uncertainty in the
  data for each element on the array.  This APE places no restriction on
  what type of uncertainty this is (e.g. variance,  standard deviation,
  or posisson count rate), nor does it require the attribute to be set
  at all (other than defaulting to ``None``). It places only one restriction
  on ``uncertainty``: it must have an attribute ``uncertainty_type``,
  which should be a human-readable string.

  While not a requirement, the following ``uncertainty_type`` strings
  are strongly recommended for common ways of specifying normal
  distributions:

  * ``"std"``: if ``uncertainty`` stores the standard deviation/sigma
    (either a single value or on a per-pixel basis).
  * ``"var"``: if ``uncertainty`` stores the variance (either a single
    value or on a per-pixel basis).
  * ``"ivar"``: if ``uncertainty`` stores the inverse variance (either a
    single value or on a per-pixel basis).

The only **required** attribute is ``data``; all others default to ``None`` if
not initialized or overridden in a subclass.

Proposal for the ``NDData`` class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``NDData`` class would be a concrete class that is far less restrictive
than the current ``NDData`` while still providing some basic functionality as
a container. It will place loose restrictions on what ``data`` can be set to
and is described further in `Implementation`_.


The ``NDData`` class would **not** include methods such as ``__array__``,
``__array_prepare__``, and so on which allow a class to be treated as a Numpy
array. This behavior has been identified as being potentially ambiguous in
the general case because it will depend on the details of e.g. how masks are
handled and also does not make it explicit in what units the data is required.

Proposal for mixin classes to provide additional functionality
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Specific functionality such as uncertainty handling and arithmetic can be
developed as mix-in classes that can be used by ``NDData`` sub-classes.

Generic slicing capabilities, further described in `Implementation`_, will be
provided as a mixin class called ``NDSlicing``

Handling of ``NDData`` in Astropy and affiliated packages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If a user has a data object such as an image, it would be nice if they can use
functions directly on this image and have them return an image object. At the
same time, we do not want to force people to use special data containers if
they have for example a Numpy array and a WCS object. This raises the question
of whether we should duplicate the API for all functions, to provide one
interface for ``NDData`` subclasses, and one for separate attributes. The
proposal in this APE is that functions should only define a single API that
takes separate keyword arguments for e.g. ``data``, ``mask``, and so on, but
that we then provide a way for users to be able to call these functions with
``NDData`` sub-classes (see `Implementation`_).

Implementation
--------------

``NDDataBase`` class
^^^^^^^^^^^^^^^^^^^^

``NDDataBase`` will be implemented as an abstract base class. The only input
validation it will provide is enforcing the existence of an
``uncertainty_type`` attribute if ``uncertainty`` is not ``None``, as described
above.

We will not include setters for properties except ``mask`` and ``uncertainty``
because it is ambiguous what the meaning of setting e.g. the unit or WCS after
initialization means: it could either mean to change the unit or WCS, or it
could mean that the user wants to convert the data to this new unit or WCS.
Given this ambiguity, it is safer to not have setters for the core attributes
and this is consistent with e.g. ``Quantity``.


``NDData`` class
^^^^^^^^^^^^^^^^

``NDData`` will be a concrete subclass of ``NDDataBase`` that provides some logic for handling the setting of ``data``:

* If the object passed in as ``data`` has a ``shape`` attribute, is
  sliceable, and has an ``__array__`` method, so that it can be easily used as
  a numpy array, then ``NDData.data`` will be set to that object.
* Otherwise, ``NDData`` will attempt to create a ``numpy.ndarray`` from the
  input ``data`` and use that as the internal representation of the data.

I/O mixin
^^^^^^^^^

The ``read`` and ``write`` methods will be developed via a mixin class.

Slicing mixin
^^^^^^^^^^^^^

This APE suggests adding a mixin class, ``NDSlicing``, to handle basic
slicing. This could be done by simply having code similar to the following
inside ``__getitem__``::

    def __getitem__(self, slice):

        new = self.__class__()

        if self.data is not None:
            new._data = self.data[slice]

        if self.mask is not None:
            new._mask = self.mask[slice]

        if self.wcs is not None:
            new._wcs = self.wcs[slice]
        ...

Note that this is only meant as an illustration of the idea suggested here,
and the final implementation will likely differ from this - but the basic
idea is that the slicing would be delegated to the member attributes. For
example, the WCS class would need to define itself how it should be sliced.
Some attributes (such as ``meta``) would not necessarily need to be sliceable.

Note the effect of slicing on attributes presumably returns a similar object,
e.g., for wcs, it returns a new WCS appropriate to the sliced data attribute.


Arithmetic mixin
^^^^^^^^^^^^^^^^

The arithmetic methods currently in ``NDData`` will be implemented in a mixin
called ``NDArithmetic``.

Facilitating the use of ``NDData`` sub-classes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

One question that has come up as part of several affiliated packages is how
to deal with ``NDData`` objects in functions. For example, if we consider a
``downsample`` function that can downsample an image, should the function
accept only ``NDData`` (or sub-class) objects? Should it also
accept plain Numpy arrays? If so, how do we pass any additional meta-data
such as WCS? Should we return a downsampled Numpy array and downsampled WCS,
or a single downsampled ``NDData`` (or sub-class) instance? In this example, one option would
be to provide two APIs, one for ``NDData`` and/or sub-classes and one for separate Numpy arrays
and attributes, but maintaining two parallel APIs is not an ideal solution.
An alternative is for each function to encode the logic of checking the input
type and deciding on the output type based on the output type. However, this
means repeating a lot of similar code such as::

    def downsample(data, wcs=None)

        if isinstance(data, NDData):
            if wcs is not None:
                raise ValueError("wcs cannot be specified if NDData instance was passed")
            wcs = data.wcs
            data = data.data

and this will become a lot more complex once more attributes are needed by
the function.

In order to make it easier for functions to accept ``NDData`` sub-classes and
return these, we can implement a decorator that will automatically split up an
``NDData`` object as needed. Let us consider the following function::

    def test(data, wcs=None, unit=None, n_iterations=3):
        ...

We can provide a decorator called e.g. ``support_nddata``::

    @support_nddata
    def test(data, wcs=None, unit=None, n_iterations=3):
        ...

which makes it so that if the user passes an ``NDData`` sub-class called e.g.
``nd``, the function would automatically be called with::

    test(nd.data, wcs=nd.wcs, unit=nd.unit)

That is, the decorator looks at the signature of the function and checks if any
of the arguments are also properties of the ``NDData`` object, and passes them
as individual arguments.

An error could be raised if an ``NDData`` property is set but the function does
not accept it - for example, if ``wcs`` is set, but the function cannot support
WCS objects, an error would be raised. On the other hand, if an argument in the
function does not exist in the ``NDData`` object or is not set, it is simply
left to its default value. This behavior could be customizable but the
details are beyond the scope of this APE document.

If the function call succeeds, then the decorator will make a new ``NDData``
object (with the correct class) and will populate the properties as needed. In
order to figure out what is returned by the function, the decorator will need
to accept a list which gives the name of the output values::

    @support_nddata(returns=['data', 'wcs'])
    def test(data, wcs=None, unit=None, n_iterations=3):
        ...

Finally, the decorator could be made to restrict input to specific ``NDData``
sub-classes (and sub-classes of those)::

    @support_nddata(accepts=CCDImage, returns=['data', 'wcs'])
    def test(data, wcs=None, unit=None, n_iterations=3):
        ...

With this decorator, the functions could be seamlessly used either with
separate arguments (e.g. Numpy array and WCS) or with subclasses of
``NDData`` such as ``CCDImage``.

Example of subclassing from both NDData and Quantity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``Quantity`` class  would benefit from the ability to share the same
interface that NDData provides and to tap into NDData's metadata and WCS
handling.

Because subclassing from ``numpy.ndarray`` involves subtleties that differ
from typical subclassing in python, an example subclass called ``NDQuantity``
may be implemented as part of ``astropy.nddata``. Should it turn out to be
unreasonably difficult to do, an attempt may be made to implement a class
which uses ``Quantity`` as the data store, with the ability to link ``NDData``
properties like ``unit`` to the underlying properties in ``Quantity``. Should
that also prove to be unworkable, an explanation of the issues that prevented
implementation may be provided in the documentation for ``NDData``.

Branches and pull requests
--------------------------

Initial decorator implementation: https://github.com/astropy/astropy/pull/2855

Initial refactoring of NDData: https://github.com/astropy/astropy/pull/2905

Backward compatibility
----------------------

This APE will require packages such as ``specutils`` and ``ccdproc`` to
completely refactor how they use the ``NDData`` class. This will also break
compatibility with users currently using ``NDData`` directly, but this is
assumed to be a very small fraction (if any) of users.

Alternatives
------------

Eliminate ``NDData``
^^^^^^^^^^^^^^^^^^^^

One alternative is to remove the ``NDData`` class altogether and to start
the base classes at the level of ``Spectrum`` or ``Image``. In this case many
of this ideas of this APE (including the attribute names, decorators, etc.)
would still apply to these base classes. The benefits of having a base
``NDData`` class instead of starting at the ``Image`` and ``Spectrum`` level
are that:

* The ``NDData`` class enforces the naming of the base properties to ensure
  consistency across all sub-classes.

* It allows slicing to be implemented at the core level as a mixin, whereas
  this would need to be repeated in each base class if we had e.g.
  ``Spectrum``, ``Image``, ``SpectralCube`` as the base classes.

* It allows the connection to the unified I/O framework to be defined once,
  whereas this would also need to be repeated in each base class otherwise.

On the other hand, the downsides of having a core ``NDData`` class is that it
reduces flexibility of the sub-classes - for instance ``Spectrum`` has to be
implemented taking into consideration the restrictions on e.g. attribute
names defined by the sub-classes. In the
`spectral-cube <https://spectral-cube.readthedocs.io>`_ package, at the moment
we do not have a ``data`` attribute because we have a custom masking
framework and define attributes like ``unmasked_data``. Of course, we should
aim to make this more compliant with what is decided here, but this is just
to demonstrate that this type of flexibility may be lost. However, this may
be a good thing as it enforces consistency for users.

Subclass NDData from ``astropy.units.Quantity`` or ``numpy.ndarray``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The original implementation of the ``NDData`` class behaved like a numpy
``ndarray``; an alternative to making ``NDData`` a more generic container is
to make it a full-fledged subclass of ``ndarray`` or of ``Quantity``. The
advantage of this approach is that it potentially reduces duplication of code
by using the infrastructure of ``Quantity`` and/or ``nddata``.

It has the disadvantage of reducing the flexibility of ``NDData`` and presents
the challenge of handling the attributes (especially ``meta``, ``mask`` and
``wcs``) in a sensible way for arbitrary operations on an ``NDData``. Even in
one of the most straightforward cases, the addition of two ``NDData`` objects
with metadata, it is unclear what the ``meta`` of the result should be.

There is a need for a more generic container with metadata than would be
possible if subclassing from ``ndarray``. In addition, it would be
straightforward to implement a subclass of the ``NDData`` proposed in this APE
that ties the ``unit`` and (when they are available in ```Quantity``) ``mask``
and ``uncertainty`` to those properties of the ``data`` attribute. In other
words, a subclass which is essentially a ``Quantity`` with ``meta`` wrapped in
the ``NDData`` interface is straightforward.

If ``NDData`` subclasses from ``ndarray`` then it will be difficult or
impossible to subclass a more generic container from it, which is likely to
lead, down the road, to the need for the type of generic container proposed in
this APE.

Decision rationale
------------------

This APE led to lengthy discussion both
`on the mailing list <https://groups.google.com/forum/#!topic/astropy-dev/3KeYQqbNblo>`_
and in the `astropy-APEs Pull Request <https://github.com/astropy/astropy-APEs/pull/8>`_.
This produced some major changes to the APE, leading to the current form.
A final vote was held, with unanimous support.  Hence this APE was accepted
2014 December 17.
