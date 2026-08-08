---
title: "Monitoring and CloudWatch"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Overview

Monitoring is **CRITICAL** for production systems. In this step, you will set up:
- CloudWatch Logs aggregation
- Custom metrics
- CloudWatch Dashboard
- Alarms for critical events
- X-Ray tracing (optional)

## Why Monitoring?

**Without monitoring:**
- ❌ Don't know when system fails
- ❌ Can't detect performance degradation
- ❌ Can't track usage patterns
- ❌ Debugging is very hard during issues

**With monitoring:**
- ✅ Real-time error detection
- ✅ Automatic alerts when thresholds exceeded
- ✅ Analyze trends and patterns
- ✅ Fast debugging with detailed logs

## Monitoring Architecture

```
[All Services]
    ↓ Logs
[CloudWatch Logs]
    ↓ Metrics Filter
[CloudWatch Metrics]
    ↓ Threshold
[CloudWatch Alarms]
    ↓ Notification
[SNS Topic] → Email/SMS
```

## Step 1: Create CloudWatch Log Groups

**Lambda logs (auto-created, but verify):**
```bash
aws logs describe-log-groups \
  --log-group-name-prefix /aws/lambda/smart-campus \
  --region ap-southeast-1
```

Expected:
```
/aws/lambda/smart-campus-api
```

**API Gateway logs (created in step 6):**
```bash
aws logs describe-log-groups \
  --log-group-name-prefix /aws/apigateway/smart-campus \
  --region ap-southeast-1
```

**Application logs (custom):**
```bash
aws logs create-log-group \
  --log-group-name /smart-campus/application \
  --region ap-southeast-1

# Set retention (30 days)
aws logs put-retention-policy \
  --log-group-name /smart-campus/application \
  --retention-in-days 30 \
  --region ap-southeast-1
```

## Step 2: Setup Structured Logging

**Best practice: Log JSON format for easy parsing**

```python
# app/core/logger.py
import json
import logging
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'function': record.funcName,
            'line': record.lineno
        }
        
        # Add exception info if present
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)
        
        # Add custom fields
        if hasattr(record, 'user_id'):
            log_data['user_id'] = record.user_id
        if hasattr(record, 'request_id'):
            log_data['request_id'] = record.request_id
        
        return json.dumps(log_data)

# Configure logger
logger = logging.getLogger('smart-campus')
logger.setLevel(logging.INFO)

handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
```

**Usage:**
```python
from app.core.logger import logger

# Basic log
logger.info("User logged in")

# Log with context
logger.info("Attendance recorded", extra={
    'user_id': 'user-001',
    'request_id': 'req-123',
    'status': 'PRESENT'
})

# Log error
try:
    result = recognize_face(image)
except Exception as e:
    logger.error("Face recognition failed", exc_info=True, extra={
        'user_id': user_id,
        'error_type': type(e).__name__
    })
```

## Step 3: Create CloudWatch Metrics Filter

**Metric 1: Attendance success rate**

```bash
aws logs put-metric-filter \
  --log-group-name /aws/lambda/smart-campus-api \
  --filter-name AttendanceRecorded \
  --filter-pattern '{ $.message = "Attendance recorded*" }' \
  --metric-transformations \
    metricName=AttendanceSuccess,metricNamespace=SmartCampus,metricValue=1,defaultValue=0 \
  --region ap-southeast-1
```

**Metric 2: Face recognition errors**

```bash
aws logs put-metric-filter \
  --log-group-name /aws/lambda/smart-campus-api \
  --filter-name FaceRecognitionError \
  --filter-pattern '{ $.level = "ERROR" && $.message = "*Face recognition*" }' \
  --metric-transformations \
    metricName=FaceRecognitionErrors,metricNamespace=SmartCampus,metricValue=1,defaultValue=0 \
  --region ap-southeast-1
```

**Metric 3: Unknown faces detected**

```bash
aws logs put-metric-filter \
  --log-group-name /aws/lambda/smart-campus-api \
  --filter-name UnknownFaceDetected \
  --filter-pattern '{ $.message = "*Unknown face*" }' \
  --metric-transformations \
    metricName=UnknownFaces,metricNamespace=SmartCampus,metricValue=1,defaultValue=0 \
  --region ap-southeast-1
```

**Metric 4: API latency (high latency requests)**

```bash
aws logs put-metric-filter \
  --log-group-name /aws/apigateway/smart-campus-api \
  --filter-name HighLatency \
  --filter-pattern '[..., latency > 3000]' \
  --metric-transformations \
    metricName=HighLatencyRequests,metricNamespace=SmartCampus,metricValue=1,defaultValue=0 \
  --region ap-southeast-1
```

## Step 4: Create Custom CloudWatch Dashboard

**Create dashboard:**

```bash
cat > dashboard-config.json <<'EOF'
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "title": "API Requests",
        "metrics": [
          ["AWS/ApiGateway", "Count", {"stat": "Sum", "label": "Total Requests"}],
          [".", "4XXError", {"stat": "Sum", "label": "Client Errors"}],
          [".", "5XXError", {"stat": "Sum", "label": "Server Errors"}]
        ],
        "period": 300,
        "stat": "Sum",
        "region": "ap-southeast-1",
        "yAxis": {"left": {"min": 0}}
      }
    },
    {
      "type": "metric",
      "properties": {
        "title": "Lambda Performance",
        "metrics": [
          ["AWS/Lambda", "Duration", {"stat": "Average", "label": "Avg Duration"}],
          ["...", {"stat": "Maximum", "label": "Max Duration"}],
          [".", "Errors", {"stat": "Sum", "label": "Errors"}],
          [".", "Throttles", {"stat": "Sum", "label": "Throttles"}]
        ],
        "period": 300,
        "region": "ap-southeast-1"
      }
    },
    {
      "type": "metric",
      "properties": {
        "title": "Attendance Metrics",
        "metrics": [
          ["SmartCampus", "AttendanceSuccess", {"stat": "Sum", "label": "Successful Check-ins"}],
          [".", "FaceRecognitionErrors", {"stat": "Sum", "label": "Recognition Errors"}],
          [".", "UnknownFaces", {"stat": "Sum", "label": "Unknown Faces"}]
        ],
        "period": 300,
        "region": "ap-southeast-1"
      }
    },
    {
      "type": "metric",
      "properties": {
        "title": "DynamoDB Performance",
        "metrics": [
          ["AWS/DynamoDB", "UserErrors", {"stat": "Sum"}],
          [".", "SystemErrors", {"stat": "Sum"}],
          [".", "ConsumedReadCapacityUnits", {"stat": "Sum"}],
          [".", "ConsumedWriteCapacityUnits", {"stat": "Sum"}]
        ],
        "period": 300,
        "region": "ap-southeast-1"
      }
    },
    {
      "type": "metric",
      "properties": {
        "title": "SQS Queue Depth",
        "metrics": [
          ["AWS/SQS", "ApproximateNumberOfMessagesVisible", {"stat": "Average", "label": "Analytics Queue"}],
          ["...", {"stat": "Average", "label": "Notification Queue"}],
          [".", "NumberOfMessagesSent", {"stat": "Sum"}],
          [".", "NumberOfMessagesDeleted", {"stat": "Sum"}]
        ],
        "period": 300,
        "region": "ap-southeast-1"
      }
    },
    {
      "type": "log",
      "properties": {
        "title": "Recent Errors",
        "query": "SOURCE '/aws/lambda/smart-campus-api'\n| fields @timestamp, @message\n| filter @message like /ERROR/\n| sort @timestamp desc\n| limit 20",
        "region": "ap-southeast-1"
      }
    }
  ]
}
EOF

aws cloudwatch put-dashboard \
  --dashboard-name SmartCampusDashboard \
  --dashboard-body file://dashboard-config.json \
  --region ap-southeast-1
```

**View dashboard:**
```
https://console.aws.amazon.com/cloudwatch/home?region=ap-southeast-1#dashboards:name=SmartCampusDashboard
```

## Step 5: Create CloudWatch Alarms

**Alarm 1: High error rate**

```bash
# Create SNS topic for alerts
aws sns create-topic \
  --name smart-campus-alerts \
  --region ap-southeast-1

# Subscribe email
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-southeast-1:${AWS_ACCOUNT_ID}:smart-campus-alerts \
  --protocol email \
  --notification-endpoint your-email@example.com \
  --region ap-southeast-1
```

**Confirm subscription** (check email inbox)

**Create alarm:**
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name HighErrorRate \
  --alarm-description "Alert when error rate > 5%" \
  --metric-name 5XXError \
  --namespace AWS/ApiGateway \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions arn:aws:sns:ap-southeast-1:${AWS_ACCOUNT_ID}:smart-campus-alerts \
  --region ap-southeast-1
```

**Alarm 2: High Lambda duration**

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name HighLambdaDuration \
  --alarm-description "Alert when Lambda duration > 10s" \
  --metric-name Duration \
  --namespace AWS/Lambda \
  --dimensions Name=FunctionName,Value=smart-campus-api \
  --statistic Average \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 10000 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions arn:aws:sns:ap-southeast-1:${AWS_ACCOUNT_ID}:smart-campus-alerts \
  --region ap-southeast-1
```

**Alarm 3: DLQ not empty**

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name DLQNotEmpty \
  --alarm-description "Alert when messages in DLQ" \
  --metric-name ApproximateNumberOfMessagesVisible \
  --namespace AWS/SQS \
  --dimensions Name=QueueName,Value=smart-campus-analytics-dlq \
  --statistic Average \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --alarm-actions arn:aws:sns:ap-southeast-1:${AWS_ACCOUNT_ID}:smart-campus-alerts \
  --region ap-southeast-1
```

**Alarm 4: Unknown face spike**

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name UnknownFaceSpike \
  --alarm-description "Alert when too many unknown faces" \
  --metric-name UnknownFaces \
  --namespace SmartCampus \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions arn:aws:sns:ap-southeast-1:${AWS_ACCOUNT_ID}:smart-campus-alerts \
  --region ap-southeast-1
```

## Step 6: CloudWatch Insights Queries

**Query 1: Top errors in last 1 hour**

```
SOURCE '/aws/lambda/smart-campus-api'
| fields @timestamp, @message, @logStream
| filter @message like /ERROR/
| stats count() by @message
| sort count desc
| limit 20
```

**Execute:**
```bash
aws logs start-query \
  --log-group-name /aws/lambda/smart-campus-api \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | stats count() by @message | sort count desc | limit 20' \
  --region ap-southeast-1
```

**Query 2: Slowest API requests**

```
SOURCE '/aws/apigateway/smart-campus-api'
| fields @timestamp, httpMethod, path, latency
| filter latency > 1000
| sort latency desc
| limit 50
```

**Query 3: Users with most check-ins**

```
SOURCE '/aws/lambda/smart-campus-api'
| fields @timestamp, user_id
| filter @message like /Attendance recorded/
| stats count() by user_id
| sort count desc
| limit 20
```

**Query 4: Error rate by hour**

```
SOURCE '/aws/lambda/smart-campus-api'
| fields @timestamp, level
| filter level = "ERROR"
| stats count() as error_count by bin(5m)
```

## Step 7: Setup X-Ray Tracing (Advanced)

**Enable X-Ray for Lambda:**

```bash
aws lambda update-function-configuration \
  --function-name smart-campus-api \
  --tracing-config Mode=Active \
  --region ap-southeast-1
```

**Enable X-Ray for API Gateway:**

```bash
aws apigatewayv2 update-stage \
  --api-id ${API_ID} \
  --stage-name prod \
  --default-route-settings '{
    "DetailedMetricsEnabled": true,
    "DataTraceEnabled": true
  }' \
  --region ap-southeast-1
```

**Add X-Ray SDK to Python code:**

```python
# requirements.txt
aws-xray-sdk==2.12.1

# app/main.py
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

app = FastAPI()
xray_recorder.configure(service='smart-campus-api')
XRayMiddleware(app, xray_recorder)

# Instrument AWS SDK calls
from aws_xray_sdk.core import patch_all
patch_all()
```

**View traces:**
```
https://console.aws.amazon.com/xray/home?region=ap-southeast-1#/service-map
```

## Step 8: Cost Optimization for Logs

**Reduce log retention:**

```bash
# Keep Lambda logs 7 days only
aws logs put-retention-policy \
  --log-group-name /aws/lambda/smart-campus-api \
  --retention-in-days 7 \
  --region ap-southeast-1

# Keep API Gateway logs 3 days
aws logs put-retention-policy \
  --log-group-name /aws/apigateway/smart-campus-api \
  --retention-in-days 3 \
  --region ap-southeast-1
```

**Export old logs to S3:**

```bash
aws logs create-export-task \
  --log-group-name /aws/lambda/smart-campus-api \
  --from $(date -u -d '7 days ago' +%s)000 \
  --to $(date -u +%s)000 \
  --destination smart-campus-logs-archive-${AWS_ACCOUNT_ID} \
  --destination-prefix lambda-logs/ \
  --region ap-southeast-1
```

## Monitoring Best Practices

1. **Log levels:**
   - ERROR: Actual errors needing action
   - WARN: Potential issues
   - INFO: Normal operations (attendance, login...)
   - DEBUG: Detailed debugging (only when debugging)

2. **Correlation IDs:**
   - Add request_id to every log
   - Trace request across multiple services

3. **Alert fatigue:**
   - Only alert for critical issues
   - Tune thresholds to avoid false positives

4. **Dashboard design:**
   - Top: Business metrics (attendance, tasks)
   - Middle: Technical metrics (latency, errors)
   - Bottom: Infrastructure metrics (CPU, memory)

## Testing Alarms

**Trigger high error rate alarm:**

```bash
# Generate errors
for i in {1..20}; do
  curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/invalid-endpoint
done
```

**Check alarm state:**
```bash
aws cloudwatch describe-alarms \
  --alarm-names HighErrorRate \
  --region ap-southeast-1 \
  --query 'MetricAlarms[0].StateValue'
```

Expected: `ALARM` (after 2 evaluation periods)

**Check SNS notification in email**

## Cost Estimation

**CloudWatch pricing:**
- Logs ingestion: $0.50/GB
- Logs storage: $0.03/GB/month
- Custom metrics: $0.30/metric/month
- Alarms: $0.10/alarm/month
- Dashboard: $3/dashboard/month

**Monthly estimate (100K requests):**
- Logs: 1GB ingested + 500MB stored = $0.515
- Custom metrics: 5 metrics × $0.30 = $1.50
- Alarms: 4 alarms × $0.10 = $0.40
- Dashboard: $3.00
- **Total: ~$5.42/month**

**Free Tier:**
- 5GB logs ingestion/month
- 10 custom metrics
- 10 alarms

→ Workshop likely stays in free tier!

## Troubleshooting

**Issue: Metrics not showing up**
- Wait 5-10 minutes (metrics delayed)
- Check metric filter pattern correct
- Verify log group name correct

**Issue: Alarm not triggering**
- Check threshold settings
- Verify SNS subscription confirmed
- Check evaluation periods

**Issue: Dashboard blank**
- Check region correct
- Verify metrics exist
- Check time range

## Verify Setup

Checklist:
- [ ] All log groups exist and configured
- [ ] Metrics filters created
- [ ] Dashboard showing data
- [ ] Alarms created and ACTIVE
- [ ] SNS topic subscribed
- [ ] Test alarm triggered successfully

## Next Step

Proceed to [Step 11: Security and Optimization](../5.11-optimization) to optimize the system!