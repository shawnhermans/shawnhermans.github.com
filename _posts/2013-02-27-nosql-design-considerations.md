---
layout: post
title: "NoSQL Design Considerations"
author: Shawn Hermans
date: 2017-02-27 12:00:00 -0600
categories: [nosql]
---
Originally posted as an answer to [What are some good resources for people considering using a NoSQL database for a web app?](https://www.quora.com/What-are-some-good-resources-for-people-considering-using-a-NoSQL-database-for-a-web-app/answer/Shawn-Hermans?srid=hLq3)

There are a lot of NoSQL options out there so things like design considerations will depend a lot on the type of NoSQL option and your use cases.  

## Reasons why you might want to use NoSQL

* Your data does not have a well-defined schema.  NoSQL options normally deal with schema-less data a lot better than traditional RDBMS systems.
* You have an application that requires high write rates. As an example, real-time analytics is a good example where a NoSQL solution is a good option.  As an example, something like Redis is going to give you high read/write rates as compared to traditional SQL options.
* Your application requires an advanced full-text search capability. In these situations something like Apache Solr or Elasticsearch might be a good choice.  
* SQL databases generally require more expertise to manage and scale. NoSQL options generally require less expertise. Therefore NoSQL options may be the right choice for a startup effort.
Y* our data is highly linked and you want to do social network analysis.  A graph database might be a good choice.

## Design Considerations
* Most NoSQL databases do not support JOINS very well. This means you will need to avoid many-to-many relationships.  This means you may need to redesign your current schema.
* There are some good object relational mapping tools for NoSQL, but they are not as mature as SQL ORMs.  I generally do web development in Python, so there is Django Nonrel, MongoEngine and I think there are a few others.  I noticed Spring had a few tools available for NoSQL Java options.
* You may want to consider some hybrid approaches depending upon your needs. One of the other posters mentioned combining CouchDB with Redis.  That is a pretty good design pattern. In general using an in-memory cache like Redis or Memcached with a persistent data store works well for a lot of applications.  Or maybe you could use a NoSQL search engine like Elasticsearch with a traditional database.
* NoSQL options vary in their reliability. Some support transactions and some do not.  Most support a model of eventually consistency, but check into the specifics of your NoSQL choice.

If you are looking to replace a traditional SQL database with a NoSQL one, SimpleDB, MongoDB, CouchDB, Google App Engine’s datastore, and HBase are decent options. HBase is probably the most reliable and well supported of them. SimpleDB and Google App Engine are both cloud options.  MongoDB and CouchDB are both document-oriented datastores.  Both are good at storing JSON or JSON-like data structures.  This makes them good for designing REST APIs.  

If you are looking to augment a web app with NoSQL capabilities, Elasticsearch is a good option for adding full-text search.  Redis is a great option for in-memory caching.  Both Elasticsearch and Redis are very simple and have good performance.
