Astropy Release Cycle and Version Numbering
-------------------------------------------

author: Erik Tollerud

date: Nov 18, 2013

type: Process

status: Discussion

Abstract
--------

This APE describes the general plan for timing of releases and their version
numbers.  The basic premise is a 6-month cycle for new feature releases, with
releases designated "long-term support" (LTS) releases every two years.  Bugfix
releases occur as needed, on both the current feature release and the LTS
releases.

Detailed description
--------------------

The release plan through version 0.3 has been somewhat haphazard, with releases
when a certain set of features are judged "ready", but without a clear sense of
how long those features will take.  This has lead to concerns that new features
are spending a long time in the development version because *other* features are
not yet complete.  At the same time, some larger institutions and facilities have
expressed concern that there is not a clearly-stated support cycle for Astropy
versions.  That is, it is not clear if a given version will be supported via
critical bugfixes if it is not the latest version.  This was voiced as a conern
because facilities will not always want to upgrade to the latest version, but
do want to be sure major bugs will get addressed.

Release scheduling and LTS
^^^^^^^^^^^^^^^^^^^^^^^^^^
These issues were discussed in the 2013 Coordination Meeting, and the following
proposal represents a consensus from that meeting on the solution.  Starting
with the version after v0.3, Astropy will have a feature release every six
months.  Releases between feature releases will only contain bugfixes, not new
features. (See the "Releases and backwards compatibility" section below for more
on what is considered bugfixes).

Additionally, some feature releases will be designated
"long-term support" (LTS) releases, and these will continue to receive
bugfixes for two years (i.e., typically 3 regular feature releases will
follow an LTS release).  This addresses the first issue by applying a
time-based release cycle, meaning no feature will need to wait more than six
months before reaching users.  Simultaneously, it addresses the second
concern by providing a guarantee to facilities that if they use an LTS, they
can count on releases fixing critical bugs for at least two years.

Version numbering
^^^^^^^^^^^^^^^^^
To match this release plan, Astropy will adopt a version numbering scheme
of the form ``x.y.z``, where ``z`` is advanced on a bugfix release, ``y`` is
advanced for a feature release that is *not* an LTS release, and ``x`` is
advanced on an LTS (which also resets ``y`` to 0).  So starting from the
first LTS release, the version number schedule might be::

* 1.0.0 (LTS release)
* 1.0.1
* 1.0.2
* 1.1.0 (six months after 1.0.0)
* 1.1.1
* 1.0.3
* 1.1.2
* 1.2.0 (six months after 1.1.0)
* 1.2.1
* 1.3.0 (six months after 1.2.0)
* 1.0.4
* 1.3.1
* 2.0.0 (LTS release, six months after 1.3.0)

Feature freeze/ testing calendar
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Because bug checking/fixing and preparation for release take time, a
feature freeze will need to be enacted *before* the time of a given feature
release.  Such a feature freeze will mean no more major new feature pull
requests will be accepted for that version, but minor improvements, bugfixes,
or documentation additions are still acceptable.  The standard calendar for
a release will be:

* -6 months: previous release
* -1 month: feature freeze, beta release, branch created
* -1 week: release candidate

This is not meant to be an exactly strict calendar, as the day of the
week, existence of holidays, and degree of readiness of key features or
critical bug fixes necessitate slop in the exact timing.  So more what you'd
call guidelines than actual rules.


Releases and backwards compatibility
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As defined here, a "bugfix" release must not break backwards compatibility as
defined by the public API, and it also should not introduce new major features.
The exception to this is cases where the API has an actual mistake that needs to
be fixed, such as a function argument that is missing but was clearly intended
to be there. Further, a bugfix release *can* include documentation improvements,
and in the future there may be some sort of "sandbox" package to act as a
preview or testbed for new additions to come in the next feature release.

Feature releases do *not* guarantee backwards compatibility with previous
releases.  However, all backwards incompatible changes should always be
mentioned in the release notes for a new version.  Furthermore, when
possible, a backwards incompatible change should generate an
`AstropyDeprecationWarning` for at least one feature release version before
actually making the change permanent, to allow an opportunity for
codes to transition to the new interface.



Branches and pull requests
--------------------------

N/A

Implementation
--------------

If this APE is ratified, a few trivial steps will be required:

* The currently under-develpment version that is currently called "v0.4" will
  be re-named "v1.0".  This will require updating the milestone name on github
  and changing the version number in the ``CHANGES.rst`` file.
* A decision will need to be made for when to "start the clock" for v1.0, as
  some features in the works may not have been planned fully assuming the six
  month release cycle.
* The documentation should be updated to mention the existence of LTS releases
  and make it clear that they come with a promise of two years of bugfixes (as
  needed).


Backward compatibility
----------------------

N/A

Alternatives
------------

An alternative version numbering scheme mentioned on astropy-dev was to continue
the current scheme of ``0.x.y``, and simply append `-LTS` for LTS releases.
The disadvantage of this approach is that the first version number is then
rendered essentially meaningless.  That is, given its nature as a research
library, it is unlikely to ever be "feature-complete", and hence there is no
clear break point for a 1.0 version.  The scheme proposed here simply assigns
a different meaning to the major version as "number of LTSs since the start of
the project".

Another alternative is to use `semantic versioning <http://semver.org/>`_.
The disadvantage of that approach for Astropy is that it is very likely that
*all* versions in the forseeable future will break backwards compatibility to
some extent.  In semantic versioning, this would mean all versions would need
to be of the form ``x.0.y``, rendering the minor version number meaningless.

Decision rationale
------------------

<To be filled in when the APE is accepted or rejected>
