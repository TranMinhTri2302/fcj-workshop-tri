---
title: "EventBridge và Event-Driven Architecture"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Tổng quan

Amazon EventBridge là serverless event bus cho phép xây dựng kiến trúc event-driven. Trong bước này, bạn sẽ:
- Tạo Custom Event Bus
- Tạo SQS Queues (Analytics + Notification) với DLQ
- Tạo EventBridge Rules để route events
- Test event publishing và processing

#### Tại sao Event-Driven Architecture?

**Monolithic approach (bad):**
```
API → Check-in → [Write DynamoDB + Send email + Write analytics] → Response
                  ↑ Blocking, slow (5-10 seconds)
```

**Event-driven approach (good):**
```
API → Check-in → [Write DynamoDB + Publish event] → Response (fast!)
                                    ↓
                        EventBridge Event Bus
                         ↓              ↓
                   Analytics Queue  Notification Queue
                         ↓              ↓
                   Worker Lambda    Worker Lambda
```

**Benefits:**
- ✅ Fast response time (< 500ms)
- ✅ Async processing (không block user)
- ✅ Decoupling (services độc lập)
- ✅ Retry mechanism (DLQ)
- ✅ Scale independently

#### Kiến trúc Event Flow

```
[Lambda API]
    ↓ PutEvents
[EventBridge Custom Bus: smart-campus-events]
    ↓
[EventBridge Rules]
    ├─ Rule 1: AttendanceEvents → Analytics SQS
    ├─ Rule 2: AttendanceEvents → Notification SQS
    ├─ Rule 3: TaskEvents → Analytics SQS
    └─ Rule 4: TaskEvents → Notification SQS
    ↓
[SQS Queues]
    ├─ smart-campus-analytics-queue → Analytics Worker Lambda
    │   └─ DLQ: analytics-dlq
    └─ smart-campus-notification-queue → Notification Worker Lambda
        └─ DLQ: notification-dlq
```

#### Bước 1: Tạo Custom Event Bus

```bash
aws events create-event-bus \
  --name smart-campus-events \
  --region ap-southeast-1 \
  --tags Key=Project,Value=SmartCampus
```

**Expected output:**
```json
{
    "EventBusArn": "arn:aws:events:ap-southeast-1:123456789012:event-bus/smart-campus-events"
}
```

**Verify:**
```bash
aws events describe-event-bus \
  --name smart-campus-events \
  --region ap-southeast-1
```

#### Bước 2: Tạo Dead Letter Queues (DLQ)

DLQ lưu trữ messages failed sau nhiều lần retry.

**Analytics DLQ:**
```bash
aws sqs create-queue \
  --queue-name smart-campus-analytics-dlq \
  --attributes MessageRetentionPeriod=1209600 \
  --region ap-southeast-1 \
  --tags Project=SmartCampus
```

**Notification DLQ:**
```bash
aws sqs create-queue \
  --queue-name smart-campus-notification-dlq \
  --attributes MessageRetentionPeriod=1209600 \
  --region ap-southeast-1 \
  --tags Project=SmartCampus
```

**Note:** MessageRetentionPeriod=1209600 = 14 days

**Save DLQ ARNs:**
```bash
ANALYTICS_DLQ_ARN=$(aws sqs get-queue-attributes \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-analytics-dlq \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' \
  --output text)

NOTIFICATION_DLQ_ARN=$(aws sqs get-queue-attributes \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-notification-dlq \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' \
  --output text)
```

#### Bước 3: Tạo Main SQS Queues với DLQ

**Analytics Queue:**
```bash
aws sqs create-queue \
  --queue-name smart-campus-analytics-queue \
  --attributes '{
    "VisibilityTimeout": "60",
    "MessageRetentionPeriod": "345600",
    "ReceiveMessageWaitTimeSeconds": "20",
    "RedrivePolicy": "{\"deadLetterTargetArn\":\"'${ANALYTICS_DLQ_ARN}'\",\"maxReceiveCount\":\"3\"}"
  }' \
  --region ap-southeast-1 \
  --tags Project=SmartCampus
```

**Notification Queue:**
```bash
aws sqs create-queue \
  --queue-name smart-campus-notification-queue \
  --attributes '{
    "VisibilityTimeout": "60",
    "MessageRetentionPeriod": "345600",
    "ReceiveMessageWaitTimeSeconds": "20",
    "RedrivePolicy": "{\"deadLetterTargetArn\":\"'${NOTIFICATION_DLQ_ARN}'\",\"maxReceiveCount\":\"3\"}"
  }' \
  --region ap-southeast-1 \
  --tags Project=SmartCampus
```

**Parameters explained:**
- **VisibilityTimeout: 60** → Message invisible trong 60s khi đang xử lý
- **MessageRetentionPeriod: 345600** → Keep messages 4 days
- **ReceiveMessageWaitTimeSeconds: 20** → Long polling (giảm cost)
- **maxReceiveCount: 3** → Retry 3 lần trước khi vào DLQ

**Save Queue URLs:**
```bash
export ANALYTICS_QUEUE_URL=https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-analytics-queue
export NOTIFICATION_QUEUE_URL=https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-notification-queue
```

#### Bước 4: Grant EventBridge Permission to SQS

**Policy for Analytics Queue:**
```bash
aws sqs set-queue-attributes \
  --queue-url ${ANALYTICS_QUEUE_URL} \
  --attributes '{
    "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"events.amazonaws.com\"},\"Action\":\"sqs:SendMessage\",\"Resource\":\"arn:aws:sqs:ap-southeast-1:'${AWS_ACCOUNT_ID}':smart-campus-analytics-queue\",\"Condition\":{\"ArnEquals\":{\"aws:SourceArn\":\"arn:aws:events:ap-southeast-1:'${AWS_ACCOUNT_ID}':rule/smart-campus-events/*\"}}}]}"
  }' \
  --region ap-southeast-1
```

**Policy for Notification Queue:**
```bash
aws sqs set-queue-attributes \
  --queue-url ${NOTIFICATION_QUEUE_URL} \
  --attributes '{
    "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"events.amazonaws.com\"},\"Action\":\"sqs:SendMessage\",\"Resource\":\"arn:aws:sqs:ap-southeast-1:'${AWS_ACCOUNT_ID}':smart-campus-notification-queue\",\"Condition\":{\"ArnEquals\":{\"aws:SourceArn\":\"arn:aws:events:ap-southeast-1:'${AWS_ACCOUNT_ID}':rule/smart-campus-events/*\"}}}]}"
  }' \
  --region ap-southeast-1
```

#### Bước 5: Tạo EventBridge Rules

**Rule 1: Attendance events → Analytics Queue**

```bash
aws events put-rule \
  --name attendance-to-analytics \
  --event-bus-name smart-campus-events \
  --event-pattern '{
    "source": ["smart-campus.attendance"],
    "detail-type": ["AttendanceRecorded", "AttendanceLate", "AttendanceAbsent"]
  }' \
  --state ENABLED \
  --description "Route attendance events to analytics queue" \
  --region ap-southeast-1
```

**Add target:**
```bash
ANALYTICS_QUEUE_ARN=$(aws sqs get-queue-attributes \
  --queue-url ${ANALYTICS_QUEUE_URL} \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' \
  --output text)

aws events put-targets \
  --rule attendance-to-analytics \
  --event-bus-name smart-campus-events \
  --targets "Id"="1","Arn"="${ANALYTICS_QUEUE_ARN}" \
  --region ap-southeast-1
```

**Rule 2: Attendance events → Notification Queue**

```bash
aws events put-rule \
  --name attendance-to-notification \
  --event-bus-name smart-campus-events \
  --event-pattern '{
    "source": ["smart-campus.attendance"],
    "detail-type": ["AttendanceRecorded", "AttendanceLate"]
  }' \
  --state ENABLED \
  --region ap-southeast-1
```

**Add target:**
```bash
NOTIFICATION_QUEUE_ARN=$(aws sqs get-queue-attributes \
  --queue-url ${NOTIFICATION_QUEUE_URL} \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' \
  --output text)

aws events put-targets \
  --rule attendance-to-notification \
  --event-bus-name smart-campus-events \
  --targets "Id"="1","Arn"="${NOTIFICATION_QUEUE_ARN}" \
  --region ap-southeast-1
```

**Rule 3: Task events → Analytics Queue**

```bash
aws events put-rule \
  --name task-to-analytics \
  --event-bus-name smart-campus-events \
  --event-pattern '{
    "source": ["smart-campus.tasks"],
    "detail-type": ["TaskCreated", "TaskCompleted", "TaskReassigned"]
  }' \
  --state ENABLED \
  --region ap-southeast-1

aws events put-targets \
  --rule task-to-analytics \
  --event-bus-name smart-campus-events \
  --targets "Id"="1","Arn"="${ANALYTICS_QUEUE_ARN}" \
  --region ap-southeast-1
```

**Rule 4: Security events → Notification Queue (High Priority)**

```bash
aws events put-rule \
  --name security-alert \
  --event-bus-name smart-campus-events \
  --event-pattern '{
    "source": ["smart-campus.security"],
    "detail-type": ["UnknownFaceDetected", "UnauthorizedAccess"]
  }' \
  --state ENABLED \
  --region ap-southeast-1

aws events put-targets \
  --rule security-alert \
  --event-bus-name smart-campus-events \
  --targets "Id"="1","Arn"="${NOTIFICATION_QUEUE_ARN}" \
  --region ap-southeast-1
```

#### Bước 6: Configure Lambda SQS Triggers

**Analytics Worker trigger:**
```bash
aws lambda create-event-source-mapping \
  --function-name smart-campus-api \
  --event-source-arn ${ANALYTICS_QUEUE_ARN} \
  --batch-size 10 \
  --maximum-batching-window-in-seconds 5 \
  --enabled \
  --region ap-southeast-1
```

**Notification Worker trigger:**
```bash
aws lambda create-event-source-mapping \
  --function-name smart-campus-api \
  --event-source-arn ${NOTIFICATION_QUEUE_ARN} \
  --batch-size 5 \
  --maximum-batching-window-in-seconds 1 \
  --enabled \
  --region ap-southeast-1
```

**Note:** Notification có batch size nhỏ hơn (5) để send email nhanh hơn.

#### Bước 7: Test Event Publishing

**Test event structure:**
```json
{
  "source": "smart-campus.attendance",
  "detail-type": "AttendanceRecorded",
  "detail": {
    "attendanceId": "att-test-123",
    "userId": "user-001",
    "userName": "Nguyen Van A",
    "status": "PRESENT",
    "sessionType": "MORNING",
    "timestamp": "2026-08-06T07:55:00+07:00",
    "cameraId": "camera-01",
    "confidence": 98.5
  }
}
```

**Publish test event via CLI:**
```bash
aws events put-events \
  --entries '[
    {
      "Source": "smart-campus.attendance",
      "DetailType": "AttendanceRecorded",
      "Detail": "{\"attendanceId\":\"att-test-123\",\"userId\":\"user-001\",\"userName\":\"Nguyen Van A\",\"status\":\"PRESENT\",\"sessionType\":\"MORNING\",\"timestamp\":\"2026-08-06T07:55:00+07:00\",\"cameraId\":\"camera-01\",\"confidence\":98.5}",
      "EventBusName": "smart-campus-events"
    }
  ]' \
  --region ap-southeast-1
```

**Expected output:**
```json
{
    "FailedEntryCount": 0,
    "Entries": [
        {
            "EventId": "abc123-event-id-xyz789"
        }
    ]
}
```

#### Bước 8: Verify Event Delivery

**Check Analytics Queue:**
```bash
aws sqs receive-message \
  --queue-url ${ANALYTICS_QUEUE_URL} \
  --max-number-of-messages 1 \
  --wait-time-seconds 10 \
  --region ap-southeast-1
```

**Expected output:**
```json
{
    "Messages": [
        {
            "MessageId": "msg-123",
            "ReceiptHandle": "...",
            "Body": "{\"version\":\"0\",\"id\":\"abc123-event-id\",\"detail-type\":\"AttendanceRecorded\",\"source\":\"smart-campus.attendance\",\"account\":\"123456789012\",\"time\":\"2026-08-06T00:55:00Z\",\"region\":\"ap-southeast-1\",\"resources\":[],\"detail\":{\"attendanceId\":\"att-test-123\",\"userId\":\"user-001\",\"userName\":\"Nguyen Van A\",\"status\":\"PRESENT\",\"sessionType\":\"MORNING\",\"timestamp\":\"2026-08-06T07:55:00+07:00\",\"cameraId\":\"camera-01\",\"confidence\":98.5}}"
        }
    ]
}
```

**Check Notification Queue:**
```bash
aws sqs receive-message \
  --queue-url ${NOTIFICATION_QUEUE_URL} \
  --max-number-of-messages 1 \
  --region ap-southeast-1
```

**Check Lambda logs (workers should process automatically):**
```bash
aws logs tail /aws/lambda/smart-campus-api --follow
```

Expected in logs:
```
✅ Wrote to S3: s3://smart-campus-datalake-.../attendance/year=2026/month=08/day=06/att-test-123.json
✅ Sent notification to user-001
```

#### Bước 9: Python Code Integration

**File: `app/shared/aws/eventbridge.py`**

```python
import boto3
import json
from datetime import datetime

eventbridge = boto3.client('events', region_name='ap-southeast-1')
EVENT_BUS_NAME = 'smart-campus-events'

def publish_event(source: str, detail_type: str, detail: dict):
    """
    Publish event to EventBridge.
    
    Args:
        source: Event source (e.g., 'smart-campus.attendance')
        detail_type: Event type (e.g., 'AttendanceRecorded')
        detail: Event detail (dict)
    """
    try:
        response = eventbridge.put_events(
            Entries=[{
                'Source': source,
                'DetailType': detail_type,
                'Detail': json.dumps(detail, default=str),
                'EventBusName': EVENT_BUS_NAME,
                'Time': datetime.utcnow()
            }]
        )
        
        if response['FailedEntryCount'] > 0:
            print(f"❌ Failed to publish event: {response['Entries'][0].get('ErrorMessage')}")
            return False
        
        print(f"✅ Event published: {detail_type}")
        return True
        
    except Exception as e:
        print(f"❌ Error publishing event: {e}")
        return False

# Usage example
def publish_attendance_event(attendance_data: dict):
    """Publish attendance recorded event."""
    return publish_event(
        source='smart-campus.attendance',
        detail_type='AttendanceRecorded',
        detail=attendance_data
    )
```

**Integrate vào attendance service:**

```python
# app/modules/attendance/service.py
from app.shared.aws.eventbridge import publish_attendance_event

def record_attendance(user_id: str, image_bytes: bytes):
    # 1. Recognize face
    result = search_face_by_image(image_bytes)
    
    # 2. Apply rules
    attendance = create_attendance_record(result)
    
    # 3. Save to DynamoDB
    save_attendance(attendance)
    
    # 4. Publish event (async, non-blocking)
    publish_attendance_event({
        'attendanceId': attendance['attendance_id'],
        'userId': user_id,
        'status': attendance['status'],
        'sessionType': attendance['session_type'],
        'timestamp': attendance['timestamp']
    })
    
    # 5. Return immediately
    return attendance
```

#### Monitoring EventBridge

**View EventBridge metrics:**
```bash
aws cloudwatch get-metric-statistics \
  --namespace AWS/Events \
  --metric-name Invocations \
  --dimensions Name=EventBusName,Value=smart-campus-events \
  --start-time 2026-08-06T00:00:00Z \
  --end-time 2026-08-06T23:59:59Z \
  --period 3600 \
  --statistics Sum \
  --region ap-southeast-1
```

**Monitor DLQ:**
```bash
# Check DLQ depth (should be 0)
aws sqs get-queue-attributes \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-analytics-dlq \
  --attribute-names ApproximateNumberOfMessages \
  --region ap-southeast-1
```

**If DLQ has messages → investigate:**
```bash
aws sqs receive-message \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-analytics-dlq \
  --max-number-of-messages 10 \
  --region ap-southeast-1
```

#### Troubleshooting

**Issue: Events not arriving in SQS**
- Check EventBridge rule is ENABLED
- Check event pattern matches published event
- Check SQS policy allows EventBridge to send

**Issue: Lambda not processing SQS messages**
- Check Lambda trigger is enabled
- Check Lambda has permission to read from SQS
- Check Lambda timeout sufficient

**Issue: Messages going to DLQ**
- Check Lambda logs for errors
- Increase maxReceiveCount if transient errors
- Fix Lambda code bugs

#### Cost Estimation

**Pricing:**
- EventBridge: $1.00 per 1M events
- SQS: $0.40 per 1M requests (first 1M free/month)

**Monthly estimate (10K attendance/day):**
- Events published: 10K × 30 = 300K = $0.30
- SQS requests: 600K (2 queues) = $0.24
- **Total: $0.54/month**

#### Verify Setup

Checklist:
- [ ] Event Bus created
- [ ] DLQs created
- [ ] Main SQS queues created with DLQ redrive
- [ ] EventBridge rules created và ENABLED
- [ ] Lambda triggers configured
- [ ] Test event successfully delivered
- [ ] Workers processed messages

#### Bước tiếp theo

Hãy chuyển sang [Bước 8: S3 Data Lake và Athena Analytics](../5.8-analytics) để xây dựng phân tích dữ liệu!
