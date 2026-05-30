A roadmap for package infrastructure without astropy-helpers
------------------------------------------------------------

author: Stuart Mumford, Thomas Robitaille

date-created: 2019 March 7

date-last-revised: 2019 December 12

type: Process

status: Accepted

.. _APE 4: https://zenodo.org/record/1043892
.. _PEP 508: https://www.python.org/dev/peps/pep-0508/#environment-markers
.. _PEP 517: https://www.python.org/dev/peps/pep-0517/
.. _PEP 518: https://www.python.org/dev/peps/pep-0518/
.. _tox: https://tox.readthedocs.io/en/latest/
.. _pip: https://pip.pypa.io
.. _setuptools: https://setuptools.readthedocs.io/en/latest/
.. _setuptools_scm: https://github.com/pypa/setuptools_scm
.. _astropy-helpers: https://astropy-helpers.readthedocs.io/en/stable/
.. _cython: https://cython.org/
.. _pytest: https://pytest.org/
.. _jinja2: http://jinja.pocoo.org/docs/2.10/
.. _package-template: https://github.com/astropy/package-template

Abstract
--------

The `astropy-helpers`_ package contains a collection of utilities intended to
help build and distribute the core astropy package, as well as coordinated and
affiliated packages in the Astropy ecosystem, and it is also used by packages
outside of the ecosystem (at the time of writing, almost 400 repositories on
GitHub appear to use astropy-helpers).

The original APE proposing the creation of astropy-helpers (`APE 4`_) was
accepted in June 2014. Since then, the state of the Python packaging ecosystem
has changed significantly, with advances such as binary wheels that can be
provided for all major platforms, tools such as `tox`_ and `setuptools_scm`_
which are now stable and mature, and more recently support for `PEP 517`_ and
`PEP 518`_ (in pip 19.0 or later), and many incremental updates and new features
to `setuptools`_ (in setuptools 30.3 or later). The purpose of this APE is to
show how by relying on these tools, we can significantly simplify the packaging
for astropy and all the affiliated packages and other packages using
astropy-helpers, and remove the need for astropy-helpers altogether in
pure-Python packages.

Detailed description
--------------------

We now go through the various aspects of package infrastructure that
astropy-helpers has solved in the past and how we can now address these instead
using standard packaging tools.

Changes common to all packages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We first take a look at what changes are needed for all packages, regardless of
whether they have compiled extensions or not. For pure-Python packages, if all
these changes are made, astropy-helpers is no longer needed and can be removed
from the repository.

Running tests
~~~~~~~~~~~~~

Astropy-helpers provides the ability for anyone to run tests in the source
distribution to do this with the::

    python setup.py test

command, which behind the scenes built the package, installed it into a
temporary directory, installed pytest-astropy if needed, and ran the tests
there.

However, this can now be done more cleanly using `tox`_. Tox is a tool that
makes it easy to install packages into a temporary clean Python environment
along with the dependencies declared in setup.cfg, and run a set of custom
commands - in a sense it does what the test command from astropy-helpers did,
but in a more general way (including the use of an isolated test environment).
For example, by putting the following into ``tox.ini``::

    [testenv:test]
    deps =
      pytest-astropy
    commands =
      pytest aplpy

developers will then easily be able to run the tests with::

    tox -e test

The benefit of using tox over just running pytest directly is that since behind
the scenes it creates a source distribution of the package and installs it into
a clean environment, it ensures that source distributions for the package are
fully functional and that all required files and dependencies are declared. For
packages with C/Cython extensions it should also guarantee that they are always
rebuilt before running the tests.

We recognize that one of the benefits of the astropy-helpers infrastructure is
that developers and users have come to expect that packages in the astropy
ecosystem all implement ``python setup.py test``, so in order to ensure
continuity and avoid confusion, since we can't expect user to necessarily
look at detailed documentation, we recommend that packages that no longer use
astropy-helpers still make sure that ``python setup.py test`` will not necessarily
run the tests but at least print out information about how to run the tests, and
optionally include a link to more information::

    $ python setup.py test

    Note: running tests is no longer done using ``python setup.py test``, but
    instead you will need to run the following command:

        tox -e test

    If you don't already have tox installed, you can install it with:

        pip install tox

    For more information, see https://docs.astropy.org/...

    $

We could then provide a clear documentation page in a centralized place on how
to use pytest and how to do common things the test command used to allow e.g.
running tests for specific sub-packages, or with remote data.

Some flags such as ``--remote-data`` that worked with ``python setup.py test``
should continue to work with pytest, and we could ensure that options passed to
tox are also passed to pytest. Other flags such as the use of ``-P`` to indicate
which sub-packages to test (both for the code and the docs) can be replicated
as pytest plugins (see `astropy/pytest-astropy#19 <https://github.com/astropy/pytest-astropy/pull/19>`_
for an example plugin to add ``-P`` to pytest). Aliases could be added for other
flags if needed (e.g. ``-R`` for ``--remote-data``).

For developers who do not want to always use tox to run the test, it is also
possible to run pytest directly. For pure-Python packages, developers can run,
e.g.::

    pytest

Provided that the following are defined in ``setup.cfg``::

    doctest_plus = enabled
    addopts = --doctest-rst

then running ``pytest`` will pick up the tests from the ``rst`` docs in addition
to the package tests.

For developers working on packages with compiled extensions, the preferred
solution is to install the package in editable mode before running the tests::

    pip install -e .
    pytest

Alternatively, one can build the extensions in-place before running the tests::

    python setup.py build_ext --inplace
    pytest

The latter can also be simplified using a plugin such as `pytest-build
<https://github.com/astrofrog/pytest-build>`_ which allows this example to be
written as::

    pytest packagename --build-inplace

While this APE does not require it, we still recommend that tox be used as the
**primary** way to run tests for all packages (including pure-Python packages),
since it is a much more thorough test of the package, including declared
dependencies, data files, entry points, etc.

We note that these changes have no impact on the availability of the
``package.test()`` function which is unrelated to astropy-helpers and relies
instead on the astropy core package to provide a test runner.

Below we provide a table of correspondence between options passed to
``python setup.py test`` and options passed to pytest (options passed to
``package.test()`` would be unchanged):

+----------------------------------+--------------------------------------+
| **python setup.py test option**  | **pytest equivalent**                |
+----------------------------------+--------------------------------------+
| ``--args (-a)``                  | No longer needed since this just     |
|                                  | passed arguments to pytest           |
+----------------------------------+--------------------------------------+
| ``--coverage (-c)``              | ``--cov package --cov-report html``  |
|                                  | with pytest-cov plugin               |
+----------------------------------+--------------------------------------+
| ``--docs-path``                  | No longer needed since can run pytest|
|                                  | directly on rst files                |
+----------------------------------+--------------------------------------+
| ``--open-files (-o)``            | ``--open-files``                     |
|                                  | with pytest-openfiles plugin         |
+----------------------------------+--------------------------------------+
| ``--package (-P)``               | Could be added as a pytest plugin    |
+----------------------------------+--------------------------------------+
| ``--parallel (-j)``              | ``-n``                               |
|                                  | with pytest-xdist plugin             |
+----------------------------------+--------------------------------------+
| ``--pastebin (-b)``              | ``--pastebin``                       |
+----------------------------------+--------------------------------------+
| ``--pdb (-d)``                   | ``--pdb``                            |
+----------------------------------+--------------------------------------+
| ``--pep8 (-8)``                  | ``--pep8``                           |
|                                  | with pytest-pep8 plugin              |
+----------------------------------+--------------------------------------+
| ``--plugins``                    | ``PYTEST_PLUGINS`` environment       |
|                                  | variable                             |
+----------------------------------+--------------------------------------+
| ``--readonly``                   | Not easily translatable but could    |
|                                  | find other ways to prevent tests     |
|                                  | from creating files                  |
+----------------------------------+--------------------------------------+
| ``--remote-date (-R)``           | ``--remote-data``                    |
|                                  | with pytest-remotedata plugin        |
+----------------------------------+--------------------------------------+
| ``--repeat``                     | ``--count``                          |
|                                  | with pytest-repeat plugin            |
+----------------------------------+--------------------------------------+
| ``--skip-docs``                  | No longer needed since can specify   |
|                                  | ``pytest package`` to avoid docs     |
+----------------------------------+--------------------------------------+
| ``--temp-root``                  | No longer needed since equivalent is |
|                                  | to use tox which controls the        |
|                                  | temporary environment                |
+----------------------------------+--------------------------------------+
| ``--test-path``                  | Specify path to files directly       |
+----------------------------------+--------------------------------------+
| ``--verbose-results (-V)``       | ``-v``                               |
+----------------------------------+--------------------------------------+

An example implementation of the ``--package (-P)`` option as a pytest plugin
is provided in `astropy/pytest-astropy#19 <https://github.com/astropy/pytest-astropy/pull/19>`_.

In addition to the options above, using pytest directly gives easy access to the
whole ecosystem of pytest plugins and associated command-line options and flags.

Building documentation
~~~~~~~~~~~~~~~~~~~~~~

Astropy-helpers provides a ``python setup.py build_docs`` command that
behind-the-scenes built the package then added it to ``sys.path``, then ran the
documentation build. Having the package be importable is needed as documentation
often includes API sections that are dynamically created based on the package.

As for testing, we can accomplish the same process more cleanly instead using
`tox`_ - a minimal tox configuration might look like::

    [testenv:build_docs]
    deps =
      sphinx-astropy
    commands =
      sphinx-build docs docs/_build/html -W -b html

and this would be run using::

    tox -e build_docs

Developers not wishing to use tox could also accomplish the same by doing::

    pip install -e .
    cd docs
    make html

As for testing, we recognize that one of the benefits of the astropy-helpers infrastructure is
that developers and users have come to expect that packages in the astropy
ecosystem all implement ``python setup.py build_docs``, so packages that no
longer rely on astropy-helpers could make the ``python setup.py build_docs``
command print out information about how to run the tests, and
optionally include a link to more information::

    $ python setup.py build_docs

    Note: building the documentation is no longer done using ``python setup.py build_docs``,
    but instead you will need to run the following command:

        tox -e build_docs

    If you don't already have tox installed, you can install it with:

        pip install tox

    For more information, see https://docs.astropy.org/...

    $

We could then provide a clear documentation page in a centralized place on how
to use tox to build the docs, or how to use the ``make html`` approach.

These changes will have no impact on ReadTheDocs as that service never made use
of the ``build_docs`` command, instead invoking sphinx directly with
``sphinx-build``.

Version helpers
~~~~~~~~~~~~~~~

Currently, astropy-helpers includes version helpers that take the version
defined in ``setup.cfg`` or ``setup.py`` and add a developer string when in a
checked out version of a git repository. The developer string consists of
``version.devN`` where N is the number of commits since a release.

The Python Packaging Authority (PyPA) now provide a setuptools extension called
`setuptools_scm`_ which can entirely replace the version helpers in
astropy-helpers. The way this package works is that versions are no longer
specified in ``setup.cfg`` or ``setup.py`` - instead the versions are taken from
tags. Developer version strings produced in this way are much more sophisticated
and can indicate for example if the working copy is clean or has local changes,
and whether it is a stable tagged version or a developer version. We note that
the default way of setting up setuptools_scm results in the version string not
being updated automatically in ‘editable’ installs of a package (i.e. ``pip
install -e .``), however `a workaround
<https://github.com/pypa/setuptools_scm/issues/273>`_ is available.

Switching to `setuptools_scm`_ requires minimal configuration, which is well
described in its documentation and we therefore do not repeat here. However, one
subtlety is that since it relies on tags to determine versions, for packages
such as the core astropy package which have all their tags on branches (at least
in recent years), we will need to add a 'developer' tag on ``main`` straight
after branching, e.g., after creating a ``v4.0.x`` branch we should tag the next
commit on ``main`` as ``v4.1.dev``. In any case, the documentation about `how
to release the core package
<https://docs.astropy.org/en/stable/development/releasing.html>`_ will need to
be updated to reflect the use of setuptools_scm.

Package data and entry points
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At the moment, package data and entry points for packages can be defined via
``get_package_data`` and ``get_entry_points`` in ``setup_package.py`` files.
However, this adds unnecessary complexity, as even for the core package it is
simple to define the data and entry points in ``setup.cfg`` using the
``[options.entry_points]`` and ``[options.package_data]`` sections, e.g.::

    [options.entry_points]
    console_scripts =
        fits2bitmap = astropy.visualization.scripts.fits2bitmap:main
    ...

    [options.package_data]
    astropy = astropy.cfg, CITATION, **/data/**/*
    ...

We note that the package data can also be specified using a ``MANIFEST.in`` file
and the ``include_package_data`` option in ``setup.cfg``. For more details, the
[setuptools documentation](https://setuptools.readthedocs.io/en/latest/setuptools.html#including-data-files)
has a section outlining the different ways of including package data.

Removing the ability to specify package data in `setup_package.py` files removes
the dependence of the `setup.py sdist` command on astropy-helpers, which is
essential to using the PEP 518 build time dependencies discussed below.

Using ``setup.cfg`` to define package data and other options does rely on
setuptools 30.3 or later, which is now over two years old. Nevertheless, to
minimize issues for users with older Python installations, we recommend
including a version check for setuptools inside the ``setup.py`` file.

Removal of pre/post-processing hooks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A little-known feature of astropy-helpers is the ability to define hooks in
``setup_package.py`` files for steps to be carried out before/after specific
``setup.py`` commands. We propose removing this functionality since this feature
was never well advertised and likely only used in the core astropy package, and
our proof-of-concept implementation in the core package shows that we can easily
remove this.

Changes for packages with compiled extensions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Build-time dependencies
~~~~~~~~~~~~~~~~~~~~~~~

One of the common issues that packages have regularly run into and which `PEP
518`_ solves is how to define dependencies required for building a package. The
``setup()`` function provided by setuptools takes a ``setup_requires`` argument
which can include dependencies for the build, but unfortunately the ``setup.py``
file has to be executed before the ``setup()`` function is run, which leads to
the circular issue that any code in ``setup.py`` can’t rely on packages
specified in ``setup_requires`` - thus, ``setup_requires`` does not properly
solve the issue of build-time dependencies since those dependencies are only
installed part way through the build process.

PEP 518 instead specifies that build-time dependencies can be specified in a
file called ``pyproject.toml`` that contains for example::

    [build-system]
    requires = ["setuptools", "wheel", "numpy==1.13.3"]

Provided that the package is installed with a tool such as `pip`_ which
understands this file, the build-time dependencies will be installed before the
setup.py file is executed. `PEP 517`_ takes this concept further by specifying
that the build should happen in an isolated environment, which means that one
could specify a pinned version of `cython`_ or `jinja2`_ to use even if a
different version is installed in the user’s environment. In practice, this
means that the packages listed in ``pyproject.toml`` as build dependencies do
**not** end up being installed into the user's environment - they are only
installed into a temporary environment that for all intents and purposes no
longer exists once the package is installed. Therefore, if a package is needed
as a build time and a run-time dependency, it should be specified twice - once
in ``pyproject.toml`` and once in ``install_requires``.

With this in mind, if astropy-helpers was still needed, it could therefore now
be included as a build-time dependency in ``pyproject.toml`` which removes the
need for the ``ah_bootstrap.py`` file and the git submodule. Furthermore,
astropy-helpers could be pinned to specific versions in ``pyproject.toml`` and
different packages could use different versions (thanks to the build isolation,
this will not be a problem).

For packages that need Numpy to be built, ``numpy`` should also be included in
the list of build-time dependencies in ``pyproject.toml``. Currently, when
defining C extensions that need to use numpy, we need to add
``numpy.get_include()`` to the ``include_dirs`` argument of ``Extension``.
However, this can’t be done until numpy is installed, so astropy-helpers
currently provides a way for packages to specify the string ``‘numpy’`` instead,
and replacing it with ``numpy.get_include()`` on-the-fly. This workaround will
no longer be needed once Numpy is specified in ``pyproject.toml`` since Numpy
will be installed before the extensions are defined, and we suggest that
packages should instead specify ``numpy.get_include()`` explicitly.

Note that Numpy should be pinned to the **oldest** compatible version in the
``pyproject.toml`` file (but does not need to be pinned in ``install_requires``).
This is because if a user does ``pip install astropy numpy==1.14.2``, the
pinning of Numpy in the pip command only applies to the version installed
after astropy has been built, while the version taken to build astropy is always
taken from the ``pyproject.toml`` file. This means that packages such as astropy
have to be built with the oldest compatible version of Numpy since the build
will then be forward-compatible with any later version of Numpy (this is similar
to the approach taken for conda packages). We emphasize that this does not mean
that the built package has to be used with old versions of Numpy, just that the
extensions are compiled against the oldest compatible Numpy ABI which makes it
compatible with a wider range of Numpy versions at run-time.

The oldest version should be that for which wheels are available so for packages
where this depends on Python version, environment markers can be used (see `PEP
508`_), e.g.::

    "numpy==1.13.1; python_version<'3.7'",
    "numpy==1.14.5; python_version>='3.7'",

A side benefit of this is that with these pinnings in place, building wheels
with ``pip wheel .`` will automatically create wheels compatible with all
available versions of Numpy.

Note that ``setup_requires`` should no longer be used for any build-time
dependencies in ``setup.py``/``setup.cfg``, and ``install_requires`` should
require numpy to be ``>=`` than the oldest version mentioned in
``pyproject.toml``.

In the event that you want to not install build-time dependencies into a
temporary build environment, and instead want to manage the build-time
environment yourself, you can choose to run pip with the
`--no-build-isolation <https://pip.pypa.io/en/stable/reference/pip_install/#cmdoption-no-build-isolation>`_
flag.

Cython extensions
~~~~~~~~~~~~~~~~~

Currently, astropy-helpers includes functionality to auto-generate C code from
Cython extensions and ensure that when packages are released, the C code is the
one used to compile extensions, even if Cython is installed. This was originally
done to make sure that Cython was not required to install astropy, and to avoid
issues due to differences in the generated C code from different Cython
versions. When releasing stable versions of the core package for example,
developers had to remember to run the ``build`` command before ``sdist`` to
include the generated C code, otherwise this would cause issues for users that
didn’t have Cython.

However, Cython should now be included as a build-time dependency in
``pyproject.toml`` and developers should not include generated C code in
released packages. Build-time dependencies in ``pyproject.toml`` files are
always installed from wheels, so this would not have a significant performance
impact for source distributions - and since most users installing astropy with
pip will be installing astropy wheels, this will have no impact for most users.
Note that if needed, we can even pin the Cython version in ``pyproject.toml`` to
ensure consistency across all builds. With this in place, the custom
``build_ext`` command in astropy-helpers can be removed.

Extensions
~~~~~~~~~~

Astropy-helpers provides a way for developers to use ``setup_package.py`` files
throughout a package to define extensions, provides a way to auto-detect and
define Cython extensions, and also provides helpers for handling OpenMP
extensions. This is the only part of astropy-helpers which we think it makes
sense to preserve, and we argue that it is so general that it should be released
as a package with a more generic name than astropy-helpers, such as
extension-helpers - this will allow us to also avoid breaking astropy-helpers
and instead starting fresh with a clean package (although the git history could
be preserved).

However, we note that for small packages, developers can also simply define
extensions inside ``setup.py``, which would mean that astropy-helpers (or
extension-helpers) would not needed for these packages either.

External libraries
~~~~~~~~~~~~~~~~~~

As part of the extension building, astropy-helpers provides a way to define
command-line flags when calling e.g. ``python setup.py build`` such as
``--use-system-erfa``. However, while this requires complex code to add
these options to the ``setup.py`` commands, the actual logic of determining
how to link to external libraries is implemented by individual packages
inside the ``setup_package.py`` files, and this includes asking astropy-helpers
whether a particular system library was requested, e.g.::

    if setup_helpers.use_system_library('erfa'):

We propose here to remove this complex functionality from astropy-helpers and
to instead rely on environment variables such as ``ASTROPY_USE_SYSTEM_ERFA=1``
to indicate whether to opt in to using a system library. With this, the only
required modification in ``setup_package.py`` files will be to change lines like
the above one to e.g.::

    if os.environ.get('ASTROPY_USE_SYSTEM_ERFA', 0):

Note that packages could still choose to provide command-line flags in ``setup.py``
by doing e.g.::

    if '--use-system-erfa' in sys.argv:
        os.environ['ASTROPY_USE_SYSTEM_ERFA'] = 1

Branches and pull requests
--------------------------

* The pull request `astropy/astropy-helpers#503 <https://github.com/astropy/astropy-helpers/pull/503>`_
  demonstrates the code changes to change astropy-helpers into extension-helpers.
* The pull request `astropy/astropy#9726 <https://github.com/astropy/astropy/pull/9726>`_
  shows the changes necessary for the astropy core package.
* The pull request `astropy/photutils#915 <https://github.com/astropy/photutils/pull/915>`_
  shows the changes necessary for the photutils package, which includes
  defining extensions directly without using extension-helpers.
* The pull request `sunpy/sunpy#3598 <https://github.com/sunpy/sunpy/pull/3598>`_
  changes SunPy to use ``extension_helpers``.
* The following pure-Python packages have dropped astropy-helpers and adopted
  some or all of the recommendations outlined here:

  * DKIST (`3552bbeb <https://github.com/DKISTDC/dkist/commit/3552bbeb12af0e58ae48fee447d2222f0619ec03>`_)
  * MOSViz (`spacetelescope/mosviz#179 <https://github.com/spacetelescope/mosviz/pull/179>`_)
  * CubeViz (`spacetelescope/cubeviz#492 <https://github.com/spacetelescope/cubeviz/pull/492>`_).
  * aas-timeseries (`aperiosoftware/aas-timeseries#492 <https://github.com/aperiosoftware/aas-timeseries/pull/29>`_)

Implementation
--------------

All the changes to packages are already described above, but to summarize, the
following table shows the correspondence between old astropy-helpers features
and their proposed replacement:

+-------------------------------------+-------------------------------------+
| Feature                             | Replacement                         |
+-------------------------------------+-------------------------------------+
| ``astropy.version_helpers``         | `setuptools_scm`_                   |
+-------------------------------------+-------------------------------------+
| Package data in specification       | All package data should be          |
| in ``setup_package.py`` files       | specified in ``setup.cfg``          |
|                                     | or ``MANIFEST.in``                  |
+-------------------------------------+-------------------------------------+
| ``python setup.py test``            | `tox`_ or direct use of `pytest`_   |
+-------------------------------------+-------------------------------------+
| ``python setup.py build_docs``      | `tox`_ or ``make html``             |
|                                     | or ``sphinx-build``                 |
+-------------------------------------+-------------------------------------+
| Delayed import of Numpy when        | Replaced by build dependencies      |
| building C extensions               | and isolation in `PEP 517`_ and     |
+-------------------------------------+ `PEP 518`_                          |
| Transpilation of Cython to C before |                                     |
| sdist                               |                                     |
+-------------------------------------+                                     |
| Package specific versions of        |                                     |
| astropy-helpers provided by git     |                                     |
| submodule                           |                                     |
+-------------------------------------+-------------------------------------+
| ``--use-system-*`` options          | ``ASTROPY_USE_SYSTEM_*``            |
|                                     | environment variables               |
+-------------------------------------+-------------------------------------+

We propose that a new package called extension-helpers be created starting from
astropy-helpers but with only functionality related to:

* Collecting extension definitions from ``setup_package.py`` files
* Auto-defining Cython extensions
* Determining flags for OpenMP compilation

This package would need to be declared as a build-time dependency in
``pyproject.toml`` for packages that wish to use it.

In addition to the changes described here, we also recommend moving all or as
many as possible of the options for the ``setup()`` function in ``setup.py`` to
the ``setup.cfg`` file as described in `the setuptools documentation
<https://setuptools.readthedocs.io/en/latest/setuptools.html#configuring-setup-using-setup-cfg-files>`_.

Timeline for changes
--------------------

All the changes described here can be made now. The recommendation in this APE
is to first transition the core package and all coordinated packages to use this
new infrastructure. In the process, the developer documentation for the core
package as well as the package template must be updated to reflect the new
infrastructure, and a migration guide must be written to help developers of
affiliated and other packages using astropy-helpers change over to the new
infrastructure.

Once any issues have been ironed out in the core and coordinated packages, the
migration guide will be advertised widely to all maintainers of packages
using astropy-helpers. Throughout this process, astropy-helpers will continue
to be supported, and once the migration guide is advertised the final date for
support of astropy-helpers must be made clear. The recommendation in this
APE is to support astropy-helpers until the end of the 4.0 LTS cycle at the
latest (end of 2021), but this can be re-assessed depending on the ease of
migration and the feasibility of maintaining astropy-helpers if there are
extensive breaking changes in its dependencies.

Impact for users who are not developers
---------------------------------------

If done properly, these changes should have no noticeable impact for users.
Users will still be able to pip install (or conda install when available)
packages and for large packages such as the core astropy package, which contains
a lot of compiled extensions, most users will not see any
difference since they will be installing astropy from pre-built packages (wheels
or conda packages). Running tests using ``package.test()`` will still be
supported.

Impact for non-core developers
------------------------------

The immediate impact for developers of packages that use astropy-helpers is
having to update the layout and infrastructure in their packages to follow the
new guidelines presented here. However, we emphasise that these changes are
recommendations and not mandatory, and astropy-helpers will continue to work as
expected as long as it is included as a submodule. However, we recommend that
astropy-helpers no longer be actively developed, in which case guarantees could
not be made that some aspects of the astropy-helpers infrastructure will not
break with future releases of Python, setuptools, or sphinx for example.

However, we believe that the initial effort to switch over to the new guidelines
will be a worthwhile investment - in all cases it will mean being able to get
rid of astropy-helpers as a submodule and the confusion and headaches this can
cause, and it will make for example the definition of package data much simpler
and not have to worry about ``setup_package.py`` files in most cases.

The astropy `package-template`_ will be updated to reflect the latest
recommendations, which will make it easier for developers to update their
packages.

Impact for contributors
-----------------------

Users who wish to contribute fixes to the core astropy package or other packages
will be encouraged to have `tox`_ installed if they want to easily run
tests or build documentation locally. However, this is an easy package to
install with `pip`_ and we could also add code in ``setup.py`` so that running
``python setup.py test`` or ``python setup.py build_docs`` gives a helpful
error message with instructions on updating and using tox to run tests and build
the documentation.

We note however that using tox is just a convenience and will not be compulsory,
especially for pure-Python packages where running pytest directly will work.

Impact on core astropy developers
---------------------------------

One of the main benefits of these changes will be to not have to maintain
astropy-helpers any longer. Some parts of astropy-helpers have relied on hacks
that can be brittle and break with new setuptools or Sphinx releases.
Maintenance will still be needed for the proposed extension-helpers package but
this will be a much smaller package than astropy-helpers, and by making it more
generic and usable by any package, we hope to attract contributions from beyond
the Astropy team.

Impact on package managers
--------------------------

By relying on standard packaging infrastructure, this should facilitate the job
of people involved in package managers - for example, the conda-forge
infrastructure currently recommends that recipes should use pip to build
packages, but this is not possible or easy at the moment for packages using
astropy-helpers since they need to pass custom flags to setup.py to prevent the
default behavior of checking for astropy-helpers releases online.

Backward compatibility
----------------------

The changes described here are opt-in, so barring any breaking changes in
Python, setuptools, or Sphinx, everything should continue to work as expected if
developers do not make any changes.

Alternatives
------------

It is likely that continuing changes to ``pip`` and other Python packaging
infrastructure will cause things in ``astropy_helpers`` to break (see
[#501](https://github.com/astropy/astropy-helpers/issues/501)). This will
introduce a progressively increasing maintenance burden as our hacks become less
and less standard.

It is also possible but [explicitly discouraged by
tox](https://tox.readthedocs.io/en/latest/example/basic.html#integration-with-setup-py-test-command)
to overload the ``setup.py test`` command to run tox. While also possible to
continue to overload these setup commands, doing this in a way that doesn't
involve copying files between repositories would continue to mandate the use of
the submodule.

Finally, in the words of Stuart: *Do nothing, suffer submodules and maintaining the
spaghetti code of astropy-helpers until we demoralise the whole community and
Julia takes over.*

Decision rationale
------------------

Following *extensive* discussion on the mailing list and Github (141 comments on
this APE's PR...), and corresponding revision, this APE received unanimous
approval from the attendees of  the 2019 Astropy Coordination meeting (including
all members of the coordinating committee).
