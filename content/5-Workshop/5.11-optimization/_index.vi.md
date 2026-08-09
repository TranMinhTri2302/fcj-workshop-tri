---
title: "Security và Optimization"
date: 2024-01-01
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

#### Tổng quan

Đây là bước **QUAN TRỌNG** để đảm bảo hệ thống Smart Campus Platform đạt tiêu chuẩn production:
- ✅ **Secure**: IAM Least Privilege, WAF IP Whitelist, Encryption at rest/in transit
- ✅ **Performant**: Lambda tuning (memory, provisioned concurrency), Caching
- ✅ **Cost-optimized**: Right-sizing, S3 Lifecycle, Firehose buffering, Athena partition pruning
- ✅ **Reliable**: Multi-AZ, Backup/Restore, Disaster Recovery, DLQ monitoring

---

## Phần 1: Security Hardening (Cứng hóa bảo mật)

### 1.1. IAM Least Privilege Policy (Quyền tối thiểu)

**Vấn đề hiện tại:** Lambda role sử dụng FullAccess policies (quá rộng, rủi ro bảo mật)

**Giải pháp:** Tạo custom policy chỉ cấp permissions thực sự cần thiết cho từng Lambda function

**Policy cho Main API Lambda (`smart-campus-api`):**

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
        "dynamodb:Scan",
        "dynamodb:BatchWriteItem",
        "dynamodb:BatchGetItem"
      ],
      "Resource": [
        "arn:aws:dynamodb:ap-southeast-1:${AWS_ACCOUNT_ID}:table/smart-campus-users",
        "arn:aws:dynamodb:ap-southeast-1:${AWS_ACCOUNT_ID}:table/smart-campus-faces",
        "arn:aws:dynamodb:ap-southeast-1:${AWS_ACCOUNT_ID}:table/smart-campus-attendance",
        "arn:aws:dynamodb:ap-southeast-1:${AWS_ACCOUNT_ID}:table/smart-campus-tasks",
        "arn:aws:dynamodb:ap-southeast-1:${AWS_ACCOUNT_ID}:table/smart-campus-leaves",
        "arn:aws:dynamodb:ap-southeast-1:${AWS_ACCOUNT_ID}:table/smart-campus-notifications",
        "arn:aws:dynamodb:ap-southeast-1:${AWS_ACCOUNT_ID}:table/smart-campus-security",
        "arn:aws:dynamodb:ap-southeast-1:${AWS_ACCOUNT_ID}:table/smart-campus-holidays",
        "arn:aws:dynamodb:ap-southeast-1:${AWS_ACCOUNT_ID}:table/smart-campus-*/index/*"
      ]
    },
    {
      "Sid": "RekognitionAccess",
      "Effect": "Allow",
      "Action": [
        "rekognition:IndexFaces",
        "rekognition:SearchFacesByImage",
        "rekognition:CreateFaceLivenessSession",
        "rekognition:GetFaceLivenessSessionResults",
        "rekognition:ListFaces",
        "rekognition:DeleteFaces"
      ],
      "Resource": "arn:aws:rekognition:ap-southeast-1:${AWS_ACCOUNT_ID}:collection/smart-campus-faces"
    },
    {
      "Sid": "S3Access",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:GetObjectVersion"
      ],
      "Resource": [
        "arn:aws:s3:::smart-campus-images-${AWS_ACCOUNT_ID}/*",
        "arn:aws:s3:::smart-campus-datalake-${AWS_ACCOUNT_ID}/*"
      ]
    },
    {
      "Sid": "EventBridgePublish",
      "Effect": "Allow",
      "Action": "events:PutEvents",
      "Resource": "arn:aws:events:ap-southeast-1:${AWS_ACCOUNT_ID}:event-bus/smart-campus-events"
    },
    {
      "Sid": "SQSAccess",
      "Effect": "Allow",
      "Action": [
        "sqs:SendMessage",
        "sqs:SendMessageBatch"
      ],
      "Resource": [
        "arn:aws:sqs:ap-southeast-1:${AWS_ACCOUNT_ID}:smart-campus-analytics-queue",
        "arn:aws:sqs:ap-southeast-1:${AWS_ACCOUNT_ID}:smart-campus-notification-queue"
      ]
    },
    {
      "Sid": "SNSAccess",
      "Effect": "Allow",
      "Action": [
        "sns:Publish"
      ],
      "Resource": "arn:aws:sns:ap-southeast-1:${AWS_ACCOUNT_ID}:smart-campus-*"
    },
    {
      "Sid": "CognitoAccess",
      "Effect": "Allow",
      "Action": [
        "cognito-idp:AdminCreateUser",
        "cognito-idp:AdminSetUserPassword",
        "cognito-idp:AdminUpdateUserAttributes",
        "cognito-idp:AdminDeleteUser",
        "cognito-idp:AdminGetUser",
        "cognito-idp:ListUsers",
        "cognito-idp:AdminInitiateAuth",
        "cognito-idp:AdminRespondToAuthChallenge"
      ],
      "Resource": "arn:aws:cognito-idp:ap-southeast-1:${AWS_ACCOUNT_ID}:userpool/*"
    },
    {
      "Sid": "WAFAccess",
      "Effect": "Allow",
      "Action": [
        "wafv2:GetIPSet",
        "wafv2:UpdateIPSet"
      ],
      "Resource": "arn:aws:wafv2:ap-southeast-1:${AWS_ACCOUNT_ID}:regional/ipset/SmartCampusIPSet/*"
    },
    {
      "Sid": "CloudWatchLogs",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:ap-southeast-1:${AWS_ACCOUNT_ID}:log-group:/aws/lambda/smart-campus-*"
    },
    {
      "Sid": "XRayAccess",
      "Effect": "Allow",
      "Action": [
        "xray:PutTraceSegments",
        "xray:PutTelemetryRecords",
        "xray:GetSamplingRules",
        "xray:GetSamplingTargets"
      ],
      "Resource": "*"
    },
    {
      "Sid": "AthenaAccess",
      "Effect": "Allow",
      "Action": [
        "athena:StartQueryExecution",
        "athena:GetQueryExecution",
        "athena:GetQueryResults",
        "athena:GetWorkGroup"
      ],
      "Resource": "arn:aws:athena:ap-southeast-1:${AWS_ACCOUNT_ID}:workgroup/smart-campus-workgroup"
    },
    {
      "Sid": "GlueAccess",
      "Effect": "Allow",
      "Action": [
        "glue:GetTable",
        "glue:GetTables",
        "glue:GetDatabase",
        "glue:GetDatabases",
        "glue:GetPartition",
        "glue:GetPartitions"
      ],
      "Resource": [
        "arn:aws:glue:ap-southeast-1:${AWS_ACCOUNT_ID}:catalog",
        "arn:aws:glue:ap-southeast-1:${AWS_ACCOUNT_ID}:database/smart_campus_db",
        "arn:aws:glue:ap-southeast-1:${AWS_ACCOUNT_ID}:table/smart_campus_db/*"
      ]
    },
    {
      "Sid": "KMSAccess",
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "arn:aws:kms:ap-southeast-1:${AWS_ACCOUNT_ID}:key/*",
      "Condition": {
        "StringEquals": {
          "kms:ViaService": [
            "dynamodb.ap-southeast-1.amazonaws.com",
            "s3.ap-southeast-1.amazonaws.com"
          ]
        }
      }
    }
  ]
}
EOF

# Apply policy to Lambda role
aws iam put-role-policy \
  --role-name SmartCampusLambdaRole \
  --policy-name LeastPrivilegePolicy \
  --policy-document file://lambda-least-privilege-policy.json
```

**Policy cho Analytics Worker Lambda (`smart-campus-analytics-worker`):**

```bash
cat > analytics-worker-policy.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3DataLakeWrite",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:PutObjectTagging"
      ],
      "Resource": "arn:aws:s3:::smart-campus-datalake-${AWS_ACCOUNT_ID}/*"
    },
    {
      "Sid": "FirehoseAccess",
      "Effect": "Allow",
      "Action": [
        "firehose:PutRecord",
        "firehose:PutRecordBatch"
      ],
      "Resource": "arn:aws:firehose:ap-southeast-1:${AWS_ACCOUNT_ID}:deliverystream/smart-campus-attendance-stream"
    },
    {
      "Sid": "SQSReceive",
      "Effect": "Allow",
      "Action": [
        "sqs:ReceiveMessage",
        "sqs:DeleteMessage",
        "sqs:GetQueueAttributes"
      ],
      "Resource": "arn:aws:sqs:ap-southeast-1:${AWS_ACCOUNT_ID}:smart-campus-analytics-queue"
    },
    {
      "Sid": "CloudWatchLogs",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:ap-southeast-1:${AWS_ACCOUNT_ID}:log-group:/aws/lambda/smart-campus-analytics-worker*"
    }
  ]
}
EOF
```

**Policy cho Notification Worker Lambda (`smart-campus-notification-worker`):**

```bash
cat > notification-worker-policy.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "SNSAccess",
      "Effect": "Allow",
      "Action": [
        "sns:Publish"
      ],
      "Resource": "arn:aws:sns:ap-southeast-1:${AWS_ACCOUNT_ID}:smart-campus-*"
    },
    {
      "Sid": "SESAccess",
      "Effect": "Allow",
      "Action": [
        "ses:SendEmail",
        "ses:SendRawEmail"
      ],
      "Resource": "*"
    },
    {
      "Sid": "SQSReceive",
      "Effect": "Allow",
      "Action": [
        "sqs:ReceiveMessage",
        "sqs:DeleteMessage",
        "sqs:GetQueueAttributes"
      ],
      "Resource": "arn:aws:sqs:ap-southeast-1:${AWS_ACCOUNT_ID}:smart-campus-notification-queue"
    },
    {
      "Sid": "CloudWatchLogs",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:ap-southeast-1:${AWS_ACCOUNT_ID}:log-group:/aws/lambda/smart-campus-notification-worker*"
    }
  ]
}
EOF
```

### 1.2. WAF (Web Application Firewall) - IP Whitelist cho Attendance

**Mục đích:** Chỉ cho phép check-in/check-out từ mạng công ty (IP whitelist)

```bash
# 1. Tạo IP Set cho mạng công ty
aws wafv2 create-ip-set \
  --name SmartCampusIPSet \
  --scope REGIONAL \
  --ip-address-version IPV4 \
  --addresses "203.0.113.0/24" "198.51.100.0/24" \
  --description "Company office IP ranges for attendance check-in" \
  --region ap-southeast-1

# 2. Tạo Web ACL với rule IP whitelist
cat > waf-webacl.json <<'EOF'
{
  "Name": "smart-campus-waf",
  "Scope": "REGIONAL",
  "DefaultAction": {
    "Block": {}
  },
  "Description": "WAF for Smart Campus - IP whitelist for attendance",
  "Rules": [
    {
      "Name": "IPWhitelistRule",
      "Priority": 1,
      "Action": {
        "Allow": {}
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "IPWhitelistRule"
      },
      "Statement": {
        "IPSetReferenceStatement": {
          "ARN": "arn:aws:wafv2:ap-southeast-1:${AWS_ACCOUNT_ID}:regional/ipset/SmartCampusIPSet/xxx"
        }
      }
    },
    {
      "Name": "AWSManagedRulesCommonRuleSet",
      "Priority": 2,
      "OverrideAction": {
        "Count": {}
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "CommonRuleSet"
      },
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesCommonRuleSet",
          "Version": "1.0"
        }
      }
    },
    {
      "Name": "RateLimitRule",
      "Priority": 3,
      "Action": {
        "Block": {}
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "RateLimitRule"
      },
      "Statement": {
        "RateBasedStatement": {
          "Limit": 2000,
          "AggregateKeyType": "IP",
          "ScopeDownStatement": {
            "NotStatement": {
              "Statement": {
                "IPSetReferenceStatement": {
                  "ARN": "arn:aws:wafv2:ap-southeast-1:${AWS_ACCOUNT_ID}:regional/ipset/SmartCampusIPSet/xxx"
                }
              }
            }
          }
        }
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

# Create Web ACL
aws wafv2 create-web-acl \
  --cli-input-json file://waf-webacl.json \
  --region ap-southeast-1

# 3. Associate WAF với API Gateway Stage
API_ID=$(aws apigatewayv2 get-apis --region ap-southeast-1 --query "Items[?Name=='smart-campus-api'].ApiId" --output text)
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:ap-southeast-1:${AWS_ACCOUNT_ID}:regional/webacl/smart-campus-waf/xxx \
  --resource-arn arn:aws:apigateway:ap-southeast-1::/apis/${API_ID}/stages/prod \
  --region ap-southeast-1
```

**Cập nhật IP Whitelist từ Admin UI (Backend integration):**
```python
# app/modules/security/service.py
def update_waf_ip_set(new_ips: list[str]):
    """Admin gọi API này để cập nhật IP whitelist"""
    ip_set_arn = "arn:aws:wafv2:ap-southeast-1:xxx:regional/ipset/SmartCampusIPSet/xxx"
    
    # Get current lock token
    response = wafv2.get_ip_set(Name="SmartCampusIPSet", Scope="REGIONAL", Id="xxx")
    lock_token = response['LockToken']
    
    # Update IP set
    wafv2.update_ip_set(
        Name="SmartCampusIPSet",
        Scope="REGIONAL",
        Id="xxx",
        Addresses=new_ips,
        LockToken=lock_token
    )
```

### 1.3. Encryption (Mã hóa)

**S3 Buckets - SSE-S3 (AES256):**
```bash
# Images bucket
aws s3api put-bucket-encryption \
  --bucket smart-campus-images-${AWS_ACCOUNT_ID} \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }' \
  --region ap-southeast-1

# Data Lake bucket
aws s3api put-bucket-encryption \
  --bucket smart-campus-datalake-${AWS_ACCOUNT_ID} \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }' \
  --region ap-southeast-1
```

**DynamoDB - Encryption at rest (AWS managed key):**
```bash
# Enable khi tạo table (hoặc update existing)
aws dynamodb update-table \
  --table-name smart-campus-users \
  --sse-specification Enabled=true,SSEType=KMS \
  --region ap-southeast-1
```

**Athena Query Results - SSE-S3:**
```bash
aws athena update-work-group \
  --work-group smart-campus-workgroup \
  --configuration "ResultConfigurationUpdates={EncryptionConfiguration={EncryptionOption=SSE_S3}}" \
  --region ap-southeast-1
```

### 1.4. Cognito Security Enhancements

```bash
# 1. Enable MFA (Optional but recommended)
aws cognito-idp set-user-pool-mfa-config \
  --user-pool-id ap-southeast-1_xxxxxxxxx \
  --mfa-configuration ON \
  --region ap-southeast-1

# 2. Password Policy
aws cognito-idp update-user-pool \
  --user-pool-id ap-southeast-1_xxxxxxxxx \
  --policies '{
    "PasswordPolicy": {
      "MinimumLength": 12,
      "RequireUppercase": true,
      "RequireLowercase": true,
      "RequireNumbers": true,
      "RequireSymbols": true,
      "TemporaryPasswordValidityDays": 7
    }
  }' \
  --region ap-southeast-1

# 3. Advanced Security (Risk-based auth)
aws cognito-idp set-user-pool-mfa-config \
  --user-pool-id ap-southeast-1_xxxxxxxxx \
  --advanced-security-mode ENFORCED \
  --region ap-southeast-1
```

---

## Phần 2: Performance Optimization (Tối ưu hiệu năng)

### 2.1. Lambda Tuning

**Memory & CPU Optimization:**
```bash
# Test different memory sizes để tìm optimal (cost vs duration)
# 128MB, 256MB, 512MB, 1024MB, 2048MB, 3008MB

# Update function memory
aws lambda update-function-configuration \
  --function-name smart-campus-api \
  --memory-size 1024 \
  --region ap-southeast-1

# Enable Provisioned Concurrency cho cold start elimination (production)
aws lambda put-provisioned-concurrency-config \
  --function-name smart-campus-api \
  --qualifier prod \
  --provisioned-concurrent-executions 10 \
  --region ap-southeast-1
```

**Lambda Power Tuning (Automated):**
```bash
# Deploy AWS Lambda Power Tuning state machine
# https://github.com/aws-samples/aws-lambda-power-tuning
# Chạy để tìm memory size tối ưu cho cost/duration
```

### 2.2. API Gateway Caching

```bash
# Enable caching cho GET endpoints (users list, reports)
aws apigatewayv2 update-stage \
  --api-id ${API_ID} \
  --stage-name prod \
  --cache-cluster-enabled \
  --cache-cluster-size 0.5 \
  --region ap-southeast-1

# Cache TTL: 300 seconds (5 minutes)
# Cache key: query string parameters + headers (Authorization excluded)
```

### 2.3. DynamoDB Optimization

**On-Demand vs Provisioned:**
```bash
# For predictable workloads, switch to Provisioned với Auto Scaling
aws dynamodb update-table \
  --table-name smart-campus-attendance \
  --billing-mode PROVISIONED \
  --provisioned-throughput ReadCapacityUnits=100,WriteCapacityUnits=50 \
  --region ap-southeast-1

# Enable Auto Scaling
aws application-autoscaling register-scalable-target \
  --service-namespace dynamodb \
  --resource-id table/smart-campus-attendance \
  --scalable-dimension dynamodb:table:ReadCapacityUnits \
  --min-capacity 10 \
  --max-capacity 1000 \
  --region ap-southeast-1
```

**GSI Optimization:**
- Chỉ project attributes cần thiết (KEYS_ONLY hoặc INCLUDE specific attributes)
- Avoid projecting large attributes (images, long descriptions)

### 2.4. Frontend Optimization (React + Vite)

```javascript
// vite.config.js - Production build optimization
export default defineConfig({
  build: {
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true
      }
    },
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom', 'react-router-dom'],
          charts: ['recharts'],
          icons: ['lucide-react'],
          aws: ['@aws-sdk/client-cognito-identity-provider', '@aws-sdk/client-s3']
        }
      }
    },
    chunkSizeWarningLimit: 1000
  },
  // Enable gzip/brotli compression
  plugins: [
    react(),
    compressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 1024
    })
  ]
})
```

---

## Phần 3: Cost Optimization (Tối ưu chi phí)

### 3.1. S3 Lifecycle Policies

```bash
cat > lifecycle-policy.json <<'EOF'
{
  "Rules": [
    {
      "Id": "TransitionImagesToIA",
      "Status": "Enabled",
      "Filter": { "Prefix": "faces/raw/" },
      "Transitions": [
        { "Days": 30, "StorageClass": "STANDARD_IA" },
        { "Days": 90, "StorageClass": "GLACIER" },
        { "Days": 365, "StorageClass": "DEEP_ARCHIVE" }
      ]
    },
    {
      "Id": "TransitionDataLakeToIA",
      "Status": "Enabled",
      "Filter": { "Prefix": "attendance/" },
      "Transitions": [
        { "Days": 90, "StorageClass": "STANDARD_IA" },
        { "Days": 180, "StorageClass": "GLACIER" }
      ]
    },
    {
      "Id": "TransitionTasksToIA",
      "Status": "Enabled",
      "Filter": { "Prefix": "tasks/" },
      "Transitions": [
        { "Days": 90, "StorageClass": "STANDARD_IA" },
        { "Days": 180, "StorageClass": "GLACIER" }
      ]
    },
    {
      "Id": "DeleteAthenaResults",
      "Status": "Enabled",
      "Filter": { "Prefix": "athena-results/" },
      "Expiration": { "Days": 7 }
    },
    {
      "Id": "DeleteMultipartUploads",
      "Status": "Enabled",
      "AbortIncompleteMultipartUpload": { "DaysAfterInitiation": 7 }
    }
  ]
}
EOF

aws s3api put-bucket-lifecycle-configuration \
  --bucket smart-campus-images-${AWS_ACCOUNT_ID} \
  --lifecycle-configuration file://lifecycle-policy.json \
  --region ap-southeast-1

aws s3api put-bucket-lifecycle-configuration \
  --bucket smart-campus-datalake-${AWS_ACCOUNT_ID} \
  --lifecycle-configuration file://lifecycle-policy.json \
  --region ap-southeast-1
```

### 3.2. Kinesis Firehose Buffering (Cost vs Latency Trade-off)

```bash
# Update delivery stream buffering hints
aws firehose update-destination \
  --delivery-stream-name smart-campus-attendance-stream \
  --current-delivery-stream-version-id 1 \
  --destination-id destinationId \
  --s3-destination-update '{
    "BucketARN": "arn:aws:s3:::smart-campus-datalake-xxx",
    "BufferingHints": {
      "SizeInMBs": 5,
      "IntervalInSeconds": 60
    },
    "CompressionFormat": "GZIP",
    "Prefix": "attendance/year=!{timestamp:yyyy}/month=!{timestamp:MM}/day=!{timestamp:dd}/",
    "ErrorOutputPrefix": "firehose-errors/"
  }' \
  --region ap-southeast-1
```

**Buffering Guidelines:**
| Workload | Size (MB) | Interval (s) | Latency | Cost |
|:---|:---|:---|:---|:---|
| Real-time | 1 | 60 | Low | Higher |
| **Balanced (Recommended)** | **5** | **60** | **Medium** | **Optimal** |
| Batch/Analytics | 128 | 300 | High | Lowest |

### 3.3. Athena Cost Optimization

**Partition Pruning (QUAN TRỌNG):**
```sql
-- LUÔN filter partition keys trong WHERE clause
-- GOOD: Partition pruning works
SELECT * FROM attendance 
WHERE year = '2026' AND month = '08' AND day = '06'

-- BAD: Full table scan - EXPENSIVE!
SELECT * FROM attendance 
WHERE user_id = 'user-001'

-- GOOD: Partition + filter
SELECT * FROM attendance 
WHERE year = '2026' AND month = '08' AND user_id = 'user-001'
```

**Workgroup Configuration:**
```bash
aws athena update-work-group \
  --work-group smart-campus-workgroup \
  --configuration '{
    "ResultConfigurationUpdates": {
      "OutputLocation": "s3://smart-campus-datalake-xxx/athena-results/",
      "EncryptionConfiguration": { "EncryptionOption": "SSE_S3" }
    },
    "EngineVersion": { "SelectedEngineVersion": "Athena engine version 3" },
    "WorkGroupConfigurationUpdates": {
      "BytesScannedCutoffPerQuery": 1000000000,  -- 1GB limit per query
      "EnforceWorkGroupConfiguration": true,
      "PublishCloudWatchMetricsEnabled": true,
      "RequesterPaysEnabled": false
    }
  }' \
  --region ap-southeast-1
```

### 3.4. CloudWatch Logs Retention

```bash
# Set appropriate retention per log group
aws logs put-retention-policy --log-group-name /aws/lambda/smart-campus-api --retention-in-days 30 --region ap-southeast-1
aws logs put-retention-policy --log-group-name /aws/lambda/smart-campus-analytics-worker --retention-in-days 14 --region ap-southeast-1
aws logs put-retention-policy --log-group-name /aws/lambda/smart-campus-notification-worker --retention-in-days 14 --region ap-southeast-1
aws logs put-retention-policy --log-group-name /aws/apigateway/smart-campus --retention-in-days 30 --region ap-southeast-1
aws logs put-retention-policy --log-group-name /aws/events/smart-campus --retention-in-days 30 --region ap-southeast-1
aws logs put-retention-policy --log-group-name /smart-campus/application --retention-in-days 90 --region ap-southeast-1
```

### 3.5. Estimated Monthly Cost Breakdown (ap-southeast-1)

| Service | Configuration | Est. Cost/Month |
|:---|:---|:---|
| **Lambda** | 1M invocations, 1GB-sec, 100ms avg | ~$3.50 |
| **API Gateway** | 1M requests, HTTP API | ~$1.00 |
| **DynamoDB** | On-demand, 10GB storage, 1M reads/writes | ~$15.00 |
| **S3** | 50GB Standard + 100GB IA + 10GB Glacier | ~$3.50 |
| **Rekognition** | 10K face searches + 1K liveness | ~$12.00 |
| **EventBridge** | 100K events | ~$1.00 |
| **SQS** | 100K messages | ~$0.40 |
| **SNS** | 10K notifications | ~$0.50 |
| **Athena** | 100GB scanned/month | ~$5.00 |
| **Glue** | 3 crawlers, 10 DPU-hours | ~$2.00 |
| **CloudWatch** | Logs 10GB, 10 alarms, 1 dashboard | ~$5.00 |
| **X-Ray** | 100K traces | ~$5.00 |
| **CloudFront** | 1TB transfer, 10M requests | ~$10.00 |
| **WAF** | 1 Web ACL, 1M requests | ~$5.00 |
| **Cognito** | 1000 MAU | ~$0 (Free tier) |
| **CodeBuild** | 100 build minutes | ~$1.50 |
| **Kinesis Firehose** | 10GB ingested | ~$0.30 |
| **TOTAL** | **Production workload** | **~$70-80/month** |

---

## Phần 4: Reliability & Disaster Recovery

### 4.1. Multi-AZ (Automatic cho Serverless)
- Lambda: Automatic Multi-AZ
- DynamoDB: Automatic Multi-AZ replication
- S3: Automatic Multi-AZ (99.999999999% durability)
- SQS/SNS: Automatic Multi-AZ

### 4.2. Backup Strategy

**DynamoDB Point-in-Time Recovery (PITR):**
```bash
aws dynamodb update-continuous-backups \
  --table-name smart-campus-users \
  --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true \
  --region ap-southeast-1

# Repeat for all 8 tables
```

**S3 Cross-Region Replication (CRR):**
```bash
# Replicate critical buckets to backup region (ap-northeast-1)
# Requires versioning enabled on both source and destination
```

### 4.3. DLQ Monitoring & Recovery

```bash
# CloudWatch Alarm cho DLQ (already created in 5.10)
# Recovery procedure:
# 1. Check DLQ messages
aws sqs receive-message --queue-url https://sqs.ap-southeast-1.amazonaws.com/xxx/smart-campus-analytics-dlq --max-number-of-messages 10

# 2. Analyze failure reason (check message attributes)
# 3. Fix root cause (code bug, permission, throttling)
# 4. Re-drive messages
aws sqs start-message-move-task \
  --source-arn arn:aws:sqs:ap-southeast-1:xxx:smart-campus-analytics-dlq \
  --destination-arn arn:aws:sqs:ap-southeast-1:xxx:smart-campus-analytics-queue \
  --region ap-southeast-1
```

### 4.4. Chaos Engineering (Game Days)

```bash
# Simulate failures để test resilience:
# 1. Lambda throttle: aws lambda put-function-concurrency --function-name smart-campus-api --reserved-concurrent-executions 1
# 2. DynamoDB throttle: Reduce RCU/WCU temporarily
# 3. Network failure: VPC endpoint deletion
# 4. Dependency failure: Mock Rekognition 500 errors

# Verify: System degrades gracefully, DLQ catches messages, alarms fire, auto-recovery works
```

---

## Checklist Security & Optimization Summary

| Category | Item | Status |
|:---|:---|:---|
| **IAM** | Least privilege policies per Lambda | ☐ |
| **IAM** | No wildcard (*) resources where avoidable | ☐ |
| **WAF** | IP Whitelist for attendance endpoints | ☐ |
| **WAF** | Rate limiting + AWS Managed Rules | ☐ |
| **Encryption** | S3 SSE-S3 (AES256) | ☐ |
| **Encryption** | DynamoDB Encryption at rest | ☐ |
| **Encryption** | Athena results encrypted | ☐ |
| **Cognito** | MFA enabled | ☐ |
| **Cognito** | Strong password policy | ☐ |
| **Cognito** | Advanced security (risk-based) | ☐ |
| **Lambda** | Memory optimized (power tuning) | ☐ |
| **Lambda** | Provisioned concurrency (prod) | ☐ |
| **API GW** | Caching enabled for GET | ☐ |
| **DynamoDB** | Auto Scaling configured | ☐ |
| **DynamoDB** | PITR enabled | ☐ |
| **S3** | Lifecycle policies (IA/Glacier) | ☐ |
| **S3** | CRR for critical data | ☐ |
| **Firehose** | Buffering optimized (5MB/60s) | ☐ |
| **Athena** | Partition pruning enforced | ☐ |
| **Athena** | Workgroup query limits | ☐ |
| **CloudWatch** | Log retention configured | ☐ |
| **CloudWatch** | Alarms for DLQ, Errors, Throttles | ☐ |
| **X-Ray** | Tracing enabled + service map | ☐ |
| **Cost** | Monthly budget alarm ($100) | ☐ |
| **DR** | DLQ re-drive procedure documented | ☐ |
| **DR** | Game day exercises scheduled | ☐ |

---

## Next Step

Tiến hành [Bước 12: Cleanup Resources](../5.12-cleanup) để dọn dẹp tài nguyên sau workshop!
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

# Associate với API Gateway
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

**1.3. Enable DynamoDB Point-in-Time Recovery**

```bash
# Enable PITR cho tất cả tables
for TABLE in smart-campus-users smart-campus-faces smart-campus-attendance; do
  aws dynamodb update-continuous-backups \
    --table-name ${TABLE} \
    --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true \
    --region ap-southeast-1
done
```

**1.4. Encrypt Data at Rest**

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

**DynamoDB** (encrypted by default với AWS managed keys)

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

**1.5. Secrets Management**

**Bad practice:** Hard-code API keys, passwords trong code

**Good practice:** Dùng AWS Secrets Manager

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

**Retrieve secrets trong Lambda:**
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

#### Phần 2: Performance Optimization

**2.1. Lambda Memory Tuning**

**Test different memory sizes:**

```bash
# Test với 256MB, 512MB, 1024MB
for MEMORY in 256 512 1024; do
  echo "Testing with ${MEMORY}MB..."
  
  aws lambda update-function-configuration \
    --function-name smart-campus-api \
    --memory-size ${MEMORY} \
    --region ap-southeast-1
  
  sleep 10
  
  # Invoke 10 times và measure duration
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

**2.2. Provisioned Concurrency (Eliminate Cold Starts)**

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

**2.3. API Gateway Caching**

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

**2.4. DynamoDB Auto Scaling**

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

**But:** On-Demand billing đơn giản hơn cho workshop!

**2.5. S3 Transfer Acceleration**

**Enable cho presigned URL uploads:**

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

**Speed improvement:** 50-500% faster uploads từ xa

---

#### Phần 3: Cost Optimization

**3.1. Right-Sizing Resources**

**Current costs (monthly estimate):**
- Lambda: $0.44
- API Gateway: $0.10
- DynamoDB: $2.50 (On-Demand)
- Rekognition: $300 (10K searches/day)
- S3: $0.50
- CloudWatch: $5.00
- **Total: ~$308/month**

**Optimization strategies:**

**3.1.1. Rekognition Cost Reduction**

**Problem:** Rekognition rất đắt ($0.001/search × 10K/day = $300/month)

**Solutions:**
1. **Cache results 5 phút:**
```python
import redis

redis_client = redis.Redis(host='elasticache-endpoint')

def recognize_face_cached(image_bytes):
    # Hash image để làm cache key
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

**Savings:** ~50% (avoid duplicate recognitions trong 5 phút)

2. **Chỉ dùng Liveness cho first-time registration:**
```python
if user.face_registered:
    # Skip liveness, chỉ check face match
    result = search_face_by_image(image)
else:
    # First time: full liveness check
    liveness_result = check_liveness(session_id)
    if liveness_result['is_live']:
        result = index_face(image)
```

**Savings:** Eliminate 99% liveness calls → $4,500 → $50/month

**3.1.2. DynamoDB Cost Reduction**

**Switch to Provisioned Capacity** nếu traffic predictable:

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

**3.1.3. S3 Lifecycle Policies**

**Already done in step 8:**
- Transition to Glacier after 90 days → 80% cheaper
- Delete Athena results after 7 days

**3.1.4. CloudWatch Log Retention**

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
- Rekognition: $50 (với cache + skip liveness)
- S3: $0.30 (với lifecycle)
- CloudWatch: $2.00 (reduced retention)
- **Total: ~$53.50/month** (85% reduction!)

---

#### Phần 4: Reliability & Disaster Recovery

**4.1. Multi-AZ Deployment**

**Good news:** AWS services đã Multi-AZ by default:
- ✅ Lambda: Automatically deployed across AZs
- ✅ DynamoDB: Multi-AZ replication
- ✅ API Gateway: Multi-AZ
- ✅ S3: Cross-AZ replication

**No action needed!**

**4.2. DynamoDB Backup Strategy**

**Enable automated backups:**
```bash
# Point-in-time recovery (already done)
# Restore to any point trong last 35 days

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

**4.3. Cross-Region Replication**

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

#### Phần 5: Compliance & Audit

**5.1. Enable CloudTrail**

**Track all API calls cho audit:**

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

**5.2. Enable AWS Config**

**Track resource configuration changes:**

```bash
aws configservice put-configuration-recorder \
  --configuration-recorder name=smart-campus-config,roleARN=arn:aws:iam::${AWS_ACCOUNT_ID}:role/aws-config-role \
  --recording-group allSupported=true,includeGlobalResourceTypes=true

aws configservice start-configuration-recorder \
  --configuration-recorder-name smart-campus-config \
  --region ap-southeast-1
```

**5.3. Enable GuardDuty**

**Threat detection:**

```bash
aws guardduty create-detector \
  --enable \
  --region ap-southeast-1
```

---

#### Security Checklist

Verify hệ thống đạt chuẩn security:

- [ ] IAM least privilege policies applied
- [ ] WAF enabled với rate limiting
- [ ] DynamoDB PITR enabled
- [ ] S3 encryption at rest enabled
- [ ] Secrets Manager cho sensitive data
- [ ] CloudTrail logging enabled
- [ ] VPC Flow Logs enabled (nếu có VPC)
- [ ] API Gateway throttling configured
- [ ] Lambda timeout < 30s (prevent runaway)
- [ ] No hard-coded credentials trong code

#### Performance Checklist

- [ ] Lambda memory optimized (512MB)
- [ ] Provisioned Concurrency (production only)
- [ ] API Gateway caching enabled
- [ ] DynamoDB indexes optimized
- [ ] S3 Transfer Acceleration enabled
- [ ] CloudFront CDN (nếu có frontend)
- [ ] Redis caching cho Rekognition results

#### Cost Optimization Checklist

- [ ] DynamoDB right-sized (Provisioned if predictable)
- [ ] S3 Lifecycle policies configured
- [ ] CloudWatch log retention reduced
- [ ] Lambda timeout không quá cao
- [ ] Unused resources deleted
- [ ] AWS Cost Explorer reviewed monthly
- [ ] Budget alerts configured

#### Bước tiếp theo

Hãy chuyển sang [Bước 12: Cleanup Resources](../5.12-cleanup) để xóa tất cả resources sau workshop!
Human: continue