^^^^^^^^^^^^
Introduction
^^^^^^^^^^^^

This document describes the layout of source files used with the
Abbreviation Filter, and the operation of the Filter itself.

^^^^^^
Layout
^^^^^^

An abbreviation source file consists of three layers of keys: ``jurisdiction keys``
and ``category keys``, and ``text keys``. A completely empty source list looks like this:

.. code-block:: javascript

    ABBREVS = {
      default: {
        "container-title": {},
        "collection-title": {},
        "institution-entire": {},
        "institution-part": {},
        "nickname": {},
        "number": {},
        "title": {},
        "place": {},
        "hereinafter": {},
        "classic": {}
      }
    }

^^^^^^^^^^^^^^^^^
Jurisdiction keys
^^^^^^^^^^^^^^^^^

The ``default`` jurisdiction segment is required. Abbreviations
that are not applied exclusively to legal materials of a specific
jurisdiction should be placed under this key.

Additional jurisdictions may be included in a source file, with a
jurisdiction code drawn from the `Legal Resource Registry`__ (LRR) as key.
Each jurisdiction segment must have the full set of category keys
shown above. [#]_

LRR keys have a colon-delimited path-like structure. If a key with multiple
path elements is included in a source file, keys for the parent elements
must also be included. For example, if the key ``us:c9`` is included
(corresponding to the Ninth Circuit of the U.S. federal court system),
then ``us`` must also be included.

__ https://fbennett.github.io/legal-resource-registry

.. [#] Actually, this is not true, I think, but it is good
       practice.

^^^^^^^^^^^^^^^^
Match priorities
^^^^^^^^^^^^^^^^

Values presented for abbreviation are associated with an ``Item`` and
with a variable name. The Filter will attempt a match of the value against
the keys of a category corresponding to the variable name (see below),
under the jurisdiction set on the ``Item``. If the attempt fails, and
if the jurisdiction is not ``default``, a right-side element of the
jurisdiction key is removed, and the attempt is repeated. When no
element remain, the ``default`` jurisdiction is attempted.

If no match is found, the value is returned verbatim. If a match
is found, the abbreviation is returned.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Database storage and runtime caching
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Source files are loaded to the Filter before use. When loaded, the
jurisdiction, category key, text key and value of each item is
recorded in a set of (normalized) SQLite database tables, and
associated with a particular citation style.

The Filter provides a primitive method ``getAbbreviation()`` that returns
matching values from the database. In ``citeproc-js``, this is wrapped
in a ``loadAbbreviation()`` method that first hits a cache object at
``citeproc.transform.abbrevs`` before resorting to a database call.

^^^^^^^^^^^^^
Category keys
^^^^^^^^^^^^^

The CSL variable category assignments are given below. Matching logic,
which varies according to the category, is described under each heading.

------------------
Ordinary text keys
------------------

For these categories, a simple match is attempted against the field value.

``title``
    In addition to ``title``, this includes ``event``, ``genre``, ``medium`` and ``title-short``.

``container-title``
    Only the ``container-title`` variable itself is in this category.

``collection-title``
    In addition to ``collection-title``, this includes ``archive``.

``place``
    This includes ``publisher-place``, ``event-place``, ``archive-place``, ``jurisdiction``, ``language-name``, and ``language-name-original``.
    (The latter two are extended virtual variables derived from the ``language`` field value, recognized only in
    the Juris-M extended CSL schema.)

---------------------
Institution name keys
---------------------

These categories apply to institution name values. Institution names may consist
of multiple subunits, divided by the field separator char ``|``. The
largest subunit is listed first, as "``United States|California``".

In Juris-M, the ``authority`` and ``committee`` variables are "honorary
creators." Although they enter the processor as ordinary variables, they
are parsed and handled as if they were creator variables containing
an institution name.

``institution-entire``
    This matches the entire institution name, including any field
    separator chars, literally.

``institution-part``
    This matches individual elements of an institution name.
    
----------
Number key
----------

There is just one item in this category: ``number``, which contains
only the ``number`` variable. Values appear in this category (or
*should* appear in this category) only if they test false for
``is-numeric`` in CSL.

When values are pass through to this category, they are matched
as literal strings, in the same way as ordinary text keys.

-----------
Classic key
-----------

The ``classic`` category matches items of the Juris-M extended
CSL item type ``classic``. The match is by item ID, independent
of content. This item type covers an edge case in classical
scholarship, in which works are identified by a short-form
reference that is not included in the bibliography.

------------
Nickname key
------------

This matches personal creator variables, in a normalized
rendered form. It is used to substitute matching names with
a placeholder such as "author" or an empty string, as
required by the style when referring to the author of
the manuscript. The transform is applied only (IIRC) to
creators rendered via a ``personal_communication`` or
``interview`` item.

Use of this abbreviation transform permits the entry data to be shared
among researchers, without mangling the content. List
entries in this category are not exported, and the
category should always be empty in source data files.

---------------
Hereinafter key
---------------

The ``hereinafter`` key matches the item ID of a target item. It gives
the author the ability to customize back-references. This *should*
be manuscript-specific, but we don't have a means of supporting that
at present, so it is tied to the citation style.

Like ``nickname``, abbreviations in this category are not exported,
and the segment should be empty in source files.
