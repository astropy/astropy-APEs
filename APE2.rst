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
releases designed "long-term support" (LTS) releases every two years.  Bugfix
releases occur as needed, on both the current feature release and the LTS
releases.

Detailed description
--------------------



Branches and Pull requests
--------------------------

N/A

Implementation
--------------

If this APE is ratified, a few trivial steps will be required:

* The currently under-develpment version that is currently valled "v0.4" will
  be re-named "v1.0".  This will require updating the milestone name on github
  and changing the version number in the ``CHANGES.rst`` file.
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

Decision Rationale
------------------

<To be filled in when the APE is accepted or rejected>
