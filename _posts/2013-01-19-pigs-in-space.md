---
layout: post
title: "Pigs in Space"
description: "Introduction to Pig with Space Data"
tags: [hadoop, pig, tle, space catalog] 
---

## Introduction

Alas, the subject of this post is about using Apache Pig to process satellite orbit data.
If you are an orbital analyst and want to know how to easily process data on a Hadoop cluster,
this article is for you. If you are just interested in Pig, this post should give you a good idea of
how to use Pig and write Jython-based Pig user defined functions.

## Setting Up Pig
Pig can run in either local mode or Hadoop mode. I am not going to go into great detail on how to
setup Pig using Hadoop mode. Hadoop mode requires a fully configured Hadoop cluster with MapReduce version 1.
If you are using Cloudera or some other Hadoop distribution, I recommend using whatever release of Pig that comes
bundled with their distribution.

Setting up Pig in standalone mode is much more straightforward. To setup Pig in standalone mode, download the
latest [Pig release](http://pig.apache.org/releases.html). At the time of this writing, the latest Pig
version is 10.1. Next, set the `JAVA_HOME` environment variable. On an Ubuntu based system,
add the following into `$HOME/.profile`.

    JAVA_HOME export JAVA_HOME=/usr/jvm/default-java

Decompress the Pig distribution. Common locations include `/opt/pig` or `/usr/local/pig`.
This directory is the `PIG_HOME` directory. Starting Pig in standalone mode is as simple as executing the
command `$PIG_HOME/bin/pig -x local`. This will drop you into the Pig interactive shell Grunt.
You are now ready to start processing some data in Pig!

## Getting and Preparing the Data
There are a few sources of orbital data available. [Space-Track.org](https://www.space-track.org/) hosts the
full public catalog and contains
roughly 14,000 objects. Access to this data requires registration and may not be readily accessible to all users.
As an alternative, [Celestrak](https://celestrak.com/) hosts publicly available, supplemental TLEs.
I will use these TLEs with the rest of the examples. Because Celestrak uses the same format as Space-Track.org,
these examples should work equally as well against the full satellite catalog.
Download the GPS ops TLEs. Despite what the name two-line element set would have you believe, most element sets
come with three lines. The first line is the name of the object and the other two lines are the traditional TLE
orbital elements. A typical file looks like this.

    GPS BIIA-10 (PRN 32)
    1 20959C 90103A   13019.82648148  .00000000  00000-0  ...
    2 20959  54.4316 229.8752 0117400 335.3880   9.5445 ...
    GPS BIIA-14 (PRN 26)
    1 22014C 92039A   13019.82648148  .00000000  00000-0  ...
    2 22014  56.1808 290.0954 0208678  69.4189  64.1323  ...
    .
    .
    .

The problem with this format is that Pig, and most other data processing tools, assume one record per line.
I could develop a custom Pig file loader, but that will have to wait for a later time. Instead,
I created a simple Python script to convert the traditional format into a Pig friendly
tab-separated value format.

```python
f_out = open('gps-ops.tsv', 'wb')
output_line = ''

with open('gps-ops.txt') as f:
	for line_number, line in enumerate(f):
		if line_number % 3 !=2:
			output_line += line.strip() + '\t'
		else:
			output_line += line
			f_out.write(output_line)
			output_line = ''

f_out.close()
```

After processing, the file now looks like this.

    GPS BIIA-10 (PRN 32)        1 20959U...8413 2 20959...13
    GPS BIIA-14 (PRN 26)        1 22014U...6335 2 22014...17
    .
    .
    .

The data is now in a Pig friendly format.

## Loading the Data
Our first step is loading the data into Pig. I suggest creating a Pig workspace folder. Copy all data files and
scripts into this directory. Then, while in this directory, execute `pig -x local`.

```
grunt> gps = LOAD 'gps-ops.tsv' USING PigStorage()
    AS (name:chararray, line1:chararray, line2:chararray);
```

This statement creates a relation called `gps`. It loads data from the local file `gps-ops.tsv`.
`PigStorage()` looks for the data on the local file system if in local mode.
It loads from the HDFS filesystem if run in MapReduce mode.
Loading the data as `(name:chararray, line1:chararray, line2:chararray)` tells Pig there are three fields and these
fields are all strings.
If I left that last part off, Pig would still load the data, but would interpret the columns as untyped byte arrays.
This is part of Pig’s philosophy of “pigs eat anything”. If you have type information, Pig will gladly use it.
If you don’t, Pig doesn’t care.

At this point, let’s see what Pig has actually done.
The `DESCRIBE` command will tell us information on the relation.
This is not too useful right now, but comes in handy when dealing with derived relationships.

```
grunt > DESCRIBE gps;
gps: {name: chararray,line1: chararray,line2: chararray}
```

We can take a look at the data using the DUMP command.

```
grunt > DUMP gps;

(GPS BIIA-10 (PRN 32),1 ... -3 0  8413,2 ...  2.00550870162199)
(GPS BIIA-14 (PRN 26),1 ... -3 0  6335,2 ... 2.00565888143921)
.
.
.
```

## Creating a Python User Defined Function
While the data is currently in Pig, it requires further parsing before we can do anything useful with it.
The data is encoded in the positions within the string. For example, columns 15-17 in line 1 represent the
international designator. Lines 45-52 are the second time derivative of mean motion divided by six with
the decimal point assumed. We can’t do much else with the data until we parse it.

While it may be possible to parse this using built-in Pig capabilities, it is a whole lot easier parsing this is
something like Python. Luckily, Pig supports creating user defined functions (UDFs) using Java, JavaScript,
Python and Ruby.

Pig UDFs can use most external Python libraries provided they do not have any C dependencies.
This eliminates popular data processing libraries such as Lxml and NumPy. Additionally, external libraries
cannot rely on Python features found in releases after 2.5. This is because Pig UDFs use Jython to interpret
Python UDFs. The upside of using Jython is that Pig UDFs can call Java classes as if they were native Python
libraries. I will demonstrate this feature later on.

I created a simple UDF called parseTle that takes in the name, line1, and line2 data and parses it into a map.
A map is one of Pig’s complex data types and is equivalent to a Python dictionary.

```python
import tleparser

@outputSchema("params:map[]")
def parseTle(name, line1, line2):
    params = tleparser.parse_tle(name, line1, line2)
    return params

```

The UDF is pretty simple to understand. It imports an external library called
[tleparser](https://gist.github.com/shawnhermans/4569360) to do most of the heavy lifting.
The `@outputSchema("params:map[]")` annotation tells Pig this function outputs a map named params.
The easiest way to use this UDF is to place the UDF script and all dependencies in the working folder.
Create the files `tleUDFs.py` and `tleparser.py` in this folder and restart the Grunt console.

```
grunt> gps = LOAD 'gps-ops.tsv' USING PigStorage()
    AS (name:chararray, line1:chararray, line2:chararray);
grunt> REGISTER 'tleUDFs.py' USING jython AS myfuncs;
grunt> parsed = FOREACH gps GENERATE myfuncs.parseTle(*);
```
This Pig command goes through each of the relations in gps and applies the `parseTle` UDF.
The parsed data is now a map of key/value pairs. The following is an example parsed TLE data.

```
(
    [bstar#,arg_of_perigee#333.0924,
    mean_motion#2.00559335, element_number#72,
    epoch_year#2013,inclination#54.9673,
    mean_anomaly#26.8787,rev_at_epoch#210,
    mean_motion_ddot#0.0, eccentricity#5.354E-4,
    two_digit_year#13,international_designator#12053A,
    classification#U,epoch_day#17.78040066,
    satellite_number#38833,
    name#GPS BIIF-3  (PRN 24),
    mean_motion_dot#-1.8E-6,ra_of_asc_node#344.5315]
)
```

Pig uses the `#` character to separate key/value pairs.
Having the data parsed, means we can use the power of Pig to filter, process, and otherwise manipulate it.
The following is an example of using the `FILTER` command to find all orbits with inclination between 53 and 56.

```
filtered = FILTER parsed BY (params#'inclination' > 53) 
    AND (params#'inclination' < 56);
```

At this point, we can store the results of this processing to the filesystem using the `STORE` command.

```
STORE filtered into '/tmp/filtered' using PigStorage();
```

After running this command, there should be a folder named `/tmp/filtered` with a file called part-m-00000.
For larger data sets with more reducers, this folder will contain multiple files using the same naming convention.
This file is a tab-separated representation of the filtered data.
This data can be loaded back into Pig using the PigStorage option.

#Making a Pig Script
Up until now, all of the examples used the Grunt interactive console.
This is very useful for conducting data exploration,
but not very useful for doing repeatable tasks. Luckily, creating a Pig script is as easy as saving those commands
into a text file and executing it using the Pig commandline.

To create really useful scripts we want to vary certain parameters at runtime.
This is accomplished using Pig parameter substitution. Save the previous commands into a file called `filter.pig`,
but change the last line to include the following.

```
REGISTER 'tleUDFs.py' USING jython AS tleUDFs;

gps = LOAD '$input' USING PigStorage() AS 
    (name:chararray, line1:chararray, line2:chararray);
    
parsed = FOREACH gps GENERATE tleUDFs.parseTle(*);

filtered = FILTER parsed BY
    (params#'inclination' > $lowerInclination) AND
    (params#'inclination' < $upperInclination);

STORE filtered into '$output' using PigStorage();
```

This allows the user to pass in lower and upper inclination bounds. It also stores the
results into a user defined output directory. We pass those parameters using the commandline `-param` option.

    pig -x local -param input='gps-ops.tsv' -param lowerInclination=53 \
    -param upperInclination=56 -param output='/tmp/filtered' filter.pig

#Using Java in Python UDFs
By themselves, orbital elements are not terribly useful. The most common use case for
TLEs is to use the TLEs to propagate out predicted positions and velocities for objects in space.
This requires access to a propagator that implements all the relevant mathematics and physics.
Ideally, we want to create UDF that propagates out the TLE.
Usually, when I want to propagate a TLE in Python, I use [PyEphem](http://rhodesmill.org/pyephem/).

Unfortunately, I can’t use PyEphem to write Pig UDFs. Pig uses Jython to interpret Python UDFs.
Using Jython means you can’t use any Python code requiring C extensions.
It also means you are limited to using standard library functions found in Python 2.5 as Jython does
not support the latest versions of Python. On the other hand, Jython allows for seamless integration
with existing Java code. This means you can reuse existing Java libraries or rewrite
computationally intensive code in Java.

This UDF uses the JSatTrak Java library to propagate out the TLE.
To use this UDF download
the [JSatTrak binary distribution](http://www.gano.name/shawn/JSatTrak/bin_releases/JSatTrak_v4_1_bin.zip).
Next copy the JARs from this distribution somewhere onto the Pig classpath.
The easiest way to do this is copying the JARs into the `$PIG_HOME/lib` directory.

```python
from jsattrak.objects import SatelliteTleSGP4

@outputSchema("""
    propagated:bag{
        positions:tuple(
            time:double, 
            x:double, 
            y:double, 
            z:double
        )
    }
""")
def propagateTleECEF(
    name,
    line1,
    line2,
    start_time,
    end_time,
    number_of_points
    ):
    
    try:
        satellite = SatelliteTleSGP4(name, line1, line2)
    except:
        return None

    ecef_positions = []
    
    end_time = float(end_time)
    start_time = float(start_time)
    number_of_points = float(number_of_points)

    increment = (end_time-start_time)/number_of_points
    current_time = start_time

    while current_time <= end_time:
        try:
            positions = [current_time]
            current_positions = satellite.calculateJ2KPositionFromUT(current_time)
            current_positions = list(current_positions)
            positions.extend(current_positions)
            ecef_positions.append(tuple(positions))
        except:
            pass

        current_time += increment

    return ecef_positions
```

This UDF takes a start time, end time and the number of desired points and outputs a bag of tuples.
A bag is similar to a Python list with the exception that bags in Pig are always unordered.
Tuples in Pig are like tuples in Python. They are immutable and ordered.

After adding the JARs to the classpath and adding the `propagateTleECEF`
function to `tleUDFs.py`, we are ready to propagate some orbits.

```
grunt > REGISTER 'tleUDFs.py' USING jython AS myfuncs;
grunt> gps = LOAD 'gps-ops.tsv' USING PigStorage()
    AS (name:chararray, line1:chararray, line2:chararray);
grunt> propagated = FOREACH gps GENERATE 
    myfuncs.parseTle(name, line1, line2),
    myfuncs.propagateTleECEF(
        name, line1, line2, 2454992.0, 2454993.0, 100
        );
grunt> flattened = FOREACH propagated GENERATE
    params#'satellite_number', FLATTEN(propagated);
```

The third command propagates out points for each satellite while the fourth command
flattens the results so that each position for each satellite is on its own line.
The `FLATTEN` operator will take a bag or tuple and flatten it. This operator works differently
based on whether it is flattening a bag or a tuple. Tuples are flattened in place in will not
generate additional relations. Bags are flattened into multiple additional relations.
The `DESCRIBE` command shows the new structure of each relation.

```
grunt> DESCRIBE propagated;
propagated: {params: map[],propagated:
    {positions: (time: double,x: double,y: double,z: double)}}
grunt> DESCRIBE flattened;
flattened: {bytearray,
    propagated::time: double,propagated::x:
    double,propagated::y: double,propagated::z: double}
```

Notice that the flattened relation has an unnamed, untyped field for its first item.
This is the satellite number, but Pig does not know its type or name so it treats it as an
untyped byte array. We can fix this by using the AS operator to specify its name and type.
Looking at the data using the dump flattened command shows the positions.

```
(38833,2454992.9599999785,2.278136816721697E7,7970303.195970464,-1.1066153998664627E7)
(38833,2454992.9699999783,2.2929498370345607E7,1.0245812732430315E7,-8617450.742994161)
(38833,2454992.979999978,2.2713614118860725E7,1.2358665040019082E7,-6031915.392826946)
(38833,2454992.989999978,2.213715624812226E7,1.4275325605036272E7,-3350605.7983842064)
(38833,2454992.9999999776,2.1209296863515433E7,1.5965381866069315E7,-616098.4598421039)
```

At this point we can store off the propagated data so we can use it for future processing in Pig or in an
external program. Using `STORE` flattened into 'propagated' using `PigStorage(',')`
stores the data in the folder ‘propagated’ using comma-separated value files. Pig has other
storage options as well such as HBase or JSON. Pig also allows development of custom storage drivers.
We can load this data back into Pig using `LOAD`.

```
grunt> propagated = LOAD 'propagated'
    using PigStorage(',') AS 
    (satelliteNumber:int, time:double, x:double, y:double, z:double);
```

Using the ‘AS’ operator allows us to define the schema for the data.

```
grunt> DESCRIBE propagated;
propagated: {satelliteNumber: int,time: double,x: double,y: double,z: double}
```

## Conclusion
Overall, Pig made it easy to process orbital data without needing to resort to cumbersome custom MapReduce code.
It handles all the boring details of creating and scheduling MapReduce jobs.
Pig is a valuable tool and will be a critical part of my data processing toolbox.

This post only scratches the surface of the true power of Pig. As an example,
I started down the path of developing a simple conjunction assessment algorithm
(conjunctions are when satellites in space get too close to one another and go boom)
using a Python UDF. It ran fine against small data sets, but ran out of heap memory on larger data sets.
I suspect I should be able to fine tune the processing to run efficiently against the whole data set.



