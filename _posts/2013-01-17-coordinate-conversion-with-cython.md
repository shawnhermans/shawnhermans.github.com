---
layout: post
title: "Coordinate Conversion in Cython"
category: introduction
description: ""

tags: [cython, coordinate conversion]
---
{% include JB/setup %}

One of my projects required conversion of [Earth Centered Earth Fixed](http://en.wikipedia.org/wiki/ECEF) coordinates
to geodetic latitude, longitude and height (and vice versa).
I couldn’t find any native Python libraries, so I decided to create a simple library called
[pycoordconvert](https://github.com/shawnhermans/pycoordconvert). I decided this was a good opportunity
to learn more about [Cython](http://cython.org/).

I created two versions of the code. One with Python and one with Cython extensions.
This is the Python version of the code. It uses the built-in Python math libraries and does not
use any explicitly declared types.

{% highlight python %}
from math import atan2, sqrt, cos, sin, degrees, radians


earth_radius = 6378.1*1000
a = 6378137
e2 = 6.6943799901E-3

def normal(lat_radians):
    return a/(sqrt(1-e2*pow(sin(lat_radians),2)))

def geodetic_to_ecef(lon,lat,h):
    lon = radians(lon)
    lat = radians(lat)

    x = (normal(lat)+h)*cos(lon)*cos(lat)
    y = (normal(lat)+h)*cos(lon)*sin(lat)
    z = (normal(lat)*(1-e2)+h)*sin(lat)

    return x,y,z
{% endhighlight %}

This is the equivalent code in Cython. It uses the C math libraries and static typing.

{% highlight python %}
cdef extern from "math.h":
    double cos(double theta)
    double sin(double theta)
    double acos(double theta)
    double sqrt(double theta)
    double atan2(double y,double x)
    double pow(double x,double y)

cdef double earth_radius = 6378.1*1000
cdef double a = 6378137
cdef double e2 = 6.6943799901E-3
cdef double pi = 3.141592653589793
cdef double degrees_per_radian = 57.2957795
cdef double radians_per_degree = 0.0174532925

cdef double degrees(double theta):
    return degrees_per_radian*theta

cdef double radians(double theta):
    return radians_per_degree*theta

cdef double normal(lat_radians):
    return a/(sqrt(1-e2*pow(sin(lat_radians),2)))

cdef object geodetic_to_ecef(lon,lat,h):
    cdef double lon_rads,lat_rads,x,y,z
    lon_rads = radians(lon)
    lat_rads = radians(lat)

    x = (normal(lat_rads)+h)*cos(lon_rads)*cos(lat_rads)
    y = (normal(lat_rads)+h)*cos(lon_rads)*sin(lat_rads)
    z = (normal(lat_rads)*(1-e2)+h)*sin(lat_rads)

    return (x,y,z)

{% endhighlight %}

The reverse transforms can be found in the [pycoordconvert](https://github.com/shawnhermans/pycoordconvert)
repository.

The following setup file enables building of the Cython module.

{% highlight python %}
from distutils.core import setup
from distutils.extension import Extension
from Cython.Distutils import build_ext

setup(
    cmdclass = {'build_ext': build_ext},
    ext_modules = [Extension("cython_coordconvert", ["cython_coordconvert.pyx"])]
)
{% endhighlight %}

Build the extensions using the command `python setup.py build_ext --inplace`.
A quick comparison of the two approaches shows a significant speedup for a small amount of work.

The ECEF to Geodetic ran in 4.41 usec/pass in Cython and 39.81 in CPython for 9x speedup.
The Geodetic to ECEF ran in 1.01 usec/pass in Cython and 4.84 for CPython for a 4.8x speedup.

Overall my first experience with Cython was favorable. It was a lot easier than writing native C code and
provided a huge performance improvement.




