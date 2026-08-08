---
title: "Security and Optimization"
date: 2024-01-01
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

## Overview

This is a **CRITICAL** step to ensure the system:
- ✅ **Secure**: IAM least privilege, WAF, encryption
- ✅ **Performant**: Lambda tuning, caching
- ✅ **Cost-optimized**: Right-sizing, reserved capacity
- ✅ **Reliable**: Multi-AZ, backup, disaster recovery

## Part 1: Security Hardening

### 1.1. IAM Least Privilege Policy

**Current issue:** Lambda role has overly broad permissions (FullAccess policies)

**Solution:** Create custom policy with only required permissions

```bash
cat > lambda-least-privilege-policy.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DynamoDBAccess",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:Query",
        "dynamodb:Scan"
      ],
      "Resource": [
        "arn:aws:dynamodb:ap-southeast-1:*:table/smart-campus-users",
        "arn:aws:dynamodb:ap-southeast-1:*:table/smart-campus-faces",
        "arn:aws:dynamodb:ap-southeast-1:*:table/smart-campus-attendance",
        "arn:aws:dynamodb:ap-southeast-1:*:table/smart-campus-*/index/*"
      ]
    },
    {
      "Sid": "RekognitionAccess",
      "Effect": "Allow",
      "Action": [
        "rekognition:IndexFaces",
        "rekognition:SearchFacesByImage",
        "rekognition:CreateFaceLivenessSession",
        "rekognition:GetFaceLivenessSessionResults"
      ],
      "Resource": "*"
    },
    {
      "Sid": "S3Access",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::smart-campus-*/*"
    },
    {
      "Sid": "EventBridgePublish",
      "Effect": "Allow",
      "Action": "events:PutEvents",
      "Resource": "arn:aws:events:ap-southeast-1:*:event-bus/smart-campus-events"
    },
    {
      "Sid": "SQSAccess",
      "Effect": "Allow",
      "Action": [
        "sqs:ReceiveMessage",
        "sqs:DeleteMessage",
        "sqs:GetQueueAttributes"
      ],
      "Resource": "arn:aws:sqs:ap-southeast-1:*:smart-campus-*"
    },
    {
      "Sid": "CloudWatchLogs",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:ap-southeast-1:*:log-group:/aws/lambda/smart-campus-*"
    }
  ]
}
EOF

# Update Lambda role
aws iam put-role-policy \
  --role-name SmartCampusLambdaRole \
  --policy-name LeastPrivilegePolicy \
  --policy-document file://lambda-least-privilege-policy.json
```

### 1.2. Enable WAF (Web Application Firewall)

```bash
# Create IP set for rate limiting
aws wafv2 create-ip-set \
  --name smart-campus-blocked-ips \
  --scope REGIONAL \
  --ip-address-version IPV4 \
  --addresses \
  --region ap-southeast-1

# Create Web ACL
cat > waf-rules.json <<'EOF'
{
  "Name": "smart-campus-waf",
  "Scope": "REGIONAL",
  "DefaultAction": {
    "Allow": {}
  },
  "Rules": [
    {
      "Name": "RateLimitRule",
      "Priority": 1,
      "Statement": {
        "RateBasedStatement": {
          "Limit": 2000,
          "AggregateKeyType": "IP"
        }
      },
      "Action": {
        "Block": {}
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "RateLimitRule"
      }
    },
    {
      "Name": "AWSManagedRulesCommonRuleSet",
      "Priority": 2,
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesCommonRuleSet"
        }
      },
      "OverrideAction": {
        "None": {}
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "CommonRuleSet"
      }
    },
    {
      "Name": "AWSManagedRulesKnownBadInputsRuleSet",
      "Priority": 3,
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesKnownBadInputsRuleSet"
        }
      },
      "OverrideAction": {
        "None": {}
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "KnownBadInputs"
      }
    }
  ],
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "SmartCampusWAF"
  }
}
EOF

aws wafv2 create-web-acl \
  --cli-input-json file://waf-rules.json \
  --region ap-southeast-1

# Associate with API Gateway
WEB_ACL_ARN=$(aws wafv2 list-web-acls \
  --scope REGIONAL \
  --region ap-southeast-1 \
  --query "WebACLs[?Name=='smart-campus-waf'].ARN" \
  --output text)

aws wafv2 associate-web-acl \
  --web-acl-arn ${WEB_ACL_ARN} \
  --resource-arn arn:aws:apigateway:ap-southeast-1::/apis/${API_ID}/stages/prod \
  --region ap-southeast-1
```

**WAF rules explained:**
- **Rate limit:** Block IPs sending > 2000 requests/5 minutes
- **Common Rule Set:** Protect against OWASP Top 10
- **Known Bad Inputs:** Block malformed requests

### 1.3. Enable DynamoDB Point-in-Time Recovery

```bash
# Enable PITR for all tables
for TABLE in smart-campus-users smart-campus-faces smart-campus-attendance; do
  aws dynamodb update-continuous-backups \
    --table-name ${TABLE} \
    --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true \
    --region ap-southeast-1
done
```

### 1.4. Encrypt Data at Rest

**S3 buckets** (already done in step 8):
```bash
aws s3api put-bucket-encryption \
  --bucket smart-campus-images-${AWS_ACCOUNT_ID} \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'
```

**DynamoDB** (encrypted by default with AWS managed keys)

**Upgrade to Customer Managed Keys (CMK):**
```bash
# Create KMS key
KEY_ID=$(aws kms create-key \
  --description "Smart Campus encryption key" \
  --region ap-southeast-1 \
  --query 'KeyMetadata.KeyId' \
  --output text)

aws kms create-alias \
  --alias-name alias/smart-campus \
  --target-key-id ${KEY_ID} \
  --region ap-southeast-1

# Update DynamoDB table encryption (requires re-encryption)
aws dynamodb update-table \
  --table-name smart-campus-attendance \
  --sse-specification Enabled=true,SSEType=KMS,KMSMasterKeyId=${KEY_ID} \
  --region ap-southeast-1
```

### 1.5. Secrets Management

**Bad practice:** Hard-code API keys, passwords in code

**Good practice:** Use AWS Secrets Manager

```bash
# Store Cognito credentials
aws secretsmanager create-secret \
  --name smart-campus/cognito \
  --secret-string '{
    "user_pool_id": "ap-southeast-1_ABC123",
    "client_id": "xyz789",
    "client_secret": "secret123"
  }' \
  --region ap-southeast-1

# Store SES credentials
aws secretsmanager create-secret \
  --name smart-campus/ses \
  --secret-string '{
    "smtp_username": "AKIAIOSFODNN7EXAMPLE",
    "smtp_password": "password123"
  }' \
  --region ap-southeast-1
```

**Retrieve secrets in Lambda:**
```python
import boto3
import json

secrets_client = boto3.client('secretsmanager')

def get_secret(secret_name):
    response = secrets_client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

# Usage
cognito_config = get_secret('smart-campus/cognito')
USER_POOL_ID = cognito_config['user_pool_id']
```

---

## Part 2: Performance Optimization

### 2.1. Lambda Memory Tuning

**Test different memory sizes:**

```bash
# Test with 256MB, 512MB, 1024MB
for MEMORY in 256 512 1024; do
  echo "Testing with ${MEMORY}MB..."
  
  aws lambda update-function-configuration \
    --function-name smart-campus-api \
    --memory-size ${MEMORY} \
    --region ap-southeast-1
  
  sleep 10
  
  # Invoke 10 times and measure duration
  for i in {1..10}; do
    aws lambda invoke \
      --function-name smart-campus-api \
      --payload '{"httpMethod":"GET","path":"/health"}' \
      --region ap-southeast-1 \
      --log-type Tail \
      /dev/null \
      --query 'LogResult' \
      --output text | base64 -d | grep Duration
  done
done
```

**Expected results:**
- 256MB: ~800ms (cold start ~2s)
- 512MB: ~400ms (cold start ~1s) ← **Best balance**
- 1024MB: ~300ms (cold start ~800ms, but 2x cost)

**Recommendation:** Use **512MB** for balance of performance/cost

### 2.2. Provisioned Concurrency (Eliminate Cold Starts)

**For production with consistent traffic:**

```bash
# Enable Provisioned Concurrency (5 instances always warm)
aws lambda put-provisioned-concurrency-config \
  --function-name smart-campus-api \
  --qualifier prod \
  --provisioned-concurrent-executions 5 \
  --region ap-southeast-1
```

**Cost:** ~$10/month for 5 instances, but eliminates cold starts

### 2.3. API Gateway Caching

**Enable caching for read-only endpoints:**

```bash
aws apigatewayv2 update-stage \
  --api-id ${API_ID} \
  --stage-name prod \
  --route-settings '{
    "GET /api/users/me": {
      "DataTraceEnabled": false,
      "ThrottlingBurstLimit": 100,
      "ThrottlingRateLimit": 50,
      "CachingEnabled": true,
      "CacheTtlInSeconds": 300
    }
  }' \
  --region ap-southeast-1
```

**Cache TTL:** 5 minutes (300 seconds)

### 2.4. DynamoDB Auto Scaling

**Enable auto-scaling (for Provisioned mode):**

```bash
# Register scalable target
aws application-autoscaling register-scalable-target \
  --service-namespace dynamodb \
  --resource-id "table/smart-campus-attendance" \
  --scalable-dimension "dynamodb:table:ReadCapacityUnits" \
  --min-capacity 5 \
  --max-capacity 100 \
  --region ap-southeast-1

# Create scaling policy
aws application-autoscaling put-scaling-policy \
  --service-namespace dynamodb \
  --resource-id "table/smart-campus-attendance" \
  --scalable-dimension "dynamodb:table:ReadCapacityUnits" \
  --policy-name "ReadAutoScaling" \
  --policy-type "TargetTrackingScaling" \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "DynamoDBReadCapacityUtilization"
    }
  }' \
  --region ap-southeast-1
```

**But:** On-Demand billing is simpler for workshop!

### 2.5. S3 Transfer Acceleration

**Enable for presigned URL uploads:**

```bash
aws s3api put-bucket-accelerate-configuration \
  --bucket smart-campus-images-${AWS_ACCOUNT_ID} \
  --accelerate-configuration Status=Enabled \
  --region ap-southeast-1
```

**Update presigned URL code:**
```python
s3_client = boto3.client('s3', config=Config(s3={'use_accelerate_endpoint': True}))
```

**Speed improvement:** 50-500% faster uploads from afar

---

## Part 3: Cost Optimization

### 3.1. Right-Sizing Resources

**Current costs (monthly estimate):**
- Lambda: $0.44
- API Gateway: $0.10
- DynamoDB: $2.50 (On-Demand)
- Rekognition: $300 (10K searches/day)
- S3: $0.50
- CloudWatch: $5.00
- **Total: ~$308/month**

**Optimization strategies:**

### 3.1.1. Rekognition Cost Reduction

**Problem:** Rekognition very expensive ($0.001/search × 10K/day = $300/month)

**Solutions:**

1. **Cache results for 5 minutes:**
```python
import redis

redis_client = redis.Redis(host='elasticache-endpoint')

def recognize_face_cached(image_bytes):
    # Hash image for cache key
    image_hash = hashlib.sha256(image_bytes).hexdigest()
    
    # Check cache
    cached = redis_client.get(f"face:{image_hash}")
    if cached:
        return json.loads(cached)
    
    # Call Rekognition
    result = search_face_by_image(image_bytes)
    
    # Cache 5 minutes
    redis_client.setex(f"face:{image_hash}", 300, json.dumps(result))
    
    return result
```

**Savings:** ~50% (avoid duplicate recognitions within 5 minutes)

2. **Only use Liveness for first-time registration:**
```python
if user.face_registered:
    # Skip liveness, only check face match
    result = search_face_by_image(image)
else:
    # First time: full liveness check
    liveness_result = check_liveness(session_id)
    if liveness_result['is_live']:
        result = index_face(image)
```

**Savings:** Eliminate 99% liveness calls → $4,500 → $50/month

### 3.1.2. DynamoDB Cost Reduction

**Switch to Provisioned Capacity** if traffic predictable:

```bash
aws dynamodb update-table \
  --table-name smart-campus-attendance \
  --billing-mode PROVISIONED \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
  --region ap-southeast-1
```

**Cost comparison:**
- On-Demand: $2.50/month (100K operations)
- Provisioned: $0.65/month (5 RCU + 5 WCU)

**Savings:** 74%

### 3.1.3. S3 Lifecycle Policies

**Already done in step 8:**
- Transition to Glacier after 90 days → 80% cheaper
- Delete Athena results after 7 days

### 3.1.4. CloudWatch Log Retention

```bash
# Reduce retention from 30 days → 7 days
aws logs put-retention-policy \
  --log-group-name /aws/lambda/smart-campus-api \
  --retention-in-days 7 \
  --region ap-southeast-1
```

**Savings:** ~$3/month

**Optimized costs:**
- Lambda: $0.44
- API Gateway: $0.10
- DynamoDB: $0.65 (Provisioned)
- Rekognition: $50 (with cache + skip liveness)
- S3: $0.30 (with lifecycle)
- CloudWatch: $2.00 (reduced retention)
- **Total: ~$53.50/month** (85% reduction!)

---

## Part 4: Reliability & Disaster Recovery

### 4.1. Multi-AZ Deployment

**Good news:** AWS services already Multi-AZ by default:
- ✅ Lambda: Automatically deployed across AZs
- ✅ DynamoDB: Multi-AZ replication
- ✅ API Gateway: Multi-AZ
- ✅ S3: Cross-AZ replication

**No action needed!**

### 4.2. DynamoDB Backup Strategy

**Enable automated backups:**
```bash
# Point-in-time recovery (already done)
# Restore to any point in last 35 days

# Create on-demand backup
aws dynamodb create-backup \
  --table-name smart-campus-attendance \
  --backup-name smart-campus-attendance-$(date +%Y%m%d) \
  --region ap-southeast-1
```

**Schedule weekly backups:**
```bash
# Create Lambda function to trigger backups
# Invoke via EventBridge schedule: cron(0 0 ? * SUN *)
```

### 4.3. Cross-Region Replication

**For disaster recovery, replicate to another region:**

```bash
# Enable DynamoDB Global Tables
aws dynamodb update-table \
  --table-name smart-campus-attendance \
  --stream-specification StreamEnabled=true,StreamViewType=NEW_AND_OLD_IMAGES \
  --region ap-southeast-1

# Create replica in ap-northeast-1 (Tokyo)
aws dynamodb create-global-table \
  --global-table-name smart-campus-attendance \
  --replication-group RegionName=ap-southeast-1 RegionName=ap-northeast-1 \
  --region ap-southeast-1
```

**Cost:** ~2x storage cost, but full DR capability

---

## Part 5: Compliance & Audit

### 5.1. Enable CloudTrail

**Track all API calls for audit:**

```bash
aws cloudtrail create-trail \
  --name smart-campus-audit \
  --s3-bucket-name smart-campus-audit-logs-${AWS_ACCOUNT_ID} \
  --is-multi-region-trail \
  --enable-log-file-validation \
  --region ap-southeast-1

aws cloudtrail start-logging \
  --name smart-campus-audit \
  --region ap-southeast-1
```

### 5.2. Enable AWS Config

**Track resource configuration changes:**

```bash
aws configservice put-configuration-recorder \
  --configuration-recorder name=smart-campus-config,roleARN=arn:aws:iam::${AWS_ACCOUNT_ID}:role/aws-config-role \
  --recording-group allSupported=true,includeGlobalResourceTypes=true

aws configservice start-configuration-recorder \
  --configuration-recorder-name smart-campus-config \
  --region ap-southeast-1
```

### 5.3. Enable GuardDuty

**Threat detection:**

```bash
aws guardduty create-detector \
  --enable \
  --region ap-southeast-1
```

---

## Security Checklist

Verify system meets security standards:

- [ ] IAM least privilege policies applied
- [ ] WAF enabled with rate limiting
- [ ] DynamoDB PITR enabled
- [ ] S3 encryption at rest enabled
- [ ] Secrets Manager for sensitive data
- [ ] CloudTrail logging enabled
- [ ] VPC Flow Logs enabled (if using VPC)
- [ ] API Gateway throttling configured
- [ ] Lambda timeout < 30s (prevent runaway)
- [ ] No hard-coded credentials in code

## Performance Checklist

- [ ] Lambda memory optimized (512MB)
- [ ] Provisioned Concurrency (production only)
- [ ] API Gateway caching enabled
- [ ] DynamoDB indexes optimized
- [ ] S3 Transfer Acceleration enabled
- [ ] CloudFront CDN (if frontend exists)
- [ ] Redis caching for Rekognition results

## Cost Optimization Checklist

- [ ] DynamoDB right-sized (Provisioned if predictable)
- [ ] S3 Lifecycle policies configured
- [ ] CloudWatch log retention reduced
- [ ] Lambda timeout not too high
- [ ] Unused resources deleted
- [ ] AWS Cost Explorer reviewed monthly
- [ ] Budget alerts configured

## Next Step

Proceed to [Step 12: Cleanup Resources](../5.12-cleanup) to delete all resources after workshop!