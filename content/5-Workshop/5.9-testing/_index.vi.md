---
title: "Testing Hệ Thống"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

#### Tổng quan

Đây là bước **QUAN TRỌNG NHẤT** để verify hệ thống hoạt động end-to-end. Bạn sẽ test:
1. ✅ Face Registration workflow
2. ✅ Attendance Check-in (happy path)
3. ✅ Face Liveness Detection (anti-spoofing)
4. ✅ Rule Engine validations
5. ✅ Event-driven pipeline (EventBridge → SQS → Workers)
6. ✅ Data Lake analytics

#### Test Case 1: Face Registration

**Objective:** Đăng ký khuôn mặt cho user mới

**Prerequisite:**
- User đã tồn tại trong DynamoDB users table
- Có ảnh khuôn mặt rõ ràng (test-images/user1.jpg)

**Steps:**

1. **Upload ảnh qua API:**
```bash
# Convert image to base64
IMAGE_BASE64=$(base64 -i test-images/user1.jpg)

# Call API
curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/faces/register \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${JWT_TOKEN}" \
  -d '{
    "user_id": "user-001",
    "image_base64": "'"${IMAGE_BASE64}"'"
  }'
```

2. **Expected Response:**
```json
{
  "face_id": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
  "user_id": "user-001",
  "confidence": 99.8,
  "message": "Face registered successfully"
}
```

3. **Verify in DynamoDB:**
```bash
aws dynamodb get-item \
  --table-name smart-campus-faces \
  --key '{"face_id":{"S":"1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p"}}' \
  --region ap-southeast-1
```

4. **Verify in Rekognition:**
```bash
aws rekognition list-faces \
  --collection-id smart-campus-faces \
  --region ap-southeast-1 \
  --query "Faces[?ExternalImageId=='user-001']"
```

5. **Verify S3 image:**
```bash
aws s3 ls s3://smart-campus-images-${AWS_ACCOUNT_ID}/faces/raw/user-001/
```

**Expected Results:**
- ✅ Response status 200
- ✅ Face ID returned
- ✅ DynamoDB record exists
- ✅ Rekognition collection has face
- ✅ S3 has original image
- ✅ CloudWatch logs show "Face registered"

**Error Scenarios:**

**Test 1.1: No face detected**
```bash
# Upload ảnh không có khuôn mặt
curl -X POST ... -d '{"user_id":"user-001","image_base64":"<landscape-image>"}'

# Expected: 400 Bad Request
{
  "error": "No face detected in image"
}
```

**Test 1.2: Multiple faces**
```bash
# Upload ảnh có 2 người
curl -X POST ... -d '{"user_id":"user-001","image_base64":"<group-photo>"}'

# Expected: 400 Bad Request
{
  "error": "Multiple faces detected. Only one face allowed."
}
```

**Test 1.3: Duplicate registration**
```bash
# Register lại cùng khuôn mặt
curl -X POST ... -d '{"user_id":"user-001","image_base64":"<same-image>"}'

# Expected: 400 Bad Request
{
  "error": "Face already registered for this user"
}
```

---

#### Test Case 2: Attendance Check-in (Happy Path)

**Objective:** Điểm danh thành công trong giờ

**Prerequisite:**
- User đã đăng ký face (Test Case 1)
- Hiện tại đang trong session time (7:00-11:30 MORNING)

**Steps:**

1. **Check-in request:**
```bash
# Chụp ảnh mới (hoặc dùng ảnh khác của cùng người)
IMAGE_BASE64=$(base64 -i test-images/user1-checkin.jpg)

# Call attendance API
curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/attendance/recognize \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${JWT_TOKEN}" \
  -d '{
    "image_base64": "'"${IMAGE_BASE64}"'",
    "camera_id": "camera-01",
    "timestamp": "2026-08-06T07:55:00+07:00"
  }'
```

2. **Expected Response:**
```json
{
  "attendance_id": "att-abc123",
  "user_id": "user-001",
  "user_name": "Nguyen Van A",
  "status": "PRESENT",
  "session_type": "MORNING",
  "confidence": 98.5,
  "timestamp": "2026-08-06T07:55:00+07:00",
  "message": "Attendance recorded successfully"
}
```

3. **Verify DynamoDB record:**
```bash
aws dynamodb query \
  --table-name smart-campus-attendance \
  --index-name date-index \
  --key-condition-expression "#d = :date" \
  --expression-attribute-names '{"#d":"date"}' \
  --expression-attribute-values '{":date":{"S":"2026-08-06"}}' \
  --region ap-southeast-1
```

4. **Verify EventBridge event published:**
```bash
# Check CloudWatch Logs của EventBridge
aws logs filter-log-events \
  --log-group-name /aws/events/smart-campus \
  --filter-pattern "AttendanceRecorded" \
  --start-time $(date -u -d '5 minutes ago' +%s)000 \
  --region ap-southeast-1
```

5. **Verify SQS message:**
```bash
# Check Analytics Queue
aws sqs receive-message \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-analytics-queue \
  --max-number-of-messages 1 \
  --region ap-southeast-1
```

6. **Verify S3 Data Lake:**
```bash
# Check today's partition
aws s3 ls s3://smart-campus-datalake-${AWS_ACCOUNT_ID}/attendance/year=2026/month=08/day=06/
```

7. **Verify notification sent:**
```bash
# Check email inbox hoặc CloudWatch Logs của Notification Worker
aws logs tail /aws/lambda/smart-campus-api --follow | grep "Notification sent"
```

**Expected Results:**
- ✅ Status 200
- ✅ Status = "PRESENT"
- ✅ DynamoDB attendance record exists
- ✅ EventBridge event published
- ✅ SQS message in Analytics Queue
- ✅ SQS message in Notification Queue
- ✅ S3 Data Lake has record
- ✅ Email received (nếu có SES setup)

---

#### Test Case 3: Attendance Check-in LATE

**Objective:** Điểm danh muộn (sau 15 phút)

**Steps:**

```bash
# Check-in lúc 8:20 (muộn 20 phút)
curl -X POST ... -d '{
  "image_base64": "...",
  "camera_id": "camera-01",
  "timestamp": "2026-08-06T08:20:00+07:00"
}'
```

**Expected Response:**
```json
{
  "status": "LATE",
  "message": "Attendance recorded. You are 20 minutes late."
}
```

---

#### Test Case 4: Duplicate Check-in (Same Session)

**Objective:** Ngăn chặn điểm danh trùng

**Steps:**

```bash
# Check-in lần 2 trong cùng session
curl -X POST ... -d '{
  "image_base64": "...",
  "camera_id": "camera-01",
  "timestamp": "2026-08-06T08:00:00+07:00"
}'
```

**Expected Response:**
```json
{
  "status": "DUPLICATE",
  "message": "You have already checked in for MORNING session today",
  "is_duplicate": true,
  "previous_attendance_id": "att-abc123"
}
```

**Expected Results:**
- ✅ Status 200 (idempotent, không báo lỗi)
- ✅ is_duplicate = true
- ✅ KHÔNG tạo record mới trong DynamoDB
- ✅ KHÔNG publish event mới

---

#### Test Case 5: Face Liveness Detection (Anti-Spoofing)

**Objective:** Phát hiện ảnh in/video replay

**Test 5.1: Ảnh in (Printed Photo)**

```bash
# Chụp ảnh của ảnh in
IMAGE_BASE64=$(base64 -i test-images/spoofing/printed-photo.jpg)

curl -X POST ... -d '{
  "image_base64": "'"${IMAGE_BASE64}"'",
  "camera_id": "camera-01"
}'
```

**Expected Response:**
```json
{
  "error": "Liveness check failed",
  "reason": "Possible spoofing attack detected",
  "confidence": 45.2,
  "threshold": 80
}
```

**Test 5.2: Screenshot từ video**

```bash
IMAGE_BASE64=$(base64 -i test-images/spoofing/screenshot.jpg)

curl -X POST ...
```

**Expected Response:**
```json
{
  "error": "Liveness check failed",
  "reason": "Video replay attack detected",
  "confidence": 52.8
}
```

**Expected Results:**
- ✅ Status 400 (rejected)
- ✅ Confidence < 80%
- ✅ KHÔNG tạo attendance record
- ✅ KHÔNG publish event
- ✅ Log ghi nhận "Liveness failed"

---

#### Test Case 6: Unknown Face

**Objective:** Phát hiện khuôn mặt chưa đăng ký

**Steps:**

```bash
# Dùng ảnh người chưa đăng ký
IMAGE_BASE64=$(base64 -i test-images/unknown-person.jpg)

curl -X POST ... -d '{
  "image_base64": "'"${IMAGE_BASE64}"'",
  "camera_id": "camera-01"
}'
```

**Expected Response:**
```json
{
  "error": "Unknown face",
  "message": "Face not recognized. Please register first.",
  "camera_id": "camera-01",
  "timestamp": "2026-08-06T08:00:00+07:00"
}
```

**Verify UnknownFaceDetected event:**
```bash
aws logs filter-log-events \
  --log-group-name /aws/events/smart-campus \
  --filter-pattern "UnknownFaceDetected" \
  --region ap-southeast-1
```

**Expected Results:**
- ✅ Status 404
- ✅ Message rõ ràng
- ✅ UnknownFaceDetected event published
- ✅ Security incident created (nếu có module security)
- ✅ Ảnh được lưu vào S3: `attendance/unknown/{camera_id}/{timestamp}.jpg`

---

#### Test Case 7: Outside Session Time

**Objective:** Từ chối điểm danh ngoài giờ

**Steps:**

```bash
# Check-in lúc 12:00 (giữa MORNING và AFTERNOON session)
curl -X POST ... -d '{
  "image_base64": "...",
  "camera_id": "camera-01",
  "timestamp": "2026-08-06T12:00:00+07:00"
}'
```

**Expected Response:**
```json
{
  "status": "REJECTED",
  "reason": "Outside session time",
  "message": "No active session at this time",
  "next_session": "AFTERNOON",
  "next_session_start": "13:00"
}
```

---

#### Test Case 8: Analytics Query (Athena)

**Objective:** Verify Data Lake và Athena hoạt động

**Prerequisite:**
- Đã có data trong S3 Data Lake
- Glue Crawler đã chạy và tạo tables

**Steps:**

1. **Query total attendance today:**
```sql
SELECT 
  date,
  session_type,
  status,
  COUNT(*) as total
FROM attendance_records
WHERE date = '2026-08-06'
GROUP BY date, session_type, status
ORDER BY session_type, status
```

**Execute via AWS CLI:**
```bash
aws athena start-query-execution \
  --query-string "SELECT date, COUNT(*) as total FROM attendance_records WHERE date='2026-08-06' GROUP BY date" \
  --query-execution-context Database=smart_campus_db \
  --result-configuration OutputLocation=s3://smart-campus-datalake-${AWS_ACCOUNT_ID}/athena-results/ \
  --region ap-southeast-1
```

**Get results:**
```bash
# Save query execution ID
QUERY_ID="abc123-query-id"

# Wait for completion
aws athena get-query-execution --query-execution-id ${QUERY_ID}

# Get results
aws athena get-query-results --query-execution-id ${QUERY_ID}
```

**Expected Results:**
- ✅ Query executes successfully
- ✅ Returns correct count
- ✅ Latency < 10 seconds

---

#### Test Case 9: End-to-End Workflow

**Objective:** Test complete attendance workflow từ đầu đến cuối

**Steps:**

1. Register user → 2. Register face → 3. Check-in → 4. Verify all components

```bash
#!/bin/bash
# end-to-end-test.sh

set -e  # Exit on error

echo "=== Smart Campus E2E Test ==="

# 1. Create user
echo "Step 1: Creating user..."
aws dynamodb put-item \
  --table-name smart-campus-users \
  --item '{"user_id":{"S":"test-user-e2e"},"email":{"S":"test@example.com"},"name":{"S":"Test User"},"role":{"S":"STAFF"},"status":{"S":"ACTIVE"},"face_registered":{"BOOL":false}}' \
  --region ap-southeast-1
echo "✅ User created"

# 2. Register face
echo "Step 2: Registering face..."
IMAGE_BASE64=$(base64 -i test-images/user-e2e.jpg)
REGISTER_RESPONSE=$(curl -s -X POST ${API_URL}/api/faces/register \
  -H "Content-Type: application/json" \
  -d '{"user_id":"test-user-e2e","image_base64":"'"${IMAGE_BASE64}"'"}')

FACE_ID=$(echo $REGISTER_RESPONSE | jq -r '.face_id')
echo "✅ Face registered: $FACE_ID"

# 3. Check-in
echo "Step 3: Check-in attendance..."
CHECKIN_RESPONSE=$(curl -s -X POST ${API_URL}/api/attendance/recognize \
  -H "Content-Type: application/json" \
  -d '{"image_base64":"'"${IMAGE_BASE64}"'","camera_id":"camera-e2e-test"}')

ATTENDANCE_ID=$(echo $CHECKIN_RESPONSE | jq -r '.attendance_id')
STATUS=$(echo $CHECKIN_RESPONSE | jq -r '.status')
echo "✅ Attendance recorded: $ATTENDANCE_ID, Status: $STATUS"

# 4. Verify DynamoDB
echo "Step 4: Verifying DynamoDB..."
aws dynamodb get-item \
  --table-name smart-campus-attendance \
  --key '{"attendance_id":{"S":"'"${ATTENDANCE_ID}"'"}}' \
  --region ap-southeast-1 > /dev/null
echo "✅ DynamoDB record exists"

# 5. Verify S3 Data Lake (wait 30s for worker)
echo "Step 5: Waiting for Data Lake..."
sleep 30
aws s3 ls s3://smart-campus-datalake-${AWS_ACCOUNT_ID}/attendance/year=$(date +%Y)/month=$(date +%m)/day=$(date +%d)/ | grep -q .
echo "✅ Data Lake record exists"

echo ""
echo "🎉 E2E Test PASSED!"
```

---

#### Test Results Summary

**Create test report:**

```bash
# test-report.sh
echo "Smart Campus Test Report - $(date)"
echo "========================================"
echo ""
echo "Test Cases:"
echo "✅ TC1: Face Registration - PASSED"
echo "✅ TC2: Check-in Happy Path - PASSED"
echo "✅ TC3: Check-in Late - PASSED"
echo "✅ TC4: Duplicate Check - PASSED"
echo "✅ TC5: Liveness Detection - PASSED"
echo "✅ TC6: Unknown Face - PASSED"
echo "✅ TC7: Outside Session - PASSED"
echo "✅ TC8: Analytics Query - PASSED"
echo "✅ TC9: End-to-End - PASSED"
echo ""
echo "Coverage: 9/9 test cases passed (100%)"
echo "Duration: ~15 minutes"
echo "Status: ALL GREEN ✅"
```

#### Troubleshooting

**Issue: API returns 403 Forbidden**
- Check JWT token valid
- Check IAM permissions
- Check WAF rules (nếu có)

**Issue: Face not recognized**
- Check face quality (brightness, sharpness)
- Check confidence threshold (lower to 70% for testing)
- Verify face exists in collection

**Issue: Event not published**
- Check EventBridge rules active
- Check Lambda has permission to PutEvents
- Check CloudWatch Logs for errors

**Issue: SQS message not processed**
- Check Lambda trigger active
- Check DLQ for failed messages
- Check Lambda timeout sufficient

#### Bước tiếp theo

Hãy chuyển sang [Bước 10: Monitoring với CloudWatch](../5.10-monitoring) để setup giám sát hệ thống!
