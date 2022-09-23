.. _Black: https://github.com/psf/black
.. _Django DEP-0008: https://github.com/django/deps/blob/main/final/0008-black.rst
.. _pandas pre-commit: https://pandas.pydata.org/docs/development/contributing_codebase.html#pre-commit
.. _coding style: https://docs.astropy.org/en/latest/development/codeguide.html#coding-style-conventions
.. _PEP8: https://peps.python.org/pep-0008/
.. |Black| replace:: *Black*

Formatting Code with *Black*
============================

authors: Tom Aldcroft, Erik Tollerud

date-created: 2022 July 13

date-last-revised: 2022 September 23

date-accepted: 2022 September 23

type: Standard Track

status: Accepted


Abstract
--------

This APE proposes that the `astropy core package
<https://github.com/astropy/astropy>`_ adopt |Black|_ for formatting code.

Detailed description
--------------------

Foreword
^^^^^^^^^

This APE presents a rationale that code formatting with |Black|_ can be beneficial
both for the ``astropy`` code development process and for the community.
Hereafter we refer to the astropy core package as ``astropy``, noting that it is
distinct from the Astropy project as a whole and coordinated and affiliated
Astropy packages. While coordinated or affiliated packages may choose to adopt
this APE's conventions in the future, this APE is concerned solely with the core
package itself.

|Black|_ is a fast and reliable Python automated code formatter supported by the
Python Software Foundation. The project motto "The uncompromising code
formatter" reflects that the code is strongly opinionated about formatting and
there are very few configuration options. From their documentation, "By using
*Black*, you agree to cede control over minutiae of hand-formatting. In return,
*Black* gives you speed, determinism, and freedom from ``pycodestyle`` nagging
about formatting."

Code formatting is controversial and the `astropy-dev thread`_ about *Black*
generated a spirited discussion but did not result in a clear consensus. This
strong division of opinions about automated code formatting is common and has
been documented in other projects, for instance in `Django DEP-0008`_.

Other projects, including NumPy and SciPy, have considered using *Black* for
formatting (see e.g. `SciPy PR #14330
<https://github.com/scipy/scipy/pull/14330>`_). Neither of those projects felt
that auto-formatting with the current version of *Black* (circa 2022) was a good
choice for their codebase, with particular emphasis on needing improvements
in formatting of mathematical expressions.

We do not expect unanimous agreement on this APE but do hope to demonstrate the
value of *Black* for the collective needs of the community and Astropy Project.

.. _astropy-dev thread: https://groups.google.com/g/astropy-dev/c/6cRJCMgaFyM/

Note that ideas and some text for this APE have been adapted from a similar
enhancement proposal for Django `Django DEP-0008`_.

Rationale
^^^^^^^^^

**Consistent and readable code formatting** reduces effort in both code review
and in future maintenance. Code is read far more often than it is written, so it
makes sense to invest effort to ensure that the code is easy to read later.

Ensuring that code formatting meets ``astropy`` standards is currently done by
applying CI checks based on :pep:`8`. Learning and consistently applying all
these rules is not easy. There are at least 200 commits in ``astropy`` which
were solely address PEP8 violations, so this is a real issue.

Because of the need to follow these format guidelines, ``astropy`` developers
must pay close attention to formatting, constantly making small decisions about
code formatting based on rules, on taste, or on a mix of both. While some of
:pep:`8` is focused on guidelines like naming conventions that cannot be
automated, much of it is essentially algorithmic and hence these decisions are a
needless burden on the developers.

With **automated code formatting**, the workload of code formatting is done by
computers, saving human bandwidth for higher value and complexity activities
like the logic or interface design. Collectively, contributors no longer need to
think about code formatting when writing or in code reviews. By ceding control
to |Black|_ they can focus on the content of the code and not worry about details
like where to break a line or how to arrange a list.  Whatever the formatter
does is the expected result, by definition.

Automated code formatting will increase consistency across the ``astropy`` code
base. Currently the style of each subpackage reflects the preferences of the
maintainers. For instance, the ``time``, ``table``, and ``io.ascii`` subpackages
allow line lengths of up to 100, while most others use 80 or 88. There are other
more subtle differences and astropy contributors find this confusing. This has
come up in the past.

While there are multiple code formatters for Python, the *Black* Python code
formatter produces good enough results that it is getting significant traction
in many open source projects, including: scikit-learn, pytest, tox, Pyramid,
Django, Hypothesis, SQLAlchemy, virtualenv, pandas, Pillow, and Twisted. While
adoption by this list of high-profile community-based open source projects does
not automatically imply that it is a good fit for ``astropy``, it demonstrates
unequivocally several points:

- Technical issues related to the initial transition and subsequent maintenance of
  *Black* formatting are surmountable. There is plenty of prior art and good
  documentation.
- Sociological issues related to developer preferences are surmountable.
  Evidence from other projects is that developers either embrace or
  at least accept autoformatting.
- Developer documentation can provide clear instructions to contributors on how
  to implement *Black* formatting in their development process. Issues with this
  process are no more difficult (and arguably easier) to resolve than enforcing
  PEP-8 standards.

Therefore, this APE proposes to adopt *Black* for ``astropy``.

Concerns and mitigations
^^^^^^^^^^^^^^^^^^^^^^^^

Foolish consistency
~~~~~~~~~~~~~~~~~~~
A concern that has been highlighted by some members of the community (including,
initially, one of the co-authors of this APE, and highlighed in detail in `this
blog post
<https://luminousmen.com/post/my-unpopular-opinion-about-black-code-formatter>`_)
is that adopting *Black* violates the PEP8 principle that "A Foolish Consistency
is the Hobgoblin of Little Minds". I.e., that by using a code formatter, the
developer thinks less about readability because they think they can let the
formatter do this for them. While this concern has some validity, this APE
explicitly recognizes that **developers and code reviewers are ultimately
responsible for maintaining a readable code base**. That is, there are many
choices concerning logic and readability of code that *Black* does *not* affect,
and those choices are more important for developers to focus on than mechanistic
decisions that *Black* makes about whitespace and newlines.

Undesirable formatting
~~~~~~~~~~~~~~~~~~~~~~
On occasion *Black*'s formatting produces undesirable results. The code may look
very different from expectation due to mathematical or other domain-specific
conventions, or some specific formatting that conveys meaning may be lost. A
simple example of this is a constant identity matrix which would typically be
written as::

    matrix = [[1, 0, 0],
              [0, 1, 0],
              [0, 0, 1]]

*Black* reformats this as shown below in a way that is very different from the
typical representation for a matrix and makes it harder to read the structure::

    matrix = [[1, 0, 0], [0, 1, 0], [0, 0, 1]]

The mitigation here is the exception policy:  In this case the author could
surround the matrix with the ``#fmt: off``/``#fmt: on`` escape hatch, as this
clearly makes the code more readable.  While many examples are less clear than
the above, and indeed in most cases "readability" is a subjective concept, this
mechanism allows for such cases to be explicitly excepted where the developer
thinks the improved readability warrants it.

History
~~~~~~~
This changes almost every Python file and a large fraction of code lines (for
example about one third of non-blank lines in the ``astropy.time`` subpackage).
This is more than previous bulk changes and it is an impact that needs to be
accepted with this APE.

Code correctness
~~~~~~~~~~~~~~~~
*Black* guarantees semantic equivalence of the formatted code, and will not
(`except in a few limited cases
<https://black.readthedocs.io/en/stable/the_black_code_style/current_style.html#ast-before-and-after-formatting>`_)
change the code AST.

Migration effort
~~~~~~~~~~~~~~~~
Enthusiastic champions of the effort will largely do the work, but we recognize
and appreciate that additional effort will be required from maintainers that are
neutral or against the adoption of *Black*.

Backports / release branches
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This is addressed in the Implementation section.

New contributors
~~~~~~~~~~~~~~~~
New contributors may be more discouraged by more steps. However, this *replaces*
the existing PEP8 check, so that it is not "another step". To make it better,
leave very clear explicit messages in the status checks on GitHub to reduce the
barrier. Scikit-learn had a few issues with contributors but not at a level that
caused regret about the decision to adopt *Black* formatting. One developer
described the change as a "net benefit".

Removing the humanity
~~~~~~~~~~~~~~~~~~~~~
Some developers may feel that individual style and humanity gets lost in
autoformatting. We hope to convey that humanity and individuality is expressed
more fully in the elegance and clarity of the code logic and implementation. In
most cases the placement of white space and line breaks can be viewed as a
mechanistic process that humans need not control.

Exception Policy
^^^^^^^^^^^^^^^^
As described above, there are cases *Black* formatting makes the code
substantially more difficult to read or understand. These should be treated
explicitly as exceptions, using the ``#fmt: off``/``#fmt: on`` escape hatch for
multi-line exceptions, or ``#fmt: skip`` for single-line exceptions. **This is
allowed if the code author and PR reviewer(s) agree that the code line or block
should be excluded from *Black* formatting.**


Branches and pull requests
--------------------------

- `Add configuration for black autoformatter <https://github.com/astropy/astropy/pull/13253>`_
- `Apply black to modeling <https://github.com/astropy/astropy/pull/13254>`_

Implementation
--------------

All Python code in ``astropy`` is formatted with |Black|_, using its default
settings, that is, 88 characters per line and double quotes. Note that, by
design, most of |Black|_ is inflexible, and there is not a way to pick-and-choose
some elements and not others of *Black*'s format.

Implementing this change requires:

1. Updating the `coding style`_:

   * Adding documentation about *Black*, with a particular emphasis on the
     rationale expressed here and the exception policy described above.
   * Document the *Black* exception guidelines (as described in this APE)
   * Removing other references to code formatting in the documentation.

2. Updating flake8 configuration to be compatible with *Black*.

   This is straightforward and well documented. 

3. Checking *Black* in CI.

   * Add a CI check that runs *Black* to ensure consistency, failing if it meets a violation.

4. Reformatting Astropy's code.  This will be done on a subpackage-by-subpackage
   basis over a transition period. The process for each subpackage to be
   transitioned is as follows:

   * The sub package maintainer(s) create a branch (e.g.
     ``blackify-coordinates``) to do the re-formatting.
   * Edit ``pyproject.toml`` to remove the *Black* exclusion for the sub-package.
   * Run ``black --skip-string-normalization astropy/<sub-package>`` on the
     sub-package directory.  Commit the changes *Black* makes and make a draft
     pull request.
   * The maintainer(s) should review all changes and highlight for discussion
     any  changes that are judged to be valid exceptions (see above for what
     "valid exception" means). These can be reverted by copying-and-pasting
     from the "changes" interfaces to the local copy and applying the
     appropriate *Black* directives to prevent auto-formatting. Commit and push
     these changes for review.
   * Once the PR submitter and reviewer are both satisfied with the changes
     then squash the commit(s) down to a single commit and force push.
   * Run ``black astropy/<sub-package>`` to apply just the string normalization
     stage and commit and push.
   * Edit .git-blame-ignore-revs to include the commits comprising PR to this
     point.
   * Convert the draft PR to a normal PR (ready for review). Once it passes CI
     then it can be merged without further review.

A sub-package will not have *Black* checked in CI until it has completed these
steps.

Backward compatibility
----------------------
This APE would supersede any previous code formatting styles recommended for the
core package. In practice this has little impact because the "standard" style
has grown organically and is not applied consistently beyond the PEP8 checks
(which *Black* is a superset of).

Alternatives
------------

Other automated code formatting tools exist for Python code. The main players
are ``autopep8`` and ``yapf``.  There are many good posts on this topic -
entering "black vs autopep8 vs yapf" into your favorite search engine will
likely get you several good options. To establish the goal of consistency
without endless debate, however, *Black* is the clear winner: Many other Python
projects have adopted it. It is officially supported by the Python Software
Foundation. The algorithm *Black* uses is generally more performant than the other
two on large code bases like ``astropy``.

Decision rationale
------------------

Although opinions were fairly divided on the initial `astropy-dev thread`_
discussing of adopting *Black*, the discussion has converged in favor of the
implementation described here. In the cases where *Black* formatting will hurt
readability, those situations will  be handled on a case-by-case basis to
exclude those lines from auto-formatting.

It was initially unclear whether this proposal was also about the adoption of
pre-commit hooks in astropy. Those hooks were adopted by the project prior to
this APE, and while *Black* would be added to the pre-commit hooks in CI, the
use of hooks is independent of this APE.

The majority of the community who reviewed the PR supports auto-formatting
code in general. There were mixed feeling as to whether *Black*’s formatting
choices were optimal, and more broadly has some drawbacks. However, no better
alternative was identified.

Important to the decision to adopt *Black* over other auto-formatters  is its
guarantee not to alter the generated bytecode. There is not developer
capacity to handle breakage caused by autoformatting. *Black*’s speed on a
large code base and its lack of configurability were also elements in its
favor.

Based on the broad agreement in the discussion on the APE pull request, the
CoCo has approved the APE. The community was well informed of this APE and
discussion via several mechanisms, including the mailing list and the
developer telecons. The CoCo thinks that the discussion in this APE includes
enough voices to be representative of the broader developer community.
