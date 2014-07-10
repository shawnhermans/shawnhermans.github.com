---
layout: post
title: "Pig Tricks"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

# Performance

## Use Replicated JOINs for Small Tables

```
big_data = LOAD '/user/example/big-data/' USING PigStorage() AS (key1:chararray, values1:map[]);
small_data = LOAD '/user/example/small-data/' USING PigStorage() AS (key2:chararray, values2:map[]);
joined_data = JOIN big_data BY key1, small_data BY key2 USING 'replicated';
```

But then
## Use Compression

```
SET mapred.output.compress true
SET mapred.output.compression.codec org.apache.hadoop.io.compress.GzipCodec
```

What about this?

```
SET mapred.output.compress true
SET mapred.output.compression.codec org.apache.hadoop.io.compress.SnappyCodec
SET avro.output.codec snappy
```

And some more

# Map side cross

profiles_with_join_key = FOREACH labeled_profiles GENERATE *, 1 AS joinKey:int;
grouped_campaign_info_with_join_key = FOREACH grouped_campaign_info GENERATE campaign_info AS campaign_info, 1 AS joinKey:int;


profiles_joined_with_campaign_info = JOIN profiles_with_join_key BY joinKey,
	grouped_campaign_info_with_join_key BY joinKey USING 'replicated';

flatten_campaign_info = FOREACH profiles_joined_with_campaign_info GENERATE jsonProfile, labels,
	FLATTEN(campaign_info);

# User Defined Functions
Text

## Avoid using Jython UDFs across multiple scripts
More text

## Limitations of Jython standard libraries

1.  datetime
2.  json

# Testing
Testing is great.

## HBase Scripts
Why doesn't this work?

```
SET hbase.client.scanner.caching 1000;

profiles1 = LOAD 'hbase://profile' USING org.apache.pig.backend.hadoop.hbase.HBaseStorage(
    'e:*','-loadKey true -gt 38700000-0000-0000-0000-000000000000 -lt 39000000-0000-0000-0000-000000000000 -minTimestamp=1401653161000')
    AS (profileId:chararray, events:map[chararray]);

STORE profiles1 INTO '/tmp/sample-1' USING PigStorage();

profiles2 = LOAD 'hbase://profile' USING org.apache.pig.backend.hadoop.hbase.HBaseStorage(
    'e:*','-loadKey true -gt 38700000-0000-0000-0000-000000000000 -lt 39000000-0000-0000-0000-000000000000 -maxTimestamp=1401653161000')
    AS (profileId:chararray, events:map[chararray]);

STORE profiles2 INTO '/tmp/sample-2' USING PigStorage();
```

# Developing

## Prefer AvroStorage to PigStorage for Primary Data Storage
Stuff

## Pigbootup

```
REGISTER 'hdfs://nn1.sojern.com:8020/etl/workspace/lib/avro-1.7.4.jar';
REGISTER 'hdfs://nn1.sojern.com:8020/etl/workspace/lib/json-simple-1.1.jar';
REGISTER 'hdfs://nn1.sojern.com:8020/etl/workspace/lib/piggybank.jar';

DEFINE AvroStorage org.apache.pig.piggybank.storage.avro.AvroStorage();
```

## Dependency Conflicts


```
SET mapreduce.task.classpath.user.precedence true;

REGISTER 'hdfs://nn1.sojern.com:8020/etl/workspace/campaign-processing/lib/guava-14.0.1.jar';
```

## Limitations of Macros

1.  Cannot do REGISTER statements

## Working with S3

SET fs.s3n.awsAccessKeyId XXXXXXXXXXXXXXXXXX
SET fs.s3n.awsSecretAccessKey xxxxxxxxxxxxxxxxxxxxxx

REGISTER 's3n://sojern-workflow/lib/piggybank.jar';
REGISTER 's3n://sojern-workflow/lib/avro.jar';
REGISTER 's3n://sojern-workflow/lib/json-simple.jar';


# DataFu

```
git clone https://github.com/apache/incubator-datafu
cd incubator-datafu/
./gradlew assemble
```

The built JAR can be found under `datafu-pig/build/libs` by the name `datafu-pig-x.y.z.jar`, where x.y.z is the version.

## Percentile




```
REGISTER 'hdfs://nn1.sojern.com:8020/etl/workspace/lib/datafu-1.1.0.jar';

DEFINE Percentile datafu.pig.stats.Quantile(
    '0.0','0.010000','0.020000','0.030000','0.040000','0.050000','0.060000','0.070000','0.080000','0.090000',
    '0.100000','0.110000','0.120000','0.130000','0.140000','0.150000','0.160000','0.170000','0.180000',
    '0.190000','0.200000','0.210000','0.220000','0.230000','0.240000','0.250000','0.260000','0.270000',
    '0.280000','0.290000','0.300000','0.310000','0.320000','0.330000','0.340000','0.350000','0.360000',
    '0.370000','0.380000','0.390000','0.400000','0.410000','0.420000','0.430000','0.440000','0.450000',
    '0.460000','0.470000','0.480000','0.490000','0.500000','0.510000','0.520000','0.530000','0.540000',
    '0.550000','0.560000','0.570000','0.580000','0.590000','0.600000','0.610000','0.620000','0.630000',
    '0.640000','0.650000','0.660000','0.670000','0.680000','0.690000','0.700000','0.710000','0.720000',
    '0.730000','0.740000','0.750000','0.760000','0.770000','0.780000','0.790000','0.800000','0.810000',
    '0.820000','0.830000','0.840000','0.850000','0.860000','0.870000','0.880000','0.890000','0.900000',
    '0.910000','0.920000','0.930000','0.940000','0.950000','0.960000','0.970000','0.980000','0.990000',
    '1.0');

all_events = LOAD '/etl/smp/flume-avro/2014/03' USING AvroStorage();
events = LOAD '/etl/smp/flume-avro/2014/03/31' USING AvroStorage();

no_conversions_or_impressions = FILTER all_events BY (eventType != 'CREDITED_CONVERSION' AND eventType != 'IMPRESSION' AND eventType != 'CONVERSION');

events_with_conversions = JOIN  no_conversions_or_impressions BY profileId, conversions BY profileId;
grouped = GROUP events_with_conversions BY conversions::profileId;

counted = FOREACH grouped GENERATE COUNT($1) AS eventCount;

group_all = GROUP counted ALL;

stats = FOREACH group_all {
 sorted = ORDER counted BY eventCount;
 GENERATE Percentile(sorted.eventCount);
};

dump stats;
```

Then show plotting it using Python


## User Agent Parsing
The [ua-parser](https://github.com/tobie/ua-parser/) project is a
multi-language library for parsing user agent strings.  To compile the Pig JAR use the following steps:

```
git clone https://github.com/tobie/ua-parser.git
cd ua-parser/pig/
mvn package
```

These steps fetches a copy of the latest code from Github and compiles the Pig JAR.
After the `mvn package` step completes the JAR can be found in `target/ua-parser-pig-0.1-SNAPSHOT-job.jar`.
Move this to the appropriate directory on HDFS or your local filesystem.

```
REGISTER 'ua-parser-pig-0.1-SNAPSHOT-job.jar';

DEFINE DeviceFamily     ua_parser.pig.device.Family;
DEFINE OsFamily         ua_parser.pig.os.Family;
DEFINE OsMajor          ua_parser.pig.os.Major;
DEFINE OsMinor          ua_parser.pig.os.Minor;
DEFINE OsPatch          ua_parser.pig.os.Patch;
DEFINE OsPatchMinor     ua_parser.pig.os.PatchMinor;
DEFINE UserAgentFamily  ua_parser.pig.useragent.Family;
DEFINE UserAgentMajor   ua_parser.pig.useragent.Major;
DEFINE UserAgentMinor   ua_parser.pig.useragent.Minor;
DEFINE UserAgentPatch   ua_parser.pig.useragent.Patch;

browser_data = LOAD '/user/example/browser-data' USING PigStorage() AS userId:chararray, userAgentString:chararray;

browsers = FOREACH browser_data GENERATE 
    userId,
	DeviceFamily(userAgentString)	AS DeviceFamily:chararray,
	OsFamily(userAgentString) AS OsFamily:chararray,
	OsMajor(userAgentString) AS OsMajor:chararray,
	OsMinor(userAgentString) AS OsMinor:chararray,
	OsPatch(userAgentString) AS OsPatch:chararray,
	OsPatchMinor(userAgentString) AS OsPatchMinor:chararray,
	UserAgentFamily(userAgentString) AS UserAgentFamily:chararray,
	UserAgentMajor(userAgentString) AS UserAgentMajor:chararray,
	UserAgentMinor(userAgentString) AS UserAgentMinor:chararray,
	UserAgentPatch(userAgentString) AS UserAgentPatch:chararray,
	eventType, data#'campaignId' AS campaignId;
```
This code extracts

## Sessionization

Foo

## Training Models with Mahout

See if you can find the old code for this. Will need to attach the Java code

# Style

aliases - snake case
fields - lower camel case
Java UDFs - upper camel case
Jython UDFs - lower camel case




