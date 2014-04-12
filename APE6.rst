Data Table Text Interchange Format
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
discussing existing widely-used standards and why they are substantially
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


Branches and pull requests
--------------------------

Any pull requests or development branches containing work on this APE should be
linked to from here.  (An APE does not need to be implemented in a single pull
request if it makes sense to implement it in discrete phases). If no code is yet
implemented, just put "N/A"


Implementation
--------------

This section lists the major steps required to implement the APE.  Where
possible, it should be noted where one step is dependent on another, and which
steps may be optionally omitted.  Where it makes sense, each  step should
include a link related pull requests as the implementation progresses.


Backward compatibility
----------------------

This section describes the ways in which the APE breaks backward compatibility.


Alternatives
------------

If there were any alternative solutions to solving the same problem, they should
be discussed here, along with a justification for the chosen approach.


Decision rationale
------------------

<To be filled in when the APE is accepted or rejected>
