.. contents:: Table of Contents
.. _APETemplate: https://github.com/astropy/astropy/wiki/APETemplate

APE Purpose and Process
-----------------------

author: Perry Greenfield

date: 2013 October 11

type: process

status: Discussion

Abstract
--------

APE stands for Astropy Proposal for Enhancement. An APE is a design document providing information to the Astropy community, or describing a new feature for Astropy or its processes or environment. The APE should provide a concise technical specification of the feature and a rationale for the feature.

We intend APEs to be the primary mechanisms for proposing major new features, for collecting community input on an issue, and for documenting the design decisions that have gone into Python. The APE author is responsible for building consensus within the community and documenting dissenting opinions.

Because the APEs are maintained as text files in a versioned repository (indirectly since this wiki is versioned within github), their revision history is the historical record of the feature proposal

Detailed description
--------------------

APE Types
.........

Chimp, Gorilla, Bonobo...

But more seriously.

There are three kinds of APE:

A Standards Track APE describes a new feature or implementation for Astropy. It may also describe an interoperability standard that will be supported outside the standard library for current Astropy versions before a subsequent APE adds standard library support in a future version.

An Informational APE describes a Astropy design issue, or provides general guidelines or information to the Python community, but does not propose a new feature. Informational APEs do not necessarily represent an Astropy community consensus or recommendation, so users and implementers are free to ignore Informational APEs or follow their advice. We await a Zen of Astropy APE.

A Process APE describes a process surrounding Astropy, or proposes a change to (or an event in) a process. Process APEs are like Standards Track APEs but apply to areas other than the Astropy package itself. They may propose an implementation, but not to Astropy's codebase; they often require community consensus; unlike Informational PEPs, they are more than recommendations, and users are typically not free to ignore them. Examples include procedures, guidelines, changes to the decision-making process, and changes to the tools or environment used in Astropy development. Any meta-APE is also considered a Process APE.

Submitting an APE
.................

The APE process begins with a new idea for Astropy. It is highly recommended that a single APE contain a single key proposal or new idea. Small enhancements or patches often don't need a APE and can be injected into the Astropy development workflow with a patch submission to the Astropy issue tracker. The more focused the APE, the more successful it tends to be. If in doubt, split your APE into several well-focused ones.

Each APE must have a champion -- someone who writes the APE using the style and format described below, shepherds the discussions in the appropriate forums, and attempts to build community consensus around the idea. The APE champion (a.k.a. Author) should first attempt to ascertain whether the idea is APE-able. Posting to the astropy-dev mailing list is the best way to go about doing this.

Vetting an idea publicly before going as far as writing a APE is meant to save the potential author time. Many ideas have been brought forward for changing Astropy that have been rejected for various reasons. Asking the Astropy community first if an idea is original helps prevent too much time being spent on something that is guaranteed to be rejected based on prior discussions (searching the internet does not always do the trick). It also helps to make sure the idea is applicable to the entire community and not just the author. Just because an idea sounds good to the author does not mean it will work for most people in most areas where Astropy is used.

Once the champion has asked the Astropy community as to whether an idea has any chance of acceptance, a draft APE should be presented to astropy-dev. This gives the author a chance to flesh out the draft APE to make properly formatted, of high quality, and to address initial concerns about the proposal.

Following a discussion on astropy-dev, the proposal should be submitted as a Pull Request to astropy-APEs with the name APE<n>.rst where <n> is an appropriately assigned number. The draft must use the APEtemplate.rst file. 

As updates are necessary, the APE author can update the APE as needed (noting edit dates in the APE). 

Standards Track APEs consist of two parts, a design document and a reference implementation. It is generally recommended that at least a prototype implementation be co-developed with the APE, as ideas that sound good in principle sometimes turn out to be impractical when subjected to the test of implementation. This is not required when too onerous, but some indication of implementation practicality is highly recommended by actual code. The best way to provide that code is via a github pull request either to the `astropy/astropy <https://github.com/astropy/astropy>`_  repository, or `astropy/astropy-api <https://github.com/astropy/astropy-api>`_, as appropriate.

APE Review
..........

Normally APEs are discussed on astropy-dev and perhaps in other forums.  Sometimes APEs will grow out of an existing pull request, but it's better to have any discussion after the APE is generated on the list, as it has a wider audience.  The final decision on any APE is made by the coordinating committee, though usually a consensus in the development is sufficient (but in unusual cases may be overridden by the coordinating committee). The decision may require changes to the APE and any implementation. Final acceptance is not done until the required changes are made to the APE and implementation.

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
