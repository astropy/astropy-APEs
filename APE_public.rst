Astropy's Public API Policy
---------------------------

author: Nathaniel Starkman

date-created: 2013 November 5 <replace with the date you submit the APE>

date-last-revised: 2013 December 17 <keep this up to date anytime something
changes>

date-accepted: 2014 January 20 <replace with accepted date>

type: <one of these three: Standard Track, Informational, Process>

status: Discussion


Abstract
--------

This APE establishes Astropy's policy for communicating what are public versus
internal interfaces.  This policy is implemented in Astropy core and coordinated
packages, and highly recommended -- though not mandatory -- in affiliated
packages.


Detailed description
--------------------

Pure Python has no language-level means to establish and enforce separations
between public versus internal interfaces -- all code objects can be accessed.
This is in contrast to languages like C++ and Java, which have language-level
constructs to establish public versus private interfaces. Differentiating
between public and internal interfaces is important for everyone, users and
developers alike, to know what interfaces are stable and supported, and which
are not. Consequently, Python has developed conventions for communicating what
is public versus internal. Nominally there are three means, (discussed more
formally later):

1. **Underscore Prefix**.  By convention, names that begin with an underscore
   are considered internal.
2. ``__all__``.  The dunder attribute ``__all__`` of a module is a list of
   names that are considered public.
3. **Documentation**.  Public interfaces are documented and internal interfaces
   are not, or are specifically noted.

Astropy partially implements all of these conventions: some private classes and
methods are prefixed with an underscore, while some are not; some modules have
an ``__all__`` attribute, and some do not;  some (debatably) internal interfaces
are in the public documentation, like ``astropy.utils``, and some are not.
However, Astropy does not have a uniform approach to public vs internal, which
leads to confusion for users and developers alike. This longstanding issue has
steadily become more pressing with the development of assistive code tools, like
auto-fill in IDEs or more recently with generative coding tools, like GitHub's
CoPilot, which can suggest code to users.  This can be problematic if that code
is internal and users do not realize. If we do not have a uniform and systematic
approach to communicating what is public vs internal, then we cannot expect
users and especially their tools to know what is public vs internal.

Having established that we need a uniform and systematic approach to public vs
internal interfaces, we now decide what that approach should be. The approach
should fulfill a few criteria:

1. It should be unambiguous and easy to understand.
2. It should be straightforward to implement and maintain, so that it is not a
   burden on developers.
3. It should align closely with established conventions, so that it is familiar
   to users, developers, and hopefully detected by assistive tools.

We start by considering established pactice, for criteria 3, and how to ensure
that it satisfies the first two criteria and can be implemented in a uniform and
systematic way. There are three important documents that describe established
practice as it relates to Astropy:

1. `PEP 8 <https://peps.python.org/pep-0008/#public-and-internal-interfaces>`_ :
   the Python style guide.
2. `scipy
   <https://docs.scipy.org/doc/scipy/reference/index.html#importing-from-scipy>`_:
   the largest scientific Python project.
3. `Python's typing guide
   <https://github.com/python/typing/blob/master/docs/source/libraries.rst#library-interface-public-and-private-symbols>`_:
   the official guide for how to statically analyze Python code, which is import
   for assistive tools.

Starting with PEP 8, it states that ::

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
   are an explicitly documented part of the containing module’s API, such as
   os.path or a package’s __init__ module that exposes functionality from
   submodules.


This is a more detailed and nuanced description of the three means discussed
above.  It is also the most authoritative source, as it is the official Python
style guide.  However, it is not as unambiguous as we require: which takes
precedence, the ``__all__`` attribute or the underscore prefix?

Scipy tries to resolve this ambiguity by stating that ::

   - Methods / functions / classes and module attributes whose names begin with
     a leading underscore are private.
   - If a class name begins with a leading underscore, none of its members are
     public, whether or not they begin with a leading underscore.
   - If a module name in a package begins with a leading underscore none of its
     members are public, whether or not they begin with a leading underscore.
   - If a module or package defines ``__all__``, that authoritatively defines
     the public interface.
   - If a module or package doesn’t define ``__all__``, then all names that
     don’t start with a leading underscore are public.

This is a good solution, and is consistent with PEP 8, but for our goal of
complete unambiguity and uniformity it falls short on two counts:  first, it
does not mention the documentation; second, PEP 8 recommends "modules should
explicitly declare the names in their public API using the ``__all__``
attribute. Setting ``__all__`` to an empty list indicates that the module has no
public API." SciPy allows modules to lack an ``__all__`` attribute, meaning a
user and their tools must understand the nuances of the previous rules. Having
an ``__all__`` attribute in every module is simpler, unambiguous, and better for
introspection by both users and automated systems.

Finally, let's consider what `Python typing guide
<https://github.com/python/typing/blob/master/docs/source/libraries.rst#library-interface-public-and-private-symbols>`
adds. :: 

   - Symbols whose names begin with an underscore (but are not dunder names) are
     considered private.

   - Imported symbols are considered private by default. If they use the
     ``import A as A`` (a redundant module alias), from ``X import A as A``
     (a redundant symbol alias), or ``from . import A`` forms, symbol ``A`` is
     not private unless the name begins with an underscore. If a file
     ``__init__.py`` uses form ``from .A import X``, symbol ``A`` is treated
     likewise. If a wildcard import (of the form ``from X import *``) is used,
     all symbols referenced by the wildcard are not private.
   
   - A module can expose an ``__all__`` symbol at the module level that provides
     a list of names that are considered part of the interface. This overrides
     all other rules above, allowing imported symbols or symbols whose names
     begin with an underscore to be included in the interface.
   
   - Local variables within a function (including nested functions) are always
     considered private.

This is consistent with PEP 8 and SciPy, but it does not mention the
documentation. This typing recommendations reinforces the idea that the
``__all__`` attribute is the authoritative source of what is public vs internal.
Importantly, the ``__all__`` attribute is a module-level attribute, so it only
applies to the module in which it is defined.  This means that a module can
define an ``__all__`` attribute but if the module itself is not public, then
anything in the ``__all__`` attribute cannot be publicly accessed from outside
the module. For example, consider the following module::

   src/
      __init__/
         __all__ = []

      _foo/
         __all__ = ['bar']
         def bar():
            pass

In this case, ``bar`` is public in ``_foo``, but  ``src._foo`` is not public, so
``src._foo.bar`` is not public either.

We propose the following:

1. That Astropy adopts all the PEP 8 rules on public versus internal interfaces
   (the only one I'm not sure about is adding prefixes to private classes and
   functions.)
2. That Astropy disambiguates these rules by adopting the subsequent rules for
   its API:
3. That Astropy ensures its documentation is consistent with its code, the
   latter being the authoritative source.

   - That Astropy explains these rules in its documentation
4. That Astropy also adopts these rules for coordinated packages, and recommends
   that affiliated packages follow these rules as well.

Rules:

1. A symbol is public if all containing namespaces (packages, modules, classes,
   functions, attributes or other names) are public. A symbol is private if any
   containing namespace is private.
2. All modules must have an ``__all__`` attribute, even if it is empty. The
   ``__all__`` attribute defines the public and private interface of the module
   in which it is defined. Anything in ``__all__`` is public, including
   underscore-prefixed symbols. Anything not in ``__all__`` is private in that
   module. The exception to this rule is implicit namespace packages, which
   cannot have a top-level ``__all__`` attribute because they do not have a
   top-level ``__init__.py`` file. In these cases, the public interface is
   defined by the following rule.
3. A symbol is public if it does not start with an underscore, and private
   otherwise. The only exception to this rule is dunder symbols (``__<...>__``),
   which follow their rules not determined in this APE.


Branches and pull requests
--------------------------

N/A


Implementation
--------------

This APE will be implemented in 3 phases:

1. **Add** ``__all__``: An ``__all__`` dunder attribute will be added to all
   modules that do not have it.
2. **Update the documentation** to reflect the new rules.

   - Add a scipy-like section to the developer documentation.
   - Make sure the developer documentation is consistent with the new rules.
   - Fix links to always point to the public interface, not the internal
     interface.
   - Clearly state if a documented object is actually private.
3. **Add prefixes**: 1. Add prefixes to all modules that are not public. 2. Add
   prefixes to all classes, functions, and attributes that are not public.
      :note:`I'm less enthusiastic on this point.`

The 3rd phase will be broken into the two listed parts.


Backward compatibility
----------------------

This APE breaks backward compatibility in two ways:

1. Changes ``__all__`` in many modules. Many modules define an ``__all__`` but
   include symbols that are intended to be private because they should be
   imported from other modules (generally the top of the module). This will not
   break code that directly imports the symbols (as Python does not use
   ``__all__`` for this purpose), but it will break code that uses ``from module
   import *``.
2. Adds prefixes to many objects that are not public. This can be done in a
   backward compatible way by adding a ``__getattr__`` method to the module that
   raises an warning for any object that is not public. This will allow existing
   code to continue to work, because it will force people to fix their code to
   use the public interface.


Alternatives
------------

**We do nothing:**

This is the status quo.  It is not a good option because it is not solve the
issue.  The aforementioned problems of not i) knowing what is stable and
supported, and what is not, remain.

**We allow** ``__all__`` **to be optional:**

Not great.

The only time this might be good is when a mudule has dynamic symbols from a
`PEP 562 <https://peps.python.org/pep-0562/>`_ module-level ``__getattr__``
method. However if it truly dynamic then it cannot be statically analyzed, which
is undesirable for other reasons. We haven't encountered this situation in
Astropy yet, so I don't think it's a good reason to allow ``__all__`` to be
optional.


Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or
rejected>
