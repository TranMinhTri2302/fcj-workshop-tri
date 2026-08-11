---
title : "Data Analytics (S3 & Athena)"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

### Goal

In the previous sections, whenever a user checkin, the system saves the result into DynamoDB (for fast per-user querying) and emits an event via EventBridge to SQS/SNS. 
Simultaneously, we also have an **Analytics Worker** Lambda function (responding to the SQS queue) that automatically pulls these attendance events and **writes directly** as JSON files into an **S3 Data Lake**.

Once the data safely resides in the Data Lake, we can use AWS Glue to automatically infer the table schema, and use Amazon Athena to query the data using SQL. This entire process is fully automated, serverless, and isolated from the main API system (OLTP). Instead of having to query DynamoDB (which is very expensive and unsuitable for large statistical queries), we will use a Serverless Data Lake architecture with **AWS Glue** and **Amazon Athena**.

- **AWS Glue (Crawler):** Automatically scans Log files in the S3 Data Lake to identify the data structure (Schema) and create a virtual table.
- **Amazon Athena:** Allows you to write familiar SQL statements to query data residing in S3 directly through the virtual Glue table, without needing to install any Database Server. You pay per amount of data scanned (Pay per query).

### Detailed Practice Content

Please click on each item below in the left menu bar or click directly on the links below to follow the detailed steps:

{{% children /%}}
