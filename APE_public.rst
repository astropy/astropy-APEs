
Astropy's API Policy on Modules and their Contents
--------------------------------------------------

author: Nathaniel Starkman

date-created: 2013 November 5 <replace with the date you submit the APE>

date-last-revised: 2013 December 17 <keep this up to date anytime something
changes>

date-accepted: 2014 January 20 <replace with accepted date>

type: <one of these three: Standard Track, Informational, Process>

status: Discussion


Abstract
--------

This APE establishes Astropy's policy for public versus internal modules and
their contents.


Scope
-----

A library's API is the publicly accessible "surface": the set of modules,
classes, functions, and other symbols provided to the library's users. This APE
is narrowly focused on which modules are public and how to communicate which
contents of those modules are public. This APE does not address any other
questions of API design, e.g. public versus internal methods in classes, leaving
these for future APEs or future revisions to this APE.


.. _description_of_the_problem:

Description of the Problem
--------------------------

Pure Python has no language-level means to establish and enforce separations
between public versus internal interfaces -- all code objects can be accessed.
This is in contrast to languages like C++ or Java, which have language-level
constructs to establish public versus private interfaces. Differentiating
between public and internal interfaces is important for everyone, users and
developers alike, to know what interfaces are stable and supported, and which
are not. Consequently, Python has developed conventions for communicating what
is public versus internal. Nominally there are three means (discussed more
formally later):

1. **Underscore Prefix**.  By convention, names that begin with a single underscore
   are considered internal.
2. ``__all__``.  The dunder attribute ``__all__`` of a module is a list of
   names that are considered public.
3. **Documentation**.  Public interfaces are documented and internal interfaces
   are not, or are specifically noted.

These 3 means can be ambiguous and contradictory. Let's explore this point with
an example. Consider an example library with the following layout::

   src/
      __init__.py::
         __all__ = ["foo", "_bar"]

      foo.py::
         __all__ = ["_func"]
         def func():
            pass

         def _func():
            pass
      
      _bar.py::
         __all__ = ['bunc']
         def bunc():
            pass

         def _bunc():
            pass

      baz.py::
         __all__ = ['qux']

         def qux():
            pass

- Is the ``foo`` module public? Probably, but what if it's not documented?
- Is ``foo.func`` public? It is not in ``foo.__all__``. 
- Is ``foo._func`` public?  It is prefixed with an underscore, but in ``foo.__all__``. 
- Is the ``_bar`` module public? It is prefixed with an underscore, but in ``src.__all__``.
- Is ``_bar.bunc`` public? It is in ``_bar.__all__``, but what if ``_bar`` is private?
- Is ``_bar._bunc`` public? Probably not, but what if it's documented?
- Is ``baz`` public? It does not start with an underscore, but is not in ``src.__all__``.
- Is ``baz.qux`` public? It does not start with an underscore and is in
  ``baz.__all__``, but what if ``baz`` is private?


Astropy partially implements all three of the conventions: some private classes
and functions are prefixed with an underscore, while some are not; some modules
have an ``__all__`` attribute, and some do not;  some (debatably) internal
interfaces are in the public documentation, like ``astropy.utils``, and some are
not. However, Astropy does not have a uniform approach to differentiating and
communicating public vs internal symbols, which leads to confusion for users and
developers alike. This longstanding issue has steadily become more pressing with
the development of assistive code tools, like auto-fill in IDEs or more recently
with generative coding tools, like GitHub CoPilot, which suggest code to users
based on a corpus of code examples.  When users unintentionally use internal
code, code tools will then suggest that internal code to other users. If we do
not have a uniform and systematic approach to communicating what is public vs
internal, then we cannot expect users and especially their tools to know what is
public vs internal.

The ambiguity between public and internal interfaces also creates problems for
developers. When private APIs are not clearly distinguished from public ones,
they get used downstream -- as was common with NumPy -- thereby becoming
effectively frozen even though they were never intended to be stable. This
happens because:

1. Users (knowingly or unknowingly) start depending on private APIs.
2. Maintainers become hesitant to refactor or improve private code for fear of
   breaking downstream users.
3. The cost of cleanup increases over time as more code depends on the
   quasi-public private APIs.
4. The boundary between public and private becomes meaningless.

This creates bad incentives: the behavior is individually rational ("don't break
anyone today") but collectively harmful ("downstream increasingly relies on what
should never have been relied on"). The whole point of having private APIs is
precisely so that they can be changed as needed. When private APIs are
confusingly public, improving them becomes tortuous for developers, and the
library suffers as a result.


Solution Criteria
-----------------

Having established that we need a uniform and systematic approach to public vs
internal interfaces, we now decide what that approach should be. The approach
should fulfill a few criteria:

1. Align closely with established conventions, so that it is familiar
   to users and developers, and hopefully detected by assistive tools.
2. Be unambiguous and easy to understand.
3. Be straightforward to implement and maintain, so that it is not a
   burden on developers.

We start by considering established practice, for criterion 1, and how to ensure
that we satisfy the latter two criteria and can be implemented in a uniform and
systematic way. There are three important documents that describe established
practice as it relates to Astropy:

1. `PEP 8 <https://peps.python.org/pep-0008/#public-and-internal-interfaces>`_ :
   the Python style guide.
2. `SciPy
   <https://docs.scipy.org/doc/scipy/reference/index.html#importing-from-scipy>`_:
   the largest scientific Python project.
3. `Python's typing guide
   <https://github.com/python/typing/blob/master/docs/source/libraries.rst#library-interface-public-and-private-symbols>`_:
   the official guide for how to statically analyze Python code, which is important
   for assistive tools.

Starting with `PEP 8
<https://peps.python.org/pep-0008/#public-and-internal-interfaces>`_, it states
that::

   Documented interfaces are considered public, unless the documentation
   explicitly declares them to be provisional or internal interfaces exempt
   from the usual backwards compatibility guarantees. All undocumented
   interfaces should be assumed to be internal.

   To better support introspection, modules should explicitly declare the names
   in their public API using the ``__all__`` attribute. Setting ``__all__`` to
   an empty list indicates that the module has no public API.

   Even with ``__all__`` set appropriately, internal interfaces (packages,
   modules, classes, functions, attributes or other names) should still be
   prefixed with a single leading underscore.

   An interface is also considered internal if any containing namespace
   (package, module or class) is considered internal.

   Imported names should always be considered an implementation detail. Other
   modules must not rely on indirect access to such imported names unless they
   are an explicitly documented part of the containing module's API, such as
   ``os.path`` or a package's ``__init__`` module that exposes functionality from
   submodules.


This is a more detailed and nuanced description of the three means mentioned in
:ref:`description_of_the_problem`.  It is also the most authoritative source, as
it is the official Python style guide.  However, it is not as unambiguous as we
require: which takes precedence, the ``__all__`` attribute or the underscore
prefix?

`SciPy
<https://docs.scipy.org/doc/scipy/reference/index.html#importing-from-scipy>`_
tries to resolve this ambiguity by stating that ::

   - Methods / functions / classes and module attributes whose names begin with
     a leading underscore are private.
   - If a class name begins with a leading underscore, none of its members are
     public, whether or not they begin with a leading underscore.
   - If a module name in a package begins with a leading underscore none of its
     members are public, whether or not they begin with a leading underscore.
   - If a module or package defines ``__all__``, that authoritatively defines
     the public interface.
   - If a module or package doesn't define ``__all__``, then all names that
     don't start with a leading underscore are public.

This is a good solution, and is consistent with PEP 8, but for our goal of
complete unambiguity and uniformity it falls short on two counts: first, it
does not mention the documentation; second, PEP 8 recommends "modules should
explicitly declare the names in their public API using the ``__all__``
attribute. Setting ``__all__`` to an empty list indicates that the module has no
public API." SciPy allows modules to lack an ``__all__`` attribute, meaning a
user and their tools must understand the nuances of the previous rules. Having
an ``__all__`` attribute in every module is simpler, clearer, and better for
introspection by both users and automated systems.

Finally, let's consider what `Python typing guide
<https://github.com/python/typing/blob/master/docs/source/libraries.rst#library-interface-public-and-private-symbols>`
adds.:: 

   - Symbols whose names begin with an underscore (but are not dunder names) are
     considered private.
   
   - A module can expose an ``__all__`` symbol at the module level that provides
     a list of names that are considered part of the interface. This overrides
     all other rules above, allowing imported symbols or symbols whose names
     begin with an underscore to be included in the interface.

This is consistent with PEP 8 and SciPy, but it does not mention the
documentation. The Python typing guide reinforces the idea that the ``__all__``
attribute is the authoritative source of what is public vs internal.
Importantly, the ``__all__`` attribute is a module-level attribute, so it only
applies to the module in which it is defined. We will refer to this as "locally"
public. This means that a module can define an ``__all__`` attribute but if the
module itself is not public, then anything in the ``__all__`` attribute cannot
be publicly accessed from outside the module.


Solution
--------

We propose the following:

1. That Astropy adopts the PEP 8 rules on public versus internal interfaces
2. That Astropy disambiguate these rules similarly to SciPy.
3. In addition, that Astropy ensures its documentation is consistent with its
   code, the latter being the authoritative source.

**Rules for Public Interfaces:**

1. A symbol is public if and only if all containing namespaces are public.
   A symbol is internal if any containing namespace is internal.

   For example, `astropy/path/to/code` is public if and only if all `astropy`,
   `astropy/path`, `astropy/path/to`, and `astropy/path/to/code` are all public.
   If any of these namespaces is internal, then `astropy/path/to/code` is
   internal.

2. All modules must have an ``__all__`` attribute, even if it is empty. The
   ``__all__`` attribute specifies the public and internal interface of the
   module in which it is defined. Anything in ``__all__`` is *locally* public,
   including underscore-prefixed symbols. Anything not in ``__all__`` is
   internal in that module. The exception to this rule are modules that cannot
   have an ``__all__`` attribute, for example implicit namespace packages, which
   lack an ``__init__.py``. In these cases, the public interface is defined by
   the sub-rules:

   - A symbol is *locally* public if it does not start with an underscore, and
     internal otherwise, unless it is a dunder symbol (``__<...>__``).
   - Dunder symbols (``__<...>__``) have their own public vs internal rules,
     which are not determined in this APE.

3. Public symbols must not be prefixed with an underscore.

   - Internal symbols need not be prefixed with an underscore, but it is often
     recommended. An example of when it is not needed are symbols defined in
     internal modules, making the prefix redundant.

4. Public symbols must be documented.

   - If an internal symbol is documented (which is not recommended), it must be
     unambiguously, explicitly, (and preferably repeatedly) noted as internal.


Let's consider an example::

   src/
      __init__.py::
         __all__ = ["foo", "spam"]

      foo.py::
         __all__ = ["func"]
         def func(): pass
         def _func(): pass
      
      _bar.py::
         __all__ = ['bunc']
         def bunc(): pass
         def _bunc(): pass

      baz.py::
         __all__ = ['qux']
         def qux(): pass

      spam/  # implicit namespace package

         ham.py::
            __all__ = ['eggs']
            def eggs(): pass

In this case, ``func`` is public in ``foo``, and  ``foo`` is public in ``src``,
so ``src.foo.func`` is public. In contrast, while ``bunc`` is public in
``_bar``, ``src._bar`` is not public, so ``src._bar.bunc`` is not public either.
This is ''locally'' public, as in internal. Both ``_func`` and ``_bunc`` are
internal.  Note that ``src.baz`` is internal because it is not in
``src.__all__``, even though it is not prefixed with an underscore. This follows
the strict rules for public interfaces established above, but as is also noted,
it is a very bad idea to name an easily accessed module in such a way that it
appears public but is not.

=======  =========  ==========  ==========  =========================  =========
Symbol                                      Status                             Good Idea
=======  =========  ==========  ==========  =========================  =========
``src``                                     public                     ✅
         ``.foo``                           public                     ✅
                    ``.func``               public                     ✅
                    ``._func``              internal                   ✅
         ``._bar``                          public                     ✅
                    ``.bunc``               internal (locally public)  ✅
                    ``._bunc``              internal                   ✅
         ``.baz``                           internal                   ❌
                    ``.qux``                internal (locally public)  ✅
         ``spam``                           public                     ✅
                    ``.ham``                public                     ✅
                                ``.eggs``   public                     ✅
=======  =========  ==========  ==========  =========================  =========


Branches and pull requests
--------------------------

N/A


Implementation
--------------

The process of adopting this APE will change Astropy's API. This APE formally
establishes Astropy's API, so what is meant by "change"?  There is already a *de
facto* API, which is what users *think* is the API.  This is determined
primarily by the documentation, but also by tab-completion within an interactive
session.  In the implementation of this APE to establish a *formal* API, we aim
to minimize changes to the *de facto* API. This will be accomplished in the
following 5 phases:

1. **Snapshot the documentation**. As of the adoption of the APE a snapshot of
   the documentation will be saved. This will be used to determine what is
   currently public and what is not. On a per-sub-package basis, until the APE's
   adoption is complete, this snapshot is authoritative, e.g. dictating what
   must be added and removed from ``__all__``, for deciding what must undergo a
   deprecation process, etc.

2. **Add / update** ``__all__``. The ``__all__`` in each module will be updated
   to reflect phase 1. Any modules' missing ``__all__`` will have one added.

3. **Update the documentation** and **implement deprecations**.

   - Make sure every public symbol is in the documentation, unless it is being
     deprecated.
   - Deprecate any public symbol that is not intended to be public, but was made
     public as part of steps 1 and 2.
   - Add a SciPy-like section to the developer documentation explaining the
     public vs internal rules.
   - Fix links to always point to the public interface, not the internal
     interface. (This will presumably require a similar implementation as used
     in NumPy to change the ``__module__`` attribute of many objects.)
   - Add a reminder to the maintainer checklist that to be public a symbol must be
     in ``__all__`` and documented.
   - Clearly state if a documented object is actually private.

4. **Add a pre-commit hook**. This will ensure that the ``__all__`` attribute is
   always present and up-to-date. It will also ensure that the documentation is
   always up-to-date with the ``__all__`` attribute. This can be accomplished
   with a pre-commit hook that runs a script that checks the ``__all__``
   attribute and, if the module is public, searches in the ``docs/api``
   directory for symbols in ``__all__``.

5. **Add prefixes**: Add prefixes to the top-most private symbols. For modules
   this makes all their contents private.

In Astropy core each phase will be implemented on a per-sub-package basis with
individual pull requests.


Implementation Examples
-----------------------

1. Astropy's cosmology sub-package

The ``astropy.cosmology`` sub-package serves as an example implementation of
this APE. In ``cosmology``, all internal code was moved into a ``_src`` folder,
making everything within it private. For backwards-compatibility, the old public
API was then exposed through backwards-compatibility modules that use
``__getattr__`` to import and deprecate the contents of ``_src``. For example,
``flrw.py`` was moved to ``_src/flrw.py``, and a new ``flrw.py`` was created
with a ``__getattr__`` that imports and deprecates the contents of
``_src/flrw.py``. This deprecation process was completed and the
backwards-compatibility modules removed after a full year for version 8.0, with
no user complaints. The result is a clear and robust separation between the
public API and the private implementation (anything in ``_src``).

Other sub-packages may prefer to underscore-prefix individual modules rather
than use the ``_src/module`` pattern. Both approaches provide clear
communication to users and developers about what is public and what is internal.

2. NumPy

NumPy has adopted a similar approach to this proposal, by moving most of their
internal code into a ``_core`` folder.

3. SciPy

SciPy has adopted a similar approach to this proposal, by moving most of their
internal code into underscore-prefixed modules and packages.
For example, see the `scipy.fft` package.


Teaching This
-------------

This APE requires very little teaching. For users, the rules are simple:

1. Check the tree of ``__all__`` to see if a symbol is public (Rules 1 and 2). This is the normative definition of what is public.
2. Everything conforms to normal expectations, e.g. public modules don't start
   with underscores (Rule 3).
3. The documentation follows the code (Rule 4).

So in practice users can just tab-complete in an IDE or check the documentation
to see what is public.

For most developers using modern code tools, this APE requires no teaching at
all. Code tools will automatically pick up public vs private API.

For developers using code tools that don't autocomplete underscore-prefixed
files (typically only old editors) but want this functionality, there is an easy
solution: create a symlink from the private file to a public name and then
git-exclude it. For example, create ``cosmology/src`` as a symlink to
``cosmology/_src``. What's nice about this solution is that it's permanent -- do
it once and forget about it.


Backward compatibility
----------------------

This APE breaks backward compatibility in two ways:

1. Changes ``__all__`` in many modules. Many modules define an ``__all__`` but
   include symbols that are intended to be private because they should be
   imported from other modules (generally the top of the module). This will not
   break code that directly imports the symbols (as Python does not use
   ``__all__`` for this purpose), but it will break code that expects private
   symbols and uses ``import *``.
2. Adds prefixes to many symbols that are not public. This can be done in a
   backward compatible way by adding a ``__getattr__`` method to the module that
   raises a warning for any object that is not public. This will allow existing
   code to continue to work, while encouraging people to fix their code to use
   the public interface.


Alternatives
------------

**We do nothing:**

This is the status quo.  It is not a good option because it does not solve the
issue.  The aforementioned problems of not knowing what is stable and
supported, and what is not, remain.

**We allow** ``__all__`` **to be optional:**

This is not great.

The only time this might be good is when a module has dynamic symbols from a
`PEP 562 <https://peps.python.org/pep-0562/>`_ module-level ``__getattr__``
method. However, if it is truly dynamic then it cannot be statically analyzed, which
is undesirable for other reasons. If it is not truly dynamic, then the "dynamic"
symbols can be added to ``__all__`` and an ``__init__.pyi`` file used to
communicate the public interface to static analyzers.

**We make the documentation the authoritative definition of the API.**

Good code is well documented in the code. Beyond code comments and docstrings, what is
public versus internal is an important aspect of the code, and should be
communicated in the code itself. It is vitally important that the code and
user-facing documentation are consistent, and the user-facing documentation
should reflect the code.

**We make tab-completion the authoritative definition of the API.**

This is not a good option because it does not work with implicit namespace
packages nor `PEP 562 <https://peps.python.org/pep-0562/>`_ module-level
``__getattr__`` methods.


Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or
rejected>
