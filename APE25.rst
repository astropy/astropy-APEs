An initial roadmap for static type checking
-------------------------------------------

author: Nick Murphy

date-created:

date-last-revised:

date-accepted:

type: Standard Track

status: Discussion


Abstract
--------

This APE describes the initial process for gradually adding static
type checking to Astropy.

Detailed description
--------------------



.. This section describes the need for the APE.  It should describe
   the existing problem that it is trying to solve and why this APE
   makes the situation better.  It should include examples of how the
   new functionality would be used and perhaps some use cases.


Benefits of adding type hint annotations with static type checking
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Can find programming errors that would be difficult to find
   otherwise
#. Helps introspection and tab completion in Jupyter notebooks
#. Helps IDEs to make things easier for humans
#. Helpful for downstream package maintainers

Disadvantages of adding type hint annotations with static type checking
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Requiring type hint annotations that mypy doesn't complain about
   adds an extra step to contributing to Astropy...

1. An additional step is required to contribute to Astropy.

1. Type hint annotations may add a barrier to entry for contributing
   to Astropy.





Branches and pull requests
--------------------------

.. Any pull requests or development branches containing work on this
   APE should be linked to from here.  (An APE does not need to be
   implemented in a single pull request if it makes sense to implement
   it in discrete phases). If no code is yet implemented, just put
   "N/A"


Implementation
--------------

#. Create an initial ``mypy.ini`` configuration file that ignores all
   existing errors and warnings from mypy. The configuration file must
   enable static type checking on a single Python module, and disable
   static type checking for all other modules and subpackages.

#. Create a tox environment for running mypy.

#. Create a GitHub Action that runs the tox environment as a GitHub
   Action for each pull request.

#. Create a section in Astropy's developer documentation on the
   process for adding type hint annotations to Astropy and using
   static type checking.

#. Set the GitHub Action for running mypy as required for a pull
   request to be merged.

#. Gradually enable static type checking in ``astropy.cosmology`` and
   ``astropy.units`` by expanding the number of files checked by mypy,
   enabling new mypy rules on a per-file or per-subpackage basis,
   adding type hint annotations, creating type stub files for
   dynamically generated content, and using ``# type: ignore`` style
   comments to indicate when static type checking should be skipped on
   particular lines.

#. Update the developer documentation with lessons learned and best
   practices for adding type hint annotations.

At this point, subpackage maintainers may gradually enable static type
checking for the subpackages that they maintain.



.. This section lists the major steps required to implement the APE.  Where
   possible, it should be noted where one step is dependent on another, and which
   steps may be optionally omitted.  Where it makes sense, each  step should
   include a link related pull requests as the implementation
   progresses.


Backward compatibility
----------------------

This APE is expected to maintain backward compatibility.


Alternatives
------------

.. If there were any alternative solutions to solving the same
   problem, they should be discussed here, along with a justification
   for the chosen approach.

1. **Do not enable static type checking of Astropy.** Type hint
   annotations have begun to be added to Astropy. At present, there is
   no way to check the validity or accuracy of type hint annotations. 
2. **Take a more aggressive approach to static type checking.** Most
   recommendations 
3. **Delay adding a static type checker to Astropy's continuous
   integration suite.** 

Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
