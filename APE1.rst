APE Purpose and Process
-----------------------

author: Perry Greenfield, Lia Corrales, Thomas Robitaille

date-created: 2013 October 11

date-last-revised: 2021 February 26

date-accepted: 2013 November 8

type: Process

status: Accepted

Abstract
--------

APE stands for Astropy Proposal for Enhancement. An APE is a design document
providing information to the Astropy community, or describing a new feature
for Astropy or its processes or environment. The APE should provide a concise
technical specification of the feature and a rationale for the feature.

We intend APEs to be the primary mechanisms for proposing major new features,
for collecting community input on an issue, and for documenting the design
decisions that have gone into Python. The APE author is responsible for
building consensus within the community and documenting dissenting opinions.

Because the APEs are maintained as text files in a versioned repository
(indirectly since this wiki is versioned within github), their revision
history is the historical record of the feature proposal

Detailed description
--------------------

APE Types
.........

Chimp, Gorilla, Bonobo...

But more seriously.

There are three kinds of APE:

* A "Standard Track" APE describes a new feature or implementation for
  Astropy. It may also describe an interoperability standard that will be
  supported  in current Astropy versions before a subsequent APE adds the
  feature in the future.

* An "Informational" APE describes a Astropy design issue, or provides general
  guidelines or information to the Python community, but does not propose a new
  feature. Informational APEs do not necessarily represent an Astropy community
  consensus or recommendation, so users and implementers are free to ignore
  Informational APEs or follow their advice. We await a Zen of Astropy APE.

* A "Process" APE describes a process surrounding Astropy, or proposes a change
  to (or an event in) a process. Process APEs are like Standard Track APEs but
  apply to areas other than the Astropy package itself. They may propose an
  implementation, but not to Astropy's codebase; they often require community
  consensus; unlike Informational PEPs, they are more than recommendations, and
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

Each APE must have a champion -- someone who writes the APE using the style
and format described below, shepherds the discussions in the appropriate
forums, and attempts to build community consensus around the idea. The APE
champion (a.k.a. Author) should first attempt to ascertain whether the idea is
APE-able. Posting to the astropy-dev mailing list is the best way to go about
doing this.

Following a discussion on astropy-dev, the proposal should be submitted as a
Pull Request to astropy-APEs with the name APE<n>.rst where <n> is an
appropriately assigned number. The draft must use the APEtemplate.rst file.
That a formal proposal has been submitted as a PR should be announced to the
astropy-dev list.

The APE author may update the APE as needed.

Standard Track APEs consist of two parts, a design document and a reference
implementation. It is generally recommended that at least a prototype
implementation be co-developed with the APE, as ideas that sound good in
principle sometimes turn out to be impractical when subjected to the test of
implementation. This is not required when too onerous, but some indication of
implementation practicality is highly recommended by actual code. The best way
to provide that code is via a github pull request either to the
`astropy/astropy <https://github.com/astropy/astropy>`_ repository, or
`astropy/astropy-api <https://github.com/astropy/astropy-api>`_, as
appropriate.

APE Review
..........

Normally APEs are discussed on astropy-dev and perhaps in other forums.
Sometimes APEs will grow out of an existing pull request, but it's better to
have any discussion after the APE is generated on the list, as it has a wider
audience. The final decision on any APE is made by the coordinating committee,
though usually a consensus in the development is sufficient (but in unusual
cases may be overridden by the coordinating committee). The decision may
require changes to the APE and any implementation. Final acceptance is not
done until the required changes are made to the APE and implementation.

APE Status
..........

An APE's status can

* "Discussion": New APE pull requests should always start in this status.  This
  means the APE is currently being considered and a decision has not been made
  regarding what should be done.

* "Accepted": If an APE is accepted, it will be merged - either the original
  author can do this if they wish to fill in the "decision rationale" section,
  or the coordination committee member who merges it can change the status and
  write the rationale.  Regardless, if the APE is an informational or process
  APE, it is now done. If it is standard track, this status means it is in the
  process of being implemented.

* "Implemented": Only valid for a Standard Track APE.  This means the feature
  discussed in the APE is complete and has been fully merged into the main
  Astropy repository.

* "Rejected": If it is decided that an APE should be rejected, the person
  who merges it should change its status to "Rejected."  The "decision
  rationale" should also be filled in, either by the merger, the original
  author, or another community member who voiced objections to the APE.
  The goal is to try to reflect the overall community opinion in these
  rationales, so that new community members can understand why a decision was
  made.

Modifications to an APE
.......................

An APE can be modified, with one person acting as "champion" for the APE update.
Similar to the APE development procedure, the update champion should discuss
their desired changes with the Astropy community (e.g., through the astropy-dev
mailing list). Following a discussion on astropy-dev, the APE update should be
submitted as a Pull Request which should be advertised on astropy-dev.

The APE should be modified directly without annotating the changes. If not
already listed, the update champion can add themselves to the author list for
the APE being updated. A new section, **Previous versions of this APE** should
be added to the bottom of the APE (if not already there) with a bullet point
list of dates which link to the DOI of the previous versions of APEs.

The Astropy Coordinating Committee is responsible for reviewing and ultimately
accepting Pull Requests for changes to individual APEs, unless otherwise
specified within the APE document of interest. For non-trivial changes,
community consensus should generally be reached before acceptance.

Branches and Pull requests
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

Decision Rationale
------------------

The coordinating committee thought it was a honking great idea.

Previous versions of this APE
-----------------------------

* `2013-11-08 <https://zenodo.org/record/1043886>`_