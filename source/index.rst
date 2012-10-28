Introduction
============

The following describes a data catalog schema and access protocol. The schema
describes how metadata from the catalog should be presented (and, if
neceessary, consumed) while the protocol provides a specification of how to
interact with a data catalog in order to extract relevant information from it
such as metadata on the datasets it contains.

Schema
======

The Schema is directly based on DCAT_ with some minor recommendations re
specific usage. In addition we provide a mapping of the DCAT_ vocabulary (which
is RDF-oriented) to pure JSON (and JSON-LD).

.. _DCAT: http://www.w3.org/TR/vocab-dcat/

The following classes from DCAT_ are used: dcat:Dataset and dcat:Distribution
(Resource). The following are optional and are not used by default in the
outline below: dcat:Catalog and dcat:CatalogRecord.

JSON serialization
------------------

A dataset would be presented as follows in JSON::

  {
    # required attributes
    id: [string or integer] [dcterms:identifier] identifier of the dataset
    title: [string] [dc:title] title for the dataset
    license: [string] [dcterms:license] identifier for the license for this dataset
    resources: [list] [reference] a list of resource objects (see below) 

    # optional attributes
    name: [string] [] short name or slug suitable for use in a URL
    author: [string] [dc:creator] author / creator of this dataset
    maintainer: [string] = dcterms:publisher
    tags: [ list-of-strings ] = dcat:keyword
    spatial: [GeoJSON Object] = dcterms:spatial
    temporal: [string] [dcterms:temporal] as per dcterms:temporal
    version: [string] [] string specifying version of the data 

    # CatalogRecord (required)
    metadata_modified: [iso8601 datetime] [dc:modified] when catalog was last modified
    metadata_created: [iso8601 datetime] [dc:issued]

  }

Resource are a dcat:Distribution (and sub-types thereof)::

  {
    # required
    resource_type: [ file | file.upload | api | doc | ... ] = defines the subclass of Distribution
    url: [string] [dcat:downloadUrl] url download this file
    
    # optional
    format: [string] [dc:format] format of the file
    size: [integer] [dcat:size] file size in bytes
    ## additions compared to DCAT
    title: [string] [dc:title] title of this resource (e.g. file name /title )
    mimetype: [string] [] the mimetype of the file
    hash: [string] [] md5 hash of the file
    last_modified: [iso8601 datetime] [dc:modified] last modified for this resource
    name: [string] [] short name / slug suitable for use in a url
  }


n3 Serialization
----------------

The n3 serialization follows directly from DCAT_ since DCAT_ is an RDF vocabulary. Full details can be found in the DCAT_ specification but we provide  one example here::

  :dataset/001
     a       dcat:Dataset ;
     dct:title "Imaginary dataset" ;
     dcat:keyword "accountability","transparency" ,"payments" ;
     dcat:theme :themes/accountability ;
     dct:issued "2011-12-05"^^xsd:date ;
     dct:updated "2011-12-05"^^xsd:date ;
     dct:publisher :agency/finance-ministry ;
     dct:accrualPeriodicity "every six months" ;
     dct:language "en"^^xsd:language ;
     dcat:Distribution :dataset/001/csv ;
     .

Proposals for changes to DCAT
-----------------------------

Various changes to DCAT have been suggested to as a result of in practice
usage. The following summarize the proposed changes.

.. note:: The following are under discussion with the W3C Government
          Linked Data working group who are managing the DCAT specification. A
          detailed discussion took place at the `GLD WG meeting on 26th July`_
          and consensus resolution has been reached on almost all of them at
          the recent GLD meeting in October - see `minutes and resolutions of
          the meeting on 25th October 2012`_.

.. _minutes and resolutions of the meeting on 25th October 2012: http://www.w3.org/2011/gld/meeting/2012-10-25
.. _GLD WG meeting on 26th July: http://www.w3.org/2011/gld/meeting/2012-07-26

Dataset concept
~~~~~~~~~~~~~~~

* Remove dcat:accessURL and just use Resource (Distribution)

* Remove dcat:dataDictionary (leave for v2 or v1.1)

  * Better to introduce once practice has established a need and consistent
    usage. One should be parsimonious in generating new properties at this
    early stage.
  * Also currently has inconsistent usage

* Remove dcat:dataQuality (ditto)

  * As previous

* Remove dcat:granularity (or specify better)

  * As previous

* Remove dc:references (is it used and how would it be used)

  * Suggest removal since for linking datasets we should have (at some point):
    derives, links_to, sibling, partof
  * Remember that people can always add other attributes they want ...

* (Correction) dc:updated versus dc:modified (example uses dc:updated)

* Make clear what is optional versus required (?) e.g.

  * Designate as optional: dcterms:accrualPeriodicity
  * Designate as optional: dcat:theme

Possibly to add (but will not happen for the present):

* version
* partof

Distribution / Resources concept
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Rename dcat:Distribution to dcat:Resource

  * Distribution has a strong connotation from software of a packaged version
    of the entire dataset whereas, in fact, in most cases it will be a data
    file or API associated to the Dataset for which the term Resource is more
    appropriate.
  * Status: ticket and discuss

* Size: define it as bytes and add sizeString. That is:

  * dcat:size = number / size in bytes
  * [Add] dcat:sizeString: informal string description size e.g. > 1Mb

* Extend the set of attributes a Resource may have

  * [Optional] Add dc:title to Resource
  * [Optional] dcat:mimetype - see http://docs.ckan.org/en/latest/domain-model-resource.html

    * http://docs.ckan.org/en/latest/domain-model-resource.html#resource-format-strings
    * could also have mimetypeInner

  * [Optional]: hash (md5 or sha1, must be of form md5:{hash} or sha1:{hash})
  * [Optional]: dc:created and dc:modified

Catalog Access, Federation and Harvesting Mechanism
===================================================

**Status: early draft**

This portion of the specification details a protocol for accessing catalog
metadata and supporting automated harvesting and federation.

*This specification is at a very early stage and is intended as a basis for discussion rather than a finished document*.

API
---

A catalog MUST provide the following API. The API base location is specified by the following meta tag in the site home page::

  <meta content="data-catalog-api" value="http://my-data-catalog.org/api" />

Relative to this base URL there are the following endpoints::

  /changes.json # changes API
  /dataset/{id}.json # dataset API

Changes API
~~~~~~~~~~~

Get all changes since X::

  /api/changes.json?since=date&page=3

Two optional parameters:

  * since: date to specify when to retrieve changes since
  * page: page option

Dates in the API are assumed to be formatted in according to ISO 8601 (e.g. 2012-12-12T12.12.342342). 

Returns a list of objects like::

  {
      dataset_id:
      modified_date: 2012-12-12T12.12.342342
      change_type: update | deleted | created | ...
  }

Format of returned results is determined by extension. An implementor MUST implement JSON and MAY implement others such as turtle, n3 etc.

When the request is invalid or the requested range not available, return a 400 Bad Request with a message saying something sensible like "the turtle API is not available. Use the JSON API here: http://xxx"

Attempts to access a format that is not supported MUST return 400 Bad Request.

Dataset API
~~~~~~~~~~~

This returns object corresponding to the Schema specified above. The desired representation can be specified both via the file extension on the URI as well as via an Accept header. Supported types are:

  * .json - application/json 
  * .n3 - text/n3
  * .rdf - application/rdf+xml

To Discuss
----------

* Rate limiting based on the values in a ROBOTS.txt
* Notification (push) APIs

