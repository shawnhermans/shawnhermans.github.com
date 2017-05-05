---
layout: post
title: "Migrating GCS to AWS S3"
author: Shawn Hermans
date: 2017-02-23 12:00:00 -0600
categories: [aws, gcs, hadoop, distcp]
---
Originally posted as an answer to [How can I migrate data from Google cloud storage into AWS S3 buckets?](https://www.quora.com/How-can-I-migrate-data-from-Google-cloud-storage-into-AWS-S3-buckets/answer/Shawn-Hermans?srid=hLq3)

Hadoop Distcp is a good way of moving large amounts of data between different file systems. Here are the steps I used to transfer data between Google Cloud Storage and S3 using distcp.

1. I created a 3-node Hadoop cluster using Google Dataproc. If it is configured correctly, it should have access to your GCS files without having to add any additional configuration.
2. Once the cluster finishes initializing, SSH into the master node and run the following command.

```
hadoop distcp \
  http://gs://<bucket-name>/<folder>/ \
  http://s3a://<aws_access_key>:<aws_secret_key>@<s3-bucket>/<folder>
```

This should copy the data from GCS to S3.
