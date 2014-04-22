Configuration
-------------

author: Michael Droettboom

date: 2013-12-10

type: Standard Track

status: Discussion

Abstract
--------

We need a new approach to configuration that makes it easier to modify
and extend configuration over time, plays well with other tools in the
Python ecosystem, and most importantly, encourages the reproducibility
of science results.

Detailed description
--------------------

Toward a configuration policy
`````````````````````````````

The most important goal of the configuration system should be the
reproducibility of scientific results.  Some of the existing
configuration parameters relate to peculiarities of platforms and
environments (let's call this "platform configuration"), and others
relate to actual scientific data processing (let's call this "science
state").  This APE argues that the existing configuration system is
well-suited to platform configuration (providing that some modest
enhancements are made), whereas scientific state should be handled in
more directly in code in a way that keeps those settings closer to a
given application and not in user- or machine-global settings.

For example, ``utils.console.use_color`` configures whether ANSI color
codes will be output to the terminal.  This might be changed to
``False`` if the user's system does not support them, but that would
have no bearing on the output of science processing.  In contrast,
``cosmology.core.default_cosmology`` changes the default cosmology to
use, which could put sources in different locations, changing the
ultimate science result.

Global state of any kind (not just in a config file) that relates to
scientific results should be discouraged wherever possible, but there
are cases where it is just too convenient.  An example might be the
list of URLs to use in a ``vo`` query (the configuration item
``vo.validator.validate.cs_urls``).  Another example of global state
that doesn't currently use the configuration system is the set of
active units (``astropy.units.set_enabled_units``).  These sorts of
things should be settable from Python code, and ideally have context
managers, but should not use the configuration system, since storing
them in user- and machine-global files hinders reproducibility.  We
should instead encourage the use of Python code, and the various
facilities that Python has to reuse code, rather than config files.

Current system
``````````````

The current configuration system stores uses the ``ConfigObj`` library
to store configuration in a file in the user's home directory
(``~/.astropy`` or ``~/.config/astropy``, depending on platform and
configuration).  The configuration items are defined within each
subpackage using declarations that look like::

    SESAME_URL = ConfigurationItem("sesame_url",
                        ["http://cdsweb.u-strasbg.fr/cgi-bin/nph-sesame/",
                         "http://vizier.cfa.harvard.edu/viz-bin/nph-sesame/"],
                        "The URL to Sesame's web-queryable database.",
                        cfgtype='string_list')

At build time, all of these configuration items are collated into a
template config file that is copied to the user's home directory (if
it doesn't already exist) upon first import of astropy.

Configuration items may be set and saved to disk directly from
Python::

    In [1]: from astropy.coordinates import name_resolve
    In [2]: name_resolve.NAME_RESOLVE_TIMEOUT.set(42)
    In [3]: name_resolve.NAME_RESOLVE_TIMEOUT.save()

The config file may also be edited by hand.  It provides a convenient
summary and description of each of the available options.

There are some things about this system that would be nice to retain:

    - It's easy to save values to the file from Python.

    - It's easy to edit the config file by hand.

    - It can be loaded very early on during the import of astropy so
      it can affect things, such as logging, that are not very
      dynamic.

However, there are number of shortcomings of this approach:

    - The entire astropy package must be imported to determine what
      the config items are, and generate the template configuration
      file.  This is slow, and can place unnecessary restrictions on
      the code in order to make it importable at build time.

    - The template config file is copied to the user's home directory
      with all values uncommented.  This means we don't know which
      values the user has explicitly changed, and can't change default
      values in future versions of astropy.

    - We don't know the version of astropy that the config file was
      created with, so we can't update it when astropy is updated.

Unrelated to the system itself, but to how it's been used thus far, it
is difficult to find the configuration items using introspection in
Python, since many are defined rather "deep" within the subpackages.
For example, we have ``cosmology.core.DEFAULT_COSMOLOGY`` instead of
``cosmology.DEFAULT_COSMOLOGY``.  All of these configuration items
should be moved to the ``config`` namespace within each subpackage.
To use this example, ``cosmology.config.DEFAULT_COSMOLOGY``.

Branches and pull requests
--------------------------

#2094

Implementation
--------------

The following improvements to the base configuration system will be
made:

    - Since, as a result of this APE, the number of configuration
      options will be much smaller, we will just include a template
      config file directly in the source code repository, install that
      alongside the source code, and copy that into the user's
      configuration directory upon first import (see below for the
      details of how and when it is copied to the user's config
      directory).  Obviously, it will require some care to keep it in
      sync with the declaration of the configuration options in the
      source code.  However, it is the simplest possible thing that
      can work and resolves the issues with generating the config file
      template at build time.

    - The template config file will have all key/value pairs commented
      out by default.  This way, default values can be changed in
      astropy and will only be overridden by the config file if the
      user explicitly does so.

    - Store a version string in the config file.  This, at a later
      date, but not as part of this APE, may allow for graceful
      automatic upgrades of the config file.  This version should
      correspond to the release version of astropy, and not include a
      githash.

    - Upgrading of the config file will roughly follow the `Debian
      configuration file guidelines
      <http://raphaelhertzog.com/2010/09/21/debian-conffile-configuration-file-managed-by-dpkg/>`__,
      which strikes a good balance between safety and simplicity.  It
      prevents the user's config file changes from being accidentally
      overwritten, but doesn't try to be too clever about
      automatically updating the content.  In short:

      - If there is no user config file, copy the one from astropy's
        current version.

      - If there is a user config file, and it matches *exactly* the
        config file template from any previous version of astropy,
        overwrite it with the one from astropy's current version.
        This implies that hashes of previously released configuration
        templates must be stored somewhere.  A simplification to that
        might be to just determine if everything in the config file is
        commented out, and if so, overwrite the file.  (This will not
        work for an astropy 0.3.x config file where the values were
        not commented out, but should work going forward.)

      - If the user config file is different from the config file
        template of a previous astropy version, don't touch it.
        Install alongside it ``astropy.cfg.ver`` where ``ver`` is the
        current version.  Optionally, install a
        ``astropy.cfg.ver.diff`` which is a diff of the user's config
        file and the current config file template.  Display a warning
        that the config file has changed and the user may want to
        manually resolve the differences between their file and the
        new one.  This warning should only be displayed once (when
        ``astropy.cfg.ver`` doesn't already exist) so that user's who
        frequently switch between versions of astropy are not
        bombarded with warnings.

Once doing that, each existing configuration item will be determined
to be either "platform" or "science".  (That will be done in this APE
in a subsequent revision).

For "platform" configuration items:

    - Include the item within the new config file template in the
      source repository.

    - Move the configuration item to the ``subpackage.config``
      namespace (which may be a class instance of some sort or a real
      module, TBD).

    - For backward compatibility, keep special delegation objects that
      delegate from the existing location to the new location and
      raise deprecation warnings when used.

    - The configuration items may still be set by their old keys in
      the config file for one major release cycle, but a deprecation
      warning will be shown.

    - The configuration item should be documented in the subpackages
      documentation in a standardized section ("Configuration")

For "science" configuration items:

    - Define a standard Python context manager for setting the global
      state associated with each configuration item.  For example,
      this should work::

          from astropy import cosmology
          with cosmology.set_default_cosmology('WMAP9'):
              # do something

          # This also works, but doesn't automatically "reset" itself
          # at the end of the block
          cosmology.set_default_cosmology('WMAP9')

    - These context managers will be documented in the API section of
      the subpackage in the standard way along with the rest of the
      API.

    - Retain special delegation objects at the existing location of
      the configuration items that call these new Python context
      managers.  These will raise deprecation warnings describing how
      to update code.

    - When these configuration items are found in the config file,
      deprecation warnings will be shown, but only if they are
      different from the defaults as specified in astropy 0.3.  Doing
      this without checking against the defaults would give everyone a
      warning, since all users currently have an astropy 0.3 config
      file with all values set.

To support the new way of dealing with scientific configuration, ways
of conveniently running Python code at the start of every script
should be documented.  This should include, in order of increasing
"broadness":

    - Making state changes at the top of your script.

    - Having a Python module that all your scripts explicitly import.

    - Using IPython's "profiles"

    - Using Python's "sitecustomize" (though this would be the least
      desirable, as it has many of the reproducibility problems that
      plague the current configuration system).

For a subsequent release, we will remove all of the deprecated
backward-compatibility delegation objects.

A set of guidelines about the difference between "platform
configuration" and "science state" will be added to the relevant
developer documentation.

As an optional follow-on to all of the above, a
``astropy.reset_science_defaults()`` function may be added, that will
reset all of the science state to their defaults.  If all of the
science state context managers inherit from the same base class,
presumably providing that should be fairly automatic and
straightforward.

Backward compatibility
----------------------

The delegation objects should retain backward compatibility for at
least one release.

Alternatives
------------

In an earlier draft of this proposal, I proposed that we throw out the
existing configuration system altogether.  By doing so, however, we
would lose the ability to easily update and save values to the file
from Python.  We also would require users to use IPython profiles
(which are a fairly advanced feature) just to do basic things.  It
also makes it harder to convey to the user which things are
recommended for user- or machine-wide configuration and which things
may lead to scripts and applications not being portable.  In the
present proposal it's obvious: the config file is for user-global
stuff; everything else is just Python code.

The first draft of this APE surmised an elaborate automatic
configuration file update system.  This draft proposes much like
package upgrading on Debian, which preserves the state of a
user-modified configuration file, while otherwise staying out of the
way and providing manual intervention to upgrade the config file.
That should probably be "good enough" and is far simpler, and
presumable less prone to errors or surprising bugs.

Decision rationale
------------------

<To be filled in when the APE is accepted or rejected>
