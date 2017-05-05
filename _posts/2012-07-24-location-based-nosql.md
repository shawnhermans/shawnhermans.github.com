---
layout: post
title: "NoSQL Database for location-based app?"
author: Shawn Hermans
date: 2012-07-24 12:00:00 -0600
categories: [gis, nosql]
---
Originally posted as an answer to [Which NoSQL Database would be a good backend for a location-based app?](https://www.quora.com/Which-NoSQL-Database-would-be-a-good-back-end-for-a-location-based-app/answer/Shawn-Hermans?srid=hLq3)

As mentioned before MongoDB and CouchDB both have geospatial support.  Apache Solr and Elasticsearch are search engines which can double as a NoSQL data store.

One option is to use a traditional database (or NoSQL datastore) with Elasticsearch or Solr as an external search index.  Elasticsearch does a slightly better job of doing high volumes of real-time updates.  Elasticsearch supports what are called rivers.  Basically, rivers are a way to update Elasticsearch in real-time.  They have rivers for RSS, Twitter, Mongo, and Couch.
