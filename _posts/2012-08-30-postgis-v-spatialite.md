---
layout: post
title: "PostGIS vs Spatialite"
author: Shawn Hermans
date: 2012-08-30 12:00:00 -0600
categories: [gis, postgres, sqlite, postgis, spatialite]
---
Originally posted as an answer to [How does PostGIS compare to Spatialite?](https://www.quora.com/How-does-PostGIS-compare-to-Spatialite/answer/Shawn-Hermans?srid=hLq3)

PostGIS offers many more features. [GeoDjango's documentation](https://docs.djangoproject.com/en/dev/ref/contrib/gis/db-api/#compatibility-tables) has a comparison of the spatial features it supports from each database.  Please note that this is not a direct comparison of the databases. Rather it reflects what GeoDjango supports.  However, it will still give you a rough idea of the differences between the two.

The choice between PostGIS and Spatialite comes down to your particular use case.  Spatialite is probably the best choice when you need a standalone, embedded database.  For example, maybe you are building a mobile or desktop application.  Also, Spatialite may be used during the development of a web-based application. I have used it during the development of GeoDjango applications.  

PostGIS is the better choice if you need high read/write rates, multiple database users, clustering, network accessibility and any other standard database feature.  

Regarding Spatialite's licensing, the library is licensed under the LGPL and the GUI tools are licensed under the GPL (the same license as MySQL).   This means in most cases, you do not need to worry about the viral nature of GPL.
