---
title : "Data Lake & Worker"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.7.1. </b> "
---

#### 5.7.1. Initialize Data Lake and Analytics Worker
Before using Glue and Athena for data analysis, we need a data storage location (S3 Data Lake) and a piece of code to automatically push attendance events from SQS into this Data Lake. Writing directly from Lambda to S3 is a very popular architecture for small and medium data streams, saving costs compared to using Kinesis Firehose.

**Step 1: Create an S3 Data Lake Bucket**
First, we need a place to store attendance log files.
1. Search for and access the **S3** service on the AWS Console, click **Create bucket**.
> ![Search S3](/aws-image/setupS3/setups3-1.png)
> ![Click Create bucket](/aws-image/setupS3/s3-2.png)
2. **Bucket name**: Name it `smart-campus-datalake-[your-name]` (e.g., `smart-campus-datalake-danhdct`).
> ![Enter bucket name](/aws-image/setupS3Worker/s31.png)
3. Keep all default settings intact (leaving this bucket in Private mode, blocking all Public Access is the safest).
> ![Object Ownership](/aws-image/setupS3Worker/s32.png)
> ![Block Public Access](/aws-image/setupS3Worker/s33.png)
4. Scroll down to the bottom and click **Create bucket**.
> ![Create S3 Data Lake](/aws-image/setupS3Worker/s34.png)
5. The screen indicates the Bucket was created successfully.
> ![Successfully created](/aws-image/setupS3Worker/s35.png)

**Step 2: Create the Analytics Worker Lambda function**
Now we need to create a Lambda function acting as a "worker": Reading events from the SQS queue and writing JSON files directly to the S3 Data Lake.
1. Go to the **Lambda** service -> **Create function**.
> ![Search for Lambda](/aws-image/setupLambda/lambda3.png)
> ![Go to Create function](/aws-image/setupLambda/lambda4.png)
2. **Function name**: `smart-campus-analytics-worker`. Select the Runtime as **Python 3.11** (or 3.x).
3. Click **Create function**.
> ![Fill info and Create function](/aws-image/setupLambdaWorker/lambda1.png)
> ![Successfully created](/aws-image/setupLambdaWorker/lambda2.png)
4. In the Lambda screen, scroll down to the **Code** tab: Delete all default content in the `lambda_function.py` file and paste the following code into it (this is a **standalone** version using only `boto3`, which can run directly on the AWS Console without packaging):



```python
import json
import logging
import os
import uuid
import boto3

logger = logging.getLogger()
logger.setLevel(logging.INFO)

# Read bucket name from environment variable (will be configured in Step 5)
DATA_LAKE_BUCKET = os.environ.get("DATA_LAKE_BUCKET", "")
AWS_REGION = os.environ.get("AWS_REGION", "ap-southeast-1")

s3_client = boto3.client("s3", region_name=AWS_REGION)


def _write_to_s3(record: dict) -> str:
    """Writes an attendance record to S3 Data Lake following a partition structure."""
    data = json.dumps(record, ensure_ascii=False, default=str)

    year  = record.get("year",  "0000")
    month = record.get("month", "00")
    day   = record.get("day",   "00")
    file_name = f"year={year}/month={month}/day={day}/{uuid.uuid4().hex}.json"

    s3_client.put_object(
        Bucket=DATA_LAKE_BUCKET,
        Key=file_name,
        Body=data.encode("utf-8"),
        ContentType="application/json"
    )
    return file_name


def lambda_handler(event, context):
    """
    Entry point for Lambda Analytics Worker.
    Triggered by SQS Queue (smart-campus-analytics-queue)
    containing 'AttendanceRecorded' events from EventBridge.
    """
    records = event.get("Records", [])
    logger.info("AnalyticsWorker received %d records from SQS", len(records))

    failed_message_ids = []

    for sqs_record in records:
        message_id = sqs_record.get("messageId")
        try:
            # EventBridge Payload is embedded in SQS message body
            eb_event  = json.loads(sqs_record.get("body", "{}"))
            detail_type = eb_event.get("detail-type", "")
            detail      = eb_event.get("detail", {})

            if detail_type != "AttendanceRecorded":
                logger.info("Skipping non-attendance event: %s", detail_type)
                continue

            ts = detail.get("timestamp", "")
            analytics_record = {
                "event_type":    detail_type,
                "attendance_id": detail.get("attendanceId"),
                "user_id":       detail.get("userId"),
                "status":        detail.get("status"),
                "timestamp":     ts,
                # Partition fields for Glue/Athena
                "year":  ts[:4]  if ts else None,
                "month": ts[5:7] if ts else None,
                "day":   ts[8:10] if ts else None,
            }

            file_key = _write_to_s3(analytics_record)
            logger.info("Written to S3 Data Lake. MsgId=%s, Key=%s", message_id, file_key)

        except Exception as exc:
            logger.error("Error processing message %s: %s", message_id, exc, exc_info=True)
            failed_message_ids.append(message_id)

    # Return standard Partial Batch Failure format of Lambda + SQS
    return {
        "batchItemFailures": [
            {"itemIdentifier": msg_id} for msg_id in failed_message_ids
        ]
    }
```
> ![Paste Code](/aws-image/setupLambdaWorker/lambda3.png)
5. Configure environment variables: Switch to the **Configuration** tab -> **Environment variables**. Click **Edit**, add the `DATA_LAKE_BUCKET` variable with the value being the bucket name you just created. Click **Save**. 
> ![Configuration Tab](/aws-image/setupLambdaWorker/lambda4.png)
> ![Enter Key Value](/aws-image/setupLambdaWorker/lambda5.png)
6. Return to the **Code** tab, click **Deploy** (A green notification will appear on the screen when successfully updated).
> ![Deploy successfully](/aws-image/setupLambdaWorker/lambda6.png)

**Step 3: Configure Trigger and Permissions**
1. Configure **Trigger** (Activation):
   - In the top overview chart (Function overview), click **+ Add trigger**.
   - From the drop-down menu, select **SQS**.
   - SQS queue: Select the `smart-campus-analytics-queue` queue you created in lesson 5.6.3. Click **Add**.
> ![Add SQS Trigger](/aws-image/setupSQS/sqs24.png)
2. Grant **IAM** permissions (Extremely important):
   - Switch to the **Configuration** tab -> **Permissions** -> Click on the Role name (e.g., `smart-campus-analytics-worker-role...`) to open the IAM window.
> ![Open IAM Role](/aws-image/setupLambdaWorker/lambda7.png)
   - In IAM, click **Add permissions** -> **Attach policies**.
> ![Attach policies](/aws-image/setupLambdaWorker/lambda8.png)
   - Find and add 2 policies: `AmazonS3FullAccess` and `AmazonSQSFullAccess`. Click **Add permissions**.
> ![Add S3 Access](/aws-image/setupLambdaWorker/lambda9.png)
> ![Add SQS Access](/aws-image/setupLambdaWorker/lambda10.png)

**Step 4: Test the data flow**
1. Try calling the attendance API using Postman (or Frontend) to simulate a student scanning their face.
2. Wait a few seconds, open the S3 Bucket `smart-campus-datalake-[your-name]`.
3. Open S3 Console, enter the newly created Data Lake Bucket. You will see an automatically generated folder structure: `attendance/year=.../month=.../day=.../`. Inside are the JSON format data files. Congratulations, your Data Lake is officially operational!
> ![S3 Result 1](/aws-image/setupLambdaWorker/lambda11.png)
> ![S3 Result 2](/aws-image/setupLambdaWorker/lambda12.png)
> ![S3 Result 3](/aws-image/setupLambdaWorker/lambda13.png)
> ![S3 Result 4](/aws-image/setupLambdaWorker/lambda14.png)
> ![S3 Result 5](/aws-image/setupLambdaWorker/lambda15.png)

Awesome! Now the data flow from marking attendance to writing into the Data Lake is completely automated. In the next lesson, we will bring AWS Glue into play to scan these JSON files.
