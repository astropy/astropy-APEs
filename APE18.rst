Adopt NEP 29 for CPython and Numpy Version Support
--------------------------------------------------

author: Stuart Mumford and Thomas Robitaille

date-created: 2020 October 16

date-last-revised: 2020 October 16

type: Standard Track

status: Discussion


Abstract
--------

After the release of astropy 4.0 APE 10 no longer specifies the supported versions of CPython.
Since APE 10 was written the wider scientific Python community have written and largely adopted `NEP 29<https://numpy.org/neps/nep-0029-deprecation_policy.html>`__ as a policy for supported versions of both CPython and Numpy.
This APE proposes that Astropy follow the projects who have already adopted this policy.


Detailed description
--------------------

This APE fully adopts NEP 29 for both CPython and Numpy versions, it supersedes APE 10 wherever there is a conflict between that APE and this APE or NEP 29.

While this APE will not replicate the whole of NEP 29, and NEP 29 should be considered the canonical source for the support policy the core policy to be adopted is as follows:

astropy will support:

* All minor versions of Python released 42 months prior to the project, and at minimum the two latest minor versions.
* All minor versions of numpy released in the 24 months prior to the project, and at minimum the last three minor versions.

This means that for the next few releases of astropy the following versions of CPython and Numpy will be supported:


+--------------------+--------------------+-----------------+--------------------+
| Astropy Release    | Release Date       | CPython Version | Numpy Version      |
+--------------------+--------------------+-----------------+--------------------+
| 4.2                | November 2020      | 3.7+            | 1.16+              |
+--------------------+--------------------+-----------------+--------------------+
| 5.0                | April 2021         | 3.7+            | 1.17+              |
+--------------------+--------------------+-----------------+--------------------+
| 5.1                | November 2021      | 3.7+            | 1.18+              |
+--------------------+--------------------+-----------------+--------------------+
| 5.2                | April 2022         | 3.8+            | 1.19+              |
+--------------------+--------------------+-----------------+--------------------+
| 6.0                | November 2022      | 3.8+            | 1.20+              |
+--------------------+--------------------+-----------------+--------------------+
| 6.1                | April 2023         | 3.9+            | 1.20+              |
+--------------------+--------------------+-----------------+--------------------+



Backward compatibility
----------------------

This version scheme is more rapid than that proposed in APE 10, however, this allows the core library to adopt modern features, and reduces burden on maintainers.
See the text of NEP 29 for further justification.

Alternatives
------------

See NEP 29.

Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
