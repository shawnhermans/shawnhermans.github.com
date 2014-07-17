---
layout: post
title: "Pig Tricks"
description: "Cool tricks with Pig"
category: tips
tags: [hadoop, pig, performance]
---
{% include JB/setup %}

# Performance

## Use Replicated JOINs for Small Tables

    big_data = LOAD '/user/example/big-data/' USING PigStorage()
        AS (key1:chararray, values1:map[]);
    small_data = LOAD '/user/example/small-data/' USING PigStorage()
        AS (key2:chararray, values2:map[]);
    joined_data = JOIN big_data BY key1, small_data BY key2 USING 'replicated';

But then

## Use Compression

    SET mapred.output.compress true
    SET mapred.output.compression.codec org.apache.hadoop.io.compress.GzipCodec

What about this?

    SET mapred.output.compress true
    SET mapred.output.compression.codec org.apache.hadoop.io.compress.SnappyCodec
    SET avro.output.codec snappy

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

    SET hbase.client.scanner.caching 1000;

    profiles1 = LOAD 'hbase://profile' USING org.apache.pig.backend.hadoop.hbase.HBaseStorage(
        'e:*','-loadKey true -gt 38700000-0000-0000-0000-000000000000 -lt 39000000-0000-0000-0000-000000000000 -minTimestamp=1401653161000')
        AS (profileId:chararray, events:map[chararray]);

    STORE profiles1 INTO '/tmp/sample-1' USING PigStorage();

    profiles2 = LOAD 'hbase://profile' USING org.apache.pig.backend.hadoop.hbase.HBaseStorage(
        'e:*','-loadKey true -gt 38700000-0000-0000-0000-000000000000 -lt 39000000-0000-0000-0000-000000000000 -maxTimestamp=1401653161000')
        AS (profileId:chararray, events:map[chararray]);

    STORE profiles2 INTO '/tmp/sample-2' USING PigStorage();

# Developing

## Prefer AvroStorage to PigStorage for Primary Data Storage
Stuff

## Pigbootup

    REGISTER 'hdfs://nn1.sojern.com:8020/etl/workspace/lib/avro-1.7.4.jar';
    REGISTER 'hdfs://nn1.sojern.com:8020/etl/workspace/lib/json-simple-1.1.jar';
    REGISTER 'hdfs://nn1.sojern.com:8020/etl/workspace/lib/piggybank.jar';

    DEFINE AvroStorage org.apache.pig.piggybank.storage.avro.AvroStorage();

## Dependency Conflicts

    SET mapreduce.task.classpath.user.precedence true;

    REGISTER 'hdfs://nn1.sojern.com:8020/etl/workspace/campaign-processing/lib/guava-14.0.1.jar';

## Limitations of Macros

1.  Cannot do REGISTER statements

## Working with S3

    SET fs.s3n.awsAccessKeyId XXXXXXXXXXXXXXXXXX
    SET fs.s3n.awsSecretAccessKey xxxxxxxxxxxxxxxxxxxxxx

    REGISTER 's3n://sojern-workflow/lib/piggybank.jar';
    REGISTER 's3n://sojern-workflow/lib/avro.jar';
    REGISTER 's3n://sojern-workflow/lib/json-simple.jar';


# DataFu

    git clone https://github.com/apache/incubator-datafu
    cd incubator-datafu/
    ./gradlew assemble

The built JAR can be found under `datafu-pig/build/libs` by the name `datafu-pig-x.y.z.jar`, where x.y.z is the version.

## Percentile

    REGISTER 'hdfs://nn1.sojern.com:8020/etl/workspace/lib/datafu-1.1.0.jar';

    DEFINE Percentile datafu.pig.stats.Quantile(
        '0.00','0.01','0.02','0.03','0.04','0.05','0.06','0.07','0.08',
        '0.09','0.10','0.11','0.12','0.13','0.14','0.15','0.16','0.17',
        '0.18','0.19','0.20','0.21','0.22','0.23','0.24','0.25','0.26',
        '0.27','0.28','0.29','0.30','0.31','0.32','0.33','0.34','0.35',
        '0.36','0.37','0.38','0.39','0.40','0.41','0.42','0.43','0.44',
        '0.45','0.46','0.47','0.48','0.49','0.50','0.51','0.52','0.53',
        '0.54','0.55','0.56','0.57','0.58','0.59','0.60','0.61','0.62',
        '0.63','0.64','0.65','0.66','0.67','0.68','0.69','0.70','0.71',
        '0.72','0.73','0.74','0.75','0.76','0.77','0.78','0.79','0.80',
        '0.81','0.82','0.83','0.84','0.85','0.86','0.87','0.88','0.89',
        '0.90','0.91','0.92','0.93','0.94','0.95','0.96','0.97','0.98',
        '0.99','1.0');

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

Then show plotting it using Python


## User Agent Parsing
The [ua-parser](https://github.com/tobie/ua-parser/) project is a
multi-language library for parsing user agent strings.  To compile the Pig JAR use the following steps:

    git clone https://github.com/tobie/ua-parser.git
    cd ua-parser/pig/
    mvn package

These steps fetches a copy of the latest code from Github and compiles the Pig JAR.
After the `mvn package` step completes the JAR can be found in `target/ua-parser-pig-0.1-SNAPSHOT-job.jar`.
Move this to the appropriate directory on HDFS or your local filesystem.

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




