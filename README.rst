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

=== =================================================== ===========
#     Title                                             Date
=== =================================================== ===========
1   `APE Purpose and Process`_                          2013-Nov-08
2   `Astropy Release Cycle and Version Numbering`_      2013-Dec-11
3   `Configuration`_                                    2013-Dec-10
4   `Astropy Setup Helpers`_                            2014-Jun-28
5   `Coordinates Subpackage Plan`_                      2014-Jan-22
6   `Enhanced Character Separated Values table format`_ 2015-Jan-26
7   `NDData Plan`_                                      2014-Dec-17
8   `Astropy Community Code of Conduct`_                2015-May-04
10  `Roadmap for Python 3-only support`_                2016-Aug-22
=== =================================================== ===========

.. _APE Purpose and Process: https://github.com/astropy/astropy-APEs/blob/master/APE1.rst
.. _Astropy Release Cycle and Version Numbering: https://github.com/astropy/astropy-APEs/blob/master/APE2.rst
.. _Configuration: https://github.com/astropy/astropy-APEs/blob/master/APE3.rst
.. _Astropy Setup Helpers: https://github.com/astropy/astropy-APEs/blob/master/APE4.rst
.. _Coordinates Subpackage Plan: https://github.com/astropy/astropy-APEs/blob/master/APE5.rst
.. _Enhanced Character Separated Values table format: https://github.com/astropy/astropy-APEs/blob/master/APE6.rst
.. _NDData Plan: https://github.com/astropy/astropy-APEs/blob/master/APE7.rst
.. _Astropy Community Code of Conduct: https://github.com/astropy/astropy-APEs/blob/master/APE8.rst
.. _Roadmap for Python 3-only support: https://github.com/astropy/astropy-APEs/blob/master/APE10.rst

Proposing a new APE
^^^^^^^^^^^^^^^^^^^

It is important to note is that there is not much point to making proposals
unless someone or some group has signed up to implement it if it is accepted
(normally this would involve the author or authors of the APE).  Just issuing
an APE in order to spur others to do work is not generally going to be received
well. Generally, an implementation is expected before an APE can be considered
fully accepted. For proposals that require extensive work that few are willing
to perform without some assurance it will be accepted, we can issue provisional
acceptance. For serious consideration it is usually good to show that detailed
technical aspects have been played with in real code rather (even if it isn't a
complete implementation).

New APEs should be created using the ``APEtemplate.rst`` file in this repository.
Just fork the repository, copy ``APEtemplate.rst`` to ``APE#.rst`` and issue a
PR with that file once you've written it up.  Be sure to look through the PRs in
this repo first so that you choose an APE# that reflects both those that are in
the repository *and* those that are under discussion.
