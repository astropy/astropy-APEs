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

Note: Throughout the rest of this APE the term ``astropy_helpers`` in teletype
or double-backticks depending on the rendering is used to refer to the actual
Python module of the same name, while "astropy_helpers" unadorned refers to
the project as a whole.

*TODO*: Consider renaming the project to "astropy-helpers" with a hyphen
instead of "astropy_helpers".  Although admittedly confusing, it is how
setuptools normalizes distribution names.


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

A major advantage of having an independent release cycle for astropy_helpers
is that we can respond quickly to installation problems by releasing a new
astropy_helpers bugfix version without having to do a full new release of
Astropy itself.  Then any affiliated packages that use astropy_helpers
immediately benefit as well.  This is especially useful for responding to
installation bugs introduced by external factors such as changes in pip or
setuptools, or platform-specific issues (such as OSX changing the default
compiler).


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

*TODO*: Currently the astropy_helpers submodule in #1563 references `my fork of
astropy_helpers <https://github.com/embray/astropy_helpers>`_ rather than the
main astropy_helpers repository.  Once the initial version of astropy_helpers
is in place the ``.gitmodules`` file in #1563 should be updated to use the URL
of the main astropy_helpers repository.

*TODO*: Create a PR to the package-template for affiliated packages to use
astropy_helpers.


Implementation
--------------

``astropy_helpers`` package contents
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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


Integration with astropy_helpers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are a few possibilities for integrating astropy_helpers into a project,
but the most agreeable solution seems to be one that combines the best of
both worlds for both audiences:  Astropy developers, and users who only wish
to install Astropy, and subcategories therein.  It also gives authors of
affiliated packages choices in how they want to use astropy_helpers in their
package.

The current sample implementation provides three means of including
astropy_helpers in a project, though there is a possibility for adding more
in the future:


astropy_helpers as a Git submodule
++++++++++++++++++++++++++++++++++

`Submodules <http://git-scm.com/docs/git-submodule>`_ are an officially
supported mechanism in Git for including an entire Git repository as a
subdirectory of another Git repository as a means of "vendoring" other
projects.  In this case, as astropy_helpers is developed in its own
repository, it can be included in Astropy's main repository as a submodule.

This solution is intended primarily for Astropy developers:  When they clone
the Astropy repository they will get astropy_helpers as a submodule.  When
running Astropy's ``setup.py`` the path of the astropy_helpers Git submodule is
added to ``sys.path`` so that the ``astropy_helpers`` package is importable.
This copy of astropy_helpers would *not* be included in the official source
distributions of Astropy as released on PyPI.

There is one complexity of using Git submodules in this manner:  When one
clones a Git repository containing submodules, by default the submodules are
not initialized.  Instead they are left as empty directories in the repository
until the user manually runs ``git submodule update --init`` in order to clone
the submodules.  Furthermore, during normal development they must manually run
``git submodule update`` to ensure they are always using the latest revision
of the submodule required by Astropy.

However, there is a workaround to this complexity that takes a lot of thinking
out of it.  It has been pointed out that the IPython project, which has also
started using submodules for some of its features, resolves this issue by
automatically running the appropriate ``git submodule`` commands whenever its
``setup.py`` script is run, thus ensuring that the submodules are always
available and up to date when developers build IPython.  In Astropy and
affiliated packages we can do something similar.  But rather than copy/paste
the same boilerplate into every ``setup.py``, we provide a bootstrap script
that projects can include in their repositories that handles this for us.
See `astropy_helpers boostrap script`_ below.

By automating the process of updating submodules, most Astropy developers need
not even be aware of the use of submodules unless they are actively involved
in updating astropy_helpers.


astropy_helpers via setup_requires
++++++++++++++++++++++++++++++++++

As explained earlier in this APE, setuptools provides a mechanism for listing
build-time dependencies that must be satisfied in order to run ``setup.py``
commands for that project.  If the user's system does not already satisify
that dependency, it is downloaded from PyPI (by default), extracted from its
source archive, and installed to a temporary location that is added to
``sys.path`` for use only for the duration of installation of the package that
requested it.

This is the generally preferred approach for providing astropy_helpers to users
installing Astropy or affiliated packages.  In general, the user does not need
to be aware of astropy_helpers or to go through an extra step of installing it
first.  So long as they are connected to the internet it will *just work*.
That said, it should still be mentioned in the documentation, in particular for
the benefit of users who need to be able to perform offline installations.

There is one major shortcoming to the ``setup_requires`` approach that is worth
explaining:  Every ``setup.py`` script ends by calling a function called
``setup()`` that is the entry point to distutils/setuptools.  All of the
project's metadata, including ``setup_requires``, is provided to distutils as
arguments to the ``setup()`` function.  However, many of the tools provided by
astropy_helpers have the purpose of *generating* that metadata that is passed
through ``setup()``.  So it is necessary to be able to import the
``astropy_helpers`` package *before* the call to ``setup()``, even though
the list of ``setup_requires`` packages is not processed *until* ``setup()``
is called, making it seemingly unhelpful for this purpose.

There is however a workaround to this problem by going through setuptools'
public API to force the temporary installation of build-time dependencies in
the same manner that it does for ``setup_requires``.  This workaround is
implemented by the `astropy_helpers bootstrap script`_, so as long as that
script is imported at the beginning of a project's ``setup.py`` it will
ensure that the ``astropy_helpers`` package is importable.


astropy_helpers as a bundled source archive
+++++++++++++++++++++++++++++++++++++++++++

Although not used by Astropy itself, projects wishing to require
astropy_helpers also have the option of including an archive containing
astropy_helpers (such as a gzipped tarball or egg file) in their repository
and/or source distribution.  This has the advantage of pinning to a
"known good" version of astropy_helpers, though it loses the advantage to their
users of automatically using the most up to date astropy_helpers.  Nonetheless
the option is provided for completeness.

This works exactly the same as ``setup_requires`` in that the source archive
is unpacked and installed to a temporary location.  The only difference is that
it does not need to download the package from PyPI, and thus can be used for
offline installations.


astropy_helpers boostrap script
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To simplify integration of packages with astropy_helpers, a "bootstrap" script,
tentatively named ``ah_bootstrap.py``, is provided.  This is similar in some
ways to the ``ez_setup.py`` bootstrap script provided by the setuptools project
to enable setuptools on systems where it is not already installed.

Projects wishing to use astropy_helpers should copy ``ah_bootstrap.py`` to the
root of their project's repository.  Then they should add to the beginning of
their ``setup.py`` (before any modules from ``distutils`` or ``setuptools`` are
imported)::

    from ah_bootstrap import use_astropy_helpers
    use_astropy_helpers()

The default behavior of the ``use_astropy_helpers()`` function depends on the
context in which the ``setup.py`` was run.  If run from a cloned Git
repository, it first checks for a submodule also named "astropy_helpers" at the
root of the repository.  If found, it ensures that submodule is initialized and
up to date, then inserts it on ``sys.path`` so that the ``astropy_helpers``
package can be imported from it.  If run from an unpacked source archive, that
does not include ``astropy_helpers``, it uses the ``setup_requires`` mechanism
to install astropy_helpers from PyPI as explained in the previous section.

The sample implementation of ``use_astropy_helpers()`` supports three optional
arguments that allow package developers to control its behavior:

* ``path``: By default ``'astropy_helpers'``, this is the path (relative to the
  to ``setup.py``) of the astropy_helpers Git submodule if it exists.
  Alternatively, this may be a path to a source archive file containing
  astropy_helpers, allowing support for the bundled source archive approach to
  supporting astropy_helpers.

* ``download_if_needed``: ``True`` by default, this allows downloading
  astropy_helpers from an online package index using the ``setup_requires``
  method as described earlier.  By default this means downloading from PyPI.

* index_url: This allows specifying a URL to an alternative package index
  from PyPI.  This option is ignored if ``download_if_needed`` is ``False``.

It should be noted that if all methods of bootstrapping astropy_helpers fail
it is generally not possible to continue the ``setup.py`` script.  In this
case the script fails with instructions to the user that they need
astropy_helpers installed, and how to do that.



Backward compatibility
----------------------

This change only affects installation and packaging of Astropy and affiliated
packages that use Astropy's package template.  It does not affect any APIs
used by users once the package is installed.

However, some affiliated packages, and especially older versions of those
packages, may rely on the ``astropy.setup_helpers`` and/or
``astropy.version_helpers`` so there will have to be a deprecation process
for those modules even after Astropy itself switches over to using
astropy_helpers.  Once those modules and the code in them are marked as
deprecated there should be no new non-critical changes to them.  I would
recommend removing them by the Astropy 1.0 release.


Alternatives
------------

If there were any alternative solutions to solving the same problem, they should
be discussed here, along with a justification for the chosen approach.


Decision rationale
------------------

<To be filled in when the APE is accepted or rejected>


.. _distutils: http://docs.python.org/2/library/distutils.html
.. _setuptools: http://pythonhosted.org/setuptools/
