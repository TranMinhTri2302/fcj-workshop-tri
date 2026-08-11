---
title : "Query with Amazon Athena"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.7.3. </b> "
---

#### 5.7.3. Data analysis with Amazon Athena
Amazon Athena is a serverless interactive query service that makes it easy to analyze data directly in S3 using standard SQL. Since the table schema has been defined by Glue, Athena can read it instantly.

**Step 1: Access Amazon Athena and Open the Query Editor**

1. From the AWS Console search bar, type **Athena** and select the service.
> ![Search Athena](/aws-image/setupAthena/athena1.png)
2. On the Athena main screen, click **Launch query editor**.
> ![Launch Query Editor](/aws-image/setupAthena/athena2.png)

**Step 2: Configure Query Result Location**

Athena saves the result of every query as a file in S3. If you use this feature for the first time, you must configure a save location.
1. In the Query editor screen, you will see a blue notification reminding you to set up the save location. Click the **Edit settings** button.
> ![Edit Settings](/aws-image/setupAthena/athena3.png)
2. On the Query settings screen that appears, click the **Manage** button.
> ![Manage Settings](/aws-image/setupAthena/athena4.png)
3. In the **Location of query result** section, click the **Browse S3** button.
> ![Browse S3](/aws-image/setupAthena/athena5.png)
4. Select your Data Lake Bucket (or a separate bucket used for Athena) and click **Choose**.
> ![Select Bucket](/aws-image/setupAthena/athena6.png)
5. You should add a folder suffix at the end of the S3 path (e.g., `/athena-results/`) so that query results are saved neatly, avoiding confusion with original data. Then click **Save**.
> ![Configure Query Result Location](/aws-image/setupAthena/athena7.png)

**Step 3: Run the SQL query**

1. Return to the **Editor** screen, ensuring the Database on the left is selected as `smart_campus_db`. In the editor box, paste the following simple SQL statement to view the formatted attendance logs on S3 (remember to change the table name to match your data, or you can double-click the table name in the left column for the system to auto-fill):
```sql
SELECT * FROM "smart_campus_db"."<your_s3_table_name>"
```
2. Click **Run** (or Run again). In the **Query results** tab below, you will see all the attendance log data with all columns like `event_type`, `attendance_id`, `user_id`, `status` displayed clearly and beautifully like a traditional database table.
> ![Query Results](/aws-image/setupAthena/athena8.png)



At this point, you have completely mastered the **Big Data (Data Pipeline)** flow in Smart Campus!
