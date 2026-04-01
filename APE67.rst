When to YOLO
------------

author: Pey Lian Lim

date-created: 2026 April 1

date-last-revised: 2026 April 1

date-accepted:

type: Process

status: Discussion


Abstract
--------

Due to shortage of maintainers, some pull requests stay open and unreviewed
for an extended period of time, until the stale-bot steps in and closes them.
This is undesirable for trivial changes submitted by maintainers who
already know very well what they are doing. This APE lays out the law of the land
for when a maintainer with enough write access can internally scream
YOLO and merge a pull request without review.


Detailed description
--------------------

What is considered a trivial pull request qualified for YOLO-ing:

* Simple documentation updates (removing dead links, fixing typos, etc).
* Code update from established process (e.g., updating bundled code
  using existing script with straightforward diff).
* Compatibility fix with upstream changes where:
   * the code is already well tested in stable and devdeps jobs, and
   * the diff is straightforward.
* Documentation change where the change is already discussed and
  agreed upon elsewhere (e.g., policy change in dev docs).
* Manual redo of an automated update (in the rare case where
  the automated pull request was closed without merge).

When is it okay to YOLO:

* Relevants checks passed (sometimes this includes the checks that
  are not marked as "required," like devdeps or one of the
  "Extra CI" checks).
* The maintainer has sufficient access, i.e., actually able to
  press the merge button.
* The maintainer is prepared to handle the wrath of other
  maintainers who disagree with the YOLO; i.e., this person
  will take responsibility and follow-up on any aftermath.

If you have *any* doubt at all, do not YOLO; Seek help.

The above mostly apply to the core astropy library. This project
has some single-maintainer packages, where YOLO is needed under
most circumstances anyway.

Branches and pull requests
--------------------------

N/A


Implementation
--------------

YOLO?


Backward compatibility
----------------------

Not backward compatible at all; opens the Gate of Chaos.


Alternatives
------------

Keep the status quo.


Decision rationale
------------------

<To be filled in when status changes, if applicable, as laid out in APE 1>
