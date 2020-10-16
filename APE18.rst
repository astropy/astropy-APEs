Adopt NEP 29 for CPython and Numpy Version Support
--------------------------------------------------

author: Stuart Mumford and Thomas Robitaille

date-created: 2020 October 16

date-last-revised: 2020 October 16

type: Standard Track

status: Discussion


Abstract
--------

APE 10 previously determined supported CPython versions, and provided a list up to the 4.0 release.
Since APE 10 was written the wider scientific Python community have written and largely adopted `NEP 29`_ as a policy for supported versions of both CPython and Numpy.
This APE proposes that Astropy adopt this policy following other projects in the scientific ecosystem.


Detailed description
--------------------

This APE fully adopts `NEP 29`_ for both CPython and Numpy versions, it supersedes APE 10 wherever there is a conflict between that APE and this APE or `NEP 29`_.

While this APE will not replicate the whole of `NEP 29`_, and `NEP 29`_ should be considered the canonical source for the support policy, the core scheme to be adopted is as follows:

The core astropy package will support:

* All minor versions of Python released 42 months prior to the project, and at minimum the two latest minor versions.
* All minor versions of numpy released in the 24 months prior to the project, and at minimum the last three minor versions.


Backward compatibility
----------------------

This version scheme is more rapid than that proposed in APE 10, however, this allows the core library to adopt modern features, and reduces burden on maintainers.
In addition to this aligning ourselves with other projects in the ecosystem is consistent and prevents potential minimum version conflicts.
See the text of `NEP 29`_ for further justification.

Alternatives
------------

See `NEP 29`_.

Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>

.. _NEP 29: https://numpy.org/neps/nep-0029-deprecation_policy.html
