CPU Threading [DRAFT]
----------------

author: Jamie Noss

date-created: 2018 October 26

date-last-revised: 2018 October 26

date-accepted: TBD

type: Standard Track

status: Discussion


Abstract
--------

As the datasets utilized in modern day astronomy become ever increasing and
continue to cross over into the realm of Big Data, so does the computational burden to process
them. At present, there is only one hardware approach to solving this demand and
that is parallel processing.

This proposal is only intended to address CPU threading and not, e.g., GPU threading.
It is also only intended to address the *allowance* of utilizing said technology and *not*
necessarily a broader directive to explicitly incorporate said technology
everywhere applicable throughout the Astropy code base.
It is intended to address at which level(s) the Astropy package interfaces
with a given threading package/library. Whether at the Python level and/or at the
C/C++ extension level. Furthermore, if at the C/C++ level, whether canonical
implementations of the OpenMP Standard are acceptable. 


Detailed description
--------------------

Disseminating parallel processing, there are, at present, only four
top level implementations:

* CPU multi-core technology - CPU threading.
* GPU multi-core technology - GPU (vector/SIMD) threading.
* Cluster machines.
* Memory modules and SSD drives with onboard CPUs.

This proposal is intended only to address CPU threading.

Since Astropy is a Python based package, its performance is limited by
the adoption of the Global Interpreter Lock (GIL) within the canonical implementations
of its interpreter , e.g. CPython. Because of this, it is common to supplement the Python
code base with Cython/C/C++ extensions to handle the compute intensive workload.
This spawns the question of how said extensions adopt parallelism?

Since the root question is *not* whether to thread but *how*, the actual root
question is at which level does the Astropy package interface with any given
threading technology? Hierarchically, this is pivoted on whether to do so
within the Python code base or within the extensions, or a healthy combination of the
two.

- Interfacing within the Python code base is the highest level whereby this
  interface interaction is with some other, non-Astropy, package/Python-module
  which it then directly interfaces with some lower-level threading API.
  this therefore removes from the Astropy developer community any and all
  responsibility of maintaining the lower-level interface interaction.

- Interfacing at the Cython/C/C++ level yields only a handful of solutions.
   - Naked interface with the lowest level, e.g. POSIX threads, C++ Std threads, TBB, etc.
   - Develop our own threading library - this is however, identical to the first
     just without the "naked".
   - Interface once removed, e.g. canonically with the OpenMP API.

Writing extensions that interface with the threading technology at the lower level is
not advisable. It would create maintenance issues and just wouldn't be necessary.
The most viable options are to interface within the Python code or within the extensions
using an OpenMP library. Some of the questions to ask and answer here are:

* Are both appropriate?
* When is one more appropriate than another?
* Are either approaches functionally complete?

Perhaps the crux of this APE is to actually question
whether to use Cython/C/C++ extensions at all. With this question at the root of
the decision tree, does the decision to use extensions inexorably lead to the
necessity to thread them? From there, it is then only a question of how, which should lead
to the simplest and canonical method of using implementations of the OpenMP standard.
The alternative to not thread within the extensions posses a serious limitation
on the scope of their performance, where performance is one of the principle reasons
they are implemented in the first place. Does it make sense to produce only trivial
extensions yet revert to parallel python for the more complex threaded tasks?


Branches and pull requests
--------------------------

At present there are no modules within Astropy that *actively* utilize OpenMP
threading support. There is, however,

* Thread ``convolution.convolve()`` with OpenMP `astropy/astropy#7293 <https://github.com/astropy/astropy/pull/7293>`_
* Added a helper function to add OpenMP compilation flags if available `astropy/astropy#346 <https://github.com/astropy/astropy-helpers/pull/346>`_
* Only link object files in add_openmp_flags_if_available `astropy/astropy-helpers#374 <https://github.com/astropy/astropy-helpers/pull/374>`_
* Make add_openmp_flags_if_available() work for clang `astropy/astropy-helpers#382 <https://github.com/astropy/astropy-helpers/pull/382>`_
* Minor updates to OpenMP infrastructure `astropy/astropy-helpers#395 <https://github.com/astropy/astropy-helpers/pull/395>`_
* Extend exception handling of OpenMP support check `astropy/astropy-helpers#399 <https://github.com/astropy/astropy-helpers/pull/399>`_
* Add functionality to disable OpenMP support `astropy/astropy-helpers#410 <https://github.com/astropy/astropy-helpers/pull/410>`_
* Recently removed OpenMP support in `astropy/astropy-healpix#97 <https://github.com/astropy/astropy-healpix/issues/97>`_


Implementation
--------------

Implementing this APE can be broken down into three aspects:

* Decide whether to use extensions.
* Investigating threading technologies and determine whether each is functionally
  complete to be useful to the project.
* Gain confidence and expertise surrounding the cons found for the above investigated
  technologies. I.e. discerning the difference between that which is actually
  an issue (has no known solution) from that where a known solution exists but
  is just not yet known to us.

For example, with OpenMP, the only current hindrance is the perspective amongst
the Astropy developer community that the build integration of said library is
non-trivial enough that it is cause for concern. It is however possible that
the astropy-helpers PRs, listed above, have solved these build issues.
In which case, nothing actually needs to be implemented other than confidence
that this is actually true. However, if it is not true, it is expected that any
further code implemented to concern only the robustness of the build rather than
use of the library itself.


Backward compatibility
----------------------

N/A

At present, no part of Astropy is explicitly threaded at the package code
base level.

Alternatives
------------

Alternatives to hardware acceleration, such as parallel processing are those, e.g., involving
algorithm and data structure design. It is typically understood that this is significantly
limited, in which case hardware acceleration is relied upon, in conjunction.
Therefore, the alternative to not utilising parallel processing is to produce an
unnecessarily under performant software product.

Alternatives to CPU threading are the other forms of parallel processing listed above.

Alternatives in respect to how Astropy adopts CPU threading are numerous.

WIP: Add lists & descriptions of current technology products, here or above, wherever is
nominally expected.


Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
