Removing data storage (representations) from coordinate frames
==============================================================

Authors (alphabetical): Jeff Jennings, Adrian Price-Whelan, Nathaniel Starkman, Marten van Kerkwijk
---------------------------------------------------------------------------------------------------

:date-created: 2024 11 04
:date-last-revised: 2024 11 04
:date-accepted: 202x xx xx
:type: Standard Track
:status: Discussion

Abstract
--------
Coordinate frames in astropy.coordinates currently store metadata used to construct the 
frame (i.e. for transforming between frames) and may also store coordinate *data* itself. 
This duplicates functionality with ``SkyCoord``, which acts as a container for both 
coordinate data and reference frame information. We propose to change the frame classes 
such that they only store metadata and never coordinate data. This would make the 
implementation more modular, remove ambiguity for users from having nearly duplicate 
functionality with slightly different APIs, and better satisfy the principle of 
Separation of Concerns.

Detailed description
--------------------
The coordinate frame classes (subclasses of ``BaseCoordinateFrame``, e.g., ``ICRS``, 
``Galactic``, ``FK4``, etc.) are used to represent astronomical reference frames. These 
may also contain position, velocity, time, or other information about the reference frame 
relative to another reference frame, which is used to transform coordinates between the 
reference frames that exist in the ``astropy.coordinates`` ecosystem. For example, the 
``AltAz`` frame class, which is used when referencing sky coordinates in altitude and 
azimuth, must contain a location on Earth and a time in order to transform to and from a 
fixed reference frame like the ICRS. In addition to reference frame information, these 
frame classes can **also** contain coordinate data (positions, velocities, or other 
differentials), which are stored internally using the ``Representation`` and 
``Differential`` classes.

The above reveals a problematic oddity with the coordinate frame implementation – an 
insufficient `separation of concerns <https://en.wikipedia.org/wiki/Separation_of_concerns>`_. 
The coordinate frame class deals with two separate issues: the definition of the frame 
and the storage of coordinate data within that frame. Ideally the code would be more 
modularly structured, where the coordinate frame class **only** defines the reference 
frame. It would then be the purview of another class – ``SkyCoord`` – to bring these 
concerns together and to represent data (using ``Representation`` and ``Differential`` 
classes) in a given reference frame (using a ``CoordinateFrame`` class). As a demonstration 
of the current state of duplicated functionality, these two initializations effectively 
represent the same thing, but return different objects with similar but not identical 
APIs:

.. code-block:: python

    c1 = SkyCoord(1., 2., frame="icrs", units="deg")
    c2 = ICRS(1. * u.deg, 2. * u.deg)

We suggest the situation should instead be analogous to what is the case for units, 
which know how to transform from one to another, but are not concerned with how the 
values are stored (that belongs to ``Quantity``). Translating to coordinates, units are 
like ``CoordinateFrame``, the values are the coordinate data (``Representations``), and 
``Quantity`` is like ``SkyCoord``.

Having both the frame classes as well as ``SkyCoord`` be able to store and handle data 
has resulted in a large amount of code duplication. Restructuring the ``Frame`` classes 
to remove data storage will allow for much more maintainable, de-duplicated code. It 
will also make it easier to contribute: if there is a problem one would like to solve 
in a given method, if one looks in ``SkyCoord``, one will likely find that it does not 
exist, but instead is defined on ``BaseCoordinateFrame`` and gets dynamically called via 
``SkyCoord.__getattr__``. Indeed, the construction of ``BaseCoordinateFrame`` ends up 
complicating ``SkyCoord``, which has to manage the coordinate data **through** the stored 
``BaseCoordinateFrame``.

Another issue with the current implementation of coordinate frames is that the optional 
inclusion of coordinate data makes the reference frames “multi-modal”. This creates 
different usage modes (with and without data), each exhibiting different behavior. For 
instance, some methods such as ``separation`` work fine with frames with data, while 
doing this on coordinate frames without data results in an error. This kind of 
multi-modal structure is considered an anti-pattern because it forces any code 
interacting with the frame classes to handle both cases (checking ``frame.has_data``), 
complicating the codebase unnecessarily.

All the points discussed thus far – separation of concerns and code duplication – 
concern maintainers. However user experience is the more important consideration. In 
this arena too, separating frames from data storage has its advantages. Perhaps most 
importantly, documentation will be more obvious: the methods and attributes are defined 
on ``SkyCoord``, and sphinx will know how to typeset those. It will also be easier: 
following the Zen of Python, there should be one clear way to do something. The present 
overlap leads to confusion where beginner users end up creating ``BaseCoordinateFrame`` 
objects such as ``ICRS``, when the docs are clear that these are for more advanced users 
and that ``SkyCoord`` is to be preferred. The system will also be less fragile. For 
example, if users manipulate the internal workings of ``SkyCoord`` (which is discouraged 
but possible), the coordinate data can become decoupled from the caching that ``SkyCoord`` 
performs for speed.

Finished Product
----------------
The end result of the implementation of this APE will be that coordinate frame classes 
only hold information pertaining to the reference frame they represent and never actual 
coordinate data in that reference frame. This is consistent with our mathematical 
framework, the reference frame mediates how coordinate data is understood (e.g. distance 
measures) or interacts (e.g. separation from other coordinates), but the coordinate data 
itself is actually independent of that information. Classes like ``SkyCoord`` will be 
composed structures bringing together the reference frame (an instance of a 
``BaseCoordinateFrame`` subclass) and the coordinate data (``Representation`` objects).

We illustrate this with the following pseudocode.

.. code-block:: python

    @dataclass
    class BaseCoordinateFrame:
        ...

    @dataclass
    class FK5(BaseCoordinateFrame):
        equinox: Time

    class SkyCoord:
        frame: BaseCoordinateFrame
        data: Representation

        def __init__(...):
            ...

Branches and pull requests
--------------------------
No direct progress on these changes has yet occurred. Discussion of these ideas has 
however arisen in multiple issues and pull requests, demonstrating the need for and 
utility of the proposed changes.

Several issues have been raised regarding topics such as confusion differentiating the 
use of ``frame`` and ``SkyCoord`` for data storage, and problems arising in other astropy 
subpackages when using frames that store data. For example:

- `Comparing Frame with data and SkyCoord with same data raises exception #13476 <https://github.com/astropy/astropy/issues/13476>`_
- `Add Frame objects without data to a Table #16823 <https://github.com/astropy/astropy/issues/16823>`_

Additionally, multiple pull requests have factored out common code between frames and 
``SkyCoord``, showing that there is no proper separation of concern:

- `Allow BaseCoordinateFrames to be stored in tables (by giving them .info) #16831 <https://github.com/astropy/astropy/pull/16831>`_
- `Masked frames and SkyCoord #17106 <https://github.com/astropy/astropy/pull/17016>`_ (this was later removed and instead methods were duplicated)

Further, pull requests have added methods to make frames and ``SkyCoord`` even more 
similar, underscoring that frames *with* data should not be separate entities from 
``SkyCoord``:

- `Implement BaseCoordinateFrame.to_table() #17009 <https://github.com/astropy/astropy/pull/17009>`_
- `Implement BaseCoordinateFrame.frame property #16356 <https://github.com/astropy/astropy/pull/16356>`_

Implementation
--------------
The direct use of coordinate frames instead of ``SkyCoord`` is common. In particular 
``ICRS`` objects are frequently created with data. Given the prevalent use, it is imperative 
to maintain backward compatibility and not break the API too quickly. Therefore, we 
propose implementing this APE through 3 steps (and substeps).

1. Splitting the frame classes into two hierarchies: ones with and without data.

2. Switching ``SkyCoord`` to use the data-less frame classes, and enabling automatic 
conversion of the with-data frames into ``SkyCoord`` objects.

3. Deprecating the legacy with-data frame classes.

   - Emitting warnings when instantiated.

   - Still warn, but return a ``SkyCoord``, not an instance of its class type (by overriding ``__new__``)

   - Remove.

The 3 steps (at stage 3a) are illustrated in the following pseudocode:

.. code-block:: python

    # === Reference Frame (no data) ===

    class AbstractReferenceFrame:
        ...

        # Like `unit.to`
        def transform_data_to(self, frame: AbstractReferenceFrame, data: Representation) -> Representation:
            """Used by `AbstractCoordinate` for transformation."""
            ...

    class ICRSFrame(AbstractReferenceFrame):
        ...

    class FK5Frame(AbstractReferenceFrame):
        equinox: Time

    # === Coordinates (data + frame) ===

    class AbstractCoordinate:
        """Base class for data in a reference frame."""
        ...

    class SkyCoord(AbstractCoordinate):
        frame: AbstractReferenceFrame
        data: Representation

        def __init__(...):
            # If the frame is a `AbstractLegacyCoordinate` then it is
            # split into a `AbstractReferenceFrame` and `Representation`
            ...

    # === Legacy Coordinate Classes ===

    class AbstractLegacyCoordinate(AbstractCoordinate):

        def __new__(self):
            warnings.warn("Please use SkyCoord")

        @abstractpropery # implemented on subclasses
        def frame(self) -> AbstractReferenceFrame:
            ...

    class ICRS(AbstractLegacyCoordinate, ICRSFrame):
        ...

    class FK5(AbstractLegacyCoordinate, FK5Frame):
        ...
