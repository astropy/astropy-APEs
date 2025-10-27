Enhanced Character Separated Values table format
------------------------------------------------

author: Tom Aldcroft

date-created: 2014 April 12

date-last-revised: 2021 May 25

date-accepted: 2015 January 26

type: Standard Track

status: Accepted

revised-by: Tom Aldcroft, Mark Taylor - 2021 May 02 - Added subtype keyword and extended data types

Abstract
--------

Data tables in astronomical analysis frequently contain additional metadata
beyond just the column names and data values.  Common attributes include the
numerical data type, the physical unit, and a longer textual description of the
column content.  These attributes can be represented in binary formats such as
FITS, but the available options for a text-only format are inadequate.

APE6 proposes to fill this void by specification of a standard for the interchange of
tabular data in a text-only format.  The format handles the key issue
of serializing column specifications and table metadata by using a YAML-encoded
data structure.  The actual tabular data are stored in a standard
character separated values (CSV) format, giving compatibility with a wide variety of
non-specialized CSV table readers.

Using YAML makes it extremely easy for applications *and humans* to read both
the standardized data format elements (e.g. column name, type, description) as
well as complex metadata structures.  YAML also lends itself to simple table
modifications by humans in a plain text editor.

The reference Python implementation in ``astropy.io.ascii`` is relatively
straightforward and will provide a significant benefit of allowing text
serialization of most astropy Table objects, persistent storage, and subsequent
interchange with other users.

Although developed in the context of Astropy, there is nothing
astronomy-specific in this format and it could be useful for other domains or
languages.  However, promoting that is beyond the scope of this APE.

Existing standards
--------------------

Following the lead of the astronomical data community in recently
documenting shortcomings of the FITS standard, we start by discussing
existing widely-used standards and why they are lacking.

CSV
^^^^^

The `Comma-Separated-Values format
<https://en.wikipedia.org/wiki/Comma-separated_values>`_ is probably the most
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
<https://vizier.u-strasbg.fr/doc/catstd.htx>`_ which is very widely used.  The
Astrophysical Journal has adopted this standard for formatting `machine readable
tables <https://aas.org/authors/machine-readable-table-standards>`_ associated
with journal articles.  The format is alternately referred to as the CDS or
Machine Readable Table (MRT) format.

The CDS format has reasonable support for basic table and column metadata,
including column types, null values, range checking, units, and labels.  From
its heritage as a catalog descriptor format, it includes table metadata
like ADC_Keywords, Literature Reference, table description, authors, notes,
and comments.  Within that context it provides a well-documented standard.

However, the CDS format does not include support for keyword values like FITS,
nor arbitrary table metadata.  This limits the fidelity of serialization
for tables that have rich metadata (e.g. WCS).

As a tool for data interchange, CDS is far from ideal.  Despite the name "Machine
Readable Table", the format is quite difficult for machines to read.  This
stems from the format of the `ReadMe file
<https://vizier.u-strasbg.fr/doc/catstd-3.1.htx>`_, which has a
non-regular format that requires tricky regex parsing and logic to fully interpret.

A few options for `reading MRT files
<https://aas.org/authors/machine-readable-table-programs>`_ are available, but
none of them are full-featured.  As far as we know the version
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
<https://irsa.ipac.caltech.edu/applications/DDGEN/Doc/ipac_tbl.html>`_ is the
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

`VOTable <https://www.ivoa.net/documents/latest/VOT.html>`_ is by design a
fully-flexible data format that can handle all of the needs for text
serialization of complex data structures, including tabular data sets.  The
issue in this context is in simplicity and data interchange with the broader
community.  In essence if someone wants to read or write a VOTable then they
must use one of a small number of implementations of this protocol.  It is not
possible for someone to directly read such a table into Excel.  Writing an
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
<https://dataprotocols.org/tabular-data-package/>`_.  This provides a
fully-formed protocol for publishing and sharing tabular-style data
which is conceptually very similar to the proposed ECSV format, with
the exception of using two files, one pure JSON for the header and one
pure CSV for the data.  The JSON header follows a schema defined by
the `JSON Table Schema
<https://dataprotocols.org/json-table-schema/>`_.


After evaluation and a brief `discussion with that community
<https://lists.okfn.org/pipermail/data-protocols/2014-April/000093.html>`_, we
find that the Tabular Data Package and associated standards would require a
fair degree of modification to fully suit our needs.  More crucially, the
decision to rely on YAML instead of JSON for serialization precludes direct use
of the TDP.


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
functional elements (e.g. column name, data type) are ubiquitous.  Generally
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
  # datatype:
  # - {name: a, unit: m / s, datatype: int64, format: '%03d'}
  # - {name: b, unit: km, datatype: int64, description: This is column b}
  a b
  1 2
  4 3

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

  >>> from collections import OrderedDict
  >>> import astropy.units as u
  >>> t = Table([[1.0, 4.0], [2, 3]], names=['a', 'b'])
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
  # datatype:
  # - {name: a, unit: m / s, datatype: float64, format: '%5.2f', description: Column A}
  # - name: b
  #   datatype: int64
  #   meta:
  #     column_meta: {a: 1, b: 2}
  # meta: !!omap
  # - keywords: !!omap
  #   - {z_key1: val1}
  #   - {a_key2: val2}
  # - comments: [Comment 1, Comment 2, Comment 3]
  # schema: astropy-2.0
  a b
  1.0 2
  4.0 3

In this case there are Ordered dictionary elements which must be preserved
during serialization.  This is done via the ``!!omap`` element tag.  The
reference Python implementation in Astropy uses custom Loader and Dumper
classes to allow serialization to and from the Python ``OrderedDict`` class.
Other implementations must likewise use an ordered mapping when reading and the
``!!omap`` tag when writing for ordered mappings in the data structure.

In addition, the reference Python implementation outputs the column attributes
in the order ``'name'``, ``'unit'``, ``'datatype'``, ``'format'``,
``'description'``, and ``'meta'``.  This is not a ECSV requirement but is
recommended for human accessibility.

Header details
^^^^^^^^^^^^^^^^

An ECSV file begins with the the table header which contains the
necessary information to define the table columns and metadata.  This
is expressed as a YAML-encoded data structure which has a small set of
required keywords and standard specifiers.

Each line of the YAML-encoded data structure must start with the hash
character ``#`` followed by a single space to indicate the presence of
header content.  All content within this header section must be parseable
as a single YAML document.  The first line which does not start with ``#``
signifies the end of the header and the start of the data section.  Subsequent
lines within the data section starting with ``#`` are to be ignored by the
parser and not provided in the header output.

Within the header section, lines which start with ``##`` are treated as
comments and must be ignored by readers.  ECSV writers shall not emit such
comment data.  Relevant comment strings should be serialized within the
``meta`` keyword structure.

Beyond the minimal standard, applications are free to
create a custom data structure as needed using the top-level ``meta``
keyword.  The specification of a corresponding ``schema`` keyword to
allow interpretation and validation of the custom data is highly
encouraged.

Standard keywords are:

``datatype``: list, required
   Top-level data type specification as a list of column specifiers.

``delimiter``: one-character string, optional, default=``space``
   Delimiter character used to separate the data fields.  Allowed
   delimiter values are the single characters ``space`` or ``comma``.

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

``datatype``: string, required
  Column data type.  Allowed types are: ``bool``, ``int8``,
  ``int16``, ``int32``, ``int64``, ``uint8``, ``uint16``, ``uint32``,
  ``uint64``, ``float16``, ``float32``, ``float64``, ``float128``,
  ``complex64``, ``complex128``, ``complex256``, and ``string``.
  Some implementations may not support all types.

``subtype``: string, optional
   The ``subtype`` keyword describes an extended data type such as a
   variable-length array or an object column. See the section below on `Subtype
   data`_ for details. This is new in version 1.0 of the ECSV standard.

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
A mismatch of the number of columns will be an error.  If there is a
name mismatch then it is recommended that the reader issue a warning,
but implementations are free to be either less or more strict.

Following the column name line the data values are serialized according to
the following rules:

- Each row must contain the same number of delimiter-separated fields.
- Fields are separated by the delimiter character, which can be either the
  space or comma character.
- Any field may be quoted with double quotes.
- Fields containing a line-break, double-quote, and/or the delimiter character
  must be quoted
- Boolean fields are represented as the case-sensitive string ``False`` or
  ``True``.
- A double quote character in a field must be represented by two double quote
  characters.

Missing values
""""""""""""""

Missing values in a data column are, by default, represented by a blank entry at
the corresponding data location. For a comma-delimited file the following
example shows the 3rd and 5th columns in this row as missing values::

  1,2.0,,Hello,

For a space-delimited file the corresponding row would be::

  1 2.0 "" Hello ""

With this convention it is not possible to have a zero-length string within a
string column.  Specialized schemas may choose to use a different convention for
missing (or masked) data. For instance it is possible to write a masked column
as two separate columns, one for the data and one for the mask. In this case the
empty string no longer serves as a marker for missing values.

Subtype data
""""""""""""

From version 1.0 and later it is possible to embed extended data types beyond
simple typed scalars in the data section. The column data in the ECSV output
shall be consistent with the specified ``datatype``, with additional details of
the data being captured in the ``subtype`` keyword.

If table readers do not recognise or support the ``subtype`` then they may
ignore it and use the ``datatype`` only.

The ECSV standard defines three types of extended data that can be represented:
multidimensional array data, variable-length array data, and object data. These
correspond to specified ``subtype`` values described below. It is also allowed
to define new custom ``subtype`` values for specific applications.

Multidimensional array data
@@@@@@@@@@@@@@@@@@@@@@@@@@@

For multidimensional columns where each data cell is an array with consistent
dimensions, the ``datatype`` is set to ``string`` and the ``subtype`` is set to
the actual data type (one of the allowed values of the ``datatype`` keyword)
followed by the `JSON <https://www.json.org/>`_ representation of the shape
(dimensions) of each cell.

The contents of each cell are represented as a string using the JSON encoding of
the array values. The encoding shall use row-major ordering with array shapes
defined accordingly.

In the example below each cell is a ``3 x 2`` array of ``float64`` type. The
shape is ``[3,2]`` so the ``subtype`` is ``float64[3,2]``::

  # %ECSV 1.0
  # ---
  # datatype:
  # - {name: array3x2, datatype: string, subtype: 'float64[3,2]'}
  # schema: astropy-2.0
  array3x2
  [[0.0,1.0],[2.0,3.0],[4.0,5.0]]
  [[6.0,7.0],[8.0,null],[10.0,11.0]]

Missing values are indicated by the ``null`` marker, as seen in the second row.

Variable-length array data
@@@@@@@@@@@@@@@@@@@@@@@@@@

For columns where the data cells are arrays which are consistent in all
dimensions *except* for the final dimension, the ``datatype`` is set to
``string`` and the ``subtype`` is set to the actual data type (one of the
allowed values of the ``datatype`` keyword specified previously)  followed by
the JSON representation of shape (dimensions) of the cells. Here the shape is
set to the consistent dimensions plus a ``null`` dimension.

The contents of each cell are represented as a string using the JSON encoding of
the array values. The encoding shall use row-major ordering with array shapes
defined accordingly.

For example a column that has 3-d arrays in each cell with shapes of
``[4,4,2]``, ``[4,4,5]`` and ``[4,4,3]``, the ``subtype`` would be
``int64[4,4,null]``. For a column that has 1-d ``int64`` arrays having lengths of
2, 5, and 3 respectively the ``subtype`` would be ``int64[null]``.

An example for a 1-d variable-length array follows::

  # %ECSV 1.0
  # ---
  # datatype:
  # - {name: array_var, datatype: string, subtype: 'int64[null]'}
  # schema: astropy-2.0
  array_var
  [1,2]
  [3,4,5,null,7]
  [8,9,10]

Missing values are indicated by the ``null`` marker, as seen in the second row.

Object columns
@@@@@@@@@@@@@@

For object-type columns, the ``datatype`` is set to ``string`` and the
``subtype`` is set to ``json``. Each object in the column is converted to a
string representation using `JSON <https://www.json.org/>`_ encoding. This
implies that the supported object types are those that can be represented in
JSON, namely integer and float numbers, boolean, null, strings, arrays, and
mappings.

As a point of clarification, "object" here refers to the common usage in the
context of object-oriented programming. In the JSON standard, "object" refers to
what we call a "mapping", for instance ``{"a":1, "b":2}``.

The example below shows writing an object array to ECSV. Note that JSON requires
a double-quote around strings, and ECSV requires ``""`` to represent
a double-quote within a string, hence the double-double quotes.
::

  # %ECSV 1.0
  # ---
  # datatype:
  # - {name: objects, datatype: string, subtype: json}
  # schema: astropy-2.0
  objects
  "{""a"":1}"
  "{""b"":[2.5,null]}"
  true

In this subtype, the ``null`` marker is decoded by JSON as the language-specific
representation of a null value. In Python this will be ``None``.

Branches and pull requests
--------------------------

`PR# 2319 <https://github.com/astropy/astropy/pull/2319>`_: "Implement support for the ECSV format proposed in APE6"

`PR# 683 <https://github.com/astropy/astropy/pull/683>`_: Initial version "Support table metadata in io.ascii"

PR#
`11569 <https://github.com/astropy/astropy/pull/11569>`_,
`11662 <https://github.com/astropy/astropy/pull/11662>`_,
`11720 <https://github.com/astropy/astropy/pull/11720>`_:
"Support reading and writing multidimensional and object columns in ECSV"


Implementation
--------------

The initial implementation is done in PR# 2319, which was based on PR# 683. PR#
11569 added support for extended data types (multidimensional and object
columns) via a new ``subtype`` keyword.

Backward compatibility
----------------------

This is a new feature and there are no issues with backward compatibility.

The 1.0 update adds a new ``subtype`` keyword. This is backward compatible with
the previous 0.9 version since that keyword will simply be ignored by older
readers that are only compliant with the 0.9 standard. In this case the
extended data values will be returned as defined by the ``datatype`` keyword.

Alternatives
------------

Alternative existing formats that support some degree of metadata have been
discussed, but none provide the necessary framework for serialization and
interchange of astropy Tables.


Decision rationale
------------------

This APE was discussed in
`astropy-APEs PR 7 <https://github.com/astropy/astropy-APEs/pull/7>`_, with
calls for comment sent out to the astropy, astropy-dev, and apps@ivoa.net
mailing lists.  There was some concern that this proposal duplicates
functionality that exists elsewhere (e.g., those discussed above), but the
general consensus of the discussion participants was that this proposal
provides a format with features that are sufficiently different from
older options that it is worth implementing.

A number of good suggestions and ideas were incorporated from the discussion,
particularly related to compatibility with the ASDF standard.  All comments
from interested parties were agreeably resolved.  As a result, the coordination
committee unanimously agreed to accept this APE on 2015 January 26.

A significant update wa made in
`astropy-APEs PR 68 <https://github.com/astropy/astropy-APEs/pull/68>`_, adding
support for extended data types. There was significant discussion in the PR
but broad agreement with the change.

Previous versions of this APE
-----------------------------

* 2019-03-08 [`DOI <https://doi.org/10.5281/zenodo.1043901>`_] [`GitHub <https://github.com/astropy/astropy-APEs/blob/d2c90199a87b59c2b3e8465586c34a029287766c/APE6.rst>`_]
