---
title: Data Catalog Interoperability Protocol
layout: default
version: 1.0-beta-3
last_update: 28 January 2014
created: 16 April 2012
author:
  - Adria Mercader (Open Knowledge Foundation,
  - Rufus Pollock (Open Knowledge Foundation),
  - James Gardner (3aims)
---


<div class="alert">
This draft is a work in progress. Comments and discussion are greatly welcomed, either:

<ul>
  <li>Adding a comment to <a href="https://docs.google.com/a/okfn.org/document/d/1JFgk-U7so4V0aecut53JS2GPtI2tgFMShUvLoINyKQY/edit#">this document</a></li>
  <li>Emailing the <a href="{{site.mailinglist}}">mailing list</a></li>
  <li>Creating an issue on the <a href="{{site.github}}/issues">issue tracker</a></li>
</ul>
</div>

{% include meta.html %}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

# Introduction

## Context and Overview

With the emergence of Open Data initiatives around the world, the need to share
metadata across different catalogs has became more evident. Sites like
<http://publicdata.eu> aggregate datasets from different portals, and there has
been a growing demand to provide a clear and standard interface to allow
incorporating metadata into them automatically.

There is growing consensus around [DCAT][] being the right way forward, but an
actual implementation is needed. In designing the following guidelines, the
main requirement has been in all cases to keep it extremely simple, making as
easy as possible for catalogs to implement them, both in terms of exposing its
metadata and consuming metadata from other catalogs.

This extension offers the following Implementation Guidelines:

* A serialization format for dataset metadata in JSON and RDF/XML format, both
  based on standard DCAT properties.
* A simple mechanism for exposing a catalog metadata dumps, with optional
  methods for pagination and filtering for large catalogs

A working implementation of this proposal for CKAN catalogs is being developed
at the following location, within the context of the [PublicData.eu] project: <https://github.com/okfn/ckanext-dcat>

[DCAT]: http://www.w3.org/TR/vocab-dcat/
[PublicData.eu]: http://publicdata.eu/

## How does this relate to the Project Open Data metadata schema?

The approach of this proposal is similar to the one described on [Project Open
Data (POD)][pod] for US agencies, where a schema is provided for dumping the catalog
metadata in a JSON file. The basic core fields in POD schema are also based on
DCAT, so those are mostly compatible with the ones on this proposal. On some
other cases this proposal follows a closer approach to the DCAT specification
and the DCAT Application profile for data portals in Europe[^3], for instance
allowing multiple resources (distributions) for each dataset.

[pod]: http://project-open-data.github.io/schema/
[^3]: See <https://joinup.ec.europa.eu/asset/dcat_application_profile/description>

## How does this relate to previous version of this proposal?

As mentioned on the previous section, the main focus of the new version has
been simplicity. Even though the previous version of this proposal provides a
more complete and feature-rich protocol, the reality is that in most cases it
needs to be a much lower entry barrier for organizations publishing data
online.

We think that this proposal is simple enough for most publishers to implement,
and at the same time flexible enough to expand it in the future or to built on
top of it if necessary.

# Proposal 

## Overview

A catalog specifies an endpoint URL for each serialization format where it
serves its datasets metadata, for instance:

    http://example.com/data.json 
    http://example.com/data.rdf

These return a dump of all or a subset of the catalog’s datasets
representations based on DCAT, either in JSON or XML-RDF form.

Catalogs with a large number of datasets can optionally implement a simple
paging mechanism and limit the results to datasets modified since a certain
date to aid client applications.

## Datasets Serialization Format

The serialization of the catalog’s datasets metadata is based on DCAT. Please
refer to the specification for usage of each field. Other core DCAT fields
still need to be added to this serialization (see Issues to discuss).

A simple example in JSON form could be:

    {
      "id": "http://example.com/data/test-dataset-1",
      "title": "A test dataset on your catalogue",
      "description": "A longer description of the dataset",
      "landingPage" : “http://url.to.dataset.home”,
      "issued": "2012-05-10",
      "modified": "2012-05-10T21:04",
      "language": ["en", "es", "ca"],
      "publisher": {
        "name": "Name of the Publishing Organization",
        "mbox": "contact@some.org"
      }, 
      "keyword": ["stats", "pollution”],
      "distribution": [
        {
          "title": "Test resource CSV file",
          "description": "A longer description of this file",
          "format": "text/csv",
          "downloadURL": "http://url.to.csv.file",
          "license": "https://url.to.license"
        },
        {
          "title": "Test resource HTML page",
          "description": "A longer description of this page",
          "format": "text/html",
          "accessURL": "http://url.to.html.page",
          "license": "https://url.to.license"
        }
      ]
    }

The same example on XML-RDF form:

    <dcat:Dataset rdf:about="http://example.com/data/test-dataset-1">
      <dct:identifier>http://example.com/data/test-dataset-1</dct:identifier>
      <dct:title>A test dataset on your catalogue</dct:title>
      <dct:description>A longer description of the dataset</dct:description>
      <dcat:landingPage>http://url.to.dataset.home</dcat:landingPage>
      <dct:issued>2012-05-10</dct:issued>
      <dct:modified>2012-05-10T21:04</dct:modified>
      <dc:language>en</dc:language>
      <dc:language>es</dc:language>
      <dc:language>ca</dc:language>
      <dct:publisher>
        <foaf:Organization>
          <foaf:name>Name of the Publishing Organization</foaf:name>
          <foaf:mbox>contact@some.org</foaf:mbox>
        </foaf:Organization>
      </dct:publisher>
      <dcat:keyword>stats</dcat:keyword>
      <dcat:keyword>pollution</dcat:keyword>
      <dcat:distribution>
        <dcat:Distribution>
          <dct:title>Test resource CSV file</dct:title>
          <dct:license>https://url.to.license</dct:license>
          <dcat:downloadURL>http://url.to.csv.file</dcat:downloadURL>
          <dct:format>
            <dct:IMT>
              <rdf:value>text/csv</rdf:value>
            </dct:IMT>
          </dct:format>
        </dcat:Distribution>
      </dcat:distribution>
      <dcat:distribution>
        <dcat:Distribution>
          <dct:title>Test resource HTML file</dct:title>
          <dct:license>https://url.to.license</dct:license>
          <dcat:accessURL>http://url.to.html.page</dcat:accessURL>
          <dct:format>
            <dct:IMT>
              <rdf:value>text/html</rdf:value>
            </dct:IMT>
          </dct:format>
        </dcat:Distribution>
      </dcat:distribution>
    </dcat:Dataset>


### Items for discussion

* Optionally add @context, @type, @id to the JSON representation to make it
  JSON-LD?
* Ids: RDF identifier http://catalog/dataset/id vs CKAN uuid. Which one to use
  for harvested datasets?
* dcat:Dataset issued/updated vs dcat:CatalogRecord issued/updated. Can we have
  them on the same object?
* Add rest of fields: spatial, temporal, accrualPeriodicity, theme
* Support multilingual properties on XML-RDF (only extract English by default).
  eg XML-RDF, what about JSON:

        <dct:title xml:lang="en">Gross value added by industry</dct:title>
        <dct:title xml:lang="es">Valor a&#xF1;adido bruto por rama de actividad</dct:title>
        <dct:title xml:lang="ca">Valor afegit brut per branca d'activitat</dct:title>

## Basic Request

When this endpoint receives a GET request it should return the listing of all
or a limited subset of the datasets in either JSON or XML-RDF format depending
on the endpoint.

http://example.com/data.json 
http://example.com/data.rdf

The JSON listing should be a list of JSON objects. For example an endpoint that
contains 3 datasets:

    [{"title": "example1",
      …
     },{"title": "example2",
      …
     },{"title": "example3",
      …
     }]

The XML-RDF listing is listing of dcat:Datatsets classes under the root RDF
tag.  For example this endpoint contains 3 datasets.

    <rdf:RDF xmlns:foaf="http://xmlns.com/foaf/0.1/" xmlns:owl="http://www.w3.org/2002/07/owl#" xmlns:rdfs="http://www.w3.org/2000/01/rdf-schema#" xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
    xmlns:dcat="http://www.w3.org/ns/dcat#"
    xmlns:dct="http://purl.org/dc/terms/">

      <dcat:Dataset rdf:about="http://example.com/dataset/example1">
       …
      </dcat:Dataset>

      <dcat:Dataset rdf:about="http://example.com/dataset/example2">
      …
      </dcat:Dataset>

      <dcat:Dataset rdf:about="http://example.com/dataset/example3">
      …
      </dcat:Dataset>

    </rdf:RDF>

The datasets should be ordered if possible with the latest modified metadata
first.

### Items for discussion

Do we want lists of elements or a root catalog object, eg JSON

    {
      "title": “Catalog title”,
      "homepage": “http://url.to.catalog”,
      ...  
      “dataset”: [
        {"title": "example1",
          …
         },{"title": "example2",
          …
         },{"title": "example3",
          …
         }]
    }

XML-RDF

    <rdf:RDF [...]>
      <dcat:Catalog rdf:about="https://data.some.org/catalog">
        <dct:title>Catalog title</dct:title>
        <foaf:homepage>https://data.some.org/the/actual/catalog</foaf:homepage>
        …
        <dcat:dataset>
          <dcat:Dataset rdf:about="http://example.com/dataset/example1">
           …
          </dcat:Dataset>
          <dcat:Dataset rdf:about="http://example.com/dataset/example2">
          …
          </dcat:Dataset>
          <dcat:Dataset rdf:about="http://example.com/dataset/example3">
          …
          </dcat:Dataset>
        </dcat:dataset>
      </dcat:Catalog>
    </rdf:RDF>


## Extra Parameters

### Page

If only a subset of all the datasets in the catalog are returned when accessing
the endpoint, it needs to accept a page parameter, i.e:

http://example.com/data.json?page=2

The page parameter should start at 1 so http://example.com/data.json is
equivalent to http://example.com/data.json?page=1

The amount of items per page will be specified by how many results are on the
first page.  So if http://example.com/data.json has 100 results then
http://example.com/data.json?page=2 should have at most 100 results too.

In order to get more datasets, the page parameter will be incremented
sequentially until one of the following conditions arises:

* The previous page is exactly the same as the last page (this will happen if
  an instance does not support the page parameter at all)
* The page receives a 404
* There are no datasets on the following page i.e the result is an empty JSON
  list or there are no contents in the RDF tag.

### Modified Since

In order to speed up repeated harvests, the endpoint can have a parameter that
specifies that only metadata that has been modified after a certain date should
be returned. It should be specified in ISO 8601 format, for example:

    http://example.com/data.json?modified_since=2010-01-31

----

# Appendix - Contributors

This proposal reflects the contribution of many people over a substantial
period. In particular, there are the participants at the first <a
href="http://lod2.okfn.org/2011/05/04/notes-from-data-catalogues-interoperability-workshop-edinburgh-3-4th-may-2011/">Data
Catalog Interoperability Workshop</a> in Edinburgh in May 2011 (organized under
the auspices of the LOD2 project), as well as the members of the <a
href="http://lists.okfn.org/mailman/listinfo/data-catalogs">data-catalogs group
and mailing list</a> where this specification has been discussed on an ongoing
basis. Among those who have provided expert feedback or input are: Phil Archer
(W3C), Martin Alvarez-Espinar (CTIC), Richard Cyganiak (DERI), John Erickson
(RPI), Chris Gutteridge (Southampton University), Jim Hendler (RPI), Faadi Mali
(DERI), Ed Summers (Library of Congress)


 

