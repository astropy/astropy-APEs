******************************************
Astropy Affiliated Packages with pyOpenSci
******************************************

author: Pey Lian Lim, Leah Wasser, William Jamieson, Derek Homeier

date-created: 2023 July 13

date-last-revised: 2023 July 15

date-accepted: TBD

type: Process

status: Discussion


Abstract
========

Since the early days of the Astropy Project, we had recognized the need
and developed an internal process for
`Astropy Affiliated Packages <https://www.astropy.org/affiliated/#affiliated-packages>`_
(hereafter, for brevity, Affiliated).
As our ecosystem thrives and grows, so does the wider scientific Python ecosystem,
which resulted in similar processes like
`JOSS <https://joss.theoj.org/>`_ and
`pyOpenSci software peer review <https://www.pyopensci.org/software-peer-review/about/intro.html>`_
(hereafter, pyOpenSci).
Between the two, while JOSS has existed longer, pyOpenSci is more closely aligned
with what we already do in Affiliated. This APE overhauls our existing Affiliated
process to partner with pyOpenSci in order to share resources and gain wider exposure.


Detailed description
====================

.. This section describes the need for the APE.  It should describe the existing
.. problem that it is trying to solve and why this APE makes the situation better.
.. It should include examples of how the new functionality would be used and
.. perhaps some use cases.

The Affiliated page has existed since 2013 (i.e,
`28d9d03 <https://github.com/astropy/astropy.github.com/commit/28d9d038054f34385563bfc0b3e2b1a5812d612c>`_)
though the concept could have been even earlier. Back then, there were a handful of
"featured" packages and the "affiliated" section was just "INSERT JSON TABLE".
JOSS was established in 2016 (source:
`Wikipedia <https://en.wikipedia.org/wiki/Journal_of_Open_Source_Software>`_)
but it was perceived as a different enough system that there was no effort
to consolidate with them. Meanwhile, pyOpenSci started a peer review process
in ???? and then collaborated with `Pangeo <https://pangeo.io/>`_ in ????.

Fast forward to July 2023, there are 47 accepted Affiliated packages and
7 undergoing the review process. We have two editors (Derek Homeier and
William Jamieson at the time of writing) and no formal/public pool of reviewers.
Given that reviewers are anonymous by default, we are unable to provide
a fully transparent review process (even though results are publicly posted
and discussed, they are posted second-hand by the editors). We are also
unable to let people publicly sign up to be reviewers, as that would break
anonymity. Furthermore, we have no resources nor infrastructure to
re-review the 47 accepted packages to ensure that they still meet the
Affiliated standards. Therefore, there is a need to restructure our Affiliated
process to be sustainable. We are confident that a partnership with
pyOpenSci is the best solution currently available.

What do we gain from a partnership with pyOpenSci?
--------------------------------------------------

* Larger exposure for Affiliated packages within scientific Python ecosystem,
  not just Astropy.
* They get to be listed in JOSS "for free" from the
  `partnership between pyOpenSci and JOSS <https://www.pyopensci.org/software-peer-review/partners/joss.html>`_.
* pyOpenSci's objective way of initial triage will automatically weed out
  packages that do not even meet basic standards (e.g., not installable),
  reducing our editors' burdens.
* We can be sure the process is consistent with other projects that have
  also partnered with pyOpenSci.
* We can share infrastructure with pyOpenSci, reducing our own maintenance burden.
* We can share reviewer pool with pyOpenSci, providing our reviewers
  opportunity to connect with peers beyond Astropy.
* Even when a package ends up not being accepted into Affiliated but satisfies
  basic pyOpenSci criteria, its application effort is not entirely wasted,
  as it could still be listed under pyOpenSci only without our Affiliated status.

Who owns this process?
----------------------

pyOpenSci already has an ongoing process and it is a community-driven one.
It is compatible with Astropy but it is up to us to decide if a package
applied to pyOpenSci also qualifies as Astropy Affiliated.

Will the new process still foster a sense of community in Astropy?
------------------------------------------------------------------

Yes, we think so! We will still keep the
`Astropy Affiliated Packages <https://www.astropy.org/affiliated/#affiliated-packages>`_
page, but now instead of hosting the content ourselves, we grab an RSS feed
from pyOpenSci for the packages marked as Affiliated. To keep the history,
we will move the existing listing to a "legacy affiliation" page and keep it static.

Funding
-------

pyOpenSci has its own funding structure beyond the control of Astropy.
At the time of writing, it is fully funded for the next 2 years.
Given the number of projects currently looking into a similar partnership
with pyOpenSci (SunPy, Heliophysics, etc.) and that it has a full-time staff
dedicated to seeking out more funding, we find that the benefits outweigh
the risks.

In a nutshell
-------------

.. TODO: Move stuff from https://docs.google.com/document/d/1hIl3y6nrcepv-tKKQLVB4u0a3fXACQ1Qrg0Y6VrJykg/view

TODO: Table from Google Doc here + Current Astropy/pyOpenSci Review Process

Rejecting packages
------------------

TODO: Fill in from Google Doc.

Rejected package may still be listed under pyOpenSci (and JOSS) if they
pass the general criteria even if it fails the Astropy-specific criteria,
e.g., a very nice Python package that has little to do with Astropy ecosystem.

"Rereview" of packages
----------------------

TODO: Fill in from Google Doc.

Finding reviewers
-----------------

TODO: Fill in from Google Doc + Astropy Requirements to be reviewers + COI

Packaging guidelines
--------------------

TODO: Fill in from Google Doc.

Listing on websites
-------------------

TODO: Fill in from Google Doc. Also mention RSS cross-posting.

Future work
-----------

- Re-review. Semi-automated, not full review.
- Live package stats.


Branches and pull requests
==========================

.. Any pull requests or development branches containing work on this APE should be
.. linked to from here.  (An APE does not need to be implemented in a single pull
.. request if it makes sense to implement it in discrete phases). If no code is yet
.. implemented, just put "N/A"


* `Affiliated Package: Consolidate with pyOpenSci? <https://github.com/astropy/astropy-project/issues/334>`_
* `Feat: add text associated with astropy partnership <https://github.com/pyOpenSci/software-peer-review/pull/225>`_
* `FEAT: Start at community mockup / astropy <https://github.com/pyOpenSci/pyopensci.github.io/pull/207>`_
* `Metrics for community partnership landing pages to track health <https://github.com/pyOpenSci/software-peer-review/issues/226>`_

Implementation
==============

.. This section lists the major steps required to implement the APE.  Where
.. possible, it should be noted where one step is dependent on another, and which
.. steps may be optionally omitted.  Where it makes sense, each  step should
.. include a link related pull requests as the implementation progresses.

TODO: William will write up something that we can link to about the new guidelines.
They need to be compatible with
`Affiliated Guidelines (Dec 23, 2021) <https://github.com/astropy/astropy-project/commit/a9ea09ccd27703ea3ef2a80a811a5f70f91bc94b>`_.

Editors will give packages under review option to try out this new process.
Regardless of the outcome of this APE, if the package is accepted during this trial,
the acceptance stands; That is, if this APE falls through but the package used
the proposed process here, it still counts as Affiliated and does not have to re-apply.

Reviewers who agree to participate will sign up via
`pyOpenSci reviewer sign-up form <https://forms.gle/GHfxvmS47nQFDcBM6>`_.

TODO: Fill in from Google Doc (Transition Notes from Their Current + TODOs)

Transition plan:

- Existing listing will be legacy but will be kept online in the lifetime of the Astropy Project.
- Already accepted packages are encouraged to go through the new process to be listed under pyOpenSci, emphasizing that they will get JOSS "for free" with little effort.
- The plan is expected to take several years to go through all the packages, however it should not take so long for any individual package from start to finish.


Backward compatibility
======================

.. This section describes the ways in which the APE breaks backward compatibility.

Somewhat compatible:

- Existing Astropy editors would join pyOpenSci editorial board.
- There would be cross-listing between Astropy editors over at pyOpenSci and our roles page.

Not backward compatible:

- Reviewers are public now.
- No color badges. But have link to full pyOpenSci review, maybe.


Alternatives
============

We keep the status quo; no changes needed but we are also not tapping into
similar effort in a wider scientific Python community.


Decision rationale
==================

<To be filled in by the coordinating committee when the APE is accepted or rejected>
