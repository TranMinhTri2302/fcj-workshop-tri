---
title: "Deploy Lambda Functions"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Overview

In this step, you will deploy 3 Lambda functions:
1. **Main API Lambda**: Main FastAPI backend (attendance, users, tasks...)
2. **Analytics Worker Lambda**: Processes events from SQS Analytics Queue, writes to S3 Data Lake
3. **Notification Worker Lambda**: Processes events from SQS Notification Queue, sends email/SMS

## Why Lambda?

**Advantages:**
- ✅ Serverless: No EC2 instance management
- ✅ Auto-scaling: 0 → 10,000 concurrent automatically
- ✅ Pay-per-use: Only pay when requests occur
- ✅ Built-in HA: Multi-AZ automatic
- ✅ Fast deployment: < 1 minute

**Limitations:**
- ❌ Cold start: 1-3 seconds when function idle
- ❌ Max timeout: 15 minutes
- ❌ Max memory: 10GB
- ❌ Stateless: No state between invocations

## Project Structure

```
smart-campus-backend/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI app entry point
│   ├── handler.py           # Lambda handler (Mangum)
│   ├── core/
│   │   ├── config.py        # Settings (env vars)
│   │   ├── exceptions.py
│   │   └── responses.py
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── router.py
│   │   │   ├── service.py
│   │   │   └── schemas.py
│   │   ├── users/
│   │   ├── faces/
│   │   ├── attendance/      # Core module
│   │   │   ├── router.py
│   │   │   ├── service.py
│   │   │   ├── repository.py
│   │   │   ├── rule_engine.py
│   │   │   └── schemas.py
│   │   └── ...
│   ├── shared/
│   │   └── aws/
│   │       ├── dynamodb.py
│   │       ├── rekognition.py
│   │       ├── s3.py
│   │       └── eventbridge.py
│   └── workers/
│       ├── analytics_worker.py
│       └── notification_worker.py
├── requirements.txt
├── Dockerfile (optional)
└── README.md
```

## Step 1: Prepare Code

**File: `requirements.txt`**
```txt
fastapi==0.104.1
mangum==0.17.0
boto3==1.29.7
pydantic==2.5.0
pydantic-settings==2.1.0
python-multipart==0.0.6
aws-xray-sdk==2.12.1
```

**File: `app/main.py`**
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api.router import api_router
from app.core.config import settings

app = FastAPI(
    title="Smart Campus API",
    version="1.0.0",
    description="Serverless Attendance System"
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(api_router, prefix="/api")

@app.get("/health")
def health_check():
    return {"status": "healthy", "timestamp": "2026-08-06T10:00:00Z"}
```

**File: `app/handler.py`** (Lambda handler)
```python
from mangum import Mangum
from app.main import app
from app.workers.analytics_worker import handler as analytics_handler
from app.workers.notification_worker import handler as notification_handler

# Main API handler
lambda_handler = Mangum(app, lifespan="off")

# Multi-handler: Route based on event source
def handler(event, context):
    """
    Main Lambda handler - routes to appropriate handler based on event source.
    """
    # Check if SQS event
    if "Records" in event and event["Records"]:
        source_arn = event["Records"][0].get("eventSourceARN", "")
        
        if "analytics-queue" in source_arn:
            return analytics_handler(event, context)
        elif "notification-queue" in source_arn:
            return notification_handler(event, context)
    
    # Default: HTTP request via API Gateway
    return lambda_handler(event, context)
```

**File: `app/workers/analytics_worker.py`**
```python
import json
import boto3
from datetime import datetime

s3 = boto3.client('s3')
DATA_LAKE_BUCKET = 'smart-campus-datalake-123456789012'

def handler(event, context):
    """
    Process events from Analytics SQS Queue.
    Write to S3 Data Lake with partitioning by date.
    """
    for record in event['Records']:
        try:
            body = json.loads(record['body'])
            detail = body.get('detail', {})
            event_type = body.get('detail-type', 'Unknown')
            
            # Extract date for partitioning
            timestamp = detail.get('timestamp', datetime.utcnow().isoformat())
            date = timestamp[:10]  # YYYY-MM-DD
            year, month, day = date.split('-')
            
            # Determine prefix based on event type
            if 'Attendance' in event_type:
                prefix = f'attendance/year={year}/month={month}/day={day}/'
            elif 'Task' in event_type:
                prefix = f'tasks/year={year}/month={month}/day={day}/'
            else:
                prefix = f'events/year={year}/month={month}/day={day}/'
            
            # Write to S3
            key = f"{prefix}{context.request_id}.json"
            s3.put_object(
                Bucket=DATA_LAKE_BUCKET,
                Key=key,
                Body=json.dumps(detail),
                ContentType='application/json'
            )
            
            print(f"✅ Wrote to S3: s3://{DATA_LAKE_BUCKET}/{key}")
            
        except Exception as e:
            print(f"❌ Error processing record: {e}")
            # Let SQS retry or send to DLQ
            raise
    
    return {"statusCode": 200, "body": "Processed"}
```

**File: `app/workers/notification_worker.py`**
```python
import json
import boto3

sns = boto3.client('sns')
ses = boto3.client('ses')

def handler(event, context):
    """
    Process events from Notification SQS Queue.
    Send email/SMS via SNS/SES.
    """
    for record in event['Records']:
        try:
            body = json.loads(record['body'])
            detail = body.get('detail', {})
            
            user_id = detail.get('userId') or detail.get('user_id')
            event_type = body.get('detail-type', 'Unknown')
            
            # Get user email from DynamoDB (simplified)
            email = get_user_email(user_id)
            
            # Send email
            if email:
                send_email(email, event_type, detail)
            
            print(f"✅ Sent notification to {user_id}")
            
        except Exception as e:
            print(f"❌ Error sending notification: {e}")
            raise
    
    return {"statusCode": 200, "body": "Notified"}

def get_user_email(user_id: str) -> str:
    # Query DynamoDB users table
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('smart-campus-users')
    response = table.get_item(Key={'user_id': user_id})
    return response.get('Item', {}).get('email')

def send_email(email: str, event_type: str, detail: dict):
    subject = f"[Smart Campus] {event_type}"
    body = json.dumps(detail, indent=2)
    
    ses.send_email(
        Source='noreply@smartcampus.edu.vn',
        Destination={'ToAddresses': [email]},
        Message={
            'Subject': {'Data': subject},
            'Body': {'Text': {'Data': body}}
        }
    )
```

## Step 2: Package Lambda

**Option 1: Manual packaging**

```bash
# Create package directory
mkdir -p lambda-package
cd lambda-package

# Install dependencies
pip install -r ../requirements.txt -t .

# Copy code
cp -r ../app .

# Create zip
zip -r ../lambda.zip .

cd ..
```

**Option 2: Docker-based packaging** (recommended)

```bash
# Dockerfile
FROM public.ecr.aws/lambda/python:3.12

# Copy requirements
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy code
COPY app/ ${LAMBDA_TASK_ROOT}/app/

# Set handler
CMD ["app.handler.handler"]
```

Build and package:
```bash
docker build -t smart-campus-lambda .
docker run --rm -v $(pwd):/out smart-campus-lambda \
  sh -c "cd /var/task && zip -r /out/lambda.zip ."
```

## Step 3: Upload Lambda Package

**Upload to S3:**
```bash
aws s3 cp lambda.zip \
  s3://smart-campus-lambda-code-${AWS_ACCOUNT_ID}/lambda.zip \
  --region ap-southeast-1
```

Verify:
```bash
aws s3 ls s3://smart-campus-lambda-code-${AWS_ACCOUNT_ID}/
```

## Step 4: Create Main API Lambda

```bash
aws lambda create-function \
  --function-name smart-campus-api \
  --runtime python3.12 \
  --role arn:aws:iam::${AWS_ACCOUNT_ID}:role/SmartCampusLambdaRole \
  --handler app.handler.handler \
  --code S3Bucket=smart-campus-lambda-code-${AWS_ACCOUNT_ID},S3Key=lambda.zip \
  --timeout 30 \
  --memory-size 512 \
  --environment Variables="{
    USERS_TABLE=smart-campus-users,
    FACES_TABLE=smart-campus-faces,
    ATTENDANCE_TABLE=smart-campus-attendance,
    FACE_COLLECTION_ID=smart-campus-faces,
    IMAGE_BUCKET=smart-campus-images-${AWS_ACCOUNT_ID},
    EVENT_BUS_NAME=smart-campus-events,
    AWS_REGION=ap-southeast-1
  }" \
  --region ap-southeast-1 \
  --tags Project=SmartCampus
```

Expected output:
```json
{
    "FunctionName": "smart-campus-api",
    "FunctionArn": "arn:aws:lambda:ap-southeast-1:123456789012:function:smart-campus-api",
    "Runtime": "python3.12",
    "Handler": "app.handler.handler",
    "CodeSize": 15728640,
    "Timeout": 30,
    "MemorySize": 512,
    "State": "Pending"
}
```

**Wait for function to be active:**
```bash
aws lambda wait function-active \
  --function-name smart-campus-api \
  --region ap-southeast-1

echo "✅ Lambda function is active!"
```

## Step 5: Test Lambda Locally (Optional)

**Test health endpoint:**
```bash
aws lambda invoke \
  --function-name smart-campus-api \
  --payload '{"httpMethod":"GET","path":"/health","headers":{}}' \
  --region ap-southeast-1 \
  response.json

cat response.json
```

Expected:
```json
{
    "statusCode": 200,
    "headers": {"content-type": "application/json"},
    "body": "{\"status\":\"healthy\",\"timestamp\":\"2026-08-06T10:00:00Z\"}"
}
```

## Step 6: Configure Lambda Triggers

**Add API Gateway trigger** (will do in step 6)

**Add SQS trigger for Analytics Worker:**
```bash
# Get SQS Queue ARN (created in step 7)
ANALYTICS_QUEUE_ARN=$(aws sqs get-queue-attributes \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-analytics-queue \
  --attribute-names QueueArn \
  --region ap-southeast-1 \
  --query 'Attributes.QueueArn' \
  --output text)

# Create event source mapping
aws lambda create-event-source-mapping \
  --function-name smart-campus-api \
  --event-source-arn ${ANALYTICS_QUEUE_ARN} \
  --batch-size 10 \
  --maximum-batching-window-in-seconds 5 \
  --region ap-southeast-1
```

## Step 7: Enable CloudWatch Logs

Lambda automatically sends logs to CloudWatch. Verify:

```bash
# List log groups
aws logs describe-log-groups \
  --log-group-name-prefix /aws/lambda/smart-campus \
  --region ap-southeast-1
```

Expected:
```
/aws/lambda/smart-campus-api
```

**View recent logs:**
```bash
aws logs tail /aws/lambda/smart-campus-api \
  --follow \
  --region ap-southeast-1
```

## Step 8: Configure Environment Variables

**Update env vars:**
```bash
aws lambda update-function-configuration \
  --function-name smart-campus-api \
  --environment Variables="{
    USERS_TABLE=smart-campus-users,
    FACES_TABLE=smart-campus-faces,
    ATTENDANCE_TABLE=smart-campus-attendance,
    FACE_COLLECTION_ID=smart-campus-faces,
    IMAGE_BUCKET=smart-campus-images-${AWS_ACCOUNT_ID},
    DATA_LAKE_BUCKET=smart-campus-datalake-${AWS_ACCOUNT_ID},
    EVENT_BUS_NAME=smart-campus-events,
    ANALYTICS_QUEUE_URL=https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-analytics-queue,
    NOTIFICATION_QUEUE_URL=https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-notification-queue,
    AWS_REGION=ap-southeast-1
  }" \
  --region ap-southeast-1
```

## Step 9: Update Lambda Code

**When code changes:**
```bash
# Re-package
zip -r lambda.zip app/

# Upload to S3
aws s3 cp lambda.zip \
  s3://smart-campus-lambda-code-${AWS_ACCOUNT_ID}/lambda.zip

# Update Lambda
aws lambda update-function-code \
  --function-name smart-campus-api \
  --s3-bucket smart-campus-lambda-code-${AWS_ACCOUNT_ID} \
  --s3-key lambda.zip \
  --region ap-southeast-1
```

**Wait for update:**
```bash
aws lambda wait function-updated \
  --function-name smart-campus-api \
  --region ap-southeast-1
```

## Performance Tuning

**Memory optimization:**

Test with different memory sizes:
```bash
# Test 256MB
aws lambda update-function-configuration \
  --function-name smart-campus-api \
  --memory-size 256

# Test 512MB
aws lambda update-function-configuration \
  --function-name smart-campus-api \
  --memory-size 512

# Test 1024MB
aws lambda update-function-configuration \
  --function-name smart-campus-api \
  --memory-size 1024
```

**Rule of thumb:**
- 256MB: Slow, cheap (~$0.42/M invocations)
- 512MB: Balanced (~$0.83/M invocations)
- 1024MB: Fast, expensive (~$1.67/M invocations)

**Cold start optimization:**
- Use Python (cold start ~500ms) instead of Java (~3s)
- Keep dependencies minimal
- Use Provisioned Concurrency (more expensive)

## Monitoring Metrics

**View Lambda metrics:**
```bash
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Duration \
  --dimensions Name=FunctionName,Value=smart-campus-api \
  --start-time 2026-08-06T00:00:00Z \
  --end-time 2026-08-06T23:59:59Z \
  --period 3600 \
  --statistics Average,Maximum \
  --region ap-southeast-1
```

**Key metrics to monitor:**
- **Invocations**: Number of function calls
- **Duration**: Execution time (aim: < 1000ms)
- **Errors**: Error count (aim: < 1%)
- **Throttles**: Throttled invocations (aim: 0)
- **ConcurrentExecutions**: Concurrent executions

## Troubleshooting

**Error: Function timed out**
```
Task timed out after 30.00 seconds
```
**Solution:** Increase timeout or optimize code

**Error: Out of memory**
```
Runtime exited with error: signal: killed Runtime.ExitError
```
**Solution:** Increase memory from 512MB → 1024MB

**Error: Import error**
```
Unable to import module 'app.handler': No module named 'fastapi'
```
**Solution:** Dependencies not packaged correctly. Re-package with pip install -t

**Error: Permission denied**
```
User: arn:aws:... is not authorized to perform: dynamodb:PutItem
```
**Solution:** Check IAM role has sufficient permissions

## Cost Estimation

**Lambda pricing (ap-southeast-1):**
- Requests: $0.20 per 1M requests
- Duration: $0.0000166667 per GB-second

**Monthly estimate (100K requests, 512MB, 500ms avg):**
- Requests: 100K × $0.20/1M = $0.02
- Duration: 100K × 0.5s × 0.5GB × $0.0000166667 = $0.42
- **Total: $0.44/month**

**Free Tier (first 12 months):**
- 1M requests/month FREE
- 400,000 GB-seconds/month FREE

→ Workshop will be within Free Tier!

## Verify Setup

Checklist:
- [ ] Lambda function `smart-campus-api` created and ACTIVE
- [ ] Package size < 50MB (unzipped < 250MB)
- [ ] Environment variables configured correctly
- [ ] IAM role has sufficient permissions
- [ ] CloudWatch Logs working
- [ ] Health endpoint returns 200 OK

## Next Step

Proceed to [Step 6: Configure API Gateway](../5.6-apigateway) to expose Lambda via HTTP endpoint!