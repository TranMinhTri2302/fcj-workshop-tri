---
title: "Configure API Gateway"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Tổng quan

Amazon API Gateway là dịch vụ managed API giúp expose Lambda functions qua HTTP/HTTPS endpoints. Trong bước này, bạn sẽ:
- Tạo HTTP API (API Gateway v2)
- Configure routes và integrations
- Setup CORS
- Integrate với Lambda function
- Test API endpoints

#### Tại sao chọn HTTP API thay vì REST API?

| Feature | HTTP API | REST API |
|---------|----------|----------|
| **Giá** | $1.00/M requests | $3.50/M requests |
| **Latency** | Thấp hơn ~30% | Cao hơn |
| **JWT Auth** | Native support | Cần custom authorizer |
| **CORS** | Auto config | Manual config |
| **Use case** | Modern apps, microservices | Legacy, advanced features |

→ Chọn **HTTP API** cho Smart Campus (rẻ hơn, đủ features)

#### Kiến trúc API Gateway

```
User/Frontend
    ↓
[CloudFront CDN] (optional)
    ↓
[API Gateway HTTP API]
    ↓
[Lambda Function]
    ↓
[DynamoDB/Rekognition/S3...]
```

#### Bước 1: Tạo HTTP API

```bash
# Create API
aws apigatewayv2 create-api \
  --name smart-campus-api \
  --protocol-type HTTP \
  --description "Smart Campus Attendance System API" \
  --cors-configuration AllowOrigins="*",AllowMethods="GET,POST,PUT,DELETE",AllowHeaders="*" \
  --region ap-southeast-1 \
  --tags Project=SmartCampus
```

**Expected output:**
```json
{
    "ApiId": "abc123xyz",
    "ApiEndpoint": "https://abc123xyz.execute-api.ap-southeast-1.amazonaws.com",
    "Name": "smart-campus-api",
    "ProtocolType": "HTTP",
    "CreatedDate": "2026-08-06T10:00:00Z"
}
```

**Save API ID:**
```bash
export API_ID=abc123xyz
echo "API_ID=${API_ID}" >> ~/.bashrc
```

#### Bước 2: Create Lambda Integration

```bash
# Get Lambda ARN
LAMBDA_ARN=$(aws lambda get-function \
  --function-name smart-campus-api \
  --region ap-southeast-1 \
  --query 'Configuration.FunctionArn' \
  --output text)

# Create integration
aws apigatewayv2 create-integration \
  --api-id ${API_ID} \
  --integration-type AWS_PROXY \
  --integration-uri ${LAMBDA_ARN} \
  --payload-format-version 2.0 \
  --region ap-southeast-1
```

**Expected output:**
```json
{
    "IntegrationId": "int123",
    "IntegrationType": "AWS_PROXY",
    "IntegrationUri": "arn:aws:lambda:ap-southeast-1:123456789012:function:smart-campus-api",
    "PayloadFormatVersion": "2.0"
}
```

**Save Integration ID:**
```bash
export INTEGRATION_ID=int123
```

#### Bước 3: Create Routes

**Route 1: Health check (public)**
```bash
aws apigatewayv2 create-route \
  --api-id ${API_ID} \
  --route-key 'GET /health' \
  --target integrations/${INTEGRATION_ID} \
  --region ap-southeast-1
```

**Route 2: Catch-all route (proxy tất cả requests)**
```bash
aws apigatewayv2 create-route \
  --api-id ${API_ID} \
  --route-key '$default' \
  --target integrations/${INTEGRATION_ID} \
  --region ap-southeast-1
```

**Note:** `$default` route sẽ forward tất cả requests (bất kỳ method/path nào) tới Lambda. FastAPI sẽ handle routing bên trong.

#### Bước 4: Create Stage và Deploy

```bash
# Create stage
aws apigatewayv2 create-stage \
  --api-id ${API_ID} \
  --stage-name prod \
  --auto-deploy \
  --description "Production stage" \
  --region ap-southeast-1
```

**Expected output:**
```json
{
    "StageName": "prod",
    "AutoDeploy": true,
    "CreatedDate": "2026-08-06T10:05:00Z"
}
```

**API endpoint bây giờ:**
```
https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/
```

#### Bước 5: Grant API Gateway Permission

API Gateway cần permission để invoke Lambda:

```bash
aws lambda add-permission \
  --function-name smart-campus-api \
  --statement-id apigateway-invoke \
  --action lambda:InvokeFunction \
  --principal apigateway.amazonaws.com \
  --source-arn "arn:aws:execute-api:ap-southeast-1:${AWS_ACCOUNT_ID}:${API_ID}/*/*" \
  --region ap-southeast-1
```

**Verify permission:**
```bash
aws lambda get-policy \
  --function-name smart-campus-api \
  --region ap-southeast-1
```

#### Bước 6: Test API Endpoints

**Test 1: Health check**
```bash
curl https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/health
```

**Expected:**
```json
{
  "status": "healthy",
  "timestamp": "2026-08-06T10:10:00Z"
}
```

**Test 2: API documentation**
```bash
curl https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/docs
```

Should return FastAPI Swagger UI HTML.

**Test 3: User registration (protected endpoint)**
```bash
# Without JWT token → 401
curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","name":"Test User","role":"STAFF"}'
```

**Expected:** 401 Unauthorized

```bash
# With JWT token → 200
curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${JWT_TOKEN}" \
  -d '{"email":"test@example.com","name":"Test User","role":"STAFF"}'
```

**Expected:** 200 OK

#### Bước 7: Configure Throttling (Rate Limiting)

**Default throttling:**
```bash
aws apigatewayv2 update-stage \
  --api-id ${API_ID} \
  --stage-name prod \
  --default-route-settings '{
    "ThrottlingBurstLimit": 100,
    "ThrottlingRateLimit": 50
  }' \
  --region ap-southeast-1
```

**Meaning:**
- Rate limit: 50 requests/second (steady state)
- Burst limit: 100 requests (token bucket)

**Per-route throttling** (nếu cần):
```bash
# Route-specific: Face recognition endpoint cần rate limit thấp hơn
aws apigatewayv2 update-route \
  --api-id ${API_ID} \
  --route-id ${ROUTE_ID} \
  --throttle-settings RateLimit=10,BurstLimit=20 \
  --region ap-southeast-1
```

#### Bước 8: Enable Access Logging

**Create CloudWatch Log Group:**
```bash
aws logs create-log-group \
  --log-group-name /aws/apigateway/smart-campus-api \
  --region ap-southeast-1
```

**Update stage với logging:**
```bash
# Get log group ARN
LOG_ARN=$(aws logs describe-log-groups \
  --log-group-name-prefix /aws/apigateway/smart-campus-api \
  --region ap-southeast-1 \
  --query 'logGroups[0].arn' \
  --output text)

# Enable logging
aws apigatewayv2 update-stage \
  --api-id ${API_ID} \
  --stage-name prod \
  --access-log-settings '{
    "DestinationArn": "'${LOG_ARN}'",
    "Format": "$context.requestId $context.routeKey $context.status $context.integrationErrorMessage"
  }' \
  --region ap-southeast-1
```

**View logs:**
```bash
aws logs tail /aws/apigateway/smart-campus-api --follow
```

#### Bước 9: Setup Custom Domain (Optional)

**Prerequisite:**
- Có domain name (e.g., api.smartcampus.edu.vn)
- SSL certificate trong ACM (Certificate Manager)

**Create custom domain:**
```bash
aws apigatewayv2 create-domain-name \
  --domain-name api.smartcampus.edu.vn \
  --domain-name-configurations CertificateArn=arn:aws:acm:... \
  --region ap-southeast-1
```

**Create API mapping:**
```bash
aws apigatewayv2 create-api-mapping \
  --domain-name api.smartcampus.edu.vn \
  --api-id ${API_ID} \
  --stage prod \
  --region ap-southeast-1
```

**Update DNS (Route 53):**
```bash
# Get API Gateway domain name
API_DOMAIN=$(aws apigatewayv2 get-domain-name \
  --domain-name api.smartcampus.edu.vn \
  --query 'DomainNameConfigurations[0].ApiGatewayDomainName' \
  --output text)

# Create CNAME record in Route 53
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "api.smartcampus.edu.vn",
        "Type": "CNAME",
        "TTL": 300,
        "ResourceRecords": [{"Value": "'${API_DOMAIN}'"}]
      }
    }]
  }'
```

**Wait for DNS propagation (~5 minutes)**

**Test custom domain:**
```bash
curl https://api.smartcampus.edu.vn/health
```

#### Bước 10: Configure CORS (Advanced)

**Nếu cần config CORS chi tiết hơn:**

```bash
aws apigatewayv2 update-api \
  --api-id ${API_ID} \
  --cors-configuration '{
    "AllowOrigins": ["https://smartcampus.edu.vn", "http://localhost:3000"],
    "AllowMethods": ["GET", "POST", "PUT", "DELETE", "OPTIONS"],
    "AllowHeaders": ["Content-Type", "Authorization", "X-Requested-With"],
    "ExposeHeaders": ["X-Request-ID", "X-Correlation-ID"],
    "MaxAge": 3600,
    "AllowCredentials": true
  }' \
  --region ap-southeast-1
```

**Test CORS:**
```bash
curl -X OPTIONS https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/users \
  -H "Origin: https://smartcampus.edu.vn" \
  -H "Access-Control-Request-Method: POST" \
  -v
```

**Expected headers in response:**
```
Access-Control-Allow-Origin: https://smartcampus.edu.vn
Access-Control-Allow-Methods: GET,POST,PUT,DELETE,OPTIONS
Access-Control-Allow-Headers: Content-Type,Authorization
Access-Control-Max-Age: 3600
```

#### Monitoring API Gateway

**View metrics:**
```bash
aws cloudwatch get-metric-statistics \
  --namespace AWS/ApiGateway \
  --metric-name Count \
  --dimensions Name=ApiId,Value=${API_ID} \
  --start-time 2026-08-06T00:00:00Z \
  --end-time 2026-08-06T23:59:59Z \
  --period 3600 \
  --statistics Sum \
  --region ap-southeast-1
```

**Key metrics:**
- **Count**: Tổng số requests
- **4XXError**: Client errors (400-499)
- **5XXError**: Server errors (500-599)
- **Latency**: Response time (aim: < 1000ms)
- **IntegrationLatency**: Lambda execution time

#### Troubleshooting

**Error: 403 Missing Authentication Token**
```
{"message": "Missing Authentication Token"}
```
**Cause:** Route không tồn tại hoặc method sai
**Solution:** Check route key: `GET /health` vs `POST /health`

**Error: 502 Bad Gateway**
```
{"message": "Internal server error"}
```
**Cause:** Lambda error hoặc timeout
**Solution:** Check Lambda CloudWatch Logs

**Error: 429 Too Many Requests**
```
{"message": "Limit Exceeded"}
```
**Cause:** Vượt throttling limit
**Solution:** Tăng rate limit hoặc implement retry logic

**Error: CORS preflight failed**
**Cause:** CORS config chưa đúng
**Solution:** Update CORS config với AllowOrigins chính xác

#### Cost Estimation

**API Gateway HTTP API pricing:**
- First 300M requests/month: $1.00 per million
- Next 700M requests: $0.90 per million

**Monthly estimate (100K requests):**
- API Gateway: 0.1M × $1.00 = $0.10
- Lambda: $0.44 (from previous step)
- **Total: $0.54/month**

**Free Tier:** 1M requests/month FREE (first 12 months)

#### Security Best Practices

1. **Enable WAF** (Web Application Firewall):
```bash
# Tạo Web ACL
aws wafv2 create-web-acl \
  --name smart-campus-waf \
  --scope REGIONAL \
  --default-action Allow={} \
  --rules file://waf-rules.json \
  --region ap-southeast-1

# Associate với API Gateway
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:... \
  --resource-arn arn:aws:apigateway:...
```

2. **Enable API keys** (nếu cần):
```bash
aws apigatewayv2 create-api-key \
  --name workshop-api-key \
  --enabled
```

3. **Enable request validation**:
- Validate request body schema
- Validate query parameters
- Reject malformed requests

#### Verify Setup

Checklist:
- [ ] API Gateway created và deployed
- [ ] Lambda integration working
- [ ] Health endpoint returns 200
- [ ] CORS configured correctly
- [ ] Throttling enabled
- [ ] Access logging enabled
- [ ] Custom domain (optional) working

#### API Endpoints Summary

**Base URL:** `https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com`

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | /health | Health check | No |
| GET | /api/docs | API documentation | No |
| POST | /api/auth/login | User login | No |
| POST | /api/users | Create user | Yes |
| GET | /api/users/me | Get current user | Yes |
| POST | /api/faces/register | Register face | Yes |
| POST | /api/attendance/recognize | Face check-in | Yes |
| GET | /api/attendance/history | Attendance history | Yes |
| GET | /api/reports/daily | Daily report | Yes |

#### Bước tiếp theo

Hãy chuyển sang [Bước 7: Setup EventBridge và SQS](../5.7-eventbridge) để xây dựng event-driven architecture!
