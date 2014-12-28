Enhanced Character Separated Values format
------------------------------------------

author: Tom Aldcroft

date-created: 2014 April 12

date-last-revised: 2014 December 28

type: Standard Track

status: Discussion


Abstract
--------

APE6 is primarily a specification of a new standard for the interchange of
tabular data in a text-only format.  The proposed format handles the key issue
of serializing column specifications and table metadata by using a YAML-encoded
data structure.  The actual tabular data are stored in a standard
character separated values (CSV) format, giving compatibility with a wide variety of
non-specialized CSV table readers.

Using YAML makes it extremely easy for applications *and humans* to read both
the standardized data format elements (e.g. column name, type, description) as
well as complex metadata structures.  YAML also lends itself to simple table
modifications by humans in a plain text editor.

The reference Python implementation in ``astropy.io.ascii`` is relatively straightforward and
will provide a significant benefit of allowing text serialization of most astropy
Table objects, persistent storage, and subsequent interchange with other users.

Existing standards
--------------------

Following the lead of the astronomical data community in recently
documenting shortcomings of the FITS standard, we start by discussing
existing widely-used standards and why they are lacking.

CSV
^^^^^

The `Comma-Separated-Values format
<http://en.wikipedia.org/wiki/Comma-separated_values>`_ is probably the most
common text-based method for representing data tables.  The proposed standard in
APE6 leverages this universality by using this format for serializing
the actual data values.

As noted in the Wikipedia article, CSV is not a rigid standard despite some
efforts in that direction.  Even the name "comma-separated" is misleading since
tab-separated or whitespace-separated tabular data generally fall in this
category.  Nevertheless nearly everyone with a computer has access to robust
tools that can read and write CSV format tables, and CSV is a de-facto standard
for importing and exporting tabular data to applications, including most
spreadsheet applications like Excel.

For science applications, pure CSV has a serious shortcoming in the complete
lack of support for table metadata.  This is frequently not a showstopper but
represents a serious limitation which has motivated alternate standards
in the astronomical community.

CDS / ApJ Machine Readable Table
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Centre de Données astronomiques de Strasbourg developed a `Standard for
Documentation of Astronomical Catalogues
<http://vizier.u-strasbg.fr/doc/catstd.htx>`_ which is very widely used.  The
Astrophysical Journal has adopted this standard for formatting `machine readable
tables <http://aas.org/authors/machine-readable-table-standards>`_ associated
with journal articles.  The format is alternately referred to as the CDS or
Machine Readable Table (MRT) format.

The CDS format has reasonable support for basic table and column metadata,
include column types, null values, range checking, units, and labels.  From
its heritage as a catalog descriptor format, it includes table metadata
like ADC_Keywords, Literature Reference, table description, authors, notes,
and comments.  Within that context it provides a well-documented standard.

However, the CDS format does not include support for keyword values like FITS, 
nor arbitrary table metadata.  This limits the fidelity of serialization
for tables that have rich metadata (e.g. WCS).

As a tool for data interchange, CDS is far from ideal.  Despite the name "Machine
Readable Table", the format is quite difficult for machines to read.  This
stems from the format of the `ReadMe file
<http://vizier.u-strasbg.fr/doc/catstd-3.1.htx>`_, which has a 
non-regular format that requires tricky regex parsing and logic to fully interpret.

A few options for `reading MRT files
<https://aas.org/authors/machine-readable-table-programs>`_ are available, but
none of them are full-featured.  [QUESTION: does TOPCAT directly read CDS?  It
isn't explicitly listed as a supported format].  As far as we know the version
in Astropy is the only one being currently maintained.  It supports much of the
standard with regards to the data values and column definitions, but makes no
attempt to parse any of the other table metadata.  Getting an MRT file directly
into Excel is onerous.

We are unaware of any publicly available application that can write in the CDS /
MRT format, and instead authors rely on data translation by online tools or data
service providers.

IPAC
^^^^^

The `IPAC Table Format
<http://irsa.ipac.caltech.edu/applications/DDGEN/Doc/ipac_tbl.html>`_ is the
standard for the NASA/IPAC Infrared Science Archive and is widely used.

This format is more in line with FITS in terms of providing support for
arbitrary keyword / value pairs and comment lines, along with specification of
column names, data types, units, and null values.  The format is succinctly
described in the standard page and is fairly straightforward to understand
and implement.

IPAC has a shortcoming as a generic data interchange format because of the
specific serialization of the column specification.  Note that in the example
below the comments are not part of the actual data file and are included for
illustration purposes only::

  \keyword = value                                                  # Keywords (optional)
  \ Comment                                                         # Comments (optional)
  |  name1    |  name2    | name3   |  name4   |     name5        | # Column Names (required)
  |   double  |   double  |   int   |   real   |     char         | # Data Types (standard)
  |   unit    |   unit    |   unit  |   unit   |     unit         | # Data Units (optional)
  |   null    |   null    |   null  |   null   |     null         | # Null Values (optional)
    165.466279  -34.704730      5      11.27      A string value    # Data Rows (1 required)
    160.1231     -4.7           3       2.3          value          # Data Rows (1 required)

In order to associate the columns names ``name1`` .. ``name5`` with the five
data columns, the reader application has to have specific knowledge of the IPAC
format.  Clearly providing full support for specific typing, units and
null-value handling requires a dedicated reader, but a significant fraction of
data tables do not require these and can make due with just column names and
automatic type inference.  Supporting the common case of falling through to
bare-bones CSV table support has value.

Another issue illustrated above is that IPAC uses fixed-width formatting for the
data values, so that if a generic CSV reader tries to parse this file it will
find an inconsistency in the number of data columns between rows 1 and 2
(because of the spaces in the ``name5`` column values).

Supporting more complex metadata structures would be possible within the IPAC
standard using keyword values, but it would be cumbersome and require inventing
a custom serialization method to work within that confine.

VOTable
^^^^^^^^

`VOTable <http://www.ivoa.net/documents/latest/VOT.html>`_ is by design a
fully-flexible data format that can handle all of the needs for text
serialization of complex data structures, including tablular data sets.  The
issue in this context is in simplicity and data interchange with the broader
community.  In essence if someone wants to read or write a VOTable then they
must use one of a small number of implementations of this protocol.  It is not
possible for someoneto directly read such a table into Excel.  Writing an
implementation of VOTable in a new language (e.g. R, Julia, or Perl) to read/write
VOTable is a major undertaking.

A smaller issue is speed, since the VOTable format requires relatively complex
parsing and validation for text serialization.

Others
^^^^^^^^

We are not aware of other widely-used standards in the astronomical
community for text representation of tabular data.

Tabular Data Package
^^^^^^^^^^^^^^^^^^^^^

Though not widely used in the astronomical community, since initially drafting
this APE we have become aware of a very similar standard known as the
`Tabular Data Package
<http://dataprotocols.org/tabular-data-package/>`_.  This provides a
fully-formed protocol for publishing and sharing tabular-style data
which is conceptually very similar to the proposed ECSV format, with
the exception of using two files, one pure JSON for the header and one
pure CSV for the data.  The JSON header follows a schema defined by
the `JSON Table Schema
<http://dataprotocols.org/json-table-schema/>`_.


After evaluation and discussion with that community, we find that the Tabular
Data Package and associated standards would require a fair degree of
modification to fully suit our needs.  More crucially, the decision to rely on
YAML instead of JSON for serialization precludes direct use of the TDP.


Detailed description
---------------------

The proposed Enhanced CSV (ECSV) format has the following
overall structure:

- A header section which consists of lines that start with the ``#`` character
  and provide the table definition and data format via a YAML-encoded data structure.
- An initial line in the header section which identifies the file as ECSV and
  provides a version number.
- A CSV-formatted data section in which the first line contains the column names
  and subsequent lines contains the data values.

Version 1.0 of the ECSV format specification and the reference Python
implementation assumes ASCII-encoded header and data sections.  Support
for unicode (in particular UTF-8) may be added in subsequent versions.

Why YAML?
^^^^^^^^^^

The fundamental issue which the CDS and IPAC formats try to address is
serializing the information which specifies the table column definitions and
other relevant metadata.  Those formats essentially invent custom serialization
specifications that must be carefully implemented from scratch by any
reader/writer application.

ECSV takes the approach of defining a minimal standard for the the underlying
data structure that is needed to define a table.  Then that structure is
encoded or decoded using YAML.  Libraries for encoding and decoding YAML are
widely used, very efficient, and easily available in all the most-frequently
used programming languages (with the notable exception of IDL).

Translating to / from the data structure provided by a ECSV header into the native
structure that an application uses should generally be quite easy because the
functional elements (e.g. column name, type) are ubiquitous.  Generally
speaking manipulating data structures programmatically is easier than parsing
textual data structure fields.

The ECSV standard does not require that the YAML encoding be "pretty", but it
is highly-recommended that applications format the YAML header to be legible
to humans.  This is important because a key feature of YAML is that it is
meant to be easily readable, and thus modifiable, by humans.

The highly readable nature of YAML is key driver for using this over JSON.
In simple cases the column definitions serialize on a single line which
makes for a compact and useful representation.

Example
^^^^^^^^^^

A couple of quick examples using the reference Python implementation
will put this in context.  First let's create a table and give it some
custom attributes::

  >>> from astropy.table import Table
  >>> t = Table([[1, 4], [2, 3]], names=['a', 'b'])
  >>> t['a'].unit = 'm/s'
  >>> t['a'].format = '%03d'
  >>> t['b'].description = 'This is column b'
  >>> t['b'].unit = 'km'
  >>> print(t)
    a    b 
  m / s  km
  ----- ---
    001   2
    004   3

Now we write this to a file using the ECSV format and print it::

  >>> t.write('example.ecsv', format='ascii.ecsv')
  >>> cat example.ecsv
  # %ECSV 1.0
  # ---
  # columns:
  # - {name: a, unit: m / s, type: int64, format: '%03d'}
  # - {name: b, unit: km, type: int64, description: This is column b}
  a b
  001 2
  004 3

We see that header starts with a header line and YAML block marker to identify
the format and the beginning of the data block.  After that comes the YAML data
structure with column definitions.  Finally the column names and data values
are included in CSV format with a space delimiter.

Now we can read back the table and see that it has survived the round-trip
to a text file::

  >>> t2 = Table.read('example.ecsv', format='ascii.ecsv')
  >>> print(t2)
    a    b 
  m / s  km
  ----- ---
    001   2
    004   3

To illustrate the full features of ECSV we create a table with meta data
(keywords and comments) as well as more detailed column meta data::

  >>> from astropy.utils import OrderedDict
  >>> t = Table([[1, 4], [2, 3]], names=['a', 'b'])
  >>> t.meta['keywords'] = OrderedDict([('z_key1', 'val1'), ('a_key2', 'val2')])
  >>> t.meta['comments'] = ['Comment 1', 'Comment 2', 'Comment 3']
  >>> t['a'].unit = u.m / u.s
  >>> t['a'].format = '%5.2f'
  >>> t['a'].description = 'Column A'
  >>> t['b'].meta = dict(column_meta={'a':1, 'b': 2})

Now we write the table to standard out::

  >>> import sys
  >>> t.write(sys.stdout, format='ascii.ecsv')
  # %ECSV 1.0
  # ---
  # columns:
  # - {name: a, unit: m / s, type: int64, format: '%5.2f', description: Column A}
  # - name: b
  #   type: int64
  #   meta:
  #     column_meta: {a: 1, b: 2}
  # meta: !!omap
  # - keywords: !!omap
  #   - {z_key1: val1}
  #   - {a_key2: val2}
  # - comments: [Comment 1, Comment 2, Comment 3]
  a b
  1.00 2
  4.00 3

In this case there are Ordered dictionary elements which must be preserved
during serialization.  This is done via the ``!!omap`` element tag.  The
reference Python implementation in Astropy uses custom Loader and Dumper
classes to allow serialization to and from the Python ``OrderedDict`` class.
Other implementations must likewise use an ordered mapping when reading and the
``!!omap`` tag when writing for ordered mappings in the data structure.

In addition, the reference Python implementation outputs the column attributes
in the order ``'name'``, ``'unit'``, ``'type'``, ``'format'``,
``'description'``, and ``'meta'``.  This is not a ECSV requirement but is
recommended for human accessibility.

Header details
^^^^^^^^^^^^^^^^

An ECSV file begins with the the table header which contains the
necessary information to define the table columns and metadata.  This
is expressed as a YAML-encoded data structure which has a small set of
required keywords and standard specifiers.

Each line of the YAML-encoded data structure must start with the two
characters ``# `` (hash followed by space) to indicate the presence of
header content.  The first line which does not start with ``#``
signifies the end of the header.  Subsequent lines starting with ``#``
are treated as file comment lines.

Within the header section, lines which start with ``##`` are treated as
comments and can be ignored by readers.  There is no requirement for
ECSV writers to emit such comment data.  Relevant comment strings
should be serialized within the ``meta`` keyword structure.

Beyond the minimal standard, applications are free to
create a custom data structure as needed using the top-level ``meta``
keyword.  The specification of a corresponding ``schema`` keyword to
allow interpretation and validation of the custom data is highly
encouraged.

Standard keywords are:

``columns``: list, required
   List of column specifiers.

``delimiter``: one-character string, optional, default=``space``
   Delimiter character used to separate the data fields.  Allowed
   delimiter values are the characters ``space`` or ``comma``.

``meta``: structure, optional
   Table meta-data as an arbitrary data structure consisting
   purely of data types that can be encoded and decoded with the YAML
   "safe" dumper and loader, respectively.  Typically the top level
   structure is a mapping (dict) or list.  Keywords, comments,
   history and so forth should be part of ``meta``.

``schema``: string, optional
   This provides a reference to a schema that can allow interpretation
   and validation of the ``meta`` values and column definitions.
   Further details of this keyword are TBD and expected in version 1.1
   of the ECSV standard.

Each column specifier is a dictionary structure with the following keys:

``name``: string, required
   Column name

``type``: string, required
  Column data type.  Allowed types are: ``bool``, ``int8``,
  ``int16``, ``int32``, ``int64``, ``uint8``, ``uint16``, ``uint32``,
  ``uint64``, ``float16``, ``float32``, ``float64``, ``float128``,
  ``complex64``, ``complex128``, ``complex256``, and ``string``.
  Some implementations may not support all types.

``unit``: string, optional
   Data unit (unit system could be part of schema?).

``format``: string, optional
   C-style formatting specification for outputting column values.
   This does not imply nor require that the values in this table are
   formatted accordingly.

``description``: string, optional
   Text description of column.

``meta``: structure, optional
   Column meta-data as an arbitrary data structure consisting
   purely of data types that can be encoded and decoded with the YAML
   "safe" dumper and loader, respectively.


Data details
^^^^^^^^^^^^^

The data section follows immediately after the header.  Lines in the
data section consisting only of zero or more whitespace characters
(space and/or tab) are ignored.

The first non-blank line in the data section must contain the column
names formatted according to the CSV writer being used.  This allows
most CSV reader applications to successfully read ECSV files and
automatically infer the correct column names.  ECSV readers should
validate that the column names in this line match those in the header.

Following the column name line the data values are serialized according to
the following rules:

- Each row must contain the same number of delimiter-separated fields.
- Fields are separated by the delimiter character, which can be either the
  space or comma character.
- Any field may be quoted with double quotes.
- Fields containing a line-break, double-quote, and/or the delimiter character
  must be quoted
- Boolean fields are represented as the string ``False`` or ``True``.
- A double quote character in a field must be represented by two double quote
  characters.


Multidimensional columns
""""""""""""""""""""""""

Multidimensional columns are not supported in version 1.0 of the ECSV format.

None of the available text data formats supports multidimensional columns
with more than one element per row.  Although in many cases
having such data would indicate using a binary storage format, there is
utility in supporting this for cases where the column shape is "reasonable",
perhaps with no more than about 10 elements.

One possible solution is to store the individual data elements as a series of
columns with a naming convention such as ``<name>__<index0>_<index1>_...``.  In
this case one would include a keyword in the column specification that
indicates the column is one element of a multidimensional column ``<name>``.
The specifics might need iteration, but again the idea is to maintain the
ability to always read a ECSV file with a simple CSV reader, even if using the
results then takes more effort.

Branches and pull requests
--------------------------

`PR# 2319 <https://github.com/astropy/astropy/pull/2319>`_: "Implement support for the ECSV format proposed in APE6"

`PR# 683 <https://github.com/astropy/astropy/pull/683>`_: Initial version "Support table metadata in io.ascii"


Implementation
--------------

Much of the implementation is done in PR# 2319, which was based on PR# 683.
Further work is pending discussion of APE6.


Backward compatibility
----------------------

This section describes the ways in which the APE breaks backward compatibility.


Alternatives
------------

Alternative existing formats that support some degree of metadata have been
discussed, but none provide the necessary framework for serialization and
interchange of astropy Tables.


Decision rationale
------------------

<To be filled in when the APE is accepted or rejected>
