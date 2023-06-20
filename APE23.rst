Guidelines for required and optional dependencies
-------------------------------------------------

author: Thomas Robitaille

date-created: 2022 April 20

date-last-revised: 2023 June 20

date-accepted: <replace with accepted date>

type: Process

status: Discussion


Abstract
--------

The original scope of the Astropy project was to develop a single core package,
and it was decided at the time that the only required dependency should be Numpy
to maximize portability across platforms and ease of installation. Since then
however, there has been three instances where a required dependency has been
added to the core package (packaging, PyERFA and PyYAML), along with 19
'optional' dependencies. The extensive use of optional dependencies can cause
frustration for users who don't realise that some dependencies are not installed
by default, and there are no clear guidelines for the core package or other
project packages regarding when dependencies should be considered required or
optional.

This APE aims to provide a set of guidelines for deciding whether dependencies
for a package should be optional or required and aspects to consider when adding
a new dependency at all, with the aim to improve user experience as well as avoid
extensive discussions amongst developers every time a new dependency is proposed.
While this APE was originally written with the core package in mind, the guidelines
are general enough to be applicable to all packages in the astropy ecosystem.

Detailed description
--------------------

Background
^^^^^^^^^^

Since the inception of the Astropy project over ten years ago, Python packaging
has progressed significantly, not least with the availability now of binary
wheels for packages as well as packaging systems such as conda. At the start of
the project, getting a scientific Python stack set up (including e.g. NumPy,
SciPy, and Matplotlib) was not straighforward and required building all these
packages from source. As a result, the original vision for the project included
the goal to only have Numpy as a required dependency for the core astropy package.

Since then, there have been three instances where a required dependency has been
added:

* PyERFA (in 2020): the core astropy package used to include ERFA wrappers, which
  were then split into a separate package under the name PyERFA. However, this
  meant adding PyERFA as a required dependency to keep the core astropy working
  as it has before.

* PyYAML (in 2021): formerly an optional dependency, PyYAML is used in the core
  astropy package in order to serialize Table information losslessly in several
  formats. The main reason this was added as a required dependency was because
  it was not an obvious dependency to users - for instance a user writing a
  table to ECSV could have no reasonable way of knowing in advance that PyYAML
  would be required for this.

* packaging (in 2021): used for comparison of versions, to replace the
  deprecated build-in module ``distutils``.

Adding these dependencies has had no reported impact on users, thanks to these
dependencies being lightweight and portable, which makes their automatic
installation by tools such as pip and conda almost unnoticeable to users.

In parallel to this evolution in required dependencies, many 'optional'
dependencies have been added to the core package (19 at the time of writing).
This has created a more problematic situation where dependencies that are
needed for some parts of the astropy core package are not installed by
default and therefore with a clean and default installation of astropy,
parts of the package do not work. Part of problem is the lack of awareness
of many users of the ability to do e.g.::

    pip install astropy[all]

but for e.g. conda, there is no equivalent and optional dependencies have
to be installed by hand.

However, potentially the bigger problem is the misuse of optional dependencies
for dependencies that should really be required.

Types of optional dependencies
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are at least three distinct types of optional dependencies being used at
the moment:

* **Performance/accuracy** dependencies: if not installed, users can
  still use astropy, but if they require the absolute best performance or
  accuracy, they can install packages that will help achieve this. An example of
  such a dependency is the `bottleneck <https://pypi.org/project/Bottleneck/>`_
  package which provides fast versions of the Numpy ``nanmin``, ``nanmax`` etc.
  functions. Without bottleneck, some parts of astropy works fine, but they can
  work faster if bottleneck is installed.

* **Interfacing** dependencies: these are dependencies
  that are needed for the user to be able to e.g. convert astropy objects
  to/from classes used by other packages or to specific file formats. An example of this is the
  `pandas <https://pypi.org/project/pandas/>`_ package which is needed for
  e.g. ``Table.to_pandas`` to work. Likewise, the `h5py <https://pypi.org/project/h5py/>`_
  package is needed for e.g. ``Table.write('data.hdf5')`` to work. A sub-type of
  these dependencies are ones where the functionality requires the user to be using
  the optional dependency anyway - for example ``Table.from_pandas`` will require
  the user to have constructed a ``DataFrame`` in any case so ``pandas`` will be
  guaranteed to be installed when ``Table.from_pandas`` is executed. Likewise,
  the WCSAxes visualization framework requires the user to import Matplotlib and
  construct a figure before they start using the WCSAxes functionality.

* **Unintutive** dependencies: these are dependencies that are required for
  parts of astropy to function but where there user could not have guessed
  that the dependency would be required. An example of this was the use of PyYAML
  for table metadata serialization mentioned above, where a user writing a table
  to e.g. ECSV would have got an error stating that they needed PyYAML (this is
  much less obvious than for example stating that h5py is required for writing
  HDF5). Another example of a current optional dependency that is in this category
  is Scipy - we will take a look at this in more detail in the following section.

Example of a unintuitive dependency in the core package
-------------------------------------------------------

There are a number of sub-packages in the core astropy package where SciPy is
used behind the scenes where users could not necessarily have guessed that this
was the case, for example in:

**astropy.coordinates**

    >>> c = SkyCoord(ra=ra1*u.degree, dec=dec1*u.degree, distance=distance1*u.kpc)
    >>> catalog = SkyCoord(ra=ra2*u.degree, dec=dec2*u.degree, distance=distance2*u.kpc)
    >>> idx, d2d, d3d = c.match_to_catalog_3d(catalog)  # REQUIRES SCIPY

**astropy.cosmology**

    >>> from astropy.cosmology import WMAP9 as cosmo
    >>> cosmo.H(0)
    <Quantity 69.32 km / (Mpc s)>
    >>> cosmo.kpc_proper_per_arcmin(3)  # REQUIRES SCIPY
    <Quantity 472.97709620405266 kpc / arcmin>

**astropy.units**

    >>> from astropy.cosmology import WMAP9
    >>> z = 1100 * cu.redshift
    >>> z.to(u.K, cu.with_redshift(WMAP9))
    <Quantity 3000.225 K>
    >>> z.to(u.Mpc, cu.with_redshift(WMAP9, distance="luminosity"))   # REQUIRES SCIPY
    <Quantity 15418438.76317008 Mpc>

**astropy.convolution**

    >>> gauss_oversample = Gaussian1DKernel(3, mode='oversample', factor=10)
    >>> gauss_integrate = Gaussian1DKernel(3, mode='integrate')   # REQUIRES SCIPY

**astropy.modeling**

    >>> g_init = models.Gaussian1D(amplitude=1., mean=0, stddev=1.)
    >>> fit_g = fitting.LevMarLSQFitter()
    >>> g = fit_g(g_init, x, y)  # REQUIRES SCIPY

With such extensive usage behind the scenes, users who do not have optional
dependencies installed experience a package that is broken in many places.
SciPy is in effect a required dependency, but was made optional because of the
original requirement of the project to minimize required dependencies and
because of concerns about the impact of making it a required dependency
in terms of installation.

SciPy is an interesting example because making it a required dependency is also
not trivial since it is not necessarily easy to install on as wide a range of
platforms as e.g. NumPy, so care has to be taken here to consider impact on
different kind of users.

Proposed Guidelines
^^^^^^^^^^^^^^^^^^^

Now that we have discussed the different types of optional dependencies and seen
an example of an unintuitive optional dependency, we make the following concrete
suggestions going forward:

* Performance/accuracy dependencies should in general be kept optional, but should
  be clearly mentioned in docstrings and the documentation for parts of astropy
  that can make use of them.

* Interfacing dependencies can also be kept optional and don't even necessarily have
  to be included in the 'all' extras, in particular in cases where these
  dependencies will likely be used/installed by the user anyway. If not included
  in 'all', they should however be included at least for some continuous
  integration jobs when running the tests.

* Unintuitive dependencies should ideally never be optional. In cases where the
  depedency is large or tricky to install on some platforms, it could at least
  be made required on some platforms, but such platform-dependent dependencies
  should be minimized as much as possible. It should be first demonstrated that
  there is a real issue with installing the dependency on some platforms and that
  this will actually impact users in practice. If a dependency is actually required
  for a lot of functionality in astropy but is not easy for a non-negligeable
  fraction of users to install, it may be better to investigate whether other
  dependencies could be used instead.

Minimal installations of astropy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There is a desire by some users to have as small an installation of astropy as
possible, and these users may want to minimize dependencies. These users typically
represent a much smaller fraction of users than the typical user who just wants
things to work out of the box and does not want to run into errors with
unintuitive dependencies. Thus, it is fair to place a higher burden on users
wishing to do a minimal installation. We should ensure that documentation is
provided to show how to achieve a minimal installation. At the time of writing,
this can be done using pip with e.g.::

    pip install numpy packaging
    pip install astropy --no-deps

This will not install e.g. PyYAML and PyERFA which are only needed for some parts
of astropy. We should then make clear that it is up to the user to install
any additional dependency they may need for their use case.

We note that there is `an extensive discussion
<https://discuss.python.org/t/adding-a-default-extra-require-environment/4898/127>`_
at http://discuss.python.org to have a way to *deselect* dependencies that may be
included by default. If implemented, this would mean that we could have::

    pip install astropy

actually default to::

    pip install astropy[recommended]

and then have the option for users wanting a minimal install to do::

    pip install astropy[-recommended]

However, this would need to first be written up as a PEP, so if this were to happen
it could still be a long time before it becomes a possibility, and we cannot rely
on it at this point.

Implementation
--------------

TBD

Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
