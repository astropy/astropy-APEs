# APEs: Astropy Proposals for Enhancement

APEs are documents to address non-trivial enhancements that require discussion
and thought beyond a single Pull Request. This is intended to mirror the
long-standing Python Enhancement  Proposal process, but generally not quite as
formally. Normally a proposal goes through various phases of consideration.
Discussion is expected to take place using existing mechanisms (astropy-dev,
github, hangouts, etc), and eventually a decision is made regarding whether the
proposal should be accepted, rejected, or modified.

## Accepted APEs

| # | Title | Date |
| :--- | :--- | :--- |
| 1   | [APE Purpose and Process](APE1.rst)                          | 2013-Nov-08 |
| 2   | [Astropy Release Cycle and Version Numbering](APE2.rst)      | 2013-Dec-11 |
| 3   | [Configuration](APE3.rst)                                    | 2013-Dec-10 |
| 4   | [Astropy Setup Helpers](APE4.rst)                            | 2014-Jun-28 |
| 5   | [Coordinates Subpackage Plan](APE5.rst)                      | 2014-Jan-22 |
| 6   | [Enhanced Character Separated Values table format](APE6.rst) | 2015-Jan-26 |
| 7   | [NDData Plan](APE7.rst)                                      | 2014-Dec-17 |
| 8   | [Astropy Community Code of Conduct](APE8.rst)                | 2015-May-04 |
| 10  | [Roadmap for Python 3-only support](APE10.rst)               | 2016-Aug-22 |

## Proposing a new APE

New APEs should be created using the `APEtemplate.rst` file in this repository.
Just fork the repository, copy `APEtemplate.rst` to `APE#.rst` and issue a
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

## Accepting APEs

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
