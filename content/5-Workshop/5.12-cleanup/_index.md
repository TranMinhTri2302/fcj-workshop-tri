---
title: "Cleanup Resources"
date: 2024-01-01
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

#### ⚠️ IMPORTANT: Cleanup Resources

**It is MANDATORY to clean up your resources after the workshop to avoid unexpected charges!**

Even within the AWS Free Tier, certain resources can still incur costs:
- API Gateway requests (beyond 1M requests)
- DynamoDB On-Demand operations
- Rekognition calls
- S3 storage (beyond 5GB)
- CloudWatch Logs (beyond 5GB)
- Kinesis Firehose ingestion
- Athena query scanning
- X-Ray traces

**Estimated cost if NOT cleaned up: $50-300/month**

---

#### Cleanup Strategy

**Recommended order** (delete in reverse order of creation - dependency order):

1. **WAF WebACL associations** (must disassociate before deleting)
2. **EventBridge Rules & Targets** (remove targets first, delete rules later)
3. **Lambda Functions & Event Source Mappings** (delete mappings first)
4. **API Gateway** (HTTP API)
5. **SQS Queues (+ DLQ)** (purge messages first)
6. **EventBridge Event Bus** (custom bus)
7. **Athena Workgroup** (recursive delete)
8. **Glue Crawlers, Tables, Database**
9. **S3 Buckets** (empty first using `--recursive`, then delete)
10. **Rekognition Collection**
11. **DynamoDB Tables** (all 8 tables)
12. **CloudWatch Log Groups, Alarms, Dashboards**
13. **IAM Roles & Policies** (detach policies first, then delete roles)
14. **Secrets Manager secrets** (force delete without recovery)
15. **SNS Topics**
16. **Kinesis Firehose Delivery Stream**
17. **Verify billing is $0** (Billing console + Cost Explorer)

---

#### Automated Script: cleanup-all.sh

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

# === Step 1: Disassociate & Delete WAF ===
echo "[1/17] Disassociating & Deleting WAF..."
WEB_ACL_ARN=$(aws wafv2 list-web-acls --scope REGIONAL --region ${AWS_REGION} --query "WebACLs[?Name=='smart-campus-waf'].ARN" --output text 2>/dev/null || true)
WEB_ACL_ID=$(aws wafv2 list-web-acls --scope REGIONAL --region ${AWS_REGION} --query "WebACLs[?Name=='smart-campus-waf'].Id" --output text 2>/dev/null || true)
WEB_ACL_LOCK=$(aws wafv2 list-web-acls --scope REGIONAL --region ${AWS_REGION} --query "WebACLs[?Name=='smart-campus-waf'].LockToken" --output text 2>/dev/null || true)

if [ -n "$WEB_ACL_ARN" ] && [ -n "$WEB_ACL_ID" ]; then
  # Disassociate from API Gateway
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
    --id ${WEB_ACL_ID} \
    --lock-token ${WEB_ACL_LOCK} \
    --region ${AWS_REGION} 2>/dev/null || true
  
  # Delete IP Set
  IP_SET_ARN=$(aws wafv2 list-ip-sets --scope REGIONAL --region ${AWS_REGION} --query "IPSets[?Name=='SmartCampusIPSet'].ARN" --output text 2>/dev/null || true)
  IP_SET_ID=$(aws wafv2 list-ip-sets --scope REGIONAL --region ${AWS_REGION} --query "IPSets[?Name=='SmartCampusIPSet'].Id" --output text 2>/dev/null || true)
  IP_SET_LOCK=$(aws wafv2 list-ip-sets --scope REGIONAL --region ${AWS_REGION} --query "IPSets[?Name=='SmartCampusIPSet'].LockToken" --output text 2>/dev/null || true)
  
  if [ -n "$IP_SET_ID" ]; then
    aws wafv2 delete-ip-set \
      --name SmartCampusIPSet \
      --scope REGIONAL \
      --id ${IP_SET_ID} \
      --lock-token ${IP_SET_LOCK} \
      --region ${AWS_REGION} 2>/dev/null || true
  fi
fi
echo "✅ WAF cleanup done"

# === Step 2: Delete EventBridge Rules & Targets ===
echo "[2/17] Deleting EventBridge rules & targets..."
for RULE in attendance-to-analytics attendance-to-notification task-to-analytics task-to-notification security-alert daily-crawler-schedule tasks-overdue-checker; do
  aws events remove-targets --rule ${RULE} --ids 1 --event-bus-name smart-campus-events --region ${AWS_REGION} 2>/dev/null || true
  aws events delete-rule --name ${RULE} --event-bus-name smart-campus-events --region ${AWS_REGION} 2>/dev/null || true
done
echo "✅ EventBridge rules deleted"

# === Step 3: Delete Lambda Functions & Event Source Mappings ===
echo "[3/17] Deleting Lambda functions & event source mappings..."
for FUNC in smart-campus-api smart-campus-analytics-worker smart-campus-notification-worker smart-campus-tasks-overdue-checker; do
  # Remove event source mappings first
  for UUID in $(aws lambda list-event-source-mappings --function-name ${FUNC} --region ${AWS_REGION} --query 'EventSourceMappings[].UUID' --output text 2>/dev/null); do
    aws lambda delete-event-source-mapping --uuid ${UUID} --region ${AWS_REGION} 2>/dev/null || true
  done
  
  # Delete function
  aws lambda delete-function --function-name ${FUNC} --region ${AWS_REGION} 2>/dev/null || true
done
echo "✅ Lambda functions deleted"

# === Step 4: Delete API Gateway ===
echo "[4/17] Deleting API Gateway..."
API_ID=$(aws apigatewayv2 get-apis --region ${AWS_REGION} --query "Items[?Name=='smart-campus-api'].ApiId" --output text 2>/dev/null || true)
if [ -n "$API_ID" ]; then
  aws apigatewayv2 delete-api --api-id ${API_ID} --region ${AWS_REGION} 2>/dev/null || true
fi
echo "✅ API Gateway deleted"

# === Step 5: Delete SQS Queues (+ DLQ) ===
echo "[5/17] Deleting SQS queues..."
for QUEUE in smart-campus-analytics-queue smart-campus-notification-queue smart-campus-analytics-dlq smart-campus-notification-dlq; do
  QUEUE_URL=$(aws sqs get-queue-url --queue-name ${QUEUE} --region ${AWS_REGION} --query 'QueueUrl' --output text 2>/dev/null || true)
  if [ -n "$QUEUE_URL" ]; then
    # Purge queue first
    aws sqs purge-queue --queue-url ${QUEUE_URL} --region ${AWS_REGION} 2>/dev/null || true
    # Delete queue
    aws sqs delete-queue --queue-url ${QUEUE_URL} --region ${AWS_REGION} 2>/dev/null || true
  fi
done
echo "✅ SQS queues deleted"

# === Step 6: Delete EventBridge Event Bus ===
echo "[6/17] Deleting Event Bus..."
aws events delete-event-bus --name smart-campus-events --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Event Bus deleted"

# === Step 7: Delete Athena Workgroup ===
echo "[7/17] Deleting Athena workgroup..."
aws athena delete-work-group --work-group smart-campus-workgroup --recursive-delete-option --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Athena workgroup deleted"

# === Step 8: Delete Glue Resources ===
echo "[8/17] Deleting Glue resources..."
# Delete crawlers
for CRAWLER in smart-campus-attendance-crawler smart-campus-tasks-crawler smart-campus-users-crawler; do
  aws glue delete-crawler --name ${CRAWLER} --region ${AWS_REGION} 2>/dev/null || true
done

# Delete tables
for TABLE in attendance tasks users leaves events; do
  aws glue delete-table --database-name smart_campus_db --name ${TABLE} --region ${AWS_REGION} 2>/dev/null || true
done

# Delete database
aws glue delete-database --name smart_campus_db --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Glue resources deleted"

# === Step 9: Delete Kinesis Firehose Delivery Stream ===
echo "[9/17] Deleting Kinesis Firehose delivery stream..."
aws firehose delete-delivery-stream \
  --delivery-stream-name smart-campus-attendance-stream \
  --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Firehose delivery stream deleted"

# === Step 10: Empty and Delete S3 Buckets ===
echo "[10/17] Deleting S3 buckets..."
for BUCKET in smart-campus-images-${AWS_ACCOUNT_ID} smart-campus-datalake-${AWS_ACCOUNT_ID} smart-campus-lambda-code-${AWS_ACCOUNT_ID} smart-campus-audit-logs-${AWS_ACCOUNT_ID} smart-campus-frontend-${AWS_ACCOUNT_ID}; do
  if aws s3 ls s3://${BUCKET} 2>/dev/null; then
    echo "  Emptying ${BUCKET}..."
    aws s3 rm s3://${BUCKET} --recursive 2>/dev/null || true
    
    echo "  Deleting ${BUCKET}..."
    aws s3 rb s3://${BUCKET} --force 2>/dev/null || true
  fi
done
echo "✅ S3 buckets deleted"

# === Step 11: Delete Rekognition Collection ===
echo "[11/17] Deleting Rekognition collection..."
aws rekognition delete-collection --collection-id smart-campus-faces --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Rekognition collection deleted"

# === Step 12: Delete DynamoDB Tables (All 8 tables) ===
echo "[12/17] Deleting DynamoDB tables..."
for TABLE in smart-campus-users smart-campus-faces smart-campus-attendance smart-campus-tasks smart-campus-leaves smart-campus-notifications smart-campus-security smart-campus-holidays; do
  aws dynamodb delete-table --table-name ${TABLE} --region ${AWS_REGION} 2>/dev/null || true
done
echo "✅ DynamoDB tables deleted"

# === Step 13: Delete CloudWatch Resources ===
echo "[13/17] Deleting CloudWatch resources..."

# Delete alarms
for ALARM in SmartCampus-HighErrorRate SmartCampus-LivenessFailureSpike SmartCampus-DLQMessages SmartCampus-API5xxErrors SmartCampus-LambdaThrottles SmartCampus-HighLambdaDuration SmartCampus-UnknownFaceSpike; do
  aws cloudwatch delete-alarms --alarm-names ${ALARM} --region ${AWS_REGION} 2>/dev/null || true
done

# Delete dashboard
aws cloudwatch delete-dashboards --dashboard-names SmartCampus-Production --region ${AWS_REGION} 2>/dev/null || true

# Delete log groups
for LOG_GROUP in /aws/lambda/smart-campus-api /aws/lambda/smart-campus-analytics-worker /aws/lambda/smart-campus-notification-worker /aws/lambda/smart-campus-tasks-overdue-checker /aws/apigateway/smart-campus /aws/events/smart-campus /smart-campus/application; do
  aws logs delete-log-group --log-group-name ${LOG_GROUP} --region ${AWS_REGION} 2>/dev/null || true
done
echo "✅ CloudWatch resources deleted"

# === Step 14: Delete IAM Resources ===
echo "[14/17] Deleting IAM resources..."

# Lambda role
aws iam detach-role-policy --role-name SmartCampusLambdaRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole 2>/dev/null || true
aws iam detach-role-policy --role-name SmartCampusLambdaRole --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess 2>/dev/null || true
aws iam detach-role-policy --role-name SmartCampusLambdaRole --policy-arn arn:aws:iam::aws:policy/AmazonRekognitionFullAccess 2>/dev/null || true
aws iam delete-role-policy --role-name SmartCampusLambdaRole --policy-name LeastPrivilegePolicy 2>/dev/null || true
aws iam delete-role --role-name SmartCampusLambdaRole 2>/dev/null || true

# Glue Crawler role
aws iam detach-role-policy --role-name SmartCampusGlueCrawlerRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole 2>/dev/null || true
aws iam delete-role-policy --role-name SmartCampusGlueCrawlerRole --policy-name S3DataLakeAccess 2>/dev/null || true
aws iam delete-role --role-name SmartCampusGlueCrawlerRole 2>/dev/null || true

# EventBridge role (for Glue crawler schedule)
aws iam delete-role-policy --role-name Amazon_EventBridge_Invoke_Glue_Crawler --policy-name GlueCrawlerAccess 2>/dev/null || true
aws iam delete-role --role-name Amazon_EventBridge_Invoke_Glue_Crawler 2>/dev/null || true

# CodeBuild role
aws iam detach-role-policy --role-name SmartCampusCodeBuildRole --policy-arn arn:aws:iam::aws:policy/AWSCodeBuildAdminAccess 2>/dev/null || true
aws iam delete-role --role-name SmartCampusCodeBuildRole 2>/dev/null || true

echo "✅ IAM resources deleted"

# === Step 15: Delete Secrets Manager ===
echo "[15/17] Deleting secrets..."
aws secretsmanager delete-secret --secret-id smart-campus/cognito --force-delete-without-recovery --region ${AWS_REGION} 2>/dev/null || true
aws secretsmanager delete-secret --secret-id smart-campus/ses --force-delete-without-recovery --region ${AWS_REGION} 2>/dev/null || true
aws secretsmanager delete-secret --secret-id smart-campus/jwt --force-delete-without-recovery --region ${AWS_REGION} 2>/dev/null || true
echo "✅ Secrets deleted"

# === Step 16: Delete SNS Topics ===
echo "[16/17] Deleting SNS topics..."
for TOPIC in smart-campus-alerts smart-campus-security-alerts smart-campus-critical-alerts; do
  TOPIC_ARN=$(aws sns list-topics --region ${AWS_REGION} --query "Topics[?contains(TopicArn, '${TOPIC}')].TopicArn" --output text 2>/dev/null || true)
  if [ -n "$TOPIC_ARN" ]; then
    aws sns delete-topic --topic-arn ${TOPIC_ARN} --region ${AWS_REGION} 2>/dev/null || true
  fi
done
echo "✅ SNS topics deleted"

# === Step 17: Delete Cognito User Pool ===
echo "[17/17] Deleting Cognito User Pool..."
USER_POOL_ID=$(aws cognito-idp list-user-pools --max-results 10 --region ${AWS_REGION} --query "UserPools[?Name=='smart-campus-users'].Id" --output text 2>/dev/null || true)
if [ -n "$USER_POOL_ID" ]; then
  aws cognito-idp delete-user-pool --user-pool-id ${USER_POOL_ID} --region ${AWS_REGION} 2>/dev/null || true
fi
echo "✅ Cognito User Pool deleted"

echo ""
echo "✅ ✅ ✅ CLEANUP COMPLETE! ✅ ✅ ✅"
echo ""
echo "Next steps:"
echo "1. Wait 5-10 minutes for all deletions to propagate"
echo "2. Go to AWS Console → Billing → Bills"
echo "3. Verify charges = \$0 for current month"
echo "4. Check Cost Explorer for any lingering charges"
echo "5. Verify no resources remain in each service console"
echo ""
```

**Save & Run the script:**
```bash
chmod +x cleanup-all.sh
./cleanup-all.sh
```

---

#### Manual Cleanup (Alternative - if script fails)

If the script fails, perform a manual cleanup following the dependency order:

**Step 1: WAF**
```bash
# List Web ACLs
aws wafv2 list-web-acls --scope REGIONAL --region ap-southeast-1

# Disassociate from API Gateway
aws wafv2 disassociate-web-acl --resource-arn arn:aws:apigateway:ap-southeast-1::/apis/${API_ID}/stages/prod --region ap-southeast-1

# Delete Web ACL (requires ID and LockToken)
aws wafv2 delete-web-acl --name smart-campus-waf --scope REGIONAL --id ${WEB_ACL_ID} --lock-token ${LOCK_TOKEN} --region ap-southeast-1

# Delete IP Set
aws wafv2 delete-ip-set --name SmartCampusIPSet --scope REGIONAL --id ${IP_SET_ID} --lock-token ${IP_SET_LOCK} --region ap-southeast-1
```

**Step 2: Lambda Functions**
```bash
# List functions
aws lambda list-functions --region ap-southeast-1 --query "Functions[?contains(FunctionName, 'smart-campus')].FunctionName"

# Delete event source mappings first
aws lambda list-event-source-mappings --function-name smart-campus-api --region ap-southeast-1
aws lambda delete-event-source-mapping --uuid ${UUID} --region ap-southeast-1

# Delete functions
aws lambda delete-function --function-name smart-campus-api --region ap-southeast-1
aws lambda delete-function --function-name smart-campus-analytics-worker --region ap-southeast-1
aws lambda delete-function --function-name smart-campus-notification-worker --region ap-southeast-1
aws lambda delete-function --function-name smart-campus-tasks-overdue-checker --region ap-southeast-1
```

**Step 3: API Gateway**
```bash
API_ID=$(aws apigatewayv2 get-apis --query "Items[?Name=='smart-campus-api'].ApiId" --output text)
aws apigatewayv2 delete-api --api-id ${API_ID}
```

**Step 4: SQS Queues**
```bash
aws sqs list-queues --queue-name-prefix smart-campus
# Purge and then delete each queue
aws sqs purge-queue --queue-url ${QUEUE_URL}
aws sqs delete-queue --queue-url ${QUEUE_URL}
```

**Step 5: EventBridge**
```bash
aws events list-rules --event-bus-name smart-campus-events
aws events remove-targets --rule ${RULE} --ids 1 --event-bus-name smart-campus-events
aws events delete-rule --name ${RULE} --event-bus-name smart-campus-events
aws events delete-event-bus --name smart-campus-events
```

**Step 6: Glue**
```bash
aws glue delete-crawler --name smart-campus-attendance-crawler
aws glue delete-crawler --name smart-campus-tasks-crawler
aws glue delete-crawler --name smart-campus-users-crawler
aws glue delete-table --database-name smart_campus_db --name attendance
aws glue delete-table --database-name smart_campus_db --name tasks
aws glue delete-table --database-name smart_campus_db --name users
aws glue delete-database --name smart_campus_db
```

**Step 7: Kinesis Firehose**
```bash
aws firehose delete-delivery-stream --delivery-stream-name smart-campus-attendance-stream
```

**Step 8: S3 Buckets**
```bash
aws s3 rm s3://smart-campus-images-${AWS_ACCOUNT_ID} --recursive
aws s3 rb s3://smart-campus-images-${AWS_ACCOUNT_ID}

aws s3 rm s3://smart-campus-datalake-${AWS_ACCOUNT_ID} --recursive
aws s3 rb s3://smart-campus-datalake-${AWS_ACCOUNT_ID}

aws s3 rm s3://smart-campus-frontend-${AWS_ACCOUNT_ID} --recursive
aws s3 rb s3://smart-campus-frontend-${AWS_ACCOUNT_ID}
```

**Step 9: Rekognition**
```bash
aws rekognition delete-collection --collection-id smart-campus-faces
```

**Step 10: DynamoDB Tables (8 tables)**
```bash
for TABLE in smart-campus-users smart-campus-faces smart-campus-attendance smart-campus-tasks smart-campus-leaves smart-campus-notifications smart-campus-security smart-campus-holidays; do
  aws dynamodb delete-table --table-name ${TABLE} --region ap-southeast-1
done
```

**Step 11: CloudWatch**
```bash
# Alarms
aws cloudwatch delete-alarms --alarm-names SmartCampus-HighErrorRate SmartCampus-LivenessFailureSpike SmartCampus-DLQMessages SmartCampus-API5xxErrors SmartCampus-LambdaThrottles

# Dashboard
aws cloudwatch delete-dashboards --dashboard-names SmartCampus-Production

# Log Groups
aws logs delete-log-group --log-group-name /aws/lambda/smart-campus-api
aws logs delete-log-group --log-group-name /aws/lambda/smart-campus-analytics-worker
aws logs delete-log-group --log-group-name /aws/lambda/smart-campus-notification-worker
aws logs delete-log-group --log-group-name /aws/lambda/smart-campus-tasks-overdue-checker
aws logs delete-log-group --log-group-name /aws/apigateway/smart-campus
aws logs delete-log-group --log-group-name /aws/events/smart-campus
aws logs delete-log-group --log-group-name /smart-campus/application
```

**Step 12: IAM**
```bash
# Lambda role
aws iam detach-role-policy --role-name SmartCampusLambdaRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam delete-role-policy --role-name SmartCampusLambdaRole --policy-name LeastPrivilegePolicy
aws iam delete-role --role-name SmartCampusLambdaRole

# Glue role
aws iam detach-role-policy --role-name SmartCampusGlueCrawlerRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole
aws iam delete-role-policy --role-name SmartCampusGlueCrawlerRole --policy-name S3DataLakeAccess
aws iam delete-role --role-name SmartCampusGlueCrawlerRole

# EventBridge role
aws iam delete-role-policy --role-name Amazon_EventBridge_Invoke_Glue_Crawler --policy-name GlueCrawlerAccess
aws iam delete-role --role-name Amazon_EventBridge_Invoke_Glue_Crawler

# CodeBuild role
aws iam detach-role-policy --role-name SmartCampusCodeBuildRole --policy-arn arn:aws:iam::aws:policy/AWSCodeBuildAdminAccess
aws iam delete-role --role-name SmartCampusCodeBuildRole
```

**Step 13: Secrets Manager**
```bash
aws secretsmanager delete-secret --secret-id smart-campus/cognito --force-delete-without-recovery
aws secretsmanager delete-secret --secret-id smart-campus/ses --force-delete-without-recovery
aws secretsmanager delete-secret --secret-id smart-campus/jwt --force-delete-without-recovery
```

**Step 14: SNS Topics**
```bash
aws sns list-topics --query "Topics[?contains(TopicArn, 'smart-campus')].TopicArn"
aws sns delete-topic --topic-arn ${TOPIC_ARN}
```

**Step 15: Cognito User Pool**
```bash
aws cognito-idp list-user-pools --max-results 10 --query "UserPools[?Name=='smart-campus-users'].Id"
aws cognito-idp delete-user-pool --user-pool-id ${USER_POOL_ID}
```

---

#### Verification Checklist (After cleanup)

| Resource Type | Verification Command | Expected Result |
|:---|:---|:---|
| **Lambda** | `aws lambda list-functions --query "Functions[?contains(FunctionName, 'smart-campus')]"` | Empty array `[]` |
| **API Gateway** | `aws apigatewayv2 get-apis --query "Items[?Name=='smart-campus-api']"` | Empty array `[]` |
| **DynamoDB** | `aws dynamodb list-tables --query "TableNames[?contains(@, 'smart-campus')]"` | Empty array `[]` |
| **S3** | `aws s3 ls | grep smart-campus` | No output |
| **EventBridge** | `aws events list-event-buses --query "EventBuses[?Name=='smart-campus-events']"` | Empty array `[]` |
| **SQS** | `aws sqs list-queues --queue-name-prefix smart-campus` | Empty `QueueUrls` |
| **Glue** | `aws glue get-databases --query "DatabaseList[?Name=='smart_campus_db']"` | Empty array `[]` |
| **Athena** | `aws athena list-work-groups --query "WorkGroups[?Name=='smart-campus-workgroup']"` | Empty array `[]` |
| **Rekognition** | `aws rekognition list-collections --query "CollectionIds[?@=='smart-campus-faces']"` | Empty array `[]` |
| **CloudWatch Logs** | `aws logs describe-log-groups --log-group-name-prefix /aws/lambda/smart-campus` | Empty `logGroups` |
| **CloudWatch Alarms** | `aws cloudwatch describe-alarms --alarm-name-prefix SmartCampus` | Empty `MetricAlarms` |
| **IAM Roles** | `aws iam list-roles --query "Roles[?contains(RoleName, 'SmartCampus')]"` | Empty array `[]` |
| **Secrets** | `aws secretsmanager list-secrets --query "SecretList[?contains(Name, 'smart-campus')]"` | Empty array `[]` |
| **SNS** | `aws sns list-topics --query "Topics[?contains(TopicArn, 'smart-campus')]"` | Empty array `[]` |
| **Cognito** | `aws cognito-idp list-user-pools --max-results 10 --query "UserPools[?Name=='smart-campus-users']"` | Empty array `[]` |
| **Firehose** | `aws firehose list-delivery-streams --query "DeliveryStreamNames[?@=='smart-campus-attendance-stream']"` | Empty array `[]` |
| **WAF** | `aws wafv2 list-web-acls --scope REGIONAL --query "WebACLs[?Name=='smart-campus-waf']"` | Empty array `[]` |

---

#### Billing Verification

```bash
# 1. Check current month charges
aws ce get-cost-and-usage \
  --time-period Start=$(date -d "$(date +%Y-%m-01)" +%Y-%m-%d),End=$(date +%Y-%m-%d) \
  --granularity MONTHLY \
  --metrics UnblendedCost \
  --group-by Type=DIMENSION,Key=SERVICE \
  --region us-east-1

# 2. Check for any unexpected charges
aws ce get-cost-and-usage \
  --time-period Start=$(date -d "$(date +%Y-%m-01)" +%Y-%m-%d),End=$(date +%Y-%m-%d) \
  --granularity DAILY \
  --metrics UnblendedCost \
  --region us-east-1
```

**Expected:** Total cost = $0.00 (or minimal < $0.01 for API calls completed prior to cleanup)

---

#### ⚠️ Common Cleanup Issues & Fixes

| Issue | Cause | Fix |
|:---|:---|:---|
| `ResourceInUseException` DynamoDB | Table still deleting | Wait 1-2 min, retry deletion |
| `DependencyViolation` IAM role | Policies still attached | Detach all policies first |
| `ConflictException` WAF | Lock token stale | Re-fetch the LockToken, then retry |
| `ResourceNotFoundException` | Resource already deleted | Ignore (idempotent operation) |
| S3 bucket not empty | Versioning is enabled | `aws s3 rm s3://bucket --recursive` includes version objects |
| Cognito delete fails | Deletion protection enabled | Disable deletion protection first |
| Firehose delete fails | Destination still processing | Wait 5 min, then retry |

---

#### Workshop Complete! 🎉

You have successfully completed all 12 steps of the Smart Campus Workshop:

1. ✅ **Prerequisites** - AWS Account, IAM, CLI, Python
2. ✅ **DynamoDB** - 8 tables with GSIs
3. ✅ **Rekognition** - Face Collection + Liveness
4. ✅ **Lambda** - 4 functions (API + 3 Workers)
5. ✅ **API Gateway** - HTTP API + CORS + JWT
6. ✅ **EventBridge + SQS** - Custom Bus + 2 Queues + DLQ
7. ✅ **Analytics Pipeline** - S3 Data Lake + Glue + Athena
8. ✅ **Testing** - 12 test cases covering WF1-WF8
9. ✅ **Monitoring** - CloudWatch + X-Ray + Alarms + Dashboard
10. ✅ **Security & Optimization** - IAM Least Privilege + WAF + Cost Optimization
11. ✅ **Cleanup** - Complete resource teardown

**Key Takeaways:**
- Serverless Event-Driven Architecture patterns
- AI/ML integration (Rekognition Face Recognition + Liveness)
- Hybrid OLTP/OLAP Analytics (DynamoDB + Athena)
- Production-grade Security (WAF, Cognito, IAM Least Privilege)
- Observability (Structured Logging, X-Ray, Custom Metrics, Alarms)
- Cost Optimization (S3 Lifecycle, Firehose Buffering, Athena Partition Pruning)
- CI/CD with CodePipeline + CodeBuild

**Next Steps for Production:**
- Add Infrastructure as Code (CloudFormation/CDK/Terraform)
- Implement Blue/Green Deployments
- Add Integration Tests (pytest + moto)
- Set up GitOps workflow
- Implement Chaos Engineering (Game Days)
- Add sophisticated ML features (Bedrock for AI Assistant)

We wish you success on your next AWS projects! ☁️🚀

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

**Verify that all resources are deleted:**

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
2. Click "Bills" in the left sidebar
3. Check "Total charges" for the current month
4. It should show **$0.00** (or remain strictly within Free Tier thresholds)

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

**Issue 1: S3 bucket cannot be deleted**
```
Error: BucketNotEmpty
```
**Solution:** Empty the S3 bucket first
```bash
aws s3 rm s3://bucket-name --recursive
aws s3 rb s3://bucket-name
```

**Issue 2: IAM role still has attached policies**
```
Error: DeleteConflict
```
**Solution:** Detach all policies before deleting the role
```bash
aws iam list-attached-role-policies --role-name RoleName
aws iam detach-role-policy --role-name RoleName --policy-arn arn:...
```

**Issue 3: DynamoDB table is locked or in use**
```
Error: ResourceInUseException
```
**Solution:** Disable streams and Point-In-Time Recovery (PITR) first
```bash
aws dynamodb update-continuous-backups \
  --table-name TableName \
  --point-in-time-recovery-specification PointInTimeRecoveryEnabled=false
```

**Issue 4: EventBridge rule still has active targets**
```
Error: Cannot delete rule with targets
```
**Solution:** Remove targets first before deleting the rule
```bash
aws events remove-targets --rule RuleName --ids 1 2 3
aws events delete-rule --name RuleName
```

---

#### Post-Cleanup Actions

**1. Delete the AWS Account** (optional, if created strictly for the workshop)
- Go to Account Settings
- Choose "Close Account"
- Confirm via confirmation email

**2. Export CloudWatch Logs** (if you need to keep logs for reporting)
```bash
aws logs create-export-task \
  --log-group-name /aws/lambda/smart-campus-api \
  --from $(date -d '7 days ago' +%s)000 \
  --to $(date +%s)000 \
  --destination s3://my-personal-backup-bucket/logs/
```

**3. Take screenshots** (for project or internship reports)
- Architecture diagram from AWS Console
- CloudWatch Dashboard
- DynamoDB tables structure
- API Gateway endpoints
- Lambda function config

---

#### Final Checklist

- [ ] Run the `cleanup-all.sh` script
- [ ] Verify all resources have been deleted (run the verification script)
- [ ] Check AWS Billing = $0
- [ ] Export logs/take screenshots for your documentation
- [ ] Remove AWS CLI credentials (optional)
- [ ] Close AWS account (optional)

---

#### Cost if You Forget to Cleanup

**Worst-case scenario (1 month):**
- Lambda: $0.44
- API Gateway: $0.10
- DynamoDB: $2.50
- Rekognition: $300 (if active traffic occurs)
- S3: $0.50
- CloudWatch: $5.00
- **Total: ~$308/month**

**Free Tier safety net:**
- First 12 months: Most foundational services are free
- After 12 months: Full standard billing rate applies

**⚠️ IMPORTANT: Set up a budget alert IMMEDIATELY:**
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

## Congratulations!

You have successfully completed the **Smart Campus Serverless Attendance System Workshop**!

**What you built:**
- ✅ Serverless attendance system with face recognition
- ✅ Event-driven architecture using EventBridge + SQS
- ✅ Data Lake analytics powered by Athena
- ✅ Production-ready observability and monitoring
- ✅ Cost-optimized, secure, and highly scalable setup

**Skills learned:**
- AWS Lambda, API Gateway, DynamoDB
- Amazon Rekognition (AI/ML)
- Event-driven architecture paradigms
- Infrastructure as Code (IaC) concepts
- Cost optimization guidelines
- Cloud security best practices

**Next steps:**
- Add advanced features: Mobile applications, Geofencing, and Multi-tenancy
- Deploy to production with a robust CI/CD pipeline
- Scale to millions of user requests
- Apply your hands-on experience to prepare for AWS certification exams!

---

**Thank you for completing this workshop!**