An Updated Model for the Affiliated Package Ecosystem
-----------------------------------------------------

author: Erik Tollerud, 2017 Astropy Coordination Meeting Attendees

date-created: 2017 November 7

date-last-revised: 2017 November 7

type: Process

status: Discussion


Abstract
--------

The abstract should be a short description of what the APE will achieve.


Detailed description
--------------------

Motivation
==========

From its inception, Astropy has included an ecosystem of astronomy packages
beyond the ``astropy`` core package(see the `original vision <xxx>`_).  The
process by which these "affiliated" packages are reviewed and accepted has
already changed a few times since the project's inception as the ecosystem has
grown. However, the current state has several significant deficiencies.

First, the affiliated packages include a mix of two different kinds of packages.
Some packages have been developed as a fairly integrated effort beside the core
package, meaning both feature planning assuming the existence of the affiliated
packages, as well assignficant overlap in the core development teams (e.g.
``photutils``, ``astroquery``, and ``specutils``). Other affiliated packages
(the majority) are developed more independently of the core package, often with
nearly disjoint development teams, while staying in contact with and making
*use* of the wider astropy ecosystem and community. It has proved difficult to
design a development and review process that works effectively with *both* of
these classes of packages.

Second, the current review process has become somewhat opaque and rather awkward
to keep organized.  The coordination committee reviews the packages, which in
practice nearly always receive a "provisional" designation, but then must be
re-reviewed as a *whole* at some future date to become "full" affiliated
packages. This "provisonal" and "full" distinction is opaque and probably
meaningless to the average user who just wants to know what a package does, and
if it is any good. Additionally, while the review standards are stated on the
affiliated package web page, some are subjective, allowing at least the
possibility of biased or exclusionary behavior to sneak in with little recourse
given the blunt nature of "reject", "provisional", and "accept".

Third, the coordination committee has found it difficult to keep up with the
rate of affiliated package review (and re-review) requests. Many reviews
languish for months before receiving even a "provisional" response, which can be
disheartening and perceived by some as unwelcoming. This is primarily driven by
the need to do a detailed and sometimes elngthy review of each package, even if
that amounts to just following a checklist (see the section on the new review
process for more details of the current process in practice).

These issues were discussed repeatedly by the coordination committee, and later
in the 2017 Astropy coordination meeting.  The new structure and processes
described below were a product of a breakout session at the coordination
meeting. This APE is thus an effort to formalize the ideas and describe how to
implement them.

Astropy "coordinated" and affiliated packages
=============================================

xxx

A criteria-based package review process
=======================================

To counter
this, the coordination committee has generally adopted an infromal
"criteria"-based approach, mainly checking if the review bullet
points are met, and considering "provisional" if some of these are met and
"full" if a larger set of them are. This de facto checklist strongly suggests a
more transparent and understandable policy would be to formally adopt this
checklist and publicize these more

xxx

A community review process for affiliated packages
==================================================

xxx


Branches and pull requests
--------------------------

N/A


Implementation
--------------

The changes outlined in this APE are best implemented as three distinct efforts.

* The new package review criteria have already been implemented by the
  coordination committee and reviewed by the community.  A new version of the
  affiliated package web site will need to be implemented to reflect these
  changes - a concept for this new site is `available here <xxx>`_.
* The Astropy web site (and to a lesser extent, affiliated package-related docs)
  will need to be updated to reflect the existence of the coordinated
  packages.  Additionally, the roles web site will need to be populated with the
  new roles for the coordinated packages (and the role descriptions).  This
  requires relatively little effort and can be completed by the Astropy web site
  maintainers upon acceptance of this APE.
* A "peer-review" process will need to be set up to enable reviews by community
  members beyond the coordination committee.  While such processes exist in
  other contexts (e.g. the `Journal of Open Source Software <xxx>`_ and
  `Ropensci <xxx>`_) that may have tools that can be re-used, it is likely that
  setting up this process will take some time.


Backward compatibility
----------------------

N/A


Alternatives
------------

The main alternative is status quo.  This does not address the concerns raised
in the "motivation" section, but requires no additional effort. Other
possibilities include any subset of the three changes outlined above.



Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
