APE Purpose and Process
-----------------------

author: Perry Greenfield, Lia Corrales, Thomas Robitaille, Erik Tollerud, Pey Lian Lim

date-created: 2013 October 11

date-last-revised: 2025 June 26

date-accepted: 2013 November 8

type: Process

status: Accepted

revised-by:

* Lia Corrales, Thomas Robitaille, Erik Tollerud - 2021 February 26 - Added APE modification process
* Pey Lian Lim - 2024 February 5 - Added APE numbering process
* Thomas Robitaille, Pey Lian Lim - 2025 June 26 - Modify APE process to merge APEs that are in the discussion stage

Abstract
--------

APE stands for Astropy Proposal for Enhancement. An APE is a design document
providing information to the Astropy community, or describing a new feature
for Astropy or its processes or environment. The APE should provide a concise
technical specification of the feature and a rationale for the feature.

We intend APEs to be the primary mechanisms for proposing major new features,
for collecting community input on an issue, and for documenting the design
decisions that have gone into Astropy. The APE author is responsible for
building consensus within the community and documenting dissenting opinions.

Because the APEs are maintained as text files in a versioned repository, their revision
history is the historical record of the feature proposal.

Detailed description
--------------------

APE Types
.........

Chimp, Gorilla, Bonobo...

But more seriously.

There are three kinds of APE:

* "Standard Track": A "Standard Track" APE describes a new feature or implementation for
  Astropy. It may also describe an interoperability standard that will be
  supported  in current Astropy versions before a subsequent APE adds the
  feature in the future.

* "Informational": An "Informational" APE describes a Astropy design issue, or provides general
  guidelines or information to the Astropy community, but does not propose a new
  feature. Informational APEs do not necessarily represent an Astropy community
  consensus or recommendation, so users and implementers are free to ignore
  Informational APEs or follow their advice. We await a Zen of Astropy APE.

* "Process": A "Process" APE describes a process surrounding Astropy, or proposes a change
  to (or an event in) a process. Process APEs are like Standard Track APEs but
  apply to areas other than the Astropy package itself. They may propose an
  implementation, but not to Astropy's codebase; they often require community
  consensus; unlike Informational APEs, they are more than recommendations, and
  users are typically not free to ignore them. Examples include procedures,
  guidelines, changes to the decision-making process, and changes to the tools
  or environment used in Astropy development. Any meta-APE is also considered a
  Process APE.

Submitting an APE
.................

The APE process begins with a new idea for Astropy. It is highly recommended
that a single APE contain a single key proposal or new idea. Small
enhancements or patches often don't need a APE and can be injected into the
Astropy development workflow with a patch submission to the Astropy issue
tracker. The more focused the APE, the more successful it tends to be. If in
doubt, split your APE into several well-focused ones.

New ideas for the following APEs should always be submitted as modifications
to the respective APE, never as a new APE:

* `APE 0: The Astropy Project Governance Charter <https://github.com/astropy/astropy-APEs/blob/main/APE0.rst>`_
* `APE 1: APE Purpose and Process <https://github.com/astropy/astropy-APEs/blob/main/APE1.rst>`_

Each APE must have a champion -- someone who writes the APE using the style
and format described below, shepherds the discussions in the appropriate
forums, and attempts to build community consensus around the idea. The APE
champion (a.k.a. Author) should first attempt to ascertain whether the idea is
APE-plicable. Posting to the `astropy-dev <https://groups.google.com/g/astropy-dev>`_
mailing list is the best way to go about doing this.

Note that there is not much point to submitting new APEs unless someone or some
group has signed up to implement it should the APE be accepted
(typically this would involve the author or authors of the APE). Just issuing
an APE in order to spur others to do work is not going to be received
well, due to the open-source nature of the Astropy Project.

While attaching additional files to an APE is discouraged, if there are no other
options, these files could go under the `assets <https://github.com/astropy/astropy-APEs/tree/main/assets>`_
sub-directory, with the filenames each starting with `ape<n>_` for clarity.
Large files should be hosted externally and linked to the APE instead.

Following a discussion on `astropy-dev <https://groups.google.com/g/astropy-dev>`_,
the proposal should be submitted as a
pull request to astropy-APEs with the name `APE<n>.rst` where `<n>` is an
appropriately assigned number (i.e., not already an accepted/proposed/rejected APE).
The draft must use the `APEtemplate.rst file`. Once the pull request is open,
the `@astropy/ape-editor-team <https://github.com/orgs/astropy/teams/ape-editor-team>`_
should be mentioned in the pull request, and
the APE will then undergo an editorial review which focuses on wording,
spelling, grammar, and clarity, but does not focus on the substance of the APE.
The editorial process ideally should not take longer than a month.
Once the APE editors are happy with the APE, they will:

#. Make sure the APE has a status of "Discussion".

#. If additional files are attached with the APE in the PR, make sure the
   file sizes are reasonable.

#. If not already, add a new entry for the APE under "APEs" table in `README.rst`
   with the:

   * Finalized number (the next available one in the `main` branch
     of the repository)
   * Title (with link to the APE RST file)
   * Last revised date (set to day of merging)
   * Status (set to "Discussion")
   * DOI left blank.

#. Preview the `README.rst` updates and test the new link to make sure they are all correct.

#. Merge the pull request.

#. Send an email to `astropy-dev <https://groups.google.com/g/astropy-dev>`_
   with a link to the APE and a short summary to kick off the discssion period.
   Emphasize that the proposal should be discussed on the mailing list, not the
   pull request.

As the discussion proceeds, the authors can choose to update the APE at any point via
new pull requests, which are as before, subject to editorial review before merging.
If needed, editors should continue to stress that discussions about the substance of the APE
should be kept on the mailing list as much as possible.

Standard Track APEs consist of two parts:

* a design document and
* a reference implementation.

For serious consideration, it is generally recommended that at least a prototype
implementation be co-developed with the APE, as ideas that sound good in
principle sometimes turn out to be impractical when subjected to the test of
implementation. This is not required when too onerous, but some indication of
implementation practicality is highly recommended by actual code. The best way
to provide that code is via a GitHub draft pull request to the affected Astropy repository
(usually `astropy/astropy <https://github.com/astropy/astropy>`_), as
appropriate.

APE Review
..........

Once the authors feel that they have reached as close to a consensus as they
will get (i.e., the community discussion on the APE has wound
down), they can contact the Coordination Committee, who are
responsible for making a final decision. The decision can be to:

* accept the APE,
* reject the APE, or
* suggest to the authors to continue the discussion before re-submitting the APE for a decision.

One of the Coordination Committee members should then:

#. Fill in the "Decision rationale" section of the APE with a description of why
   the status was chosen, including a summary of the community's discussion as relevant.
#. Update the `date-last-revised` to the day of merging and `status` to
   one of the options listed under "APE Status" section below.
#. Update the "Date (last revised)" and "Status" info for the relevant row
   in the "APEs" table in `README.txt`.
#. Leave a brief comment in the PR indicating the result.
#. Merge the PR with the above changes.
#. If the APE was accepted, then see `README.rst` for archiving instructions.
#. Send an email to `astropy-dev <https://groups.google.com/g/astropy-dev>`_
   announcing the new status. In general, this should just point to the
   APE rather than providing additional decision rationale.

APE Status
..........

For all the statuses below, the "Decision rationale" section should be updated
to reflect the overall community opinion, if applicable, so that
new community members can understand why such decision was made.

An APE's status can be:

* "Discussion": New APE pull requests should always start in this status. This
  means the APE is currently being considered and a decision has not been made
  regarding what should be done.

* "Accepted": Generally, an implementation is expected before a standard-track APE
  can be considered fully accepted; For proposals that require extensive work that
  few are willing to perform without some assurance it will be accepted, provisional
  acceptance is an option (the provision could be clarified under "Decision rationale").
  For standard track, any code implementation of this APE should
  clearly link back to the APE document in this repository. For completeness,
  the APE could also be updated later to add any additional implementation
  pull requests to its "Implementation" section.

* "Rejected"

* "Withdrawn": If the APE authors no longer feel that their proposal is feasible,
  they can choose to withdraw the APE. Reason for withdrawal should be clearly
  stated under "Decision rationale" section. A withdrawn APE is in effect equivalent
  to a Rejected APE in that it will not be implemented.

* "Superseded": An APE that was accepted but has since become irrelevant
  or entirely replaced by some other APE or event can be given this status.
  Such a status change may not require the regular APE modification process if
  whatever caused this status already followed a discussion process at least
  as long as the APE discussion process (at least 2 weeks as per the comment
  period Astropy usually follow in other processes).

* "Abandoned": If an APE is present in the repository but has been labelled
  as "Discussion" for at least a year without any sign of further discussion
  or progress, the APE can be marked as "Abandoned" (after reaching out to the
  authors to check that they are not able to continue working on it). An
  abandoned APE can be picked up at any time by other members of the community,
  and the status changed back to "Discussion".

Modifications to an existing APE
................................

An existing APE can be modified, with one person acting as champion for the APE update.
Example cases:

* Modifying the contents (including superseding) of an accepted APE.
* Withdrawing an APE under discussion.
* Reviving an abandoned APE.

Similar to the APE development procedure, the update champion should discuss
their desired changes with the Astropy community through the
`astropy-dev <https://groups.google.com/g/astropy-dev>`_ mailing list.
Following a discussion on mailing list, the APE update should be submitted
as a pull request, which should be advertised on the relevant mailing list thread.
After the community discussion period and the changes are finalized,
the Coordination Committee would follow a similar procedure in "APE Review"
should they decide to merge the pull request.

The APE should be modified directly without annotating the changes. If not
already listed, the update champions can add themselves to the author list for
the APE being updated. A new section, **Previous versions of this APE** should
be added to the bottom of the APE (if not already there) with a bullet point
list of dates which link to the DOI of the previous versions of APEs as well as
links to the merge commits of the APEs. The `date-last-revised` header for the APE
should be updated, and a `revised-by` header should be added for each
update (see the first `revised-by` section of *this* APE for a formatting
example).

Branches and pull requests
--------------------------

N/A

Implementation
--------------

N/A

Backward compatibility
----------------------

N/A

Alternatives
------------

N/A

Decision rationale
------------------

The Coordinating Committee thought it was a honking great idea.

Previous versions of this APE
-----------------------------

* 2013-11-08 [`DOI <http://doi.org/10.5281/zenodo.1043886>`_] [`GitHub <https://github.com/astropy/astropy-APEs/blob/42951733ac42c0ea178d8df30705274a43c93091/APE1.rst>`_]
* 2021-03-09 [`DOI <https://doi.org/10.5281/zenodo.10805921>`_] [`GitHub <https://github.com/astropy/astropy-APEs/blob/9896678a4a8dc7e5aeedd0230b37816953dbf800/APE1.rst>`_]
