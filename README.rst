APEs: Astropy Proposals for Enhancement
---------------------------------------

APEs are documents to address non-trivial enhancements that require discussion
and thought beyond a single Pull Request. This is intended to mirror the
long-standing Python Enhancement  Proposal process, but generally not quite as
formally. Normally a proposal goes through various phases of consideration.
Discussion is expected to take place using existing mechanisms (astropy-dev,
github, hangouts, etc), and eventually a decision is made regarding whether the
proposal should be accepted, rejected, or modified.

Accepted APEs
^^^^^^^^^^^^^

=== =================================================== =========== ============
#     Title                                             Date        DOI
=== =================================================== =========== ============
1   `APE Purpose and Process`_                          2013-Nov-08 |APE 1 DOI|
2   `Astropy Release Cycle and Version Numbering`_      2013-Dec-11 |APE 2 DOI|
3   `Configuration`_                                    2013-Dec-10 |APE 3 DOI|
4   `Astropy Setup Helpers`_                            2014-Jun-28 |APE 4 DOI|
5   `Coordinates Subpackage Plan`_                      2014-Jan-22 |APE 5 DOI|
6   `Enhanced Character Separated Values table format`_ 2015-Jan-26 |APE 6 DOI|
7   `NDData Plan`_                                      2014-Dec-17 |APE 7 DOI|
8   `Astropy Community Code of Conduct`_                2015-May-04 |APE 8 DOI|
10  `Roadmap for Python 3-only support`_                2016-Aug-22 |APE 10 DOI|
12  `Using Cookiecutter for the package-template`_      2017-Mar-28 |APE 12 DOI|
=== =================================================== =========== ============

.. _APE Purpose and Process: https://github.com/astropy/astropy-APEs/blob/master/APE1.rst
.. _Astropy Release Cycle and Version Numbering: https://github.com/astropy/astropy-APEs/blob/master/APE2.rst
.. _Configuration: https://github.com/astropy/astropy-APEs/blob/master/APE3.rst
.. _Astropy Setup Helpers: https://github.com/astropy/astropy-APEs/blob/master/APE4.rst
.. _Coordinates Subpackage Plan: https://github.com/astropy/astropy-APEs/blob/master/APE5.rst
.. _Enhanced Character Separated Values table format: https://github.com/astropy/astropy-APEs/blob/master/APE6.rst
.. _NDData Plan: https://github.com/astropy/astropy-APEs/blob/master/APE7.rst
.. _Astropy Community Code of Conduct: https://github.com/astropy/astropy-APEs/blob/master/APE8.rst
.. _Roadmap for Python 3-only support: https://github.com/astropy/astropy-APEs/blob/master/APE10.rst
.. _Using Cookiecutter for the package-template: https://github.com/astropy/astropy-APEs/blob/master/APE12.rst

.. |APE 1 DOI| image:: https://zenodo.org/badge/DOI/10.5281/zenodo.1043886.svg
   :target: https://doi.org/10.5281/zenodo.1043886

.. |APE 2 DOI| image:: https://zenodo.org/badge/DOI/10.5281/zenodo.1043888.svg
   :target: https://doi.org/10.5281/zenodo.1043888

.. |APE 3 DOI| image:: https://zenodo.org/badge/DOI/10.5281/zenodo.1043890.svg
   :target: https://doi.org/10.5281/zenodo.1043890

.. |APE 4 DOI| image:: https://zenodo.org/badge/DOI/10.5281/zenodo.1043892.svg
   :target: https://doi.org/10.5281/zenodo.1043892

.. |APE 5 DOI| image:: https://zenodo.org/badge/DOI/10.5281/zenodo.1043897.svg
   :target: https://doi.org/10.5281/zenodo.1043897

.. |APE 6 DOI| image:: https://zenodo.org/badge/DOI/10.5281/zenodo.1043901.svg
   :target: https://doi.org/10.5281/zenodo.1043901

.. |APE 7 DOI| image:: https://zenodo.org/badge/DOI/10.5281/zenodo.1043907.svg
   :target: https://doi.org/10.5281/zenodo.1043907

.. |APE 8 DOI| image:: https://zenodo.org/badge/DOI/10.5281/zenodo.1043913.svg
   :target: https://doi.org/10.5281/zenodo.1043913

.. |APE 10 DOI| image:: https://zenodo.org/badge/DOI/10.5281/zenodo.1038587.svg
   :target: https://doi.org/10.5281/zenodo.1038587

.. |APE 12 DOI| image:: https://zenodo.org/badge/DOI/10.5281/zenodo.1044484.svg
   :target: https://doi.org/10.5281/zenodo.1044484

Proposing a new APE
^^^^^^^^^^^^^^^^^^^

New APEs should be created using the ``APEtemplate.rst`` file in this repository.
Just fork the repository, copy ``APEtemplate.rst`` to ``APE#.rst`` and issue a
PR with that file once you've written it up.  Be sure to look through the PRs in
this repo first so that you choose an APE# that reflects both those that are in
the repository *and* those that are under discussion.

Note that there is not much point to making proposals unless someone or some
group has signed up to implement it if the APE is accepted
(typically this would involve the author or authors of the APE).  Just issuing
an APE in order to spur others to do work is not generally going to be received
well. Generally, an implementation is expected before an APE can be considered
fully accepted. For proposals that require extensive work that few are willing
to perform without some assurance it will be accepted, we can issue provisional
acceptance. For serious consideration it is usually good to show that detailed
technical aspects have been played with in real code rather (even if it isn't a
complete implementation).

Accepting APEs
^^^^^^^^^^^^^^

The final decision on accepting or rejecting APEs lies with the Astropy
Coordination Committee.  Once the community discussion on the APE has wound
down, the committee discusses the APE and makes a final decision on acceptance
or rejection.  One of the committee members should then:

1. Fill in the "Decision rationale" section of the APE with a description of why
   the APE was accepted or rejected, including a summary of the community's
   discussion as relevant.
2. Update the "date-last-revised" to the day of merging and "status" to
   "Accepted" or "Rejected".
3. If the APE is accepted, add a commit to the APE which puts the APE into the
   "Accepted APEs" table of the repository's README.
4. Merge the PR with the above changes.
5. Upload the APE to Zenodo to give it a DOI (see next section).
6. Send an email to astropy-dev announcing the acceptance (In general this
   should just point to the accepted APE rather than providing additional
   decision rationale).

Uploading an APE to Zenodo
^^^^^^^^^^^^^^^^^^^^^^^^^^

Go to https://zenodo.org/deposit/new, upload the .rst file, and set the fields to the following:

============================= ======================================================
Zenodo field                  Set to
============================= ======================================================
Title                         Astropy Proposal for Enhancement <number>: <title> (APE <number>)
Upload type                   Publication
Publication type              Technical note
Abstract                      The APE abstract
Publication Date              The date-last-revised of the APE (which should be the same as the accepted date for new APEs)
Authors                       The APE authors (directly from the APE text, but with ORCIDs if possible)
License                       CC-Attribution
Communities                   The Astropy Project
Related/alternate identifiers Github link to the APE file *at the specific merge commit* (e.g. https://github.com/astropy/astropy-APEs/blob/42951733ac42c0ea178d8df30705274a43c93091/APE1.rst) as "is supplemented by this upload". If this is a revised version, this should be the URL of the commit where the APE was revised.
============================= ======================================================

Updating APEs
^^^^^^^^^^^^^

In the cases where an updated APE requires updating (e.g. references to a  new
APE that supercedes it, clarifying information that emerges after the APE is
accepted, etc.), changes can be made directly via PR, but the
"date-last-revised" should be updated in the APE. Additionally, the Zenodo entry
will need to be updated with a new version of the APE (*not* a completely new
Zenodo entry), by using the "New version" button and then filling out the forms
as described above.
