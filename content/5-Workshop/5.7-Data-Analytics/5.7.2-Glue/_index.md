---
title : "Configure AWS Glue"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.7.2. </b> "
---

#### 5.7.2. Initialize Data Catalog with AWS Glue
AWS Glue is a serverless Data Integration service. We will use its **Glue Crawler** feature to automatically read the attendance log files in the S3 bucket and infer the table structure (Table Schema) to save into the Data Catalog.

**Step 1: Initialize Database in AWS Glue**

1. From the AWS Console search bar, type **Glue** and select the **AWS Glue** service.
> ![Search Glue](/aws-image/setupGlue/glue1.png)
2. On the left menu, select **Databases** under the Data Catalog section and click **Add database**.
> ![Go to Databases](/aws-image/setupGlue/glue2.png)
3. Name the database `smart_campus_db` and click **Create database**.
> ![Create Database](/aws-image/setupGlue/glue3.png)

**Step 2: Create IAM Role for Glue Crawler**

1. From the AWS Console search bar, type **IAM** and select the **IAM** service.
> ![Search IAM](/aws-image/setupGlue/glue4.png)
2. Access **Roles** on the left menu and click **Create role**.
> ![Go to IAM Roles](/aws-image/setupGlue/glue5.png)
3. Select Trusted entity type as **AWS service** and Use case as **Glue**. Click **Next**.
> ![Select Trusted entity](/aws-image/setupGlue/glue6.png)
4. At the Add permissions step, search and check the `AmazonS3ReadOnlyAccess` policy to grant read permission for the Data Lake on S3.
> ![Select S3 Policy](/aws-image/setupGlue/glue7_1.png)
5. Continue searching and checking the `AWSGlueServiceRole` policy. Then click **Next**.
> ![Select Glue Policy](/aws-image/setupGlue/glue7_2.png)
6. At the naming step, enter the Role name as `AWSGlueServiceRole-SmartCampus`.
> ![Name the Role](/aws-image/setupGlue/glue8_1.png)
7. Scroll down to review the list of selected policies and click **Create role**.
> ![Review and Create Role](/aws-image/setupGlue/glue8_2.png)
8. Ensure the Role was created successfully with a green notification displayed.
> ![Role created](/aws-image/setupGlue/glue9.png)

**Step 3: Create and Configure Glue Crawler**

1. Return to the AWS Glue page, on the left menu select **Crawlers** under the Data Catalog section and click **Create crawler**.
> ![Select Crawlers](/aws-image/setupGlue/glue10.png)
2. In the Set crawler properties section, name the crawler (e.g., `smart-campus-attendance-crawler`) and click **Next**.
> ![Name Crawler](/aws-image/setupGlue/glue11.png)
3. At the Choose data sources and classifiers step, click **Add a data source**.
> ![Add data source](/aws-image/setupGlue/glue12.png)
4. Select Data source as **S3** and paste the S3 folder path containing your Data Lake data into the **S3 path** box. Click **Add an S3 data source**.
> ![Enter S3 info](/aws-image/setupGlue/glue13.png)
5. Review the newly added data source configuration information.
> ![Review Data Source](/aws-image/setupGlue/glue14.png)
6. Click **Next** to continue.
> ![Click Next](/aws-image/setupGlue/glue15.png)
7. At the Configure security settings step, select the Existing IAM role you just created `AWSGlueServiceRole-SmartCampus` and click **Next**.
> ![Select IAM Role](/aws-image/setupGlue/glue16.png)
8. At the Set output and scheduling step, select Target database as `smart_campus_db` and Crawler schedule select **On demand**. Click **Next**.
> ![Set output](/aws-image/setupGlue/glue17.png)
9. Review all configuration information at the final step and click **Create crawler**.
> ![Create crawler](/aws-image/setupGlue/glue18.png)

**Step 4: Run the Crawler**

1. After the crawler is created, click the **Run crawler** button to proceed with scraping data.
> ![Run crawler](/aws-image/setupGlue/glue19.png)
2. Wait a few dozen seconds (depending on the amount of log files). If the crawler's status changes to **Completed**, it means the automatic schema inference process from the S3 log files was successful and a corresponding new Table was created in the Catalog!
> ![Crawler completed](/aws-image/setupGlue/glue20.png)
