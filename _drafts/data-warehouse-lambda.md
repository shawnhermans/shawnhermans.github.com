---
layout: post
title: "Data Warehousing with Hadoop and the Lambda Architecture"

tags: [metadata]
---

I’ve worked on a lot of big data integration projects.  Some involved integrating modern systems with systems 
designed in the 60’s and 70’s.  

In computing, a data warehouse (DW, DWH), or an enterprise data warehouse (EDW), is a system used for reporting 
and data analysis. Integrating data from one or more disparate sources creates a central repository of data, a 
data warehouse (DW). Data warehouses store current and historical data and are used for creating trending 
reports for senior management reporting such as annual and quarterly comparisons.

## Data Integration in the Relational Model

Teradata, Oracle, etc… provide wonderful tools.  

Relational databases are great for well structured, well understood, homogenous data sources.  
In reality, most data is messy.  

Story time.  Say we want to integrate data from two systems.  

We want to have one unified way of looking at this data.  

There are semantic mismatches.  

Spend weeks arguing over the interpretation of a single field.  

Now, we want to add in a third.  

Here is how a typical data integration effort goes.  

Let’s consider a fictional social media company called REALLY CATCHY NAME.  Recently, your company bought a few 
smaller startups and you want to combine their data with yours in order to target users for advertisements.  
You are the lead data engineer in charge of designing and implementing the data warehouse to enable this analysis.

> **Note:** While this is a fictional example, it mirrors my real world experience participating in multiple data 
integration projects.  I would rather use a real example, but confidentiality concerns prevent this.  

You decide to integrate all this data into a blah, blah, blah … traditional data warehouse. 

Everyone agrees that it makes sense clean up and combine the data to make it easier to
use for analysts and scientists.  There should be a single, correct view of the 
integrated data.  After all, each site's data is highly detailed and specialized.  We don't want them
going out and making mistakes because they misunderstood the data.  
 
You decide the best approach is to bring together the experts from each system and figure out the 
best way to organize the data into relational tables. 
To kick things off, you have a big meeting containing the experts from each site, data scientists, and business analysts.  
Everyone is so excited about being able to integrate all this data.  Business analysts are excited about being 
able to use location data from different services to identify and target local markets.  Data scientists can’t 
wait to get their hands on so much text data in order to do sentiment analysis.  

After the initial project kick-off, the real work begins.  The experts from each system bring an inventory of 
their data and systems.  Your stomach starts doing flips as you begin to realize the scope of this task.  
Each site uses different technologies and have vastly different data models.  While there are some commonalities,
there are a lot of differences.  

You decide to start with "low hanging fruit".  You find that every site has the concept of a user and posts.  
You figure it should be easy to create a common model for users and posts and have the data loaded within a few 
weeks.  

You start with users.  What fields are mandatory?  Obviously, first name and last name.  However, you have some sites
that are anonymous and don't require real names.  What about usernames?  Every site has to have a username? It 
turns out that some sites are mobile only and use a device ID instead of a username.  You decide to get clever and 
come up with a concept called *unique account identifier* that abstracts away the underlying detail.  You pat yourself
on the back for coming up with such a smart solution to a tricky problem.  

You continue down this path and discover more and more differences between the sites.  Some sites require an 
email address, others do not.  Still others may allow multiple email addresses.  Some sites distinguish between mobile,
home, and work information, while others do not.  Your user model is starting to get really big, but you are pretty
sure you are capturing every possible detail.  

You decide it is time to move onto posts. Everyone agrees that all posts contain content.  
But it turns out each site have very different content. 
One site allows posting arbitrary HTML content. Another one uses its own specialized markup language. 
Still another only allows posts using plain text. This doesn’t even cover the differences on how they each 
handle pictures, videos and links.  Everyone decides the markup won’t be important for data scientists or 
analysts. They would love to actually ask them if they needed that data, but they are all busy doing other work.  

You remember back to that initial meeting and remember how much the business analysts wanted location data.  You 
decide you should prioritize extracting location data from posts.  Sounds pretty simple right?  Just attach a 
latitude and longitude to your post and call it a day.  Except it is not that simple.  Apparently, one of the
social networking site for astronomers and another specializes in surveyors. 

> **Note:** I understand this is a stretch for the social networking use case, but it isn't for things like sensor 
networks.  

The expert from the astronomy site gives you .. The surveyors are equally unhappy and need ... After a few weeks
of intensive research and heated arguments, you reach a compromise.  Its not pretty or intuitive, but you
found a way Neither the astronomers or the surveyors are
happy, but it was the only solution they could both agree on.  

Meanwhile, the data scientists and business analysts are sitting on the sideline waiting for the experts 
to come down the mountain and hand them the cleaned, integrated dataset.  Days turn into weeks, weeks turn into 
months.  After months of waiting, they finally get their hands on the data only to discover the data scientists 
wanted to know whether or not users bolded certain words in their content.  

You finally 

A few years go by and it is time for a change. Turns out there was a systematic error

Most of the people who worked the original project have moved on.  

## What Went Wrong

The fallacy is that there is a single, correct interpretation of the data.  

## Lambda Architecture

http://jameskinley.tumblr.com/post/37398560534/the-lambda-architecture-principles-for-architecting
http://www.drdobbs.com/database/applying-the-big-data-lambda-architectur/240162604

The lambda architecture borrows key ideas from the functional programming paradigm used in languages such as 
Lisp, Scheme, Haskell, and Erlang. The two key concepts are use of functions against data and immutable data.  

The batch layer is responsible for two things. The first is to store the immutable, constantly growing master 
dataset (HDFS), and the second is to compute arbitrary views from this dataset (MapReduce).

One of the key concepts is that the master dataset is immutable.  In layperson’s terms, immutable means 
the data is written once and never changed. 

Views of the data are achieved via applications of functions to the master dataset.  

All data entering the system is dispatched to both the batch layer and the speed layer for processing.
The batch layer has two functions: (i) managing the master dataset (an immutable, append-only set of raw data), 
and (ii) to pre-compute the batch views.
The serving layer indexes the batch views so that they can be queried in low-latency, ad-hoc way. 
The speed layer compensates for the high latency of updates to the serving layer and deals with recent data only.
Any incoming query can be answered by merging results from batch views and real-time views.

## Event Model

In my experience, one can model most data as a series of events.  An event is simply a piece of time stamped 
data with an identifier.  An event can be a single instant in time or cover a start/end time. 

Banking transactions
Insurance claims
Sensor data


THE USERS OF THE DATA ARE GOING TO KNOW HOW TO USE THE DATA BETTER THAN YOU. 

As changes and transformations occur, information is typically lost.  
Quantum Data
Sounds sexy, but quantum of data is the lowest 
Organizing Data

/assets/<organization>/<department>/<application>/<version>/<year>/<month>/<day>/<hour>/<minute>

Organization
Department
Application
Version
Year
Month
Day
Hour
Minute

Not all of these fields are needed 

/etl/<organization>/<department>/<application>/<version>/….
Serialization Formats
Ideal characteristics
Schema

XML
Description

Pro
Schemas
Con

JSON
Description
Pro
Con
CSV/TSV
Description 
Pro
Con
Avro
Description
Pro
Con
Thrift
Description
Pro
Con
Protocol Buffers
Description
Pro
Con
Others
Description
Pro
Con

## Data Standardization

![XKCD](http://imgs.xkcd.com/comics/iso_8601.png)

http://xkcd.com/1179/
https://www.ietf.org/rfc/rfc3339.txt

Dates and Date Times

Tips

It is very important to maintain an untouched, “raw” copy of original data sources. 

Keep raw, untouched data 
Data is write once
Schema plus documentation should be stored with the data
Any transformations (i.e. ETL) should be version controlled and repeatable from original sources. 
Resist the urge to correct original data sources.  
Zero tolerance policy for undocumented data assumptions
Event based model works well
Use opaque identifiers
Use universally unique identifiers
You will never get everyone to agree to the same data standard! 



