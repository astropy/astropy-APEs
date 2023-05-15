Ending Long Term Support Releases
---------------------------------

author: Thomas Robitaille, Pey Lian Lim, Simon Conseil

date-created: 2023 February 16

date-last-revised: 2023 April 27

date-accepted: <replace with accepted date>

type: Process

status: Discussion


Abstract
--------

This APE proposes to no longer designate any releases of the core package as
being "long-term support" (LTS) and provide guidance on how to handle API
changes and deprecations.


Detailed description
--------------------

Motivation
^^^^^^^^^^

The original `APE 2: Astropy Release Cycle and Version Numbering
<https://doi.org/10.5281/zenodo.1043887>`_ describes the release cycle for the
core astropy package, which consists of major releases every 6 months that add
new features (e.g., v5.2), and releases in between this that are limited to
fixing bugs or updating documentation (e.g., v5.2.1). Every 2 years, major
releases are designated as being LTS (e.g., v5.0) and we have committed to
releasing bugfix releases for this version (e.g., v5.0.3) for up to 2 years.

At the start of the project, having LTS releases was critical for acceptance by
large organizations/projects for which stability is paramount, as the core
package went through significant API changes from version to version due to the
youth of the project and LTS releases were a way to shield against these
changes, or at least reduce disruption to be every 2 years.

LTS releases are however not without cost:

* While we use infrastructure to automate backporting fixes to release branches,
  in practice the LTS branch diverges more and more from the main development
  branch and automated backports are more likely to fail as time goes on,
  requiring human effort to manually do the backports, which can sometimes be
  non-trivial.
* Even once fixes are backported to the LTS branch, carrying out releases is not
  fully automated and requires manual steps, so having to do bugfix releases for
  both the latest release branch and LTS branch puts an extra burden on the core
  package release team.

In addition, it is not clear that LTS releases are needed anymore - now that the
core package is over ten years old, it has reached a much higher level of
stability. Most packages in the scientific Python ecosystem do not have LTS
releases, and there is no longer a strong argument for having this in astropy
either.

Proposed release procedure
^^^^^^^^^^^^^^^^^^^^^^^^^^

This APE proposes the following:

* No longer designate any releases as LTS.
* Refer to the x.0.0 releases (e.g., v6.0.0) as *major*, x.x.0 releases (e.g.,
  v6.1.0) as *minor*, and x.x.x releases (e.g., v6.1.2) as *bugfix* releases
  (essentially `semantic versioning <https://semver.org>`_)
* Use the x.x.0 format for the first major/minor release in a cycle, rather than
  the x.x format previously used, as this is more standard practice with
  semantic versioning.
* Keep the default 6-month release cycle for non-bugfix releases, noting that in
  line with `APE 2`_, minor versions can also be released more frequently if
  needed.
* Leave at least one year between major releases. If one year after the last
  major release there are no breaking changes to be made (such as pending
  deprecations to remove) then there is no need to release a major version.
  The one year timescale for major releases is a minimum, not a fixed
  interval.
* Minimize as much as possible any changes that break API or change results in
  minor releases, but be pragmatic and occasionally allow such changes if they
  would not affect many users.
* Add deprecation warnings or future warnings in minor and major releases, and
  make sure that these are present in released versions for a year before
  carrying out the breaking changes in a major release. Therefore, if a deprecation
  warning is added in a major release, the breaking changes can be made in the
  following major release. If a deprecation warning is added a few months before
  a major release, the breaking changes can only be made in the subsequent major
  release.

With this in place, we would effectively be striving to maximize stability over
one-year periods but users would see deprecation warnings to prepare them for
changes. This is in contrast to the current LTS system where users might not see
a deprecation warning until the next LTS release as deprecations are not typically
backported.

As astropy becomes increasingly mature and stable, and since as described above
the 1-year timescale between major releases is a minimum, the interval between
major releases may become arbitrarily longer, and the package may reach a stage
when the major version is no longer or very rarely increased.

Guidelines on deprecations and API changes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To avoid duplication, guidelines on deprecations and API changes as well as
recommendations for communication with users will be added directly to `APE 2`_.

Branches and pull requests
--------------------------

Updates to the core package documentation: https://github.com/astropy/astropy/pull/14713

Implementation
--------------

This APE proposes to continue supporting the v5.0.x LTS series but to not
designate v6.0.0 as LTS.

Changes to `APE 2`_ can be reviewed in https://github.com/astropy/astropy-APEs/pull/82

After acceptance of this APE, the documentation for the core package should be
updated appropriately: https://github.com/astropy/astropy/pull/14713

Backward compatibility
----------------------

N/A

Alternatives
------------

We could keep the current release workflow - however as mentioned above this has
a maintenance cost and it is not something that is done by most other major
packages in the scientific Python ecosystem.

Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>

.. _APE 2: https://doi.org/10.5281/zenodo.1043887
