Astropy Release Cycle and Version Numbering
-------------------------------------------

author: Erik Tollerud

date-created: 2013 November 18

date-last-revised: 2013 December 11

type: Process

status: Accepted


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
not yet complete.  At the same time, some larger institutions and facilities
have expressed concern that there is not a clearly-stated support cycle for
Astropy versions.  That is, it is not clear if a given version will be supported
via critical bug fixes if it is not the latest version.  This was voiced as a
concern because facilities will not always want to upgrade to the latest
version, but do want to be sure major bugs will get addressed.

Release scheduling and LTS
^^^^^^^^^^^^^^^^^^^^^^^^^^

These issues were discussed in the 2013 Coordination Meeting, and the following
proposal represents a consensus from that meeting on the solution, with some
modifications based on community feedback after the meeting.  Starting with
version 0.4 , Astropy will have a feature release at least as often as every six
months.  Releases between feature releases will only contain bugfixes, not new
features. (See the "Releases and backwards compatibility" section below for more
on what is considered a bug fix).

Additionally, some feature releases will be designated "long-term support" (LTS)
releases, and these will continue to receive bugfixes for at least two years
(i.e., typically 3 regular feature releases will follow an LTS release).  This
addresses the first issue by applying a time-based release cycle, meaning no
feature will need to wait more than six months before reaching users.
Simultaneously, it addresses the second concern by providing a guarantee to
facilities that if they use an LTS, they can count on releases fixing critical
bugs for at least two years.  Finally, the LTS releases also act as a convenient
time to submit new Astropy journal papers.

Version numbering
^^^^^^^^^^^^^^^^^

To match this release plan, Astropy will adopt a version numbering scheme of the
form ``x.y.z``, where ``z`` is advanced on a bugfix release, ``y`` is advanced
for a feature release that is *not* an LTS release, and ``x`` is advanced on an
LTS (which also resets ``y`` to 0).  So starting from the first LTS release, the
version number schedule might be::

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

Feature freeze/testing calendar
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Because bug checking/fixing and preparation for release take time, a feature
freeze will need to be enacted *before* the time of a given feature release.
Such a feature freeze will mean no more major new feature pull requests will be
accepted for that version, but minor improvements, bug fixes, or documentation
additions are still acceptable.  The standard calendar for a release will be:

* -6 months: previous release
* -1 month: feature freeze, beta release, branch created
* -1 week: release candidate

This is not meant to be an exactly strict calendar, as the day of the week,
existence of holidays, and degree of readiness of key features or critical bug
fixes necessitate slop in the exact timing.  So more what you'd call guidelines
than actual rules.

Rapid Feature Releases
^^^^^^^^^^^^^^^^^^^^^^

This plan is not meant to exclude the possibility of releases *more* frequent
than every six months.  Six months should be considered both the maximum and the
default timeline, but a *shorter* time between releases is allowable, if the
community agrees it is worthwhile. When/if this happens, the subsequent LTS
release should be timed such that is at least two years after the last LTS, even
though that may mean there are more than 3 releases between the LTSs.

For example, if a new feature is perceived by the Astropy developer community as
being of great importance, and is completed only three months after the v1.0
release, a v1.1 release could occur at that time.  Once that release has
completed, the release clock is "reset", so v1.2 would be released six months
after v1.1, even though it is not yet 12 months after v1.0. The next two
releases would then be v1.3 and v1.4, because v1.4 would only be 21 months after
v1.0. v2.0 would then either come three months or six months later, depending on
how much work is planned to be new in v2.0.

Releases and backwards compatibility
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As defined here, a "bugfix release" must not break backwards compatibility as
defined by the public API, and it also should not introduce new major features.
The exception to this is cases where the API has an actual mistake that needs to
be fixed, such as a function argument that is missing but was clearly intended
to be there. Further, a bugfix release *can* include documentation improvements,
and in the future there may be some sort of "sandbox" package to act as a
preview or testbed for new additions to come in the next feature release.

Feature releases do *not* guarantee backwards compatibility with previous
releases.  However, all backwards incompatible changes should always be
mentioned in the release notes for a new version.  Furthermore, when possible, a
backwards incompatible change should generate an `AstropyDeprecationWarning` for
at least one feature release version before actually making the change
permanent, to allow codes to transition to the new interface. A future APE may
add to or change this with a more concrete policy on when and for how long
something should be deprecated.


Branches and pull requests
--------------------------

N/A


Implementation
--------------

If this APE is accepted, a few organizational steps will be required:

* The currently under-development version "v0.4" will be the first following
  the release cycle.  Because v0.3 was released On the last Monday in Nov 2013,
  the feature freeze and beta for v0.4 will be scheduled for the last Monday in
  April 2014, With a release at the end of May 2014.  The milestone on GitHub
  will thus need to be updated to match this.
* The first LTS release (v1.0) will then come six months after v0.4, so an
  appropriate milestone will need to be added to GitHub.  Following that,
  milestones should be added as needed to reflect the versioning scheme
  described in this APE.
* The documentation should be updated to mention the existence of LTS releases
  and make it clear that they come with a promise of two years of bug fixes (as
  needed). It should also add a link to this APE so that those interested in the
  numbering scheme can look here for details.


Backward compatibility
----------------------

N/A


Alternatives
------------

An alternative version numbering scheme mentioned on astropy-dev was to continue
the current scheme of ``0.x.y``, and simply append `-LTS` for LTS releases. The
disadvantage of this approach is that the first version number is then rendered
essentially meaningless.  That is, given its nature as a research library, it is
unlikely to ever be "feature-complete", and hence there is no clear break point
for a 1.0 version.  The scheme proposed here simply assigns a different meaning
to the major version as "number of LTSs since the start of the project".

Another alternative is to use `semantic versioning <http://semver.org/>`_. The
disadvantage of that approach for Astropy is that it is very likely that *all*
versions in the foreseeable future will break backwards compatibility to some
extent.  In semantic versioning, this would mean all versions would need to be
of the form ``x.0.y``, rendering the minor version number meaningless.


Decision rationale
------------------

There was a fair amount of discussion on  `the GitHub PR
<https://github.com/astropy/astropy-APEs/pull/2>`_, but mainly specific details,
rather than objections to the general scheme.  The largest change from the
original proposal was the addition of the idea that *less than* six month
releases were acceptable. The above content reflects those suggestions, and the
APE was accepted 12/10/13, as there were no significant objections from the
community.
