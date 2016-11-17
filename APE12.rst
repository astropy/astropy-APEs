Using Cookiecutter for the package-template
-------------------------------------------

author: Stuart Mumford, Thomas Robitaille

date-created: 2016 November 17

date-last-revised: 2016 November 17

type: Standard Track

status: Discussion


Abstract
--------

This APE proposes migrating the `package-template
<https://github.com/astropy/package-template/>`_ to use the `Cookiecutter
<http://cookiecutter.readthedocs.io/>`_ project to provide templating facilities
to the package template to ease use of the package template and to make it easy
for other projects to customise the template for their users.


Detailed description
--------------------

Cookiecutter is a Python project that uses the Jinja2 templating engine to
define project templates for both Python and non-Python projects. When run it
presents the user with a simple set of questions to populate predefined
variables which are then used to render the Jinja2 templates and create the
project. This workflow is a massive improvement on the manual workflow currently
recommended to affiliated package authors in the
`Astropy documentation <http://docs.astropy.org/en/latest/development/affiliated-packages.html>`_.

The second advantage to using a templating system like Jinja2 is that it makes
it possible for other projects (*e.g.* SunPy) to maintain a fork of the package
template with defaults that specific to that project. Without having to make
extensive changes to the actual package template, thereby making it easy to pull
in changes from the Astropy template without lots of complex merging.

There are also secondary advantages to using a Cookiecutter project, such as
being able to include code, such as tests, in the ``package-template``
repository while not including it in the code that the person setting up a new
project receives. This is because only the templated directory in the repository
gets processed by Cookiecutter.


Branches and pull requests
--------------------------

A version of the package template converted to a Cookiecutter project is `here
<https://github.com/Cadair/package-template/tree/cookiecutter>`_. It can be
tested by installing the cookiecutter package from pip or conda-forge and
running::

  cookiecuttter -c cookiecutter gh:cadair/package-template


Implementation
--------------

The changes to the package template are to move the template into a subdirectory
and to change all occurrences of ``packagename`` and other user modifiable
variables to Jinja template placeholders such as ``{{ cookiecutter.package_name }}``.

Backward compatibility
----------------------

As implemented in
`Cadair/package-template <https://github.com/Cadair/package-template/tree/cookiecutter>`_
users who have followed the instructions where they use the package template as
a git remote will no longer be able to pull updates from the
``package_template`` git repository. This could be mitigated by automatically
generating a rendered version of the package template in another git repository
or in a different branch.

Alternatives
------------

Cookiecutter is not the only project templating system, others such as
`diecutter <https://diecutter.readthedocs.io/>`_ could be used. Cookiecutter was
chosen due to wide adoption and implementation in Python.

Decision rationale
------------------

