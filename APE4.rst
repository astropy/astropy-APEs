Astropy Setup Helpers
---------------------

author: Erik M. Bray

date-created: 2013-12-26

date-last-revised: 2014-04-11

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
setuptools_ and other third-party extensions to handle tasks related to build
and installing Astropy, as well as creating and uploading source and binary
distributions of Astropy.  Interaction with these tools is primarily through
the ``setup.py`` script that is commonly shipped with Python projects.

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
astropy`` is always a prerequisite step.

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
users who may just be trying to install an affiliated package.

For the above reasons we propose separating all of Astropy's build/installation
helpers into a separate project/Python package tentatively named
``astropy_helpers`` that is released and distributed independently of Astropy
itself.  It is intended to be used as a ``setup_requires`` package for use by
Astropy's own ``setup.py``, and equivalently by that of affiliated packages.

A major advantage of having an independent release cycle for
``astropy_helpers`` is that we can respond quickly to installation problems by
deploying a new astropy-helpers bugfix release without having to do a full new
release of Astropy itself.  Then any affiliated packages that use
astropy-helpers immediately benefit as well.  This is especially useful for
responding to installation bugs introduced by external factors such as changes
in pip or setuptools, or platform-specific issues (such as OSX changing the
default compiler).

.. note::

    Throughout the rest of this APE the term ``astropy_helpers`` in teletype or
    double-backticks depending on the rendering is used to refer to the actual
    Python module of the same name, while "astropy-helpers" with a hyphen
    instead of an underscore refers to the project as it is distributed on
    PyPI.  This is partly due to a historically pecularity in Python project
    naming that prefers hyphens over underscores.


Branches and pull requests
--------------------------

A repository for the astropy_helpers project has already been created on GitHub
at https://github.com/astropy/astropy_helpers.

astropy_helpers PR `#1 <https://github.com/astropy/astropy_helpers/pull/1>`_
contains an initial implementation of the ``astropy_helpers`` package as
described in the `Implementation`_ section below.

Astropy PR `#1563 <https://github.com/astropy/astropy/pull/1563>`_ contains
the necessary changes to Astropy for it to use ``astropy_helpers`` is its
``setup.py``.  It also includes the entire astropy_helpers repository as a
git submodule in the Astropy repository.

Finally, a PR to the package-template repostory,
`#46 <https://github.com/astropy/package-template/pull/46>`, updates the
affiliated package template to use the machinery availabe in ``astropy_helpers``
rather than depending on ``astropy`` itself.

**TODO**: Currently the astropy_helpers submodule in #1563 references `my fork
of astropy_helpers <https://github.com/embray/astropy_helpers>`_ rather than
the main astropy_helpers repository.  Once the initial version of
astropy_helpers is in place the ``.gitmodules`` file in #1563 should be updated
to use the URL of the main astropy_helpers repository.  This should happen only
once this APE is given final approval.


Implementation
--------------

``astropy_helpers`` package contents
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The initial version of this proposal recommends moving the
``astropy.setup_helpers``, ``astropy.version_helpers``, and ``astropy.sphinx``
modules into the ``astropy_helpers`` package.  These modules need not retain
the same names within the ``astropy_helpers`` and their contents may be
subdivided into new modules in later versions of astropy-helpers..  A sample
implementation including these modules is provided along with this APE (see
`Branches and pull requests`_ below).

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

* All Sphinx extensions used to build the Astropy and affiliated package
  documentation, including the HTML template used for the Astropy
  documentation.  Including the Sphinx extensions allows affiliated packages
  to run automated documentation builds without requiring all of Astropy as a
  prerequisite.

The ``astropy_helpers`` package also provides a ``._compiler`` module based
(and intended to replace) the ``astropy.utils._compiler`` module, the purpose
of which is to record information about the compiler used to compile C
extension modules.  Currently, affiliated packages that wish to have this
functionality must include their own ``packagename._compiler`` module as a
standard part of the affiliated package template.  Instead, it can now be
provided by ``astropy_helpers`` just as it generates the
``packagename.version`` module.

Astropy's bundled copy of py.test and associated plug-ins are *not* distributed
as part of astropy-helpers.  All functionality in astropy-helpers is intended
for use only when building and installing a package, that is, running its
``setup.py``.  As Astropy and most affiliated packages provide a
``packagename.test()`` facility that can be used with installed versions of
the package, the test runner should be considered a runtime dependency that
should always be installed on the host system.  Furthermore, as most Astropy
affiliated packages depend on Astropy itself to run, it is reasonable to
require Astropy to be installed in order to run the tests.


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

This solution is intended primarily for Astropy and affiliated package
developers:  When they clone the Astropy repository they will get
astropy_helpers as a submodule.  When running Astropy's ``setup.py`` the path
of the astropy_helpers Git submodule is added to ``sys.path`` so that the
``astropy_helpers`` package is importable.  This copy of ``astropy_helpers``
would *not* be included in the official source distributions of Astropy as
released on PyPI.

There is one complexity of using Git submodules in this manner:  When one
clones a Git repository containing submodules, by default the submodules are
not initialized.  Instead they are left as empty directories in the repository
until the user manually runs ``git submodule update --init`` in order to clone
the submodules.  Furthermore, during normal development they must manually run
``git submodule update`` to ensure they are always using the latest revision of
the submodule required by Astropy.

However, there is a workaround to this complexity that takes a lot of thinking
out of it.  It has been pointed out that the IPython project, which has also
started using submodules for some of its features, resolves this issue by
automatically running the appropriate ``git submodule`` commands whenever its
``setup.py`` script is run, thus ensuring that the submodules are always
available and up to date when developers build IPython.  In Astropy and
affiliated packages we can do something similar.  But rather than copy/paste
the same boilerplate into every ``setup.py``, we provide a bootstrap script
that projects can include in their repositories that handles this for us.
See `astropy_helpers bootstrap script`_ below.

By automating the process of updating submodules, most Astropy developers need
not even be aware of the use of submodules unless they are actively involved
in updating astropy_helpers.


astropy_helpers via setup_requires
++++++++++++++++++++++++++++++++++

As explained earlier in this APE, setuptools provides a mechanism for listing
build-time dependencies that must be satisfied in order to run ``setup.py``
commands for that project.  If the user's system does not already satisfy
that dependency, it is downloaded from PyPI (by default), extracted from its
source archive, and installed to a temporary location that is added to
``sys.path`` for use only for the duration of installation of the package that
requested it.

This is the generally preferred approach for providing astropy-helpers to users
installing Astropy or affiliated packages.  In general, the user does not need
to be aware of astropy-helpers or to go through an extra step of installing it
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
astropy-helpers also have the option of including an archive containing the
``astropy_helpers`` package (such as a gzipped tarball or egg file) in their
repository and/or source distribution.  This has the advantage of pinning to a
"known good" version of astropy-helpers.

This works exactly the same as ``setup_requires`` in that the source archive is
unpacked and installed to a temporary location.  The only difference is that it
does not need to download the package from PyPI, and thus can be used for
offline installations.

One disadvantage of this approach (or any other that involves "bundling"
astropy-helpers with one's source code, is that the bundled version may be
missing critical bug fixes.  To address this shortcoming, a mechanism in
provided in the `astropy_helpers boostrap script`_ to automatically update to
the latest bugfix release of astropy-helpers available on PyPI if it is newer
than the bundled version.  This auto-upgrade feature can be disabled for
offline installation.


astropy_helpers bootstrap script
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To simplify integration of packages with astropy_helpers, a "bootstrap" script,
tentatively named ``ah_bootstrap.py``, is provided.  This is similar in some
ways to the ``ez_setup.py`` bootstrap script provided by the setuptools project
to enable setuptools on systems where it is not already installed.

Projects wishing to use astropy_helpers should copy ``ah_bootstrap.py`` to the
root of their project's repository.  Then they should add to the beginning of
their ``setup.py`` (before any modules from ``distutils`` or ``setuptools`` are
imported)::

    import ah_bootstrap

And in their ``setup.cfg`` they should add::

    [ah_bootstrap]
    auto_use = True

By default simply importing ``ah_bootstrap`` does not do anything but read the
``setup.cfg`` and read options out of the ``[ah_bootstrap]`` section.  These
options are used to provide arguments to a function called
``ah_bootstrap.use_astropy_helpers()``.  But this function is not actually
called without the ``auto_use = True`` option.  Alternatively, one can forgo
the ``auto_use`` feature and explicitly call ``use_astropy_helpers()`` in their
``setup.py`` like so::

    from ah_bootstrap import use_astropy_helpers
    use_astropy_helpers()

The default behavior of the ``use_astropy_helpers()`` function depends on the
context in which the ``setup.py`` was run.  If run from a cloned Git
repository, it first checks for a submodule also named "astropy_helpers" at the
root of the repository.  If found, it ensures that submodule is initialized and
up to date, then inserts it on ``sys.path`` so that the ``astropy_helpers``
package can be imported from it.  If run from an unpacked source archive, that
does not include ``astropy_helpers``, it uses the ``setup_requires`` mechanism
to install astropy-helpers from PyPI as explained in the previous section.

The sample implementation of ``use_astropy_helpers()`` supports several
optional arguments that allow package developers to control its behavior:

* ``path``: By default ``'astropy_helpers'``, this is the path (relative to the
  to ``setup.py``) of the astropy_helpers Git submodule if it exists.
  Alternatively, this may be a path to a source archive file containing
  astropy_helpers, allowing support for the bundled source archive approach to
  supporting astropy-helpers.

* ``download_if_needed``: ``True`` by default, this allows downloading
  astropy_helpers from an online package index using the ``setup_requires``
  method as described earlier.  By default this means downloading from PyPI.

* ``index_url``: This allows specifying a URL to an alternative package index
  from PyPI.  This option is ignored if ``download_if_needed`` is ``False``.

* ``use_git``: Set to ``False`` to disable all use of git commands by
  ``ah_bootstrap``.  This effectively disables support for submodule
  initialization/updates, and is mostly useful for testing.

* ``auto_upgrade``: If set to ``False`` disables checking on PyPI for newer
  versions of astropy-helpers before using any already available versions.  By
  default the auto-upgrade feature is enabled.

Finally, although not an option to ``use_astropy_helpers()``, the
``ah_bootstrap`` module also recognizes a ``--offline`` command-line argument
when running ``setup.py``.  This disables all features that try to access the
internet.  This may be useful for offline installations, so that the process
does not hang while trying to connect to the internet.

It should be noted that if all methods of bootstrapping astropy_helpers fail
it is generally not possible to continue the ``setup.py`` script.  In this
case the script fails with instructions to the user that they need
astropy_helpers installed, and how to do that.


The future
^^^^^^^^^^

This APE and the proposed initial implementation do not, for now, make many
significant changes to the modules being moved from ``astropy`` to
``astropy_heleprs``.  This will help make for a smoother transition.  But
having an astropy_helpers project does open the door to future improvements.

In particular, although ``astropy_helpers`` provides many useful utilities,
correct use of those utilities still requires a significant amount of
boilerplate in the ``setup.py`` of every project that uses them--beyond just
the ``import ah_bootscript`` part.  In fact, almost all of the ``setup.py`` of
both Astropy and the affiliated package template is boilerplate.  See, for
example, `the latest released version as of writing
<https://github.com/astropy/astropy/blob/v0.3/setup.py>`_.  With a little
refactoring this boilerplate could be significantly simplified, while at the
same time giving developers *more* control.  This APE does not propose any
specific refactoring plans, however.

Another possibility to consider is adoption of d2to1_ or its descendant pbr_.
These are projects inspired by the since-canceled distutils2 project.  They aim
to provide a solution to the previously mentioned problem that all of a Python
project's metadata is listed in executable code.  Instead, the project is
described in plain text through an extension to the ``setup.cfg`` file.  The
project's ``setup.py`` is reduced to a stub that calls out to code that reads
the metadata from that file.

Although this sounds less flexible, it still gives developers a fair amount of
control over the process through the use of various scriptable hook points.
This is where a project like ``astropy_helpers`` comes in:  It can provide a
set of common hook scripts for use with d2to1_.  For an example of an existing
project that fits this mold, see `stsci.distutils`_.  Astropy has enough
complexities that d2to1_ may require a few enhancements before it can be used
effectively with Astropy.  These include the ability to collate multiple
``setup.cfg`` files (so that each subpackage can provide its own ``setup.cfg``)
and `support for environment markers
<https://github.com/embray/d2to1/issues/2>`_.



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

The issue that first motivated this discussion was `#31
<https://github.com/astropy/package-template/issues/31>`_ in the package
template repository.  It brought up the fact that an affiliated package
using Astropy in its ``setup.py`` could not even be installed via a
``pip-requirements`` file if Astropy is not already installed.  This is because
pip runs the ``setup.py egg_info`` command to determine the dependencies of
each requirement before installing them.  This command in turn fails if Astropy
has not already been installed.  One solution to that was provided, which
returns from ``setup.py egg_info`` early without trying to ``import astropy``,
so that the command can at least partially work in this specific case.  But
that solution only increases the complexity of the boilerplate ``setup.py``,
and leads to other problems.

There was also some discussion as to whether what we are now calling
astropy_helpers should be developed in the main Astropy repository, or should
be broken out into a separate repository.  It was decided that the latter
option would be easier for use by affiliated packages, in particular through
the use of submodules as explained earlier.

Finally, there has been some objection to the naming of "astropy_helpers"--in
particular to using "astropy" in the name at all.  The objection stems from the
fact that little to non of the functionality in astropy_helpers is specific to
Astropy, or even to astronomy-related software, and that it could be more
broadly useful for other scientific packages.  It was agreed that this line of
thinking is sound in principle, but that for now we do not want the additional
support overhead involved in advertising a tool for use by the broader
scientific community.  As the product matures through use with Astropy it might
become easier to do this, at which point either the name can be changed, or the
project can be forked by anyone who wishes to champion such a project.


Decision rationale
------------------

<To be filled in when the APE is accepted or rejected>


.. _distutils: http://docs.python.org/2/library/distutils.html
.. _setuptools: http://pythonhosted.org/setuptools/
.. _d2to1: https://pypi.python.org/pypi/d2to1
.. _pbr: https://pypi.python.org/pypi/pbr
.. _stsci.distutils: https://pypi.python.org/pypi/stsci.distutils
