---
title: "Cleanup Resources"
date: 2024-01-01
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

#### ⚠️ QUAN TRỌNG: Cleanup Resources

**BẮT BUỘC phải cleanup sau workshop để tránh bị tính phí!**

Ngay cả khi trong Free Tier, một số resources vẫn có thể phát sinh chi phí:
- 💰 API Gateway requests (sau 1M requests)
- 💰 DynamoDB On-Demand operations
- 💰 Rekognition calls
- 💰 S3 storage (sau 5GB)
- 💰 CloudWatch Logs (sau 5GB)

**Estimated cost nếu KHÔNG cleanup:** $50-300/month

---

#### Cleanup Strategy

**Recommended order** (xóa theo thứ tự ngược lại với creation):

1. Delete WAF WebACL associations
2. Delete EventBridge Rules và Targets
3. Delete Lambda Functions và Triggers
4. Delete API Gateway
5. Delete SQS Queues (+ DLQ)
6. Delete EventBridge Event Bus
7. Delete Athena Workgroup
8. Delete Glue Crawlers và Database
9. Delete S3 Buckets (empty trước)
10. Delete Rekognition Collection
11. Delete DynamoDB Tables
12. Delete CloudWatch Log Groups, Alarms, Dashboards
13. Delete IAM Roles và Policies
14. Delete Secrets Manager secrets
15. Verify billing $0

---

#### Script tự động: cleanup-all.sh

```bash
#!/bin/bash
set -e

echo "🧹 Smart Campus Cleanup Script"
echo "=============================="
echo ""
echo "⚠️  WARNING: This will DELETE ALL resources!"
echo "Are you sure? (type 'yes' to confirm)"
read CONFIRM

if [ "$CONFIRM" != "yes" ]; then
  echo "Cancelled."
  exit 0
fi

export AWS_REGION=ap-southeast-1
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

echo ""
echo "Starting cleanup in region: ${AWS_REGION}"
echo "Account: ${AWS_ACCOUNT_ID}"
echo ""

# === Step 1: Disassociate WAF ===
echo "[1/15] Disassociating WAF..."
WEB_ACL_ARN=$(aws wafv2 list-web-acls --scope REGIONAL --region ${AWS_REGION} --query "WebACLs[?Name=='smart-campus-waf'].ARN" --output text 2>/dev/null || true)
if [ -n "$WEB_ACL_ARN" ]; then
  API_ID=$(aws apigatewayv2 get-apis --region ${AWS_REGION} --query "Items[?Name=='smart-campus-api'].ApiId" --output text 2>/dev/null || true)
  if [ -n "$API_ID" ]; then
    aws wafv2 disassociate-web-acl \
      --resource-arn arn:aws:apigateway:${AWS_REGION}::/apis/${API_ID}/stages/prod \
      --region ${AWS_REGION} 2>/dev/null || true
  fi
  
  # Delete Web ACL
  aws wafv2 delete-web-acl \
    --name smart-campus-waf \
    --scope REGIONAL \
    --id $(aws wafv2 list-web-acls --scope REGIONAL --region ${AWS_REGION} --query "WebACLs[?Name=='smart-campus-waf'].Id" --output text) \
    --lock-token $(aws wafv2 list-web-acls --scope REGIONAL --region ${AWS_REGION} --query "WebACLs[?Name=='smart-campus-waf'].LockToken" --output text) \
    --region ${AWS_REGION} 2>/dev/null || true
fi
echo "✅ WAF cleanup done"

# === Step 2: Delete EventBridge Rules ===
echo "[2/15] Deleting EventBridge rules..."
for RULE in attendance-to-analytics attendance-to-notification task-to-analytics security-alert daily-crawler-schedule; do
  aws events remove-targets --rule ${RULE} --ids 1 --event-bus-name smart-campus-events --region ${AWS_REGION} 2>/dev/null || true
  aws events delete-rule --name ${RULE} --event-bus-name smart-campus-events --region ${AWS_REGION} 2>/dev/null || true
done
echo "✅ EventBridge rules deleted"

# === Step 3: Delete Lambda Function ===
echo "[3/15] Deleting Lambda function..."
# Remove event source mappings first
for UUID in $(aws lambda list-event-source-mappings --function-name smart-campus-api --region ${AWS_REGION} --query 'EventSourceMappings[].UUID' --output text 2>/dev/null); do
  aws lambda delete-event-source-mapping --uuid ${UUID} --region ${AWS_REGION} 2>/dev/null || true
done

aws lambda delete-function --function-name smart-campus-api --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Lambda deleted"

# === Step 4: Delete API Gateway ===
echo "[4/15] Deleting API Gateway..."
API_ID=$(aws apigatewayv2 get-apis --region ${AWS_REGION} --query "Items[?Name=='smart-campus-api'].ApiId" --output text 2>/dev/null || true)
if [ -n "$API_ID" ]; then
  aws apigatewayv2 delete-api --api-id ${API_ID} --region ${AWS_REGION} 2>/dev/null || true
fi
echo "✅ API Gateway deleted"

# === Step 5: Delete SQS Queues ===
echo "[5/15] Deleting SQS queues..."
for QUEUE in smart-campus-analytics-queue smart-campus-notification-queue smart-campus-analytics-dlq smart-campus-notification-dlq; do
  QUEUE_URL=$(aws sqs get-queue-url --queue-name ${QUEUE} --region ${AWS_REGION} --query 'QueueUrl' --output text 2>/dev/null || true)
  if [ -n "$QUEUE_URL" ]; then
    aws sqs delete-queue --queue-url ${QUEUE_URL} --region ${AWS_REGION} 2>/dev/null || true
  fi
done
echo "✅ SQS queues deleted"

# === Step 6: Delete EventBridge Event Bus ===
echo "[6/15] Deleting Event Bus..."
aws events delete-event-bus --name smart-campus-events --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Event Bus deleted"

# === Step 7: Delete Athena Workgroup ===
echo "[7/15] Deleting Athena workgroup..."
aws athena delete-work-group --work-group smart-campus-workgroup --recursive-delete-option --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Athena workgroup deleted"

# === Step 8: Delete Glue Resources ===
echo "[8/15] Deleting Glue resources..."
aws glue delete-crawler --name smart-campus-attendance-crawler --region ${AWS_REGION} 2>/dev/null || true
aws glue delete-crawler --name smart-campus-tasks-crawler --region ${AWS_REGION} 2>/dev/null || true

# Delete tables
aws glue delete-table --database-name smart_campus_db --name attendance --region ${AWS_REGION} 2>/dev/null || true
aws glue delete-table --database-name smart_campus_db --name tasks --region ${AWS_REGION} 2>/dev/null || true

# Delete database
aws glue delete-database --name smart_campus_db --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Glue resources deleted"

# === Step 9: Empty and Delete S3 Buckets ===
echo "[9/15] Deleting S3 buckets..."
for BUCKET in smart-campus-images-${AWS_ACCOUNT_ID} smart-campus-datalake-${AWS_ACCOUNT_ID} smart-campus-lambda-code-${AWS_ACCOUNT_ID} smart-campus-audit-logs-${AWS_ACCOUNT_ID}; do
  if aws s3 ls s3://${BUCKET} 2>/dev/null; then
    echo "  Emptying ${BUCKET}..."
    aws s3 rm s3://${BUCKET} --recursive 2>/dev/null || true
    
    echo "  Deleting ${BUCKET}..."
    aws s3 rb s3://${BUCKET} --force 2>/dev/null || true
  fi
done
echo "✅ S3 buckets deleted"

# === Step 10: Delete Rekognition Collection ===
echo "[10/15] Deleting Rekognition collection..."
aws rekognition delete-collection --collection-id smart-campus-faces --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Rekognition collection deleted"

# === Step 11: Delete DynamoDB Tables ===
echo "[11/15] Deleting DynamoDB tables..."
for TABLE in smart-campus-users smart-campus-faces smart-campus-attendance smart-campus-tasks smart-campus-leaves smart-campus-notifications smart-campus-security-incidents smart-campus-security-settings; do
  aws dynamodb delete-table --table-name ${TABLE} --region ${AWS_REGION} 2>/dev/null || true
done
echo "✅ DynamoDB tables deleted"

# === Step 12: Delete CloudWatch Resources ===
echo "[12/15] Deleting CloudWatch resources..."

# Delete alarms
for ALARM in HighErrorRate HighLambdaDuration DLQNotEmpty UnknownFaceSpike; do
  aws cloudwatch delete-alarms --alarm-names ${ALARM} --region ${AWS_REGION} 2>/dev/null || true
done

# Delete dashboard
aws cloudwatch delete-dashboards --dashboard-names SmartCampusDashboard --region ${AWS_REGION} 2>/dev/null || true

# Delete log groups
for LOG_GROUP in /aws/lambda/smart-campus-api /aws/apigateway/smart-campus-api /smart-campus/application /aws/events/smart-campus; do
  aws logs delete-log-group --log-group-name ${LOG_GROUP} --region ${AWS_REGION} 2>/dev/null || true
done
echo "✅ CloudWatch resources deleted"

# === Step 13: Delete IAM Resources ===
echo "[13/15] Deleting IAM resources..."

# Detach policies from Lambda role
aws iam detach-role-policy --role-name SmartCampusLambdaRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole 2>/dev/null || true
aws iam detach-role-policy --role-name SmartCampusLambdaRole --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess 2>/dev/null || true
aws iam detach-role-policy --role-name SmartCampusLambdaRole --policy-arn arn:aws:iam::aws:policy/AmazonRekognitionFullAccess 2>/dev/null || true
aws iam delete-role-policy --role-name SmartCampusLambdaRole --policy-name LeastPrivilegePolicy 2>/dev/null || true
aws iam delete-role --role-name SmartCampusLambdaRole 2>/dev/null || true

# Delete Glue Crawler role
aws iam detach-role-policy --role-name SmartCampusGlueCrawlerRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole 2>/dev/null || true
aws iam delete-role-policy --role-name SmartCampusGlueCrawlerRole --policy-name S3DataLakeAccess 2>/dev/null || true
aws iam delete-role --role-name SmartCampusGlueCrawlerRole 2>/dev/null || true

# Delete EventBridge role
aws iam delete-role-policy --role-name Amazon_EventBridge_Invoke_Glue_Crawler --policy-name GlueCrawlerAccess 2>/dev/null || true
aws iam delete-role --role-name Amazon_EventBridge_Invoke_Glue_Crawler 2>/dev/null || true

echo "✅ IAM resources deleted"

# === Step 14: Delete Secrets Manager ===
echo "[14/15] Deleting secrets..."
aws secretsmanager delete-secret --secret-id smart-campus/cognito --force-delete-without-recovery --region ${AWS_REGION} 2>/dev/null || true
aws secretsmanager delete-secret --secret-id smart-campus/ses --force-delete-without-recovery --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Secrets deleted"

# === Step 15: Delete SNS Topics ===
echo "[15/15] Deleting SNS topics..."
SNS_ARN=$(aws sns list-topics --region ${AWS_REGION} --query "Topics[?contains(TopicArn, 'smart-campus-alerts')].TopicArn" --output text 2>/dev/null || true)
if [ -n "$SNS_ARN" ]; then
  aws sns delete-topic --topic-arn ${SNS_ARN} --region ${AWS_REGION} 2>/dev/null || true
fi
echo "✅ SNS topics deleted"

echo ""
echo "✅ ✅ ✅ CLEANUP COMPLETE! ✅ ✅ ✅"
echo ""
echo "Next steps:"
echo "1. Wait 5-10 minutes for all deletions to propagate"
echo "2. Go to AWS Console → Billing → Bills"
echo "3. Verify charges = $0 for current month"
echo "4. Check Cost Explorer for any lingering charges"
echo ""
```

**Save script:**
```bash
chmod +x cleanup-all.sh
./cleanup-all.sh
```

---

#### Manual Cleanup (Alternative)

Nếu script fails, cleanup thủ công theo thứ tự:

**Step 1: WAF**
```bash
aws wafv2 list-web-acls --scope REGIONAL --region ap-southeast-1
# Disassociate và delete từng Web ACL
```

**Step 2: Lambda**
```bash
aws lambda delete-function --function-name smart-campus-api --region ap-southeast-1
```

**Step 3: API Gateway**
```bash
API_ID=$(aws apigatewayv2 get-apis --query "Items[?Name=='smart-campus-api'].ApiId" --output text)
aws apigatewayv2 delete-api --api-id ${API_ID}
```

**Step 4: SQS Queues**
```bash
aws sqs list-queues --queue-name-prefix smart-campus
# Delete mỗi queue URL
```

**Step 5: EventBridge**
```bash
aws events list-rules --event-bus-name smart-campus-events
# Remove targets và delete rules
aws events delete-event-bus --name smart-campus-events
```

**Step 6: Glue**
```bash
aws glue delete-crawler --name smart-campus-attendance-crawler
aws glue delete-database --name smart_campus_db
```

**Step 7: S3 Buckets**
```bash
aws s3 rm s3://smart-campus-images-${AWS_ACCOUNT_ID} --recursive
aws s3 rb s3://smart-campus-images-${AWS_ACCOUNT_ID}

aws s3 rm s3://smart-campus-datalake-${AWS_ACCOUNT_ID} --recursive
aws s3 rb s3://smart-campus-datalake-${AWS_ACCOUNT_ID}
```

**Step 8: Rekognition**
```bash
aws rekognition delete-collection --collection-id smart-campus-faces
```

**Step 9: DynamoDB**
```bash
for TABLE in smart-campus-users smart-campus-faces smart-campus-attendance; do
  aws dynamodb delete-table --table-name ${TABLE}
done
```

**Step 10: CloudWatch**
```bash
# Delete alarms
aws cloudwatch delete-alarms --alarm-names HighErrorRate HighLambdaDuration DLQNotEmpty

# Delete dashboard
aws cloudwatch delete-dashboards --dashboard-names SmartCampusDashboard

# Delete log groups
aws logs delete-log-group --log-group-name /aws/lambda/smart-campus-api
```

**Step 11: IAM Roles**
```bash
# Detach policies first
aws iam detach-role-policy --role-name SmartCampusLambdaRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Delete role
aws iam delete-role --role-name SmartCampusLambdaRole
```

---

#### Verification Checklist

**Verify tất cả resources đã xóa:**

```bash
#!/bin/bash
echo "=== Verification Checklist ==="

# Lambda
echo -n "Lambda functions: "
aws lambda list-functions --region ap-southeast-1 --query 'Functions[?contains(FunctionName, `smart-campus`)].FunctionName' --output text
echo ""

# API Gateway
echo -n "API Gateway: "
aws apigatewayv2 get-apis --region ap-southeast-1 --query "Items[?contains(Name, 'smart-campus')].Name" --output text
echo ""

# SQS
echo -n "SQS queues: "
aws sqs list-queues --region ap-southeast-1 --queue-name-prefix smart-campus --query 'QueueUrls' --output text
echo ""

# S3
echo -n "S3 buckets: "
aws s3 ls | grep smart-campus
echo ""

# DynamoDB
echo -n "DynamoDB tables: "
aws dynamodb list-tables --region ap-southeast-1 --query 'TableNames[?contains(@, `smart-campus`)]' --output text
echo ""

# Rekognition
echo -n "Rekognition collections: "
aws rekognition list-collections --region ap-southeast-1 --query 'CollectionIds[?contains(@, `smart-campus`)]' --output text
echo ""

# EventBridge
echo -n "Event buses: "
aws events list-event-buses --region ap-southeast-1 --query 'EventBuses[?contains(Name, `smart-campus`)].Name' --output text
echo ""

# CloudWatch Alarms
echo -n "CloudWatch Alarms: "
aws cloudwatch describe-alarms --region ap-southeast-1 --query 'MetricAlarms[?contains(AlarmName, `smart`) || contains(AlarmName, `High`)].AlarmName' --output text
echo ""

echo ""
echo "If all outputs are empty → ✅ Cleanup successful!"
echo "If any resources remain → ❌ Cleanup incomplete, delete manually"
```

**Expected output (all empty):**
```
=== Verification Checklist ===
Lambda functions: 
API Gateway: 
SQS queues: 
S3 buckets: 
DynamoDB tables: 
Rekognition collections: 
Event buses: 
CloudWatch Alarms: 

✅ Cleanup successful!
```

---

#### Verify Billing

**Option 1: AWS Console**
1. Go to https://console.aws.amazon.com/billing/
2. Click "Bills" in left sidebar
3. Check "Total charges" for current month
4. Should be **$0.00** (hoặc trong Free Tier)

**Option 2: AWS CLI**
```bash
aws ce get-cost-and-usage \
  --time-period Start=2026-08-01,End=2026-08-31 \
  --granularity MONTHLY \
  --metrics UnblendedCost \
  --region us-east-1
```

Expected:
```json
{
  "ResultsByTime": [{
    "Total": {
      "UnblendedCost": {
        "Amount": "0.00",
        "Unit": "USD"
      }
    }
  }]
}
```

---

#### Common Cleanup Issues

**Issue 1: S3 bucket không xóa được**
```
Error: BucketNotEmpty
```
**Solution:** Empty bucket trước
```bash
aws s3 rm s3://bucket-name --recursive
aws s3 rb s3://bucket-name
```

**Issue 2: IAM role có attached policies**
```
Error: DeleteConflict
```
**Solution:** Detach tất cả policies trước
```bash
aws iam list-attached-role-policies --role-name RoleName
aws iam detach-role-policy --role-name RoleName --policy-arn arn:...
```

**Issue 3: DynamoDB table đang được used**
```
Error: ResourceInUseException
```
**Solution:** Disable streams và PITR trước
```bash
aws dynamodb update-continuous-backups \
  --table-name TableName \
  --point-in-time-recovery-specification PointInTimeRecoveryEnabled=false
```

**Issue 4: EventBridge rule có targets**
```
Error: Cannot delete rule with targets
```
**Solution:** Remove targets trước
```bash
aws events remove-targets --rule RuleName --ids 1 2 3
aws events delete-rule --name RuleName
```

---

#### Post-Cleanup Actions

**1. Delete AWS Account** (optional, nếu chỉ dùng cho workshop)
- Go to Account Settings
- Close account
- Confirm via email

**2. Export CloudWatch Logs** (nếu cần keep cho báo cáo)
```bash
aws logs create-export-task \
  --log-group-name /aws/lambda/smart-campus-api \
  --from $(date -d '7 days ago' +%s)000 \
  --to $(date +%s)000 \
  --destination s3://my-personal-backup-bucket/logs/
```

**3. Take screenshots** (cho báo cáo thực tập)
- Architecture diagram from AWS Console
- CloudWatch Dashboard
- DynamoDB tables structure
- API Gateway endpoints
- Lambda function config

---

#### Final Checklist

- [ ] Run cleanup-all.sh script
- [ ] Verify all resources deleted (verification script)
- [ ] Check AWS Billing = $0
- [ ] Export logs/screenshots cho báo cáo
- [ ] Remove AWS CLI credentials (optional)
- [ ] Close AWS account (optional)

---

#### Cost if You Forget to Cleanup

**Worst case scenario (1 month):**
- Lambda: $0.44
- API Gateway: $0.10
- DynamoDB: $2.50
- Rekognition: $300 (nếu có traffic)
- S3: $0.50
- CloudWatch: $5.00
- **Total: ~$308/month**

**Free Tier safety net:**
- First 12 months: Most services free
- After 12 months: Charges start

**⚠️ IMPORTANT:** Setup billing alert NGAY LẬP TỨC:
```bash
aws budgets create-budget \
  --account-id ${AWS_ACCOUNT_ID} \
  --budget '{
    "BudgetName": "Monthly-Budget-Alert",
    "BudgetLimit": {
      "Amount": "10",
      "Unit": "USD"
    },
    "TimeUnit": "MONTHLY",
    "BudgetType": "COST"
  }' \
  --notifications-with-subscribers '[{
    "Notification": {
      "NotificationType": "ACTUAL",
      "ComparisonOperator": "GREATER_THAN",
      "Threshold": 80
    },
    "Subscribers": [{
      "SubscriptionType": "EMAIL",
      "Address": "your-email@example.com"
    }]
  }]'
```

---

## 🎉 Xin chúc mừng!

Bạn đã hoàn thành **Smart Campus Serverless Attendance System Workshop**!

**What you built:**
- ✅ Serverless attendance system với face recognition
- ✅ Event-driven architecture với EventBridge + SQS
- ✅ Data Lake analytics với Athena
- ✅ Production-ready monitoring
- ✅ Cost-optimized, secure, scalable

**Skills learned:**
- AWS Lambda, API Gateway, DynamoDB
- Amazon Rekognition (AI/ML)
- Event-driven architecture
- Infrastructure as Code
- Cost optimization
- Security best practices

**Next steps:**
- ⭐ Add features: Mobile app, Geofencing, Multi-tenant
- ⭐ Deploy to production với CI/CD pipeline
- ⭐ Scale to millions of users
- ⭐ Apply cho AWS certification exams!

---

**Thank you for completing this workshop!** 
