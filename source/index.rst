Data Catalog Interoperability Protocol
++++++++++++++++++++++++++++++++++++++

:Editors: Rufus Pollock (Open Knowledge Foundation), James Gardner (3aims)

.. note:: This proposal reflects the contribution of many people over a substantial period. In particular, there are the participants at the first `Data Catalog Interoperability Workshop`_ in Edinburgh in May 2011 (organized under the auspices of the LOD2 project), as well as the members of the `data-catalogs group and mailing list`_ where this specification has been discussed on an ongoing basis. Among those who have provided expert feedback or input are: Phil Archer (W3C), Martin Alvarez-Espinar (CTIC), Richard Cyganiak (DERI), John Erickson (RPI), Chris Gutteridge (Southampton University), Jim Hendler (RPI), Faadi Mali (DERI), Ed Summers (Library of Congress)

.. _Data Catalog Interoperability Workshop: http://lod2.okfn.org/2011/05/04/notes-from-data-catalogues-interoperability-workshop-edinburgh-3-4th-may-2011/
.. _data-catalogs group and mailing list: http://lists.okfn.org/mailman/listinfo/data-catalogs

Abstract
========

DCIP is a specification designed to facilitate interoperability between data catalogs published on the Web by defining:

* a JSON and RDF representation for key data catalog entities such as ``Dataset (DatasetRecord)`` and ``Resource (Distribution)`` based on the DCAT_ vocabulary
* a read only REST based protocol for achieving basic catalog interoperability

This document discusses each of the above and provides examples. The approach is designed to be a pragmatic and easily implementable. It merges existing work on DCAT with the real-life experiences of "harvesting" in various projects.

Publishers implementing this protocol for their catalog increase discoverability of their data and enable federation and interchange of the datasets they own. They also make it much easier for third parties to build applications based on their metadata leading to a richer ecosystem around their data.

.. _DCAT: http://www.w3.org/TR/vocab-dcat/

Status of This Document
=======================

This document is currently in DRAFT. It has been already been the subject of consultation with interested parties and discussion will continue before being finalised. In particular, extension discussions have been had with the W3C Government Linked Data working group who are overseeing work on the DCAT_ specification and several members of that group have provided expert input and review of this specification.

Ideally, at least two real-world implementations for the DRAFT should exist before the first version specification is finalised. Feedback from this real-world implementation will inform the draft.

A future version of this document may also include:

* a simple authorization model for making changes securely using the signed request protocol
* a streaming changes and notification API so that the next generation of catalogs can be built

1. Goal
=======

The goal of this specification is to provide a standard that meets the following user stories:

1. As an application or another catalog I want to be able to access individual dataset records in other catalogs by their ID and in an appropriate machine format so that I can use the metadata they contain without needing to resort to scraping the information from an HTML page
2. As a catalog which contains copies of certain datasets originally published in other catalogs I want to be able to determine which of my records are out of date so that I can fetch fresh copies if I need to
3. As an application or another catalog I want to be able to perform basic actions such as searching a catalog based on a dataset name or set of keywords so that I don't need to store copies of a catalog's metadata myself

The standard as well as all underlying standards on which it is based, should meet these criteria:

1. Based on existing, open, standards in wide use, with readily available libraries and broad domain expertise.
2. Able to be implemented by an application written for a limited web browser such as IE6 or a basic smart phone such as an Android 2.1 device as well as for powerful servers
3. Extremely easy for any existing catalog to be able to implement user stories 1 and 2 so that the implementation can quickly gain broad adoption
4. Easy for future catalogs to to implement user stories 1-3 and possible for existing catalogs to be upgraded
5. Should not require any special configuration of software on the device the end user is using to access a service meeting the specification

Also to consider:

1. As an application or catalog I may want to know that the information I receive from from another catalog  has not been changed by a man in the middle so that I am confident that I  can safely provide the same information to my users
2. As a catalog I want to know who is making use of my APIs so that I can give them the results they are allowed access to or adjust how my resources are used.

2. Technology Choices
=====================

JSONP vs JSON+CORS vs DCAT.N3+CORS
----------------------------------

A service providing JSONP responses can be used directly in any recent browser to access feeds from across different origins using a GET request. The issues with JSONP are:

* The HTTP 1.1 specification suggests that 255 bytes is the safest length of a GET request. In real life IE and Safari support 2KB. That means that requests larger than 2KB cannot be made. This is probably fine for retrieving data, but not for sending it to the server. A workaround is that for each request you generate a GUID on the client, use JavaScript to post a form with the data to the server then do a second GET request to the server with the GUID to return the result as JSONP.
* A malicious or compromised server can inject JavaScript into a client page with JSONP
* A service wanting to support old browsers, can always proxy information to the origin server anyway and safely serve JSON.

Because of these limitations we exclude JSONP from the specification but instead ensure what is proposed here is workable via a proxy server.

JSON+CORS is ideal in that it allows cross-origin queries to be made by a web browser but is currently supported by just 84% of browsers, with IE6 and IE7 being notable exceptions. It is well understood with broad library support.

DCAT encoded as N3 is very useful for the RDF community.

Hash Algoritms
--------------

We use SHA-256 since SHA-1 has vulnerabilities and SHA-512 takes longer to hash according to http://stackoverflow.com/a/3897457. It is also what Facebook use for their ``signed_request``.


3. Conformance
==============

The key words ``MUST``, ``MUST NOT``, ``REQUIRED``, ``SHOULD``, ``SHOULD NOT``, ``RECOMMENDED``, ``MAY``, and ``OPTIONAL`` in this specification are to be interpreted as described in [RFC2119].

* DCIP compliance means that a data catalog provides an API that is a subset of the APIs defined in one of the conformance levels.
* DCIP conformance means that a data catalog provides the entire API specified for that conformance level

4. REST API
===========

The DCIP specification defines a simple REST API. A catalog that conforms to this API exposes enough information for another catalog to store copies of the first catalog's dataset information as well as discover which datasets have changed.

.. note:: The current specification only allows for discovery of changes through regular polling of key URLs which isn't as easy for a consumer of the catalog API to use, and requires more server resources, but is very easy for the catalog owner to implement.
          
          Once agreement has been reached on the basic API, one can look at further more advanced features such as streaming changes and notifications.

Purpose
-------

REST APIs in general are for the very specific case where a client needs to create, read, update or delete an entity held in a service but no querying, partial updates of the entities are required. The Basic REST API described here only deals with the *read* operation. Thus, at this stage, all the APIs described are **read only**.

Glossary
--------

The specification uses a few terms that you should be familiar with:

REST
    Stands for "REpresentational State Transfer" but is often used to simply describe the use of the HTTP API to create, read, update or delete REST entities hosted on a server

Entity
    The object being referred to, together with an appropriate representation of any related objects. In this case we support ``Dataset`` and ``Catalog`` as two entities

API Endpoint
------------

A meta tag ``MUST`` be specified in the ``<head>`` section of the homepage of the catalog to point to the Basic REST endpoint. The ``content`` attribute ``MUST`` contain ``dcip-basic-rest-endpoint`` and the ``value`` must contain the full endpoint URL. For example:

::

    <meta content="dcip-basic-rest-endpoint" value="http://example.org/rest" />
    
Entities
--------

The Basic REST API defines just one entity at present:

* Dataset

The catalog ``MUST`` support representing the ``Dataset`` entities in JSON and ``SHOULD`` support their representation as DCAT encoded in N3. A full specification of the Dataset entity and its subcomponents can be found in the separate Entity Schemas sectio below.


URL Structure
-------------

URLs are assembled like this:

::

    <endpoint>/<entity-name>/<by-entity-attribute>/<entity-id>.<format_extension>

If the endpoint is specified with a ``/`` character, this ``MUST`` be removed before computing the URL.

The format extension specifies the format of any request body as well as the format of any response. The endpoint ``MUST`` support ``.json`` and ``.dcat.N3`` as the format extensions returning JSON and N3 encoded DCAT respectively.

If a request other than a ``GET`` is made to any URL at the endpoint, a ``400 Bad Response`` ``MUST`` be returned.

Response Headers
----------------

All ``200 OK`` successful API request responses will always contain the following headers:

``Content-Type``
    Value ``MUST`` be ``application/json; charset=utf8`` if the format extension was ``.json`` or . XXX What should it be for N3? or  ``text/plain; charset=utf8`` for text responses.

``Content-Length``
    Value ``MUST`` be the length in bytes of the UTF-8 encoded serialisation of the entity type

``Access-Control-Allow-Origin``
    Value ``MUST`` be ``*`` to allow a web browser running JavaScript served from any domain to access the response

Read Dataset API Call
---------------------------

To get a JSON representation of a ``Dataset`` with an ``id`` of ``123`` at the endpoint ``http://example.com/rest`` you would issue an HTTP GET request to this URL:

::

    http://example.com/rest/dataset/id/123.json

These are the HTTP response status's that ``MUST`` be returned given the possible outcomes of the requst:

``200 OK``

    The request was successful and the entity will be returned in the response body, encoded in whatever way is most appropriate for the file extension chosen.

``400 Bad Request``

    The request was not understood by the server.

``404 Not Found``

    There is no entity with the ID you have specified.

``429 Too Many Requests``

    You have made too many requests too quickly and rate limiting has kicked in.

``500 Internal Server Error``

    The server has crashed trying to fulfil the request

The server ``MAY NOT`` return any other response status.

No response body is returned unless the status is ``200 OK``.

The response can be HTTP 1.0 or HTTP 1.1. The response body ``MUST`` be the JSON serialised representation of the ``Dataset`` if the format extension of the request was ``.json`` and ``MUST`` be the N3 serialized representation of the ``Dataset`` if the format extension was ``.dcat.N3``. Either way, the response ``MUST`` be encoded as UTF-8.

Here's an example HTTP response:

::

    HTTP/1.1 200 OK  
    Access-Control-Allow-Origin: *
    Content-Length: 104
    Content-Type: application/json; charset=utf8  

    {
        ... Dataset information ...
    }

If no ``format_extension`` is specified on the request URL, a ``400 Bad Request`` ``MUST`` be returned.

List Dataset API Call
---------------------

To get a list of all Datasets including their ID, make a GET request as above but leave off the entity ID and format extension. For example, to list all ``Datasets`` with their IDs make a GET request to this URL:

::

    http://example.com/rest/dataset/


These are the HTTP response status's that ``MUST`` be returned given the possible outcomes of the requst:

``200 OK``

    The request was successful and the entity will be returned in the response body, encoded in whatever way is most appropriate for the file extension chosen.

``400 Bad Request``

    The request was not understood by the server.

``429 Too Many Requests``

    You have made too many requests too quickly and rate limiting has kicked in.

``500 Internal Server Error``

    The server has crashed trying to fulfil the request

The server ``MAY NOT`` return any other response status.

No response body is returned unless the status is ``200 OK`` in which case the JSON or N3 serialised list representation ``MUST`` be returned.

.. note:: At the moment no paging facility is specified in order to make the API simpler to implement.

Response Format
~~~~~~~~~~~~~~~

A catalog proving a list Datasets, ``MUST`` specify at least these attributes for each:

``id``
    The Dataset ID.

``change_type``
    ``MUST`` take one of the values ``create``, ``update`` or ``delete`` depending on whether this latest revision is as a result of an update, creation or deletion.

``modified``
   The date the update, creation or deletion occurred

``url``
    The FULL URL a client should get to obtain the serialisation of the Dataset that matches the serialization of the list of Datasets.    

It ``SHOULD`` also include these attributes if it supports such concepts:

``revision``
    An ID representing the last revision

For example as JSON we might have:

::

    [
        {
            id: "123",
            modified: "2012-01-01 13:34",
            change_type: "update",
            url: http://example.com/rest/dataset/id/123.json
        },
        {
            id: "456",
            modified: "2011-11-21 16:29",
            change_type: "delete",
            url: http://example.com/rest/dataset/id/456.json
        },
        ... etc ...
    ]

Notice that ``url`` is the full URL.



Example
~~~~~~~

Here's an example HTTP response:

::

    HTTP/1.1 200 OK  
    Access-Control-Allow-Origin: *
    Content-Length: 5604
    Content-Type: application/json; charset=utf8  

    [
        {
            id: "123",
            modified: "2012-01-01 13:34",
            change_type: "update",
            url: http://example.com/rest/dataset/id/123.json
        },
        {
            id: "456",
            modified: "2011-11-21 16:29",
            change_type: "delete",
            url: http://example.com/rest/dataset/id/456.json
        },
        ... etc ...
    ]



Help Dataset API Call
---------------------------

If no ``by-entity-attribute`` is specified but a ``/`` character remains on the end of the URL like this:

::

    http://example.com/rest/dataset/

then a 301 redirect ``SHOULD`` be made to ``http://example.com/rest/dataset/help.txt``. Likewise if a request is made to:

::

    http://example.com/rest/dataset

then a 301 redirect ``SHOULD`` also be made to ``http://example.com/rest/dataset/help.txt``

Here is a suitable response for the redirect. No response body is required:

::

    HTTP/1.1 301 Moved Permanently
    Location: http://example.com/rest/dataset/help.txt

When a request is made to the Dataset help URL at ``help.txt``, it ``MUST`` return UTF-8 encoded text that was wrapped to 78 characters and explains how the API for the entity is used.

The help text below ``MAY`` be used but the URLs ``MUST`` be suitably adjusted:

::

    Datasets Help
    
    This API is based on the DCIP specification version 1.0 DRAFT at 
    http://datacanspeak.com/ref/dcip/1.0-draft.html
    
    You can specify the Dataset you wish to return with its ID
    followed by the response format file extension. For example:

        GET http://example.com/rest/dataset/id/123.json
    
    The following file extensions are supported for setting the response
    format:
    
    .json
        The response should be in JSON format
    
    .dcat.N3
        The respose will be in N3 encoded DCAT RDF

    A list of all available Datasets can be found at this URL:

        GET http://example.com/rest/Dataset/id/

These are the HTTP response status's that ``MUST`` be returned given the possible outcomes of the requst:

``200 OK``

    The request was successful and the help text will be returned

``400 Bad Request``

    The request was not understood by the server.

``429 Too Many Requests``

    You have made too many requests too quickly and rate limiting has kicked in.

``500 Internal Server Error``

    The server has crashed trying to fulfil the request

The server ``MAY NOT`` return any other response status.

The ``Content-Type`` header ``MUST`` be set to ``text/plain; charset=utf8`` and the usual ``Content-Length`` and ``Access-Control-Allow-Origin`` headers must be set.

A server ``MAY`` present its help text in markdown format so that it can be parsed and presented as HTML by a client if necessary.

API Help Call
-------------

If no ``entity-type`` is specified and a URL like this is requested:

::

    http://example.com/rest/

then a 301 redirect ``SHOULD`` be made to ``http://example.com/rest/help.txt``. 

Here is a suitable response for the redirect. No response body is required:

::

    HTTP/1.1 301 Moved Permanently
    Location: http://example.com/rest/help.txt

When a request is made to the Dataset help URL at ``help.txt``, it ``MUST``

* return UTF-8 encoded text that was wrapped to 78 characters
* include a link to the catalog info API
* list the Dataset entities available and points to their help URLs

It ``MAY`` also include a description of what the catalog itself is for and
contact information for the catalog maintainer.

The help text below ``MAY`` be used but the URLs ``MUST`` be suitably adjusted:

::

    Welcome to the Catalog Basic REST API.
    
    This API is based on the DCIP specification version 1.0 DRAFT at 
    http://spec.datacatalogs.com/
    
    You can obtain information about this catalog by issuing a GET request to
    one of these URLs

        http://example.com/rest/catalog.json
        http://example.com/rest/catalog.dcat.N3

    The following entity types are exposed by this API:
    
    Dataset
        See http://example.com/rest/dataset/help.txt for information on its use

A server ``MAY`` present its help text in markdown format so that it can be parsed and presented as HTML by a client if necessary.

Extensions
----------

An implementing catalog ``MAY`` extend this specification in three ways:

* by implementing support for more ``entity-types``
* by implementing support for accessing entities by an attribute other than ID
* by returning additional information in the serialised Dataset

It ``MAY NOT``:

* implement alternatives to the specified API (ie the specified API must always be fully supported in its entirety too)
* give new meanings to any existing Dataset attributes

Caching
-------

No caching methodoloy is specified by this specification. It is likely a future specification will recommend Etag caching for both Dataset entities and lists of entities.


5. Entity Schema
================

The Schema is directly based on DCAT_ with some minor recommendations regarding specific usage and serialization.

The following classes from DCAT_ are used: dcat:Dataset and dcat:Distribution
(Resource). The following are optional and are not used by default in the
outline below: dcat:Catalog and dcat:CatalogRecord.

.. note:: Dataset vs Dataset Record. In this specification, the entities we are calling datasets are really objects which contain metadata about some actual data in a distributable form. Implementing catalogs might refer to these entities as "Metadata Records", "Dataset Records" or "Catalog Entries". To be consistent with DCAT and implementations such as CKAN, this specification refers to this metadata as a "Dataset".

Empty or Missing Values
-----------------------

As a guide, where a value is NULL or an empty value, the corresponding key ``SHOULD`` not be present in the serialisation of the dataset record.


Dataset
-------

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

Resources are a dcat:Distribution (and sub-types thereof)::

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

7. References
=============

RFC2119
    S. Bradner. Key words for use in RFCs to Indicate Requirement Levels. March 1997. Internet RFC 2119. URL: http://www.ietf.org/rfc/rfc2119.txt 

DCAT
    Fadi Maali, John Erickson, Phil Archer. Data Catalog Vocabulary (DCAT). URL: http://www.w3.org/TR/vocab-dcat/

8. Appendicies
==============

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

Potential Catalog Entity Attributes
-----------------------------------

The catalog data model simply exists to provide basic information about the catalog itself. Note that we don't call this a ``CatalogRecord`` since in this case the catalog provies information directly about itself and we aren't tracking metadata records about lots of other catalogs.

A Catalog ``MUST`` provide the following information:

::

    {
        id:
        description:
        contact: 
    }


 

