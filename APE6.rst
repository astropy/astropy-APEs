Data-table Text Interchange Format
----------------------------------

author: Tom Aldcroft

date-created: 2014 April 12

date-last-revised: 2014 APR 12

type: Standard Track

status: Discussion


Abstract
--------

This APE is primarily a specification of a new standard for the interchange of
tabular data in a text-only format.  The proposed format handles the key issue
of serializing column specifications and table metadata by using a JSON-encoded
data structure which is included in the file as a series of ``#``-prefixed
comment lines.  The actual tabular data are stored in a standard
comma-separated-values (CSV) format, giving compatibility with a wide variety of
non-specialized CSV table readers.  Using JSON makes it extremely easy for
applications to read both the standardized data format elements (e.g. column
name, type, description) as well as complex metadata structures.  Support for
schemas to describe and validate the metadata is part of the standard along with
support for non-ASCII unicode character encoding.

The implementation in ``astropy.io.ascii`` is relatively straightforward and
will provide a significant benefit of allowing text serialization of most astropy
Table objects, persistent storage, and subsequent interchange with other users.

Existing standards
--------------------

The well-known `XKCD comic <https://xkcd.com/927/>`_ aptly mocks the
introduction of new standards.  Let's just stipulate that this standard may be a
bad idea, but talk about it anyway.  Following the lead of the astronomical data
community in recently documenting shortcomings of the FITS standard, we start by
discussing existing widely-used standards and why they are
lacking.

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

We are not aware of other widely-used standards for text representation of
tabular data.


Detailed description
---------------------

The proposed Data-table Text Interchange Format (DTIF) has the following
overall structure:

- A header section which consists of lines that start with the "#" character
  and provide the table definition via a JSON-encoded data structure.
- A CSV-formatted data section in which the first line contains the column names
  and subsequent lines contains the data values

Why JSON?
^^^^^^^^^^

The fundamental issue which the CDS and IPAC formats try to address is
serializing the information which specifies the table column definitions and
other relevant metadata.  Those formats essentially invent custom serialization
specifications that must be carefully implemented from scratch by any
reader/writer application.

DTIF takes the approach of defining a minimal standard for the the underlying
data structure that is needed to define a table.  Then that structure is
encoded or decoded using JSON.  Libraries for encoding and decoding JSON are
widely used, very efficient, and easily available in all the most-frequently
used programming languages.

Translating to / from the data structure provided by a DTIF header into the native
structure that an application uses should generally be quite easy because the
functional elements (e.g. column name, type) are ubiquitous.  Generally
speaking manipulating data structures programmatically is easier than parsing
textual data structure fields.

The DTIF standard does not require that the JSON encoding be "pretty", but it
is highly-recommended that applications format the JSON header to be legible
to humans.

Example
^^^^^^^^^^

A quick example will put this in context.  First let's create a table
and give it some custom attributes::

  >>> from astropy.table import Table
  >>> t = Table([[1, 4], [2, 3]], names=['a', 'b'])
  >>> t['a'].unit = 'm/s'
  >>> t['a'].format = '%03d'
  >>> t['b'].description = 'This is column b'
  >>> t['b'].unit = 'km'
  >>> print t
    a    b 
  m / s  km
  ----- ---
    001   2
    004   3

Now we write this to a file using the DTIF format and print it::

  >>> t.write('example.dtif', format='ascii.dtif')
  >>> cat example.dtif
  # <DTIF encoding=ascii>
  # {
  #   "version": 1.0,
  #   "schema": "astropy.table",
  #   "table_meta": {},
  #   "columns": [
  #     {
  #       "name": "a",
  #       "unit": "m / s",
  #       "format": "%03d",
  #       "description": null,
  #       "type": "int64",
  #       "meta": {}
  #     },
  #     {
  #       "name": "b",
  #       "unit": "km",
  #       "format": null,
  #       "description": "This is column b",
  #       "type": "int64",
  #       "meta": {}
  #     }
  #   ]
  # }
  a b
  001 2
  004 3

We see that header starts with a sentinel to identify the format and provide a
required character encoding argument.  After that comes the JSON data structure
with top-level keywords and column definitions.  Finally the column names and
data values are included in CSV format with a space delimiter.

Now we can read back the table and see that it has survived the round-trip
to a text file::

  >>> t2 = Table.read('example.dtif', format='ascii.dtif')
  >>> print t
    a    b 
  m / s  km
  ----- ---
    001   2


Header details
^^^^^^^^^^^^^^^^

The table header contains the necessary information to define the table columns
and metadata.  This is expressed as a JSON-encoded data structure which has a
small set of required keywords and standard specifiers.  Beyond the minimal
standard, applications are free to create a custom data structure as needed.
The specification of a corresponding ``schema`` keyword to allow interpretation
and validation of the custom data is highly encouraged.

Standard keywords are:

``version``: required
   Version of the DTIF standard.

``columns``: required
   List of column specifiers.

``schema``: optional
   Schema name defining any data structure elements not specified in the
   minimal DTIF standard.  Details TBD.

``table_meta``: optional
   Table meta-data as an arbitrary dictionary or list type data structure.
   TDB: keywords etc as part of ``table_meta``?

OTHERS?
   Keywords, Comments, History, ???  Should these be standard?

Each column specifier is a dictionary structure with the following keys:

``name``: required
   Column name

``unit``: optional
   Data unit (unit system could be part of schema?)

``format``: optional
   C-style formatting specification for outputting column values.  This does
   not imply or require that the values in this table are formatted
   accordingly.

``description``: optional
   Text description of column

``type``: optional
   If provided this specifies the column data type.  If not available then
   automatic type inference is performed.

Data details
^^^^^^^^^^^^^

The data section follows immediately after the header.

The first line in the data section contains the column names formatted
according to the CSV writer being used.  This allows most CSV reader
applications to successfully read DTIF files and automatically infer the
correct column names.  DTIF readers should validate that the column names in
this line match those in the header.

Following the column name line the data values are serialized according to
standard CSV rules.

In this example above the delimiter is the space character.  Details of
delimiters, quote characters, etc that should be allowed / supported are TBD.

Multidimensional columns
^^^^^^^^^^^^^^^^^^^^^^^^^

None of the available text data formats supports multidimensional columns
with more than one element per row.  Although in many cases
having such data would indicate using a binary storage format, there is
utility in supporting this for cases where the column shape is "reasonable",
perhaps with no more than about 10 elements.

In this case one could store the individual data elements as a series of
columns with a naming convention such as ``<name>__<index0>_<index1>_...``.
In this case one would include a keyword in the column specification that
indicates the column is one element of a multidimensional column ``<name>``.
The specifics might need iteration, but again the idea is to maintain the
ability to always read a DTIF file with a simple CSV reader, even if using
the results then takes more effort.

Branches and pull requests
--------------------------

PR# XXXX


Implementation
--------------

Much of the implementation is done in PR# XXXX.  Further work is pending
discussion of APE6.


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
