An Updated Model for the Affiliated Package Ecosystem
-----------------------------------------------------

author: Erik Tollerud

date-created: 2018 February 22

date-last-revised: 2018 May 14

type: Process

status: Accepted


Abstract
--------

A revision of the concept of Astropy-affiliated packages (hereafter "affiliated
packages") is proposed. The idea of
affiliated packages has continued mostly unaltered since the
`original vision <http://docs.astropy.org/en/stable/development/vision.html>`_
for Astropy. However, time has revealed some potential improvements in how the
ecosystem is structured and the review process for affiliated packages. Hence,
this APE proposes to make three changes:

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
Some packages have been developed as a fairly integrated effort alongside the
core package. That is, feature planning took into account that these
affiliated packages existed, and that important features would be implemented in
them (e.g.,``photutils``, ``astroquery``, and ``specutils``). Thus, the
continued existence of those packages is understood to be important for the
Astropy project as a whole. Other affiliated
packages (the majority) are developed more independently of the core package,
which treat Astropy mainly as a base to build on, do not as strongly influence
planning for the core package, and provide functionality that is not general
enough to be considered part of the core requirements. It has proved
difficult to design a development and review process that works effectively with
*both* of these classes of packages.

Second, the current review process has become somewhat opaque and rather awkward
to keep organized. The coordination committee reviews the packages, which in
practice nearly always receive a "provisional" designation, but then must be
re-reviewed as a *whole* at some future date to become full affiliated
packages. This provisional and full distinction is opaque and probably
meaningless to the average user who just wants to know what a package does, and
if it is any good. Additionally, while the review standards are stated on the
affiliated package web page, some are subjective, allowing at least the
possibility of biased or exclusionary behavior to sneak in with little recourse
given the blunt nature of reject, provisional, and accept.

Third, the coordination committee has found it difficult to keep up with the
rate of affiliated package review (and re-review) requests. Many reviews
languish for months before receiving even a provisional response, which can be
disheartening and perceived by some as unwelcoming. This is primarily driven by
the need to do a detailed and sometimes lengthy review of each package, even if
that amounts to just following a checklist (see the section on the new review
process for more details of the current process in practice).

These issues were discussed at length by the coordination committee, and later
at the 2017 Astropy coordination meeting. The new structure and processes
described below were a product of a breakout session at the coordination
meeting. This APE is thus an effort to formalize the ideas and describe how to
implement them.

Astropy "coordinated" and affiliated packages
=============================================

The biggest-picture change is to split the currently single-category affiliated
package ecosystem into two categories: "Astropy coordinated" and "Astropy
affiliated" packages. This change is not as drastic as it sounds because it
reflects the reality of the current situation. As outlined above, two such
categories effectively exist already, and this APE simply formalizes this
status.

In practice, packages that remain as affiliated packages are essentially
unaffected by this change. They continue to be managed/organized outside of the
Astropy project core team and retain complete control of the repositories, code
base, etc. By contrast, packages in the new "coordinated package" category will
be treated as a core component of the Astropy project, with the project (via the
coordinating committee) overseeing who is maintaining the package and taking
responsibility for continued maintenance in the event the original author(s) no
longer can do so. This means maintainer roles will be created for all the
coordinated packages, and have status as
`astropy roles <http://www.astropy.org/team.html>`_.

This also means the development process for coordinated packages will generally
track close to the core package model - e.g. they will usually follow the same
package layout as the core package, APEs can be used for major
decisions/standards, etc. Note that there is *not* a requirement that
coordinated packages follow the release calendar for the core package (although
they *may* do so if desired), as a more flexible release calendar is one of the
major advantages to having a coordinated package not be in the core.

One additional consequence of this change is that coordinated packages will live
in the `astropy GitHub organization <https://github.com/astropy>`_ , for which
the project has management power, while affiliated packages, in general, should
*not*. To ease this process, however, any affiliated packages currently in the
astropy organization can be "legacied-in". This means that they may remain in
the GitHub organization even if they are not coordinated packages. Moving
forward, though, new coordinated packages should be in the astropy organization
and new affiliated packages should not be.

Note, though, that affiliated packages are still considered a part of the
"Astropy Project". That umbrella term applies to both coordinated and
affiliated packages, as well as the overall community.


A criteria-based package review process
=======================================

This APE also proposes a second change to the review process, motivated by the
concerns in the "Motivation" section above about the committee's review
process. To counter these concerns, prior to this APE the coordination committee
has informally adopted a checklist-based approach, checking if certain bullet
points are met. While the formal criteria are listed in relatively vague terms
on the
`affiliated package web site <http://www.astropy.org/affiliated/index.html#affiliated-instructions>`_,
the details of how those criteria are met
have been fairly consistent using an unwritten but roughly standard list. In
practice, in the past this has meant that affiliated packages are given
"provisional" status if some of these items are met (a status that will no
longer exist if this APE is accepted), or "full" if a larger set of them are.

This de facto checklist implies that a more transparent and understandable
policy would be to make this checklist formal and completely public, and use it
directly as the formal review criteria - i.e., a rubric. These criteria will be
laid out explicitly in a document (which has already been drafted - see the
"Implementation" section below), with various levels defined for each criteria,
generally corresponding to "good", "acceptable", and "unacceptable". This
will make the review process much faster and (eventually) open the review
process beyond the coordination committee, thereby addressing the concerns from
the "Motivation" section.


A community review process for affiliated packages
==================================================

On a longer term horizon, a criteria-based review checklist will allow the
review process to change to a more peer-review model. That is, instead of
every package being vetted by the coordination committee, the committee can act
more as editors, sometimes offering the review opportunity to other members of
the Astropy community. The details of this process will not be set in stone by
this APE, as they should be flexible enough to accomodate the experiences of the
first rounds of reviewers. But this APE sets some initial steps (see
"Implementation" below) and makes the general idea the eventual goal for the
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
  These criteria will need to be translated to a structured format to be shown
  on the affiliated
  package instructions section of the Astropy website. The affiliated package
  listing page will also need to be updated for these new criteria - a concept
  for this new site is `viewable here <assets/ape15_example_affilpkg_page.png>`_.
* The Astropy web site (and to a lesser extent, affiliated package-related docs)
  will need to be updated to reflect the existence of the coordinated
  packages. Additionally, the roles web site will need to be populated with the
  new roles for the coordinated packages (and the role descriptions). This
  requires relatively little effort and can be completed by the Astropy web site
  maintainers upon acceptance of this APE.
* A "peer-review" process will need to be set up to enable reviews by community
  members beyond the coordination committee. Such processes exist in
  other code contexts (e.g. the
  `Journal of Open Source Software <http://joss.theoj.org/>`_ and
  `Ropensci <https://ropensci.org/>`_) that may have tools that can be re-used
  to make the process smooth and efficient. Setting such tools up is likely to
  take some time, however, and needs to be adaptable enough to respond to the
  first few reviews. Hence this APE does not describe the details of this
  implementation - that is explicitly left to be developed by the Astropy
  community with the cooordination committee acting as "editors". However,
  this APE gives a suggested *starting* point: reviewers can be selected by
  the coordination committee from the pool of existing affiliated package
  authors or Astropy core team members. They receive a prompt to review a new
  package via email (along with the criteria described a above), and provide
  their reviews in the most straightforward manner feasible (email, a GitHub
  issue for the review, or similar). The coordination committee then performs
  the current process of posting the result of the review as issues in the
  affiliated package's repository for discussion. In the future some of these
  processes will likely be automated, but this simplest viable start point will
  provide a place to start.


Backward compatibility
----------------------

While in the future this APE proposes that new affiliated packages not be in the
Astropy GitHub organization, affiliated packages from *before* this APE that are
in the Astropy organization will be left there unless specifically requested
otherwise.


Alternatives
------------

The main alternative is status quo. This does not address the concerns raised
in the "motivation" section, but requires no additional effort. Other
possibilities include any subset of the three changes outlined above.



Decision rationale
------------------

The concept of this APE was discussed extensively by the Astropy Coordination
Committee and approved.  The APE text was discussed and accepted by multiple
community stakeholders who have experience and project-level interest in
affiliated packages.  The APE was accepted on May 14, 2018.
