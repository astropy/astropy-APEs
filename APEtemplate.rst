.. title:: APE template

.. author:: Perry Greenfield

.. date:: 2013 October 11

.. type:: *one of three allowed types: Standards, Informational, Process*

.. status:: *various allowed status values, see below*

This APE template is a guideline of the sections that a APE should contain.  Extra sections may be added if appropriate, and unnecessary sections may be noted as such.

(note: this template was derived from the Matplotlib MEPTemplate)

Status
------

APEs go through a number of phases in their lifetime:

- **Discussion**: The APE is being actively discussed on the mailing list and it is being improved by its author.  The mailing list discussion of the APE should include the APE number (APExxx) in the subject line so they can be easily related to the APE.

- **Provisionally Accepted/Implementation underway**: Consensus was reached on the mailing list and implementation work has begun.

- **Completed**: The implementation has been merged into master.

- **Rejected**: The decision was to reject the proposal.

- **Superseded**: This APE has been abandoned in favor of another approach.

Branches and Pull requests
--------------------------

All development branches containing work on this APE should be linked to from here.

All pull requests submitted relating to this APE should be linked to from here.  (A APE does not need to be implemented in a single pull request if it makes sense to implement it in discrete phases).

Abstract
--------

The abstract should be a short description of what the APE will achieve.

Detailed description
--------------------

This section describes the need for the APE.  It should describe the existing problem that it is trying to solve and why this APE makes the situation better.  It should include examples of how the new functionality would be used and perhaps some use cases.

Implementation
--------------

This section lists the major steps required to implement the APE.  Where possible, it should be noted where one step is dependent on another, and which steps may be optionally omitted.  Where it makes sense, each  step should include a link related pull requests as the implementation progresses.

Backward compatibility
----------------------

This section describes the ways in which the APE breaks backward incompatibility.

Alternatives
------------

If there were any alternative solutions to solving the same problem, they should be discussed here, along with a justification for the chosen approach.

Decision Rationale
------------------

The reasoning behind the decision, particularly important for a rejected APE. 

