Astropy Setup Helpers
---------------------

author: Erik M. Bray

date-created: 2013-12-26

date-last-revised: 2013-12-26

type: Process

status: Discussion


Abstract
--------

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
from source, much less any affiliated-package that uses Astropy's helpers, it
must be possible to add those modules to Python's import path and import them
when running the project's ``setup.py`` script.  Since ``setup.py`` contains
executable Python code describing the project's metadata, it is necessary for
it to be executable for programs like ``pip`` to even extract basic information
about the project needed at installation time.

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

.. _distutils: http://docs.python.org/2/library/distutils.html
.. _setuptools: http://pythonhosted.org/setuptools/

Detailed description
--------------------

This section describes the need for the APE.  It should describe the existing
problem that it is trying to solve and why this APE makes the situation better.
It should include examples of how the new functionality would be used and
perhaps some use cases.


Branches and pull requests
--------------------------

Any pull requests or development branches containing work on this APE should be
linked to from here.  (An APE does not need to be implemented in a single pull
request if it makes sense to implement it in discrete phases). If no code is yet
implemented, just put "N/A"


Implementation
--------------

This section lists the major steps required to implement the APE.  Where
possible, it should be noted where one step is dependent on another, and which
steps may be optionally omitted.  Where it makes sense, each  step should
include a link related pull requests as the implementation progresses.


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
