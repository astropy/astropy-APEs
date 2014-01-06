Astropy Setup Helpers
---------------------

author: Erik M. Bray

date-created: 2013-12-26

date-last-revised: 2013-12-26

type: Process

status: Discussion


Abstract
--------

This proposes a new Python package tentatively named ``astropy_helpers`` that
would be developed in parallel with Astropy, but distributed as a separate
product with its own release cycle.  It would contain all the utilities used
to build and distribute Astropy packaged in such a way that they can be used
by Astropy itself, as well as by Astropy affiliated packages.
``astropy_helpers`` has no dependency on Astropy as it is used to build and
install Astropy itself and other packages that may or may not depend on
Astropy.


Detailed description
--------------------

Rationale
^^^^^^^^^

Astropy uses the standard Python tools of distutils_ with extensions from
setuptools_ and other third-party extensions to handle tasks related to
build and installing Astropy, as well as creating and uploading source and
binary distributions of Astropy.  Interaction with these tools is primarily
through the ``setup.py`` script that is commonly shipped with Python projects.

Astropy also includes a collection of "helpers" that extend and enhance the
built-in functionality of distutils and setuptools, as well incorporate
workarounds for some of the shortcomings of those tools and patches for
platform-specific issues that may not yet be addressed in any official Python
releases.  These helpers are currently implemented in submodules of the
``astropy`` package.  This means that in order to even build Astropy itself
from source, much less any affiliated package that uses Astropy's helpers, it
must be possible to add those modules to Python's import path and import them
when running the project's ``setup.py`` script.  Since ``setup.py`` contains
executable Python code describing the project's metadata, it is necessary for
it to be executable for programs like ``pip`` to even extract basic information
about the project needed at installation time.

In particular, when both ``pip`` and ``easy_install`` install a package from a
source distribution, they first run the ``setup.py egg_info`` command, which
generates all the project's metadata in a plain-text format.  The wisdom of
requiring essentially arbitrary code to be executed in order to determine a
package's metadata can easily be debated, but it is currently the reality we
have to work with.

Requiring the ``astropy`` package in its entirety to be importable for that
purpose places an unnecessary burden on affiliated packages that wish to take
advantage of these helpers:  It means that Astropy must be fully installed
before their users can install the affiliated package, even if the affiliated
package only has ``astropy`` as an optional runtime dependency.  This makes it
difficult to impossible for developers of affiliated packages to be able to
provide their users a simple set of instructions such as ``pip install
<projectname>`` for installing their product.  As it is, ``pip install
Astropy`` is always a prerequisite step.

There is a feature of setuptools that allows a project's ``setup.py`` to
provide a list of ``setup_requires`` packages--these are packages that must
be available at build/installation time for that project's ``setup.py`` to
fully function.  If a package listed in ``setup_requires`` is not immediately
available, the default behavior is to obtain that package from PyPI and
temporarily add it to Python's import path just for the purposes of running
the ``setup.py`` script.

However, Astropy is a rather large and cumbersome package to use in the
context of ``setup_requires``.  Because using Astropy requires a number of
C extensions to be compiled and other complexities it is entirely possible
for it to fail even temporary installation, leading to further confusion by
users who may just be trying to install an affiliated package.  Therefore we
propose separating all of Astropy's build/installation helpers into a
separate project/Python package tentatively named ``astropy_helpers`` that is
released and distributed independently of Astropy itself.  It is intended to
be used as a ``setup_requires`` package for use by Astropy's own ``setup.py``,
and equivalently by that of affiliated packages.


Branches and pull requests
--------------------------

A repository for the astropy_helpers project has already been created on GitHub
at https://github.com/astropy/astropy_helpers.

astropy_helpers PR `#1 <https://github.com/astropy/astropy_helpers/pull/1>`
contains an initial implementation of the ``astropy_helpers`` package as
described in the `Implementation`_ section below.

Astropy PR `#1563 <https://github.com/astropy/astropy/pull/1563>`_ contains
the necessary changes to Astropy for it to use ``astropy_helpers`` is its
``setup.py``.  It also includes the entire astropy_helpers repository as a
git submodule in the Astropy repository.

TODO: Currently the astropy_helpers submodule in #1563 references
`my fork of astropy_helpers <https://github.com/embray/astropy_helpers>`_
rather than the main astropy_helpers repository.  Once the initial version
of astropy_helpers is in place the ``.gitmodules`` file in #1563 should
be updated to use the URL of the main astropy_helpers repository.


Implementation
--------------

``astropy_helpers`` contents
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The initial version of this proposal recommends moving the
``astropy.setup_helpers`` and ``astropy.version_helpers`` modules into the
``astropy_helpers`` package.  These modules need not retain the same names
within the ``astropy_helpers`` and their contents may be subdivided into new
modules.  A sample implementation including these modules is provided along
with this APE (see `Branches and pull requests`_ below).

A brief overview of the functionality in these modules includes:

* Various utilities for querying ``distutils`` for information about the
  package being installed, as well as the commands and options passed to the
  ``setup.py`` script by the user.

* Workarounds to known issues in ``distutils`` and ``setuptools``, as well as
  tweaks related to platform-specific issues, such as buggy default C
  compilers.

* Tools for generating and updating a ``version.py`` submodule in the package
  being built.  This file contains information such as the release version,
  the most recent VCS revision, and other information regarding provenance of
  the installed package.

* Tools for gathering build instructions from various subpackages of the
  package, allowing the top-level ``setup.py`` to be simpler (similarly to
  per-directory Makefiles).

* Utilities for use in configuring C modules, such as tools for querying
  pkg-config for library paths.

* Extensions to the default ``setup.py`` commands, including but not limited
  to:

  - A custom ``build_ext`` with extensions for handling Cython modules (in
    particular allowing them to be built without having Cython installed so
    long as the generated C sources are included in the source distribution).

  - A custom ``build_py`` that allows skipping running 2to3 on source modules
    that already support Python 2 and 3 simultaneously.

  - A ``bdist_dmg`` command for building DMG distributions of Astropy for OSX.

The ``astropy_helpers`` package should also provide a template based on the
``astropy.utils._compiler`` module, the purpose of which is to record
information about the compiler used to compile C extension modules.  Currently,
affiliated packages that wish to have this functionality must include their
own ``packagename.utils._compiler`` module as a standard part of the affiliated
package template.  But this module could just as easily be provided by
``astropy_helpers`` just as it generates the ``packagename.version`` module.

There remain open questions whether parts of the ``astropy.tests`` and
``astropy.sphinx`` subpackages should be moved into ``astropy_helpers``, and if
so which parts.  Although these packages are also highly reusable by affiliated
packages to implement their own testing and documentation, they are not
required to build or install the package.  Testing and documentation build can
be performed post-installation, when we may also require Astropy to be
installed as a runtime dependency.  They are also activities mostly performed
by developers/contributors, who can be reasonably expected to already have
Astropy installed in their development environment.

Backward compatibility
----------------------

This section describes the ways in which the APE breaks backward compatibility.


Alternatives
------------

If there were any alternative solutions to solving the same problem, they should
be discussed here, along with a justification for the chosen approach.


Decision rationale
------------------

<To be filled in when the APE is accepted or rejected>


.. _distutils: http://docs.python.org/2/library/distutils.html
.. _setuptools: http://pythonhosted.org/setuptools/
