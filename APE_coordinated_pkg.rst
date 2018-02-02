An Updated Model for the Affiliated Package Ecosystem
-----------------------------------------------------

author: Erik Tollerud, 2017 Astropy Coordination Meeting Attendees

date-created: 2017 November 7

date-last-revised: 2017 November 7

type: Process

status: Discussion


Abstract
--------

A revision of the concept of "affiliated packages" is proposed.  Thee idea of
affiliated packages has continued mostly unaltered since the original Vision
for Astropy.  However, time has revealed some limitations in how the ecosystem
is structured and the review process for affiliated packages.  This APE proposes
to make three changes:

* This APE creates the concept of "coordinated" packages - affiliated-like packages
  that are more closely overseen by the Astropy Project. While apparently
  significant, this change is mainly a recognition of de facto status that some
  affiliated packages already have.
* The APE changes the review process to a more explicit criteria-based review
  process - again, this is primarily simply formalizing a process that was
  already in place in the coordination committee.
* It lays out guidelines for the eventual goal of converting the
  affiliated package review process into a more "peer-review" style, where
  other members of the community can contribute to the review.


Detailed description
--------------------

Motivation
==========

From its inception, Astropy has included an ecosystem of astronomy packages
beyond the ``astropy`` core package (see the
`original vision <http://docs.astropy.org/en/stable/development/vision.html>`_).
The process by which these "affiliated" packages are reviewed and accepted has
already changed a few times since the project's inception as the ecosystem has
grown. However, the current state has several significant limitations.

First, the affiliated packages include a mix of two different kinds of packages.
Some packages have been developed as a fairly integrated effort beside the core
package, meaning both feature planning assuming the existence of the affiliated
packages, as well as significant overlap in the core development teams (e.g.
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
packages. This "provisional" and "full" distinction is opaque and probably
meaningless to the average user who just wants to know what a package does, and
if it is any good. Additionally, while the review standards are stated on the
affiliated package web page, some are subjective, allowing at least the
possibility of biased or exclusionary behavior to sneak in with little recourse
given the blunt nature of "reject", "provisional", and "accept".

Third, the coordination committee has found it difficult to keep up with the
rate of affiliated package review (and re-review) requests. Many reviews
languish for months before receiving even a "provisional" response, which can be
disheartening and perceived by some as unwelcoming. This is primarily driven by
the need to do a detailed and sometimes lengthy review of each package, even if
that amounts to just following a checklist (see the section on the new review
process for more details of the current process in practice).

These issues were discussed repeatedly by the coordination committee, and later
in the 2017 Astropy coordination meeting.  The new structure and processes
described below were a product of a breakout session at the coordination
meeting. This APE is thus an effort to formalize the ideas and describe how to
implement them.

Astropy "coordinated" and affiliated packages
=============================================

The biggest-picture change is to split the currently single-category affiliated
package ecosystem into two categories: "Astropy coordinated" and "Astropy
affiliated" package.  This change is not as drastic as it sounds because it
reflects the *reality* of the current situation.  As outlined above, two such
categories effectively exist already, and this APE simply formalizes this
status.

In practice, packages that remain as affiliated packages are essentially
unaffected by this change.  They continue to be managed/organized outside of the
Astropy project core team, and retain complete control of the repositories, code
base, etc.  By contrast, packages in the new "coordinated package" category will
be treated as a core component of the Astropy project, with the project (via the
coordinating committee) overseeing who is maintaining the package and taking
responsibility for continued maintenance in the event the original author(s) no
longer can do so. This means maintainer roles will be created for all the
coordinated packages, and have full status as astropy
`roles <http://www.astropy.org/team.html>`_.

XXX Is more about exactly what "coordinated" means needed?

One additional consequence of this change is that coordinated packages will live
in the `astropy github organization <https://github.com/astropy>`_ , for which
the project has management power, while affiliated packages, in general, should
*not*.  To ease this process, however, any affiliated packages currently in the
astropy organization can be "legacied-in".  This means that they may remain in
the github organization even if they are not coordinated packages. Moving
forward, though, new coordinated packages should be in the astropy organization
and new  affiliated packages should not be.

Note, though, that affiliated packages are still considered a part of the
"Astropy Project".  That umbrella term applies to both coordinated and
affiliated packages, as well as the overall community.


A criteria-based package review process
=======================================

This APE also proposes a second change to the review process, motivated by the
concerns in the "Motivation" section above about the committee's review
process. To counter these concerns, prior to this APE the coordination committee
has informally adopted a checklist-based approach, checking if certain "bullet
points" are met.  While the formal criteria are listed in relatively vague terms
on the affiliated package web site, the details of how those criteria are met
have been fairly consistent using an unwritten but fairly standard list. In
practice, this has meant that affiliated packages are fiven "provisional" status
if some of these items are checked off are met, or "full" if a larger set of
them are.

This de facto checklist implies that a more transparent and understandable
policy would be to make this checklist formal and completely public, and use it
directly as the formal review criteria - i.e., a rubric.  These criteria will be
laid out explicitly in a document, with various levels defined for each criteria
(generally corresponding to "good", "acceptable", and "unacceptable"). This
will make the review process much faster and (eventually) open the review
process beyond the coordination committee, thereby address the concerns from the
"Motivation" section.


A community review process for affiliated packages
==================================================

On a longer term horizon, a criteria-based review rubric will allow the review
process to change to a more "peer-review" model.  That is, instead of every
package being vetted by the coordination committee, the committee can act more
as "editors", sometimes offering the review opportunity to other members of the
Astropy community.  While the details of this process will not be outlined here,
as they by necessity should be flexible enough to accomodate the experiences of
the first few reviewers, this APE sets this as the eventual *goal* for the
affiliated package review process.


Branches and pull requests
--------------------------

N/A


Implementation
--------------

The changes outlined in this APE will be implemented as three distinct efforts:

* The new package review criteria have already been implemented by the
  coordination committee and reviewed by the community (Available in Google doc
  form `here <https://docs.google.com/document/d/15PJf2PROXMa7xwTDvWnjXB_9KNuO2Ia4_kkxJ7MPazE/edit?usp=sharing>`_.
  These criteria will need to be translated to HTML and put on the affiliated
  package instructions section of the Astropy website.  The affiliated package
  listing page will also need to be updated for these new criteria - a concept
  for this new site is `viewable here <example_affilpkg_page.png>`_.
* The Astropy web site (and to a lesser extent, affiliated package-related docs)
  will need to be updated to reflect the existence of the coordinated
  packages.  Additionally, the roles web site will need to be populated with the
  new roles for the coordinated packages (and the role descriptions).  This
  requires relatively little effort and can be completed by the Astropy web site
  maintainers upon acceptance of this APE.
* A "peer-review" process will need to be set up to enable reviews by community
  members beyond the coordination committee.  While such processes exist in
  other contexts (e.g. the
  `Journal of Open Source Software <http://joss.theoj.org/>`_ and
  `Ropensci <https://ropensci.org/>`_) that may have tools that can be re-used,
  it is likely that setting up this process will take some time - this APE does
  not outline the details of this implementation, but rather gives the broad
  goals.


Backward compatibility
----------------------

While in the future this APE proposes that new affiliated packages not be in the
Astropy github organization, affiliated packages from *before* this APE that are
in the Astropy organization will be left there unless specifically requested
otherwise.


Alternatives
------------

The main alternative is status quo.  This does not address the concerns raised
in the "motivation" section, but requires no additional effort. Other
possibilities include any subset of the three changes outlined above.



Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
