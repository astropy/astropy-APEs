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
Python ecosystem, and most importantly, encourages the reproducibility of
science results.

Detailed description
--------------------

Toward a configuration policy
`````````````````````````````

The most important goal of the configuration system should be the
reproducibility of scientific results.  It's important to note that of
the existing configuration parameters, some relate to peculiarities of
platforms and environments (let's call this "platform configuration"),
and others relate to actual scientific data processing (let's call
this "science state").  This APE argues that the existing
configuration system is well-suited to platform configuration
(providing that some modest enhancements are made), whereas scientific
state should be handled in a more directly in code in a way that keeps
those settings closer to a given application and not in user- or
machine-global settings.

For example, `utils.console.use_color` configures whether ANSI color
codes will be output to the terminal.  This might be changed to
`False` if the user's system did not support them, but that would have
no bearing on the output of science processing.  In contrast,
`cosmology.core.default_cosmology` changes the default cosmology to
use.

Global state that relates to scientific results should be discouraged
wherever possible, but there are cases where it is just too
convenient.  An example might be the list of URLs to use in a `vo`
query (the configuration item `vo.validator.validate.cs_urls`).
Another example of global state that doesn't currently use the
configuration system is the set of active units
(`astropy.units.set_enabled_units`).  These sorts of things should be
settable from Python code, and ideally have context managers, but
should not use the configuration system, since storing them in user-
and machine-global files reduces reproducibility.  We should instead
encourage the use of Python code, and the various facilities that
Python has to reuse code, rather than config files.

Current system
``````````````

The current configuration system stores uses the `ConfigObj` library
to store configuration in a file in the user's home directory
(`~/.astropy` or `~/.config/astropy`, depending on platform and
configuration).  The configuration items are defined within each
subpackage using declarations that look like::

    SESAME_URL = ConfigurationItem("sesame_url",
                        ["http://cdsweb.u-strasbg.fr/cgi-bin/nph-sesame/",
                         "http://vizier.cfa.harvard.edu/viz-bin/nph-sesame/"],
                        "The URL to Sesame's web-queryable database.",
                        cfgtype='string_list')

At build time, all of these configuration items are collated into a
template configuration file that is copied to the user's home
directory (if it doesn't already exist) upon first import of astropy.

Configuration items may be set and saved to disk directly from
Python::

    In [1]: from astropy.coordinates import name_resolve

    In [2]: name_resolve.NAME_RESOLVE_TIMEOUT.set(42)

    In [3]: name_resolve.NAME_RESOLVE_TIMEOUT.save()

The configuration file may also be edited by hand.  It provides a
convenient summary and description of each the available options.

There are some things about this system that would be nice to retain:

    - It's easy to save values to the file from Python.

    - It's easy to edit the configuration file by hand.

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
is confusing to find the configuration items from Python, since many
are defined rather "deep" within the subpackages.  For example, we
have `cosmology.core.DEFAULT_COSMOLOGY` instead of
`cosmology.DEFAULT_COSMOLOGY`.  All of these configuration items
should be moved to the `config` namespace within each subpackage.

Branches and pull requests
--------------------------

N/A

Implementation
--------------

The following improvements to the base configuration system will be made:

    - Each `astropy` package will declare its configuration options in
      a declarative file (probably a `ConfigObj` spec file, if that is
      sufficient), at the root of each package.  This, rather than
      importing Python code itself, will be used to build the template
      configuration file.

    - [optional]: Rather than building the template file at build
      time, we could collate it from the declarative files at import
      time, and only if the config file doesn't exist in the user's
      home directory.  This would reduce the amount of code that needs
      to run during the build process in `astropy_helpers`.

    - The template configuration file will have all key/value pairs
      commented out by default.  This way, default values can be
      changed in astropy and will only be overridden by the config
      file if the user explicitly does so.

    - Store a version in the config file.  We can then write functions
      that will upgrade the config file from a previous version of
      astropy to a newer one.  When an old version of a config file is
      encountered, a warning is displayed.  It is then a manual
      process (`astropy.update_config_file()`), which would save a
      backup copy of the current config file and perform any upgrades
      on it in place.  For the case of upgrading from astropy 0.3 to
      0.4, if no version is found in the config file, it will be
      assumed to be a 0.3 format config file, and upgrading it will
      comment out all of the key/value pairs that were not changed
      from the 0.3 defaults, and remove any items that were declare
      "science" configuration.

Once doing that, each existing configuration item will be determined
to be either "platform" or "science".  (That will be done in this APE
in a subsequent revision).

For "platform" configuration items:

    - Define the item within the new declarative file.

    - This will automatically make it available from Python in the
      `subpackage.config` namespace (which will be a class instance of
      some sort).

    - For backward compatibility, keep special delegation objects that
      delegate from the existing location to the new location and
      raise deprecation warnings when used.

    - The configuration item should be documented in the subpackages
      documentation in a standardized section ("Configuration")

For "science" configuration items:

    - Define a standard Python context manager for setting the global
      state associated with each configuration item.  For example, this
      should work::

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
should be documented.  This should include:

    - Making state changes at the top of your script.

    - Having a Python module that all your scripts explicitly import.

    - Using IPython's "profiles"

    - Using Python's "sitecustomize" (though this would be the least
      desirable, as it has many of the reproducibility problems that
      plague the current configuration system).

For a subsequent release, we will remove all of the deprecated
backward-compatibility delegation objects.  The warnings about
existing contents of the configuration file will probably need to be
retained indefinitely.

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

Decision rationale
------------------

<To be filled in when the APE is accepted or rejected>
