Specutils Specifications
------------------------

author: S. M. Crawford, N. Earl, A. Ginsburg, E. Tollerud

date-created: 2017 April 21 <replace with the date you submit the APE>

date-last-revised: 2017 March 7 <keep this up to date anytime something changes>

type:  Standard Track

status: Discussion


Abstract
--------

Spectroscopy lies at the core of much of astronomy.  It is the goal of this APE to  coordinate and plan the development of spectroscopic software.   A number of tools have already been developed within astropy as well in individual packages, and in this APE, we propose the specifications for a Spectrum1D object, and outline a list of tools that are needed by the community.  

Detailed description
--------------------

Spectroscopy in astronomy

Spectrum 1D in specutils

problems: already has been forked into Spectrum1DRef, not widely adopted, no longer maintained

issues: order of arguments, reversible wcs/dispersion, using astropy.io 

Need for common analysis tools

Functionality we want in general spectroscopic tools (specutils, specreduce, or specviz)

- detecting lines
- measuring lines
  - fitting models to lines
  - measuring equivalent widhts
  - centroid of lines
- redshift measurementions
- heliocentric corrections
- 


Branches and pull requests
--------------------------

Any pull requests or development branches containing work on this APE should be
linked to from here.  (An APE does not need to be implemented in a single pull
request if it makes sense to implement it in discrete phases). If no code is yet
implemented, just put "N/A"


Implementation
--------------

This section lists the major steps required to implement the APE.  Where
possible, it should be noted where one step is dependent on another, and which
steps may be optionally omitted.  Where it makes sense, each  step should
include a link related pull requests as the implementation progresses.


Backward compatibility
----------------------

This section describes the ways in which the APE breaks backward compatibility.


Alternatives
------------

If there were any alternative solutions to solving the same problem, they should
be discussed here, along with a justification for the chosen approach.


Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
