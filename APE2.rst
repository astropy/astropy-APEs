Astropy Release Cycle and Version Numbering
-------------------------------------------

author: Erik Tollerud

date-created: 2013 November 18

date-last-revised: 2023 April 27

date-accepted: 2013 December 10

type: Process

status: Accepted

revised-by: Thomas Robitaille, Pey Lian Lim, Simon Conseil - 2023 April 27 - Rewritten following the approval of APE 21 to no longer have Long Term Support (LTS) versions


Abstract
--------

This APE describes the general plan for timing of releases of the core package
and their version numbers. The basic premise follows the terminology of semantic
versioning: *major* releases (e.g., 6.0.0) every year (or longer), *minor*
releases (e.g., 6.1.0) typically every six months in between, and *bugfix*
releases (e.g., 6.1.2) as needed between minor releases.

Detailed description
--------------------

The release plan prior to v0.3 was somewhat haphazard, with releases carried out when a
certain set of features were judged "ready", but without a clear sense of how
long those features would take.  This led to concerns that new features were
spending a long time in the development version because *other* features were
not yet complete.  At the same time, some larger institutions and facilities had
back then expressed concern that there was not a clearly-stated support cycle for
Astropy versions. That is, it was not clear if a given version would be supported
via critical bug fixes if it was not the latest version.  This was voiced as a
concern because facilities will not always want to upgrade to the latest
version, but do want to be sure major bugs will get addressed. This led to the
initial version of this APE following discussions at the 2013 Coordination
Meeting. This APE is a living document and is meant to reflect the latest
release guidelines (prior versions can be found in `Previous versions of this
APE`_).

Release versioning and scheduling
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The release cycle for the core package should be as follows:

* Major releases every year (or longer). These are releases that can introduce breaking
  API changes, and should be numbered as x.0.0 (e.g., 6.0.0).
* Minor releases by default every six months in between major releases (or more
  frequent if needed). These releases should minimize any API changes and focus
  on adding new features.
* Bugfix releases as needed between minor releases.

As an example, starting from the 6.0.0 release, the version number schedule might be:

* 6.0.0
* 6.0.1
* 6.0.2
* 6.1.0 (six months after 6.0.0)
* 6.1.1
* 6.1.2
* 7.0.0 (six months after 6.1.0, one year after 6.0.0)

In the above scenario, we would adopt the default 6-month release schedule, and
introduce a new major version with breaking changes one year later. However, the
release schedule could also look like the following:

* 6.0.0
* 6.0.1
* 6.0.2
* 6.1.0 (three months after 6.0.0)
* 6.1.1
* 6.1.2
* 6.2.0 (six months after 6.1.0)
* 6.2.1
* 6.3.0 (four months after 6.2.0)
* 6.3.1
* 6.3.2
* 6.3.3
* 7.0.0 (five months after 6.3.0, 18 months after 6.0.0)

In the above hypothetical scenario, the developers have decided to release minor
releases more frequently than every six months, and a year after 6.0.0 there was
no need for a major version, so this was delayed until 18 months after 6.0.0.

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
fixes necessitate slop in the exact timing.

Rapid Feature Releases
^^^^^^^^^^^^^^^^^^^^^^

This plan is not meant to exclude the possibility of minor releases *more* frequent
than every six months.  Six months should be considered both the maximum and the
default timeline, but a *shorter* time between releases is allowable, if the
community agrees it is worthwhile. When/if this happens, the subsequent major
release should be timed such that is at least one year after the last major release, even
though that may mean there are more than one minor release between the major releases.

For example, if a new feature is perceived by the Astropy developer community as
being of great importance, and is completed only three months after the v6.0.0
release, a v6.1.0 release could occur at that time.  Once that release has
completed, the release clock is "reset", so v6.2.0 would be released six months
after v6.1.0, even though it is not yet 12 months after v6.0.0. The next release
could then be v7.0.0 three or more months after v6.2.0, or could also be another
minor release.

Releases and backwards compatibility
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The guidelines in relation to API changes and backward compatibility are as follows:

* API changes (i.e., modifications as opposed to additions) should only be
  carried out in major releases.
* Any API changes should be preceded by deprecation warnings that should be
  visible to users for at least a year prior to the API change
  being carried out.
* API changes should be documented, with clear guidance on what is changing,
  why the change is being made, and how to migrate existing code to the new behavior.
* New deprecations should not be introduced in bugfix releases.
* If developers wish to make an API change at a point in time where the next
  release is a major release, they should introduce the deprecation in the major
  release and carry out the change in the following major release.
* The following will not be considered to be part of the API and can therefore
  be changed in bugfix and minor releases:

  * Changes to warning messages
  * Changes to exception messages
  * Changes to the ``__repr__`` or ``__repr_*__`` of objects

  However, exception and warning *types* will be considered to be part of the
  API.

As with all guidelines, there are exceptions where we can deviate from these guidelines:

* API changes can be carried out in major releases without deprecation if it is
  not possible to have a deprecation phase due to the nature of the change.
* Changes breaking or changing the behavior of code may be needed in order to
  fix bugs. Such changes can be made in bugfix or minor releases and do not need
  to be considered API changes.
* Changes to exception or warning types may occasionally be done in minor
  releases.

These exceptions will require judgment calls on the part of maintainers, but any
such changes should be minimized as much as possible and clearly communicated to
users (see `Communication with users`_).

Documentation and performance improvements can be made in bugfix releases as long
as they do not involve any changes or additions to the API.

Communication with users
^^^^^^^^^^^^^^^^^^^^^^^^

It is imperative that we communicate effectively with users so that they understand
what to expect from different releases:

* We should include documentation for the core package that describes the
  release policy above, namely that in general minor releases will not
  break/remove functionality, but that major releases might, and also explaining
  what is considered API.

* We should ensure that any changes in releases which deviate from the
  guidelines above are clearly communicated to users - for example any API/breaking
  changes in minor releases should be included in the "What's New" page for the
  minor release.

* Deprecations should always be mentioned in the changelog, as well as any
  removal of deprecated code/API changes. We do not specify how exactly this
  should be done, but note that, e.g., Numpy release notes have separate sections
  entitled *Deprecations*, *Expired Deprecations* and *Compatibility Notes* in
  their release notes (see `here
  <https://numpy.org/doc/stable/release/1.24.0-notes.html>`_ for an example)
  which is clearer than having a single *API changes* section.

* Maintainers should be encouraged to advertise pull requests with API changes
  that are likely to affect users and developers of other packages to the
  developer mailing list, and ensuring that the pull request has adequate time
  (at least two weeks) to be reviewed properly.

Branches and pull requests
--------------------------

N/A


Implementation
--------------

Whenever this APE is updated, the core package documentation should be updated
to reflect the latest guidelines described above: https://github.com/astropy/astropy/pull/14713

Backward compatibility
----------------------

N/A


Alternatives
------------

N/A

Decision rationale
------------------

There was a fair amount of discussion on  `the GitHub PR
<https://github.com/astropy/astropy-APEs/pull/2>`_, but mainly specific details,
rather than objections to the general scheme.  The largest change from the
original proposal was the addition of the idea that *less than* six month
releases were acceptable. The above content reflects those suggestions, and the
APE was accepted 12/10/13, as there were no significant objections from the
community.

Previous versions of this APE
-----------------------------

* 2013-12-10 [`DOI <https://doi.org/10.5281/zenodo.1043887>`_] [`GitHub <https://github.com/astropy/astropy-APEs/blob/48f949c05efa4f07ed8915eccdb0cd139c57b6f6/APE2.rst>`_]
