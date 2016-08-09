Roadmap for Python 3-only support
---------------------------------

author: Tom Robitaille

date-created: 2016 July 13

date-last-revised: 2016 July 25

type: Process

status: Draft

Abstract
--------

The purpose of this APE is to set out a roadmap for future releases of the
Astropy core package, in particular to identify the process for switching to
releases that support only Python 3 and ending support for Python 2.7 in
December 2019.

Detailed description
--------------------

Python 2.7 will cease to be supported on January 1st 2020. As a result, a
number of scientific Python projects, including IPython, Matplotlib, Sympy,
and others have now signed the `Python 3 statement
<https://python3statement.github.io>`_ which states that scientific Python
packages will drop support for Python 2 no later than 2020. For instance, the
IPython developers have now released `IPython 5.0
<http://blog.jupyter.org/2016/07/08/ipython-5-0-released/>`_, which will be the
last major release to support Python 2. This APE proposes that the Astropy
project should also sign the Python 3 statement statement, and provides a
target roadmap for future releases.

It is important to note that ending support for Python 2 by the end of 2019
does not mean that Python 2 users will no longer be able to install or use
Astropy from 2020 onwards. Existing Astropy releases will remain available
indefinitely. Instead, the proposed roadmap means that only new features
developed after mid-2017 will be available only to Python 3 users, and that
we will not fix bugs for Python 2 after 2019.

Roadmap
-------

At the time of writing, the following releases have been made since (and including) v1.0:

==========   ======================   ==========================================
Version      Release date             Notes
==========   ======================   ==========================================
v1.0 - LTS   February 19, 2015        Supported with bug fixes until June 2017
v1.1         December 11, 2015
v1.2         June 23, 2016
==========   ======================   ==========================================

The proposed roadmap for future releases is the following:

==========   ======================   ==========================================
Version      Planned release date     Notes
==========   ======================   ==========================================
v1.3         December 2016
v2.0 - LTS   June 2017                Supported with bug fixes until end of 2019
v3.0         December 2017            First release to support only Python 3+
v3.1         June 2018
v3.2         December 2018
v3.3         June 2019
v4.0 - LTS   December 2019            Supported with bug fixes for two years
==========   ======================   ==========================================

Note that v1.0, v2.0, and v4.0 are marked as long-term support releases (LTS),
which means that they are typically supported with bug fixes for at least two
years. The above roadmap proposes that the last Python 2-compatible release will
be v2.0, which we will support with bug fixes until the end of 2019. In the mean
time, v3.0, to be released in December 2017, will be the first release that does
not support Python 2.

Note that v3.0 would not be an LTS release, since we would
otherwise need to maintain two LTS releases in parallel. This is a one-off
deviation from the version numbering policy outlined in `APE 2
<https://github.com/astropy/astropy-APEs/blob/master/APE2.rst>`_, and subsequent releases will follow the pattern established by APE 2.

In order to decide which Python 3.x releases to support, we would follow the
Python support calendar: the Python 3.x versions supported by an Astropy
release would be the versions supported by the Python developers. Using the
`current Python support calendar
<https://docs.python.org/devguide/#status-of-python-branches>`_, this would
mean that the following Astropy versions would support the following Python 3.x
versions:

==========    ===============================
Version       Minimum required Python version
==========    ===============================
v2.0 - LTS    Python 2.7 or Python 3.3+
v3.0          Python 3.4+
v3.1          Python 3.4+
v3.2          Python 3.4+
v3.3          Python 3.5+
v4.0 - LTS    Python 3.5+
==========    ===============================

Implementation
--------------

In practice, dropping Python 2.7 support will involve, amongst other things:

* Removing the bundled `six <https://pythonhosted.org/six/>`_ package, and
  updating all the code that used ``astropy.extern.six`` to use the Python 3
  syntax directly.
* Removing Python 2.7 from the configuration for the continuous integration
  services (such as Travis and AppVeyor at the time of writing)
* Making sure that the documentation on ReadTheDocs is built using Python 3.x
* Removing any backports that may exist for Python 2.7 compatibility
* Updating the documentation to remove any mention of Python 2.7, as well as
  making sure that Python 3.x is not presented as a special case.
* Removing Python 2 and 2.7 from the PyPI classifiers

Challenges
----------

There are a few challenges to consider with the plan outlined above:

Developers will need to use Python 3
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once we make the switch to a Python 3-only code base, between v2.0 and v3.0,
people will no longer be able to contribute code to Astropy without using
Python 3. This means that even though users can in theory wait until the end of
2019 to switch to Python 3, developers and contributors will need to make the
switch (at least for their Astropy development environment) as soon as 2017.

Packages on PyPI
^^^^^^^^^^^^^^^^^

If we upload Astropy v3.0 to PyPI, and it does not support Python 2, then if
Python 2.7 users try and pip install Astropy, the install or import will fail
(with the current version of pip). This is because PyPI does not compare the
current Python version to the PyPI meta-data for the package (which may
indicate for example whether the package is Python 2-compatible), and will
download Astropy v3.0 regardless of the active Python version.

`PEP 345 <https://www.python.org/dev/peps/pep-0345/#requires-python>`_ defines
the ``Requires-Python`` metadata field for Python packages, which can be used
to explicitly specify which Python versions a package is compatible with.
Support for this was added to `setuptools v24.2
<https://github.com/pypa/setuptools/blob/master/CHANGES.rst#v2420>`_ via the
``python_requires`` keyword argument to ``setup``::

    setup(name=...,
          version=...,
          python_requires='>=2.7',
          ...
         )

This metadata is not yet exposed by the Python Package Index (PyPI) but it is
being worked on. Finally, support for this is `being implemented into pip
<https://github.com/pypa/pip/pull/3877>`_. These changes will likely be merged
soon, which means that versions of pip supporting this will be around in the
wild for a little more than a year before Astropy v3.0 (the first release
incompatible with Python 2) is out. Nevertheless, a fraction of Python 2 users
may still be using old versions of pip by then and run the risk of installing a
version of Astropy not compatible with Python 2.

One possible solution is described in the
`Jupyter roadmap
<https://github.com/jupyter/roadmap/blob/master/accepted/migration-to-python-3-only.md#multiple-source-distributions>`_,
which is to upload tar files that have a ``-py3.x`` suffix (one file per Python
3.x version, e.g. ``astropy-3.0-py3.5.tar.gz``). We could choose to do this if
we believe that the fraction of users with old pip installations is too high.

We will re-assess this in the weeks coming up to the v3.0 release. If we
believe that enough users have a recent enough pip installation, then we simply
need to include ``python_requires='>=3'`` in the metadata for the v3.0 release.
Otherwise, we can additionally make sure we add the ``-py3.x`` suffix to the
files we upload to PyPI. By that time, the IPython 6.0 release (incompatible
with Python 2) will be out, so we will also have the benefit of seeing how that
release went and whether it caused any issues for users.

Note that this will not be an issue for users using other package managers
(either built in to their operating system, or e.g. conda), since these are
normally always explicit about Python version requirements.

Current pull requests
^^^^^^^^^^^^^^^^^^^^^

The pull request(s) that will remove Python 2 compatibility may create
conflicts with many of the open pull requests at the time - therefore it would
make sense to try and make sure the number of open pull requests at the time is
down to a minimum. This may be easiest to do straight after the v2.0 release,
according to the calendar above.

Maintenance of v2.0 LTS
^^^^^^^^^^^^^^^^^^^^^^^

According to the proposed release calendar, the v2.0 LTS release would be
supported until the end of 2019. However, soon after the v2.0 release, the
Astropy code will no longer be Python 2-compatible. As a result, pull requests
that subsequently fix bugs in Astropy in master will in some cases not be
easily backportable. One possible solution would be to also accept bug fixes in
the v2.0.x branch and forward-port them to master.

Benefits
--------

There are several benefits to following the plan proposed above:

* Maintaining a Python 3-only code base will be significantly easier, because
  developers won't have to know both 2.x and 3.x (and the `six
  <https://pypi.python.org/pypi/six>`__ package).

* We will be able to start using Python 3-only features internally, including
  for example function annotations (e.g., for units), matrix multiplication
  (e.g., for coordinates; note that this will only be possible once we support
  only Python 3.5+)

* Since we will need to keep adding Python 3.x releases to the continuous
  integration over the coming years, we will at least be able to remove Python
  2.7, making sure that the number of builds does not grow out of control.

* Since developers/contributors will need to switch to using Python 3 for
  Astropy development, we will be training more people to do this transition,
  who will then be able to help their colleagues also make the transition.

Alternatives
------------

An alternative plan would be to continue making major releases that support
Python 2 until 2019, for example:

==========   ======================   ==========================================
Version      Planned release date     Notes
==========   ======================   ==========================================
v1.3         December 2016
v2.0 - LTS   June 2017                Supported with bug fixes until end of 2019
v2.1         December 2017            
v2.2         June 2018
v2.3         December 2018
v2.4         June 2019                Last release to support Python 2.7
v3.0 - LTS   December 2019            Supported with bug fixes for two years
==========   ======================   ==========================================

This would allow more time for the PyPI limitations mentioned above to be
resolved and more time for developers to make the transition to Python 3. On
the other hand, if we want users to all be using Python 3 by 2020, then it does
not make sense to delay the proposed release plan in this way, since the
developers and infrastructure need to be ready for Python 3-only releases
before the users are. This APE therefore does not recommend this alternative.

Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
