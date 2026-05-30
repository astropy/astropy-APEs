Using Cookiecutter for the package-template
-------------------------------------------

author: Stuart Mumford, Thomas Robitaille

date-created: 2016 November 17

date-last-revised: 2017 March 28

date-accepted: 2017 March 28

type: Standard Track

status: Accepted

Abstract
--------

This APE proposes migrating the `package-template
<https://github.com/astropy/package-template/>`_ to use the `Cookiecutter
<https://cookiecutter.readthedocs.io/>`_ project to provide templating. This
would enable users to be able to customise the package template for their
package from the command line, in a much simpler manner than the current manual
process. As well as this, having a templating system makes it easier for other
projects to change the branding or default settings of the package template.

This APE proposes major changes to the current layout of the package-template
repository, potentially breaking existing user workflows, hence the APE.


Detailed description
--------------------

Cookiecutter is a Python project that uses the Jinja2 templating engine to
define project templates for both Python and non-Python projects. When run it
presents the user with a simple set of questions to populate predefined
variables which are then used to render the Jinja2 templates and create the
project. This workflow is a massive improvement on the manual workflow currently
recommended to affiliated package authors in the
`Astropy documentation <https://docs.astropy.org/en/latest/development/affiliated-packages.html>`_.

The second advantage to using a templating system like Jinja2 is that it makes
it possible for other projects (*e.g.* SunPy) to maintain a fork of the package
template with defaults that specific to that project. Without having to make
extensive changes to the actual package template, thereby making it easy to pull
in changes from the Astropy template without lots of complex merging.

There are also secondary advantages to using a Cookiecutter project, such as
being able to include code i.e. tests, in the `package-template
<https://github.com/astropy/package-template/>`_ repository while not including
it in the code that the person setting up a new project receives. This is
because only the templated directory in the repository gets processed by
Cookiecutter.

An example of using cookiecutter with the package template is below. This
example uses the `Cadair/pacakage-template <https://github.com/Cadair/package-template/tree/cookiecutter>`_
``cookiecutter`` branch, if this work was to be merged the command would be
``cookiecutter gh:astropy/package-template`` as the ``-c`` option specifies a
non-default branch.

.. image:: https://asciinema.org/a/7ilbwr10g0rj561clfkedui4j.png
    :target: https://asciinema.org/a/7ilbwr10g0rj561clfkedui4j


.. code-block::

    ❯ cookiecutter gh:astropy/package-template
    Cloning into 'package-template'...
    remote: Counting objects: 952, done.
    remote: Compressing objects: 100% (65/65), done.
    remote: Total 952 (delta 38), reused 0 (delta 0), pack-reused 880
    Receiving objects: 100% (952/952), 258.38 KiB | 0 bytes/s, done.
    Resolving deltas: 100% (505/505), done.
    Branch cookiecutter set up to track remote branch cookiecutter from origin.
    Switched to a new branch 'cookiecutter'

    package_name [packagename]: CadairPy

    module_name [cadairpy]:

    short_description [CadairPy]: CadairPy is code written about a mountain.

    long_description []: Cadair Idris or Cader Idris is a mountain in Gwynedd, Wales, which lies at the southern end of the Snowdonia National Park near the town of Dolgellau. The peak, which is one of the most popular in Wales for walkers and hikers, is composed largely of Ordovician igneous rocks, with classic glacial erosion features such as cwms, moraines, striated rocks, and roches moutonnées.

    author_name [Astropy Developers]: Stuart Mumford

    author_email []: stuart@cadair.com

    Select license:
    1 - BSD 3-Clause
    2 - GNU GPL v3+
    3 - Apache Software Licence 2.0
    4 - BSD 2-Clause
    5 - Other
    Choose from 1, 2, 3, 4, 5 [1]:

    project_url [https://astropy.org]: https://cadair.com

    project_version [0.0.dev]: 0.0.1.dev

    include_example_code [y]:

    include_cextern_folder [n]:

    edit_on_github_extension [False]: True

    github_project [astropy/astropy]: cadair/cadairpy

    use_travis_ci [y]:

    use_appveyor_ci [y]:

    use_read_the_docs [y]:

    sphinx_theme [astropy-boostrap]:

    ❯ ls -la
    total 116K
    drwxr-xr-x  5 stuart users 4.0K Nov 17 14:41 ./
    drwx------ 69 stuart users 4.0K Nov 17 14:41 ../
    -rw-r--r--  1 stuart users  36K Nov 17 14:41 ah_bootstrap.py
    -rw-r--r--  1 stuart users 1.8K Nov 17 14:41 appveyor.yml
    drwxr-xr-x  6 stuart users 4.0K Nov 17 14:41 cadairpy/
    drwxr-xr-x  4 stuart users 4.0K Nov 17 14:41 docs/
    -rw-r--r--  1 stuart users  13K Nov 17 14:41 ez_setup.py
    -rw-r--r--  1 stuart users  655 Nov 17 14:41 .gitignore
    -rw-r--r--  1 stuart users  108 Nov 17 14:41 .gitmodules
    drwxr-xr-x  2 stuart users 4.0K Nov 17 14:41 licenses/
    -rw-r--r--  1 stuart users 1.1K Nov 17 14:41 MANIFEST.in
    -rw-r--r--  1 stuart users  742 Nov 17 14:41 README.md
    -rw-r--r--  1 stuart users   71 Nov 17 14:41 readthedocs.yml
    -rw-r--r--  1 stuart users  160 Nov 17 14:41 .rtd-environment.yml
    -rw-r--r--  1 stuart users 1.2K Nov 17 14:41 setup.cfg
    -rwxr-xr-x  1 stuart users 4.0K Nov 17 14:41 setup.py*
    -rw-r--r--  1 stuart users 4.8K Nov 17 14:41 .travis.yml




Branches and pull requests
--------------------------

A version of the package template converted to a Cookiecutter project is `here
<https://github.com/Cadair/package-template/tree/cookiecutter>`_. It can be
tested by installing the cookiecutter package from pip or conda-forge and
running::

  cookiecutter -c cookiecutter gh:cadair/package-template


Implementation
--------------

The changes to the package template are to move the template into a subdirectory
and to change all occurrences of ``packagename`` and other user modifiable
variables to Jinja template placeholders such as ``{{ cookiecutter.package_name }}``.

A draft implementation is `here <https://github.com/Cadair/package-template/tree/cookiecutter>`_. Which is in
`PR #202 <https://github.com/astropy/package-template/pull/202>`_ to the
package-template repository.

As well as the changes to the package template the documentation in the Astropy
docs regarding the package template will be significantly simplified. As a part
of the implementation it also becomes possible to keep the documentation and
instructions for using the package template in the `package-template`
repository. This change will be made with the implementation PR and the Astropy
docs changed once this is done.


Backward compatibility
----------------------

As implemented in `Cadair/package-template
<https://github.com/Cadair/package-template/tree/cookiecutter>`_ users who have
followed the instructions where they use the package template as a git remote
will no longer be able to pull updates from the ``package_template`` git
repository. To mitigate this lack of backwards compatibility a new branch on the
current package-template repository will be created which holds the cookiecutter
template version of the package-template. An automatically rendered (i.e. using
Travis CI) version of the template with the default settings will be kept in the
main branch of the package-template repository. This will result in a main
branch that looks like the current package-template repository. This could then
be used as a git remote for people who wish to continue using this update
method.

In the longer term, using cookiecutter could make the implementation of an
update helper, or automatic update process easier. This is mainly due to the
fact that cookiecutter can be called with the settings used to originally create
the project, i.e the correct name and other options. This will lead to smaller
more relevant diffs when updating and therefore make any automation easier.


Alternatives
------------

Keeping the current solution. This removes any need for changes for anyone using
the 'git update' method at the moment, but it does not provide any of the
templating benefits presented above.

Cookiecutter is not the only project templating system, others such as
`diecutter <https://diecutter.readthedocs.io/>`_ could be used. Cookiecutter was
chosen due to wide adoption and implementation in Python.

Finally, a custom templating system could be developed and maintained for the
Astropy project, however, this would be a lot of effort.


Decision rationale
------------------

There was discussion on the `APE pull request <https://github.com/astropy/astropy-APEs/pull/17>`_ and the `implementation pull request <https://github.com/astropy/package-template/pull/202>`_, but mainly specific details, rather than objections to the general scheme. The discussion resulted in figuring out a way to not break backward-compatibility for people who want to continue using the git remote method for updating their packages. The above content reflects these improvements, and the APE was accepted 2017-03-28, as there were no objections from the community.
