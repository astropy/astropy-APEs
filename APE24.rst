******************************************
Astropy Affiliated Packages with pyOpenSci
******************************************

author: Pey Lian Lim, Leah Wasser, William Jamieson, Derek Homeier, Moritz Günther

date-created: 2023 July 13

date-last-revised: 2023 November 10

date-accepted: TBD

type: Process

status: Discussion

.. NOTE: Most of this stuff is from https://docs.google.com/document/d/1hIl3y6nrcepv-tKKQLVB4u0a3fXACQ1Qrg0Y6VrJykg/view


Abstract
========

Since the early days of the Astropy Project, we had recognized the need
and developed an internal process for `Astropy Affiliated Packages`_
(hereafter, for brevity, Affiliated).
As our ecosystem thrives and grows, so does the wider scientific Python ecosystem,
which resulted in similar processes like `JOSS`_ and
`pyOpenSci software peer review <https://www.pyopensci.org/software-peer-review/about/intro.html>`_
(hereafter, pyOpenSci).
Between the two, while JOSS has existed longer, pyOpenSci is more closely aligned
with what we already do in Affiliated. This APE overhauls our existing Affiliated
process to partner with pyOpenSci in order to share resources and gain wider exposure.

.. _Astropy Affiliated Packages: https://www.astropy.org/affiliated/#affiliated-packages
.. _JOSS: https://joss.theoj.org/


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
`JOSS`_ was established in 2016 (source:
`Wikipedia <https://en.wikipedia.org/wiki/Journal_of_Open_Source_Software>`_)
but it was perceived as a different enough system that there was no effort
to consolidate with them.
Meanwhile, `pyOpenSci`_ was founded in 2018 with the peer review beginning in March 2019,
partnership with the JOSS soon after, and with `Pangeo <https://pangeo.io/>`_
in Fall 2022 using Sloan Foundation funding.

Fast forward to July 2023, there are 47 accepted Affiliated packages and
8 undergoing the review process. We have two editors (Derek Homeier and
William Jamieson as of July 2023) and recruit reviewers from the Astropy community.
Editors identify community members with sufficient experience in the Astropy ecosystem and practices
to review packages, usually asking reviewers with scientific domain experience related to a potentially
affiliated package. This process mirrors what is commonly done for scientific journals, where editors 
invite scientific peers of a study to be reviewers.
Given that reviewers are anonymous by default, we are unable to provide
a fully transparent review process (even though results are publicly posted
and discussed, they are posted second-hand by the editors).

Unlike scientific articles, which are static after publication, software continues to develop,
adding features and fixing bugs; otherwise, it will become obsolete and might be
incompatible with future versions of its dependencies (e.g., NumPy or Python).
Thus, a process of "re-review" for Affiliated packages
has been envisioned, but never executed because we have no resources nor infrastructure to
re-review the 47 accepted packages to ensure that they still meet the
Affiliated standards. Therefore, there is a need to restructure our Affiliated
process to be sustainable. We are confident that a partnership with
pyOpenSci is the best solution currently available.

.. _pyOpenSci: https://www.pyopensci.org/

What do we gain from a partnership with pyOpenSci?
--------------------------------------------------

* Larger exposure for Affiliated packages within the scientific Python ecosystem,
  not just Astropy.
* If they are in scope for JOSS, they can be fast-tracked through the JOSS
  review process and get a publication via the
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
  as it could still be listed under pyOpenSci (and JOSS) only without our Affiliated status.

Who owns this process?
----------------------

pyOpenSci already has an established software peer review process and it is community-driven.
It is compatible with Astropy but it is up to us to decide if a package
applied to pyOpenSci also qualifies as Affiliated.

Will the new process still foster a sense of community in Astropy?
------------------------------------------------------------------

Yes, we think so! We will still keep the `Astropy Affiliated Packages`_
page, but now instead of hosting the content ourselves, we grab an RSS/XML feed
from pyOpenSci for the packages marked as Affiliated. To keep the history,
we will move the existing listing to a "legacy affiliation" page and keep it static.

Funding
-------

pyOpenSci has its own funding structure beyond the control of Astropy.
As of July 2023, it is fully funded for the next 2 years.
Given the number of projects currently looking into a similar partnership
with pyOpenSci (SunPy, PlasmaPy, pyHeliophysics, etc.) and that it has a full-time staff
dedicated to seeking out more funding, we find that the benefits outweigh
the risks.

In a nutshell
-------------

+--------------------------+-----------------------------+----------------------------+
| Step                     | pyOpenSci                   | Astropy (pre-APE 24)       |
+==========================+=============================+============================+
| Editorial infrastructure | Editor-in-Chief,            | Editor, reviewer           |
| for a single review      | editor (domain specific),   |                            |
|                          | 2 reviewers (1 can be       |                            |
|                          | non-domain specific)        |                            |
+--------------------------+-----------------------------+----------------------------+
| Editorial team           | `pyOpenSci Editorial Board`_| 2 editors that rotate, see |
|                          |                             | `Astropy team roles`_      |
+--------------------------+-----------------------------+----------------------------+
| Review style             | Open review                 | Reviewer is anonymous by   |
|                          |                             | default                    |
+--------------------------+-----------------------------+----------------------------+
| Post-review checks       | We will be creating a set of| Astropy has static badges  |
|                          | checks to see if the package| that need to be manually   |
|                          | is still maintained and in  | updated.                   |
|                          | what state. We might be able|                            |
|                          | to work with Astropy (and   |                            |
|                          | other communities) on what  |                            |
|                          | this looks like.            |                            |
+--------------------------+-----------------------------+----------------------------+

.. _pyOpenSci Editorial Board: https://www.pyopensci.org/about-peer-review/#our-editorial-board
.. _Astropy team roles: https://www.astropy.org/team#roles

As of July 2023, Astropy had the following `pre-APE 24 Affiliated Package review guidelines`_
and followed this `pre-APE 24 Affiliated Package review process, with templates`_.

.. _pre-APE 24 Affiliated Package review guidelines: https://github.com/astropy/astropy-project/blob/1afe5df1728e86dac51b9b87bc30a7c2cdb289ac/affiliated/affiliated_package_review_guidelines.md
.. _pre-APE 24 Affiliated Package review process, with templates: https://github.com/astropy/astropy-project/blob/1afe5df1728e86dac51b9b87bc30a7c2cdb289ac/affiliated/affiliated_package_review_process.md

Meanwhile, pyOpenSci has its own
`submission guide for package authors <https://www.pyopensci.org/software-peer-review/how-to/author-guide.html>`_,
`accompanying GitHub issue templates <https://github.com/pyOpenSci/software-submission/issues/new/choose>`_, and
`listing of accepted packages <https://www.pyopensci.org/python-packages.html#our-accepted-python-open-source-packages>`_.
As of July 2023, they are in the process of revamping their process to handle
collaborations with different communities (e.g., Astropy). They are also:

* creating a community-specific label for package filtering
  for that community on their website package listing;
* updating the submission template so a potential Astropy affiliated package
  can select Astropy for the actual review;
* adding an Astropy label on their GitHub repository; and
* creating a new editorial bot to add community-specific tags.

Is open review a deal breaker?
------------------------------

We do not think so! In the pre-APE 24 process, the reviewer was anonymous by default
because we modeled the process after traditional astronomical journals.
Even then, the reviewer had the option to reveal themselves if they wanted to.
Given that the Astropy ecosystem is open-source anyway, it makes sense for us
to move away from the journal-style tradition and embrace a more open process that
pyOpenSci uses.

Will we no longer need Editors?
-------------------------------

We still need them! However, instead of managing the whole process themselves,
they will now be part of the `pyOpenSci Editorial Board`_ and perform their
duties within the pyOpenSci process as laid out in this APE.

Rejecting packages
------------------

A package may be "rejected" in different stages of the review, either by the
pyOpenSci Editor-in-Chief or the editor, including but not limited to causes
such as:

* the package is not in technical or domain scope;
* the package fails one of the pre-review checks done by the Editor-in-Chief;
* the package maintainer stops responding to review comments; or
* the package maintainer is unwilling to ensure the package can be used and maintained.

While in practice, rejection is rare, a rejection does *not* have to come from
the Astropy community directly and will be objective.

A package that fails the Astropy-specific criteria may still be accepted into the
pyOpenSci ecosystem (and also published by JOSS) if it is in scope;
an example of this might be a well-documented and developed Python package that
is useful to the scientific community that fails to relate well to the Astropy ecosystem.

"Re-review" of packages
-----------------------

**Note: This does not exist yet and probably will not happen before
this APE is accepted. As of July 2023, pyOpenSci had a somewhat
manual process to check in on an accepted package.**

One core goal of pyOpenSci is to support scientific Python packages that are
maintained over time. Due to the lack of resources, it does not do a *full*
re-review of packages (involving Editor-in-Chief, editor, and 2 reviewers).
However, it has plans to set up automated checks that track the "health"
and maintenance level of a package over time, with the goal of identifying
packages that have become "orphans" (i.e, unmaintained). Once a package is
identified as needing additional maintenance, it will be flagged.

"Health" might include frequency of commits, releases, CI status, etc.
Some, if not most, of this data will be collected using
`devstats <https://github.com/scientific-python/devstats>`_ and
`repo-review <https://github.com/scientific-python/repo-review>`_ from
`Scientific Python`_. This information will be displayed on public dashboards
hosted on the pyOpenSci website; we may also grab those same badges,
where applicable, for a similar dashboard on Astropy website
focused only in our Affiliated packages.

However, automation can only go so far; for instance, if automation
is deployed in GitHub Actions, then any package hosted outside of GitHub
would be excluded from these checks. In that case, manual intervention
or alternate implementation might still be necessary.

If the package is no longer maintained, one of the following can happen:

1. If it is a widely used package and the maintainer wants to see it live on,
   pyOpenSci will try to help the maintainer build a new maintainer team;
2. Otherwise, pyOpenSci will gracefully sunset it from the list of maintained/accepted
   packages.

After sunsetting, if a package becomes active again, it is up to the package
maintainer to contact Astropy or pyOpenSci in order to have it be actively
listed once more. An example scenario that might happen is when the sole
maintainer goes on a hiatus longer than the time frame set in
`maintainer responsiveness <https://www.pyopensci.org/software-peer-review/our-process/policies.html#maintainer-responsiveness>`_
and then comes back to a sunsetted package.

.. _Scientific Python: https://scientific-python.org/

Finding reviewers
-----------------

pyOpenSci currently has a list of reviewers who have signed up for this task using the
`reviewer sign-up form`_.
Because they utilize 2 reviewers for each package, they generally try to find
a reviewer with domain-specific expertise, while the other with or without.
Sometimes, the second reviewer will focus instead on general usability,
ease of installation, documentation quality, or packaging infrastructure.
For every review, they target a diversity of contributors to ensure that they
have a mix of varying gender, cultural, etc., identities.

If we decide to partner with pyOpenSci, our reviewers would sign up using
the pyOpenSci form above, specifying:

* astronomy domain expertise, and
* membership in the Astropy community.

pyOpenSci does not publicly list all the people who signed up
(i.e, a reviewer is only public during the review process)
but the `pyOpenSci Editorial Board`_ has access to the list.

pyOpenSci will respect the Astropy-specific criteria for someone to be a
reviewer for Affiliated request, namely:

* familiarity with the Astropy project,
* ability to judge whether a package integrates well with the Astropy ecosystem
  (as per pre-APE 24 guidelines), and
* having domain expertise in the area of the package (e.g., galaxy evolution).

pyOpenSci and Astropy both ask editors/reviewers disclose any potential conflict of interest (COI)
prior to agreeing to review a package. In the event where COI occurs, with this partnership,
we would follow the
`pyOpenSci COI process <https://www.pyopensci.org/software-peer-review/our-process/policies.html#conflict-of-interest-for-reviews-and-editors>`_.
pyOpenSci invites the Astropy community to review this language to ensure it meets our needs.

.. _reviewer sign-up form: https://forms.gle/GHfxvmS47nQFDcBM6

Review turnaround time
----------------------

Astropy currently does not enforce any concrete turnaround time. Reviews typically
come within weeks, but the response time for submitted packages to address the review
is very non-uniform from "within days" to "years".

If we partner with pyOpenSci, Astropy would need to adhere to their expected
timeline for each step, as laid out in `An Overview Of the Peer Review Process`_.
For example, editor is expected to find reviewers within 2-3 weeks and
a peer review should be completed within 3 weeks after that.
This is to ensure that the package maintainer have a good experience with the review
and things do not languish over a long period of time.

Generally, the editor role should not take a huge amount of time, *but* it is
important for an editor, once the review starts, to check in on the review periodically
(every few weeks and more often during wrap-up).

.. _An Overview Of the Peer Review Process: https://www.pyopensci.org/software-peer-review/our-process/review-timeline.html


Packaging guidelines
--------------------

The Astropy community has followed packaging guidelines published in the 
`OpenAstronomy packaging guide`_ for a few years,
and `Astropy package template <https://github.com/astropy/package-template>`_ before that.

pyOpenSci is also developing a `community-driven packaging guide`_
that covers modern best practices and recommendations for scientific Python packaging. 
This guide has a stringent community review process. Reviewers for this guide
represent members of PyPA, core Python, Anaconda (conda/conda-forge), and
core packages for front- and back-end tools (e.g., ``flit``, PDM, ``hatch``/``hatchling``, etc.).  

If we partner with pyOpenSci, we encourage our Affiliated packages to continue to follow the
`OpenAstronomy packaging guide`_ for packaging structure. However maintainers will find the
pyOpenSci guide to be a useful reference, given it provides community-wide guidelines and
is targeted to those newer to packaging.

Regardless, the
`Editor-in-Chief checks <https://www.pyopensci.org/software-peer-review/how-to/editor-in-chief-guide.html#editor-checklist-template>`_
are the bare minimum for a package being able to go through the pyOpenSci review process.

.. _OpenAstronomy packaging guide: https://packaging-guide.openastronomy.org/en/latest/

.. _community-driven packaging guide: https://www.pyopensci.org/python-package-guide/

Listing on websites
-------------------

The pre-APE 24 `Astropy Affiliated Packages`_ listing are listed by alphabetical order
with hard-coded badges. While these badges are color-coded to give a at-a-glance status
of packages, they become outdated over time and now could even be misleading
(i.e., they really are showing the status at the time of acceptance, not the current status).
pyOpenSci is willing to work with Astropy and the broader scientific community to develop
a more consistent standard of evaluating the "health state" of a package, using
universally accepted metrics for such measurements (see **"Re-review" of packages**);
therefore, getting rid of the need for static badges altogether.

pyOpenSci is willing to create the following specifically for Astropy
if we agree to this partnership:

* A feed of Affiliated packages that we could ingest and
  use to rebuild our own `Astropy Affiliated Packages`_ listing.
  This way, the packages would be listed both over at pyOpenSci and at Astropy
  websites.
* An Astropy-specific page on the pyOpenSci website that would be a dedicated
  link for only Affiliated packages. We could then include some other
  information about the Astropy ecosystem and branding (logo, colors, etc).
  A listed package would also have a link back to the review that was done.
* A link back to Astropy website from pyOpenSci, with the understanding that
  Astropy would do vice versa. This is similar to what pyOpenSci has done with Pangeo.


Branches and pull requests
==========================

.. Any pull requests or development branches containing work on this APE should be
.. linked to from here.  (An APE does not need to be implemented in a single pull
.. request if it makes sense to implement it in discrete phases). If no code is yet
.. implemented, just put "N/A"

Issues:

* `Affiliated Package: Consolidate with pyOpenSci? <https://github.com/astropy/astropy-project/issues/334>`_
* `Metrics for community partnership landing pages to track health <https://github.com/pyOpenSci/software-peer-review/issues/226>`_
  (as noted above in **"Re-review" of packages**, this issue is not required
  to be resolved for this APE to be accepted)

Pull requests:

* `Feat: add text associated with astropy partnership <https://github.com/pyOpenSci/software-peer-review/pull/225>`_
* `FEAT: Start at community mockup / astropy <https://github.com/pyOpenSci/pyopensci.github.io/pull/207>`_
* `Add: more specific community partners page with FAQ <https://github.com/pyOpenSci/software-peer-review/pull/253>`_
* *TODO: A future PR by William Jamieson to update our Affiliated guidelines and process,
  after consolidating what we have with that of pyOpenSci.*


Implementation
==============

.. This section lists the major steps required to implement the APE.  Where
.. possible, it should be noted where one step is dependent on another, and which
.. steps may be optionally omitted.  Where it makes sense, each  step should
.. include a link related pull requests as the implementation progresses.

If we decide to move forward with this partnership, these are the proposed steps:

1. Do a trial review period of some packages to see how it goes to make the
   Astropy community feel more comfortable with the changes
   (also see **The trial period** below).
2. Based on the trial, our current Affiliated editors would finalize
   and publish the updated guidelines and process, including new
   COI policy, packaging guidelines, and so forth.
   These new guidelines need to be compatible with
   `pre-APE 24 Affiliated Package review guidelines`_ and
   `pre-APE 24 Affiliated Package review process, with templates`_.
   At the same time, we should link to more modern
   practices over at pyOpenSci and `Scientific Python`_, in addition
   to the "old ways" in Astropy and OpenAstronomy.
3. Come up with a plan to transition already accepted packages over to
   the pyOpenSci review process. This could be a process that happens over time
   in that new packages just go through the new review process and get the value
   of this partnership (pyOpenSci and JOSS) through that transition.
   Then, we could slowly look at the older packages and evaluate their current
   health states to determine whether another review is warranted
   (also see **"Re-review" of packages**). We will encourage these older
   packages that are still actively maintained to also go through the
   new process, pointing them to the benefits mentioned in
   **What do we gain from a partnership with pyOpenSci?**
4. For the new packages, have our project website ingest a RSS/XML feed
   from pyOpenSci for cross-listing (also see **Listing on websites**).
   We would also link back to pyOpenSci on our page.
   Meanwhile, we would keep the pre-APE 24 listing on a "legacy affiliation" page
   that will be kept alive during the lifetime of the Astropy project.
5. Work with pyOpenSci to cross-list Affiliated editors, between their
   editors listing and our roles page. For example, pyOpenSci could
   mark Astropy editors on the `pyOpenSci Editorial Board`_ with Astropy logo.
6. Work with pyOpenSci to hash out a more concrete plan on how to
   get a previously sunsetted package to be listed actively again
   if the package is revived.
7. Work with pyOpenSci to come up with a process to swap Affiliated
   editors in and out of `pyOpenSci Editorial Board`_, since our
   Affiliated editor assignment is not permanent.

The goal is to have a migration that is not too disruptive to current process.
Once this APE is accepted, new package submissions will go through the pyOpenSci process.
Packages currently under review can choose to continue under the old process
(and then be treated and listed like packages accepted as Affiliated before this APE),
or be transferred to the new process and follow pyOpenSci procedures.

The trial period
----------------

**Note: Communication is very important at this stage!**

This trial is only done for new Affiliated requests that are early enough
in the process as not to have work duplicated for package maintainers,
reviewers, and editors.

Editors will give qualifying packages the option to try out this new process.
Regardless of the outcome of this APE, if the package is accepted during this trial,
the acceptance stands; That is, if this APE falls through but the package used
the proposed process here, it still counts as Affiliated and does not have to re-apply.

Reviewers who agree to participate will use the `reviewer sign-up form`_
for pyOpenSci. If a package agrees but the chosen reviewer declines this trial,
the editor assigned will find a new reviewer.


Backward compatibility
======================

.. This section describes the ways in which the APE breaks backward compatibility.

Somewhat compatible:

* Packaging guidelines are pretty similar.
* Existing Affiliated editors would join the `pyOpenSci Editorial Board`_.
* There would be cross-listing between Affiliated editors over at pyOpenSci and our roles page.
* There would be cross-listing between Affiliated packages over at pyOpenSci
  and Affiliated page.

Not backward compatible:

* Reviewers can no longer be anonymous and have to sign up via `reviewer sign-up form`_.
  The whole review process is open.
* Instead of static color badges, there will be a link to full pyOpenSci
  review for that package. More dynamic badges might come later (see **"Re-review" of packages**).
* Inactive packages will be sunsetted if revival is not an option.


Alternatives
============

We keep the status quo; no changes needed but we are also not tapping into
similar effort in a wider scientific Python community.


Decision rationale
==================

<To be filled in by the coordinating committee when the APE is accepted or rejected>
