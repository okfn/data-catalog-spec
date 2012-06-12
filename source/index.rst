Introduction
============

The following describes a data catalog schema and access protocol.

Schema
======

The Schema is directly based on DCAT_ with some minor recommendations re
specific usage.

In addition we provide a mapping of the DCAT_ vocabulary (which is
RDF-oriented) to pure JSON (and JSON-LD).

.. _DCAT: http://dvcs.w3.org/hg/gld/raw-file/default/dcat/index.html

The following classes from DCAT_ are required:

  * dcat:Dataset
    
    * dcat:Distribution (Resource)

The following are optional:
  
  * dcat:Catalog
  * dcat:CatalogRecord

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

Suggested DCAT Changes
----------------------

Dataset concept
~~~~~~~~~~~~~~~

* Remove dcat:accessURL and just use Resource (Distribution)

  * Status: agreed and in progress

* Remove dcat:dataDictionary (leave for v2 or v1.1)

  * Better to introduce once practice has established a need and consistent
    usage. One should be parsimonious in generating new properties at this
    early stage.
  * Also currently has Inconsistent usage
  * Status: ticket and discuss

* Remove dcat:dataQuality (ditto)

  * As previous

* Remove dcat:granularity (or specify better)

  * As previous

* Remove dc:references (is it used and how would it be used)

  * Suggest removal since for linking datasets we should have (at some point):
    derives, links_to, sibling, partof
  * Remember that people can always add other attributes they want ...
  * Status: ticket and discuss

* Make clear what is optional versus required (?)

  * Designate as optional: dcterms:accrualPeriodicity
  * Designate as optional: dcat:theme
  * Resolution: ticket and discuss

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

* Extend the set of attributes a Resource may have

  * [Optional] Add dc:title to Resource
  * [Optional] dcat:mimetype - see http://docs.ckan.org/en/latest/domain-model-resource.html

    * http://docs.ckan.org/en/latest/domain-model-resource.html#resource-format-strings
    * could also have mimetypeInner

  * [Optional]: hash (md5 or sha1, must be of form md5:{hash} or sha1:{hash})
  * [Optional]: dc:created and dc:modified

* Size: define it as bytes and add sizeString. That is:

  * dcat:size = number / size in bytes 
  * [Add] dcat:sizeString: informal string description size e.g. >1Mb

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

Just return a 400 Bad Request with a message saying something sensible like "the turtle API is not available. Use the JSON API here: http://xxx"

Returns a list of objects like::

  {
      dataset_id:
      modified_date: 2012-12-12T12.12.342342
      change_type: update | deleted | created | ...
  }

Format of returned results is determined by extension. An implementor MUST implement JSON and MAY implement others such as turtle, n3 etc.

Attempts to access a format that is not supported MUST return 400 Bad Request

Dataset API
~~~~~~~~~~~

This returns object corresponding to the Schema specified above.

To Discuss
----------

* Rate limiting based on the values in a ROBOTS.txt
* Notification (pus) APIs

