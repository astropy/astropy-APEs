Distributing Astropy Project Funding
====================================

author: Kelle Cruz, H. Moritz Günther, Aarya Patil, John Swinbank, Erik Tollerud

date-created: 2022 January 21

date-last-revised: 2022 February 28

date-accepted: 2022 February 28

type: Process

status: Accepted


Abstract
=========

This APE aims to provide guidance about spending Astropy Project funds by formulating a set of principles about what to pay for and who to pay. It will also touch on how expenditure decisions are made, how funds get administered, and a principle of strong transparency around these topics. It is heavily inspired by `Numpy's NEP 48 <https://numpy.org/neps/nep-0048-spending-project-funds.html>`_.


Detailed description
====================

Motivation and Scope
--------------------

Astropy is currently a fiscally sponsored project of NumFOCUS, a 501(c)(3) nonprofit organization headquartered in Austin, TX, meaning NumFOCUS is the legal entity that handles finances for Astropy. Therefore, all legal and accounting matters related to the Astropy Project have to follow the rules and regulations for American nonprofits. All nonprofit donations are classified into two categories: unrestricted funds, which may be used for any legal purpose appropriate to the organization, and restricted funds, which are reserved for a particular purpose (a specific project, educational program, etc.).

Having a clear set of principles around how to use these funds will facilitate spending them fairly and effectively. Additionally, it will make it easier to solicit donations and other contributions. The guidelines here are meant for community members to consider when proposing to use Project funds, for the community to consider when voting on proposals, and for the Finance Committee to use to guide the funding process.

A key assumption of this APE is that Astropy is developed and maintained by an institutionally and geographically distributed group of independently motivated individuals, and that the Project funds are not enough to employ most of the maintainers full-time. If funding increases to the point where that assumption is no longer true, this APE should be updated.

In scope for this APE are:

* Principles of spending project funds: what to pay for, and who to pay, and how much to pay.
* Describing how Astropy's funds get administered.
* Describing how expenditure decisions are proposed and made.


Principles for spending project funds
-------------------------------------

Astropy will likely always be a project with many times more volunteer contributors than funded people. It follows that having funded people operate in ways that attract more volunteers and enhance their participation experience is critical. That key principle motivates many of the more detailed principles given below for what to pay for and whom to pay.

We identify four distinct considerations when disbursing funds:

* Agreeing upon the types of activities that can be funded,
* Establishing a process for deciding which specific activities should be funded,
* Finding someone to do the work (if necessary),
* Determining an appropriate compensation level.

The following sections go into detail on each of these points.

What to pay for
^^^^^^^^^^^^^^^
We can pay for anything which contributes to the broad mission of the Astropy Project. This includes, for example, code development, community engagement, education, promotion, etc. Funds can be used for many purposes, for example, paid developer time, teaching buyout, development of training materials, expenses for developer meetings or sprints, hosted hardware, and CI or other software services, carbon offsets, promotion and advertising, travel to conferences or meetings to promote Project activities.

How to decide what to pay for
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
1. Pay for things that are important and otherwise will not get done. *Rationale: there is much more work to be done than there are funds to do all of that work. There are also many volunteers and externally sponsored people interested and willing to do some of the work without compensation from the Project.*
2. Do not rely on funding always being there. Plan for sustainability: how will activities be continued when the funds dry up? Invest in documentation, infrastructure, and easy to maintain workflows. For example, funded work on governance should always record the process clearly so that volunteers who follow have less work to do to figure it out.
3. Consider potential positive benefits to Astropy maintainers and contributors, maintainers of other projects, end users, and other stakeholders like packagers and educators.
4. Also consider the long-term costs to Astropy maintainers and contributors, to counterbalance item number 3.
5. Do not increase the load on volunteer maintainers by assuming they are available to review or otherwise support the work. Instead, proposals should include paid time for reviews and other necessary activities. *Rationale: the effect of spending funds should be positive for everyone, not just for the people getting paid. This is also a matter of fairness.*

Restricted / earmarked funding sources
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Much of the Project's funding comes from specific grants from private foundations and government agencies. In these cases, the fundable activities are explicitly described in the proposal text. Such funds must be spent following the proposed plan and in accordance with any other conditions imposed by the funding agency. The Finance Committee should make these restrictions clear to the community so that they may submit funding requests aligned with the existing grants.

Whom to pay
^^^^^^^^^^^
1. All else being equal, give preference to existing contributors.
2. When looking outside of the current team (from here on we use the word "team" to include anyone in the Astropy community, including code contributors, mentors, workshop facilitators, outreach, or any other activity for the benefit of the project), consider this an opportunity to make the project team more diverse.
3. Pay attention to the following when considering whether to pay someone

   * the necessary skills to execute the tasks,
   * communication and self-management skills,
   * experience contributing to and working with open source projects,
   * likelihood of long-term involvement with the Astropy community.

It will likely depend on the particular activity as to whether there's already a clear best candidate within the Astropy team, or if we must look for new people to get involved. This decision should be taken in accordance with documented Project governance procedures. If the position is not filled from within the project, it should be publicly advertised to give a broader group of people a chance to apply for it.

Compensating an appropriate amount
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Process
+++++++
In general (see next section), we put the funding level directly in the hands of the community and those making funding requests. We use a process where proposers suggest work and include a budget request which they think is required to achieve the suggested work. These suggestions are then put to the community for input and feedback. As part of the review process, the community is expected to take “value-for-money” into account. This mechanism places the trust to decide when compensation is unreasonably high or low to the community. In other cases, the Finance Committee will determine an appropriate amount taking into account the discussion and rationale below.

Discussion and Rationale
++++++++++++++++++++++++
This is motivated by the fundamental tension that we, as a community, want to get the most out of the funds available, but we also want to avoid a “race to the bottom” where developers undercut others on price until the pay is too low to live on.
Setting an appropriate level of pay for certain work is a difficult task, since a range of factors influence what people think is “fair.” This includes, to name just a few considerations,

* external factors like the cost of living in a certain place or country,
* if people are paid as independent contractors (and thus have to factor vacation, sick time, medical insurance, etc. into an hourly rate) or as a subaward to, for example, an academic institution (which usually set rates that include overheads and fringe benefits that are then provided to the employee),
* What monetary value one puts on experience (e.g., an undergraduate student vs. an experienced software developer).

`NEP 48 <https://numpy.org/neps/nep-0048-spending-project-funds.html>`_ has extensive discussion and a good list of further references on this topic.

However, we recognize that the approach of leaving the approved budget entirely up to project members also has downsides. In particular, less experienced people may be tempted to suggest work packages with a compensation that is unfairly low for the work required to increase their perceived chances to obtain funding. We place trust in the community to recognize these short-comings of the process and to advise and mentor each other to propose realistic budgets that allow the proposed work to be completed with a fair rate of pay. As the diversity of proposers increases, we recognize that this approach will likely need to be modified and potentially be made more prescriptive. This may come in the form of a future APE or more incremental changes as experience grows.

Defining fundable activities and projects
-----------------------------------------
We define three separate categories of expense, by value:

* Up to $2000 (US);
* $2000 to $10000 (US);
* More than $10000 (US).

Items in the first category — up to $2000 — may be approved by majority vote of the Finance Committee.

Items in the second category — up to $10000 — can be approved by majority vote of the Finance Committee. If, in the judgment of one or more Finance Committee members, the item would not receive widespread support, the Committee will communicate these concerns to the Coordination Committee (CoCo). The CoCo may provide a decision, or may request that the Finance Committee organize a vote among the Astropy membership to approve the expense.

Items in the third category — more than $10000 — should follow approved Project decision making methods from `APE0 <https://github.com/astropy/astropy-APEs/blob/main/APE0.rst>`_, or a vote among the Astropy voting membership unless specifically allocated in a grant to a specific person or organization already. In the latter case, the Finance Committee is tasked to ensure it is spent according to the provisions of the grant. The Finance Committee will periodically solicit suggestions from the community for projects and other major expenses and coordinate a selection process.

*Rationale: We want the community to make the big and important decisions, but also not overburden community members with frequent discussions and voting.*

Strategy for spending/saving funds
----------------------------------
There is an expectation from Astropy funders that the funds will be used for the benefit of the project and the community. Therefore, we should spend available funds thoughtfully, strategically, and fairly, as they become available. For emergencies, we should keep a reserve of $10,000 to $15,000 (US) that could cover, for example, a year of CI and hosting services, a few weeks of full-time maintenance work, or engaging a consultant to address a specific need.

How project funds get administered
-----------------------------------
Astropy wants to administer project funds as transparently as possible while remaining efficient for those receiving and administering funds, complying with the terms of specific grant programs or funding agencies, and providing appropriate protection for personal information.  The administration process is therefore publicly visible and open to input from the Astropy community whenever possible. In some cases, though, specific details or personal information will be redacted or otherwise hidden from public view.

Beyond these broad guidelines, the details of administration are intentionally not specified in this APE, as experience shows they need to be adapted to changing circumstances and different funding sources. Specific policies on administration of funds live in `the finance section of the Astropy Project Github repo <https://github.com/astropy/astropy-project/tree/main/finance>`_. Changes to these policies follow the same Pull Request process as for code contributions in the Project.

Acknowledgements
-----------------
Thanks to NumPy for NEP 48 (https://numpy.org/neps/nep-0048-spending-project-funds.html) which is used as the basis for this document.


Alternatives
============

The main alternative to having this APE is **not** having this APE and continue to follow a general agreement to work with funds along the lines described above, but without a written policy on the level of an APE.


Decision rationale
==================

Following discussion and comments on GitHub in the `APE pull request <https://github.com/astropy/astropy-APEs/pull/72>`_, this APE was accepted by the Coordinating Committee after community consensus was reached.
