---
layout: post
title: "Django Rest Framework, TastyPie, and Piston Compared"
author: Shawn Hermans
date: 2012-07-20 12:00:00 -0600
categories: [python, REST, django, django-rest-framework]
---

Originally posted on [Quora](https://www.quora.com/Can-someone-compare-advantages-and-disadvantages-of-using-Django-REST-framework-Piston-and-Tastypie/answer/Shawn-Hermans?srid=hLq3). Note: In the time between now and when I originally wrote this answer, I have had considerable experience developing using Django Rest Framework and think it is one of the best Django-based API frameworks available.

A while back I implemented a REST-based API in Django.  I looked into a lot of options before I ended up writing my own.

I first started by using Django Piston.  I also looked at Django REST framework, but it seemed to be a little light on documentation. I was able to get a basic REST API up and running in a short period of time.  My primary issue with Piston was that I had a difficult time trying to get it to support custom serialization.  My API needed to support geospatial encoding in both GeoJSON and GeoRSS/GeoAtom.  I tried to get the custom serialization to work in Piston, but did not have much luck.

Additionally, I was using Django Haystack with Elasticsearch within my project.I wanted the API to be able to use Elasticsearch for advanced search and filtering.     

Right now I am in the process of re-evaluting these frameworks. Based upon my current research, I plan to explore TastyPie first.  It seems to have better support for custom serialization methods. Also, it supports geospatial using GeoJSON. Out of the three frameworks it appears to have the best documentation.  They have a section on using Haystack as well.  Lastly, out of all of those options TastyPie has the best name.  

I guess the forth option not listed here is rolling your own without a framework.  I went with this option with mixed results.  The overall experience was useful in that it taught me a lot about designing and implementing REST APIs.

## Summary

### Django Piston

Advantages - Easy to get setup and started.  Works well if you use the default configuration.

Disadvantages - Had a hard time implementing custom serialization methods.

### Django REST Framework

Advantages - I didn't actually use this

Disadvantages - Seems to have the least documentation out of the the three.

### TastyPie

Advantages - Best name out of the three options.  Seems to have the most features for implementing customized serialization and tying into non-ORM sources of data.

Disadvantages - The reason I avoided TastyPie the first time around is it seemed too complex for my needs.  TastyPie might be overkill for simple use cases.

### Rolling your own

Advantages - Path of least resistance if you need to get something up and running for a proof of concept and don't care about having a lot of features.  
Disadvantages - End up doing a lot of rework.  Time is better spent learning or improving existing framework.
