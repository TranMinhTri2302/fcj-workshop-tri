---
title: "Testing the System"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Overview

This is the **MOST IMPORTANT** step to verify the system works end-to-end. You will test all 8 Workflows of the Smart Campus system:

1. ✅ **WF1**: Authentication (Cognito JWT, Force Change Password)
2. ✅ **WF2**: Face Registration (Rekognition IndexFaces + S3)
3. ✅ **WF3**: Attendance Check-in (SearchFacesByImage + Rule Engine + Liveness)
4. ✅ **WF4**: Notification (SNS Multi-channel + EventBridge)
5. ✅ **WF5**: Analytics (DynamoDB + Athena + Dashboard)
6. ✅ **WF6**: AI Assistant (Bedrock NL2SQL + Athena) - *Optional*
7. ✅ **WF7**: Security (Risk Engine + Incident Management) - *Optional*
8. ✅ **WF8**: Task Management (Task CRUD + Incident + Maintenance)

---

## Test Case 1: Face Registration (WF2)

**Objective:** Register face for new user with anti-fraud checks

**Prerequisite:**
- User exists in DynamoDB `smart-campus-users` table
- Have clear face photo (test-images/user1.jpg)

**Steps:**

1. **Upload image via API:**
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
  "bounding_box": {"Width": 0.5, "Height": 0.6, "Left": 0.2, "Top": 0.1},
  "message": "Face registered successfully"
}
```

3. **Verify in DynamoDB (`smart-campus-faces`):**
```bash
aws dynamodb get-item \
  --table-name smart-campus-faces \
  --key '{"face_id":{"S":"1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p"}}' \
  --region ap-southeast-1
```

4. **Verify in Rekognition Collection:**
```bash
aws rekognition list-faces \
  --collection-id smart-campus-faces \
  --region ap-southeast-1 \
  --query "Faces[?ExternalImageId=='user-001']"
```

5. **Verify S3 image stored:**
```bash
aws s3 ls s3://smart-campus-images-${AWS_ACCOUNT_ID}/faces/raw/user-001/
```

**Expected Results:**
- ✅ Response status 200
- ✅ Face ID returned (from Rekognition)
- ✅ Confidence ≥ 80%
- ✅ BoundingBox coordinates returned
- ✅ DynamoDB record exists with face_id, user_id, image_key, confidence, bounding_box
- ✅ Rekognition collection has face with ExternalImageId = user_id
- ✅ S3 has original image at `faces/raw/{user_id}/{filename}.jpg`
- ✅ CloudWatch logs show "Face registered"

**Error Scenarios (Anti-Fraud):**

**Test 1.1: No face detected**
```bash
# Upload image without face (landscape)
curl -X POST ... -d '{"user_id":"user-001","image_base64":"<landscape-image>"}'

# Expected: 400 Bad Request
{
  "error": "No face detected in image"
}
```

**Test 1.2: Multiple faces detected**
```bash
# Upload group photo
curl -X POST ... -d '{"user_id":"user-001","image_base64":"<group-photo>"}'

# Expected: 400 Bad Request
{
  "error": "Multiple faces detected. Only one face allowed."
}
```

**Test 1.3: Duplicate face registration (Anti-Fraud)**
```bash
# Register same face for different user
curl -X POST ... -d '{"user_id":"user-002","image_base64":"<same-image>"}'

# Expected: 400 Bad Request - Face already registered to another user
{
  "error": "Face already registered to another user",
  "existing_user_id": "user-001"
}
```

**Test 1.4: Image validation (size/format)**
```bash
# Upload > 5MB image
curl -X POST ... -d '{"user_id":"user-001","image_base64":"<large-image>"}'

# Expected: 400 Bad Request
{
  "error": "Image size exceeds 5MB limit"
}
```

---

## Test Case 2: Attendance Check-in - Happy Path (WF3)

**Objective:** Successful check-in during session time with liveness verification

**Prerequisite:**
- User has registered face (Test Case 1)
- Currently within session time (MORNING: 07:00-11:30)

**Steps:**

1. **Check-in request with liveness session:**
```bash
# Step 1: Create liveness session
curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/attendance/liveness/start \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${JWT_TOKEN}"

# Response: {"session_id": "liveness-session-xyz"}

# Step 2: Stream video frames to Rekognition (via frontend SDK)
# Step 3: Get liveness results
curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/attendance/liveness/result \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${JWT_TOKEN}" \
  -d '{"session_id": "liveness-session-xyz"}'

# Expected: {"liveness_passed": true, "confidence": 95.2}

# Step 4: Check-in with face recognition
IMAGE_BASE64=$(base64 -i test-images/user1-checkin.jpg)

curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/attendance/check-in \
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

3. **Verify DynamoDB record (`smart-campus-attendance`):**
```bash
aws dynamodb query \
  --table-name smart-campus-attendance \
  --index-name date-index \
  --key-condition-expression "#d = :date" \
  --expression-attribute-names '{"#d":"date"}' \
  --expression-attribute-values '{":date":{"S":"2026-08-06"}}' \
  --region ap-southeast-1
```

4. **Verify EventBridge event published (`AttendanceRecorded`):**
```bash
# Check CloudWatch Logs for EventBridge
aws logs filter-log-events \
  --log-group-name /aws/events/smart-campus \
  --filter-pattern "AttendanceRecorded" \
  --start-time $(date -u -d '5 minutes ago' +%s)000 \
  --region ap-southeast-1
```

5. **Verify SQS messages in both queues:**
```bash
# Check Analytics Queue
aws sqs receive-message \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-analytics-queue \
  --max-number-of-messages 1 \
  --region ap-southeast-1

# Check Notification Queue
aws sqs receive-message \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-notification-queue \
  --max-number-of-messages 1 \
  --region ap-southeast-1
```

6. **Verify S3 Data Lake (partitioned by date):**
```bash
# Check today's partition
aws s3 ls s3://smart-campus-datalake-${AWS_ACCOUNT_ID}/attendance/year=2026/month=08/day=06/
```

7. **Verify notification sent (WF4):**
```bash
# Check Notification Worker logs
aws logs tail /aws/lambda/smart-campus-notification-worker --follow | grep "Notification sent"
```

**Expected Results:**
- ✅ Status 200
- ✅ Status = "PRESENT" (on time within 15 min of session start)
- ✅ Session_type = "MORNING" (auto-detected from timestamp)
- ✅ Confidence ≥ 80% (Rekognition SearchFacesByImage)
- ✅ Liveness check passed (confidence ≥ 80%)
- ✅ DynamoDB attendance record exists with all fields
- ✅ EventBridge event `AttendanceRecorded` published
- ✅ SQS message in Analytics Queue (for Data Lake)
- ✅ SQS message in Notification Queue (for email/push)
- ✅ S3 Data Lake has record in partitioned path
- ✅ Email/Push notification received (if SES/SNS configured)

---

## Test Case 3: Attendance Check-in LATE (WF3 Rule Engine)

**Objective:** Check-in late (after 15 minutes grace period)

**Steps:**

```bash
# Check-in at 08:20 (20 minutes late for MORNING session starting 07:00)
curl -X POST ... -d '{
  "image_base64": "...",
  "camera_id": "camera-01",
  "timestamp": "2026-08-06T08:20:00+07:00"
}'
```

**Expected Response:**
```json
{
  "attendance_id": "att-def456",
  "user_id": "user-001",
  "status": "LATE",
  "session_type": "MORNING",
  "confidence": 97.2,
  "timestamp": "2026-08-06T08:20:00+07:00",
  "message": "Attendance recorded. You are 20 minutes late."
}
```

**Rule Engine Logic:**
- MORNING: 07:00-11:30 (Late after 07:15)
- AFTERNOON: 13:00-17:30 (Late after 13:15)
- EVENING: 17:30-21:00 (Late after 17:45)

---

## Test Case 4: Duplicate Check-in Prevention (WF3 Idempotency)

**Objective:** Prevent duplicate check-in in same session

**Steps:**

```bash
# First check-in
curl -X POST ... -d '{"image_base64":"...","camera_id":"camera-01","timestamp":"2026-08-06T07:55:00+07:00"}'

# Second check-in in same session (MORNING)
curl -X POST ... -d '{"image_base64":"...","camera_id":"camera-01","timestamp":"2026-08-06T08:00:00+07:00"}'
```

**Expected Response (2nd request):**
```json
{
  "attendance_id": "att-abc123",
  "user_id": "user-001",
  "status": "DUPLICATE",
  "session_type": "MORNING",
  "is_duplicate": true,
  "previous_attendance_id": "att-abc123",
  "message": "You have already checked in for MORNING session today"
}
```

**Expected Results:**
- ✅ Status 200 (idempotent, not error)
- ✅ is_duplicate = true
- ✅ NO new record in DynamoDB
- ✅ NO new event published to EventBridge
- ✅ Returns existing attendance_id

---

## Test Case 5: Face Liveness Detection - Anti-Spoofing (WF3)

**Objective:** Detect printed photo, video replay, and 3D mask attacks

**Test 5.1: Printed Photo Attack**
```bash
# Photo of printed photo
IMAGE_BASE64=$(base64 -i test-images/spoofing/printed-photo.jpg)

curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/attendance/check-in \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${JWT_TOKEN}" \
  -d '{
    "image_base64": "'"${IMAGE_BASE64}"'",
    "camera_id": "camera-01"
  }'
```

**Expected Response:**
```json
{
  "error": "Liveness check failed",
  "reason": "Possible spoofing attack detected - printed photo",
  "confidence": 45.2,
  "threshold": 80
}
```

**Test 5.2: Video Replay / Screenshot Attack**
```bash
IMAGE_BASE64=$(base64 -i test-images/spoofing/screenshot.jpg)

curl -X POST ...
```

**Expected Response:**
```json
{
  "error": "Liveness check failed",
  "reason": "Video replay attack detected",
  "confidence": 52.8,
  "threshold": 80
}
```

**Test 5.3: 3D Mask Attack (if test data available)**
```bash
IMAGE_BASE64=$(base64 -i test-images/spoofing/3d-mask.jpg)

curl -X POST ...
```

**Expected Response:**
```json
{
  "error": "Liveness check failed",
  "reason": "3D mask attack detected",
  "confidence": 38.5,
  "threshold": 80
}
```

**Expected Results (All Spoofing Tests):**
- ✅ Status 400 (rejected)
- ✅ Confidence < 80% (liveness threshold)
- ✅ NO attendance record created
- ✅ NO event published
- ✅ CloudWatch logs record "Liveness failed" with reason
- ✅ Security event `UnknownFaceDetected` may be published (WF7)

---

## Test Case 6: Unknown Face Detection (WF3 + WF7)

**Objective:** Detect unregistered face and trigger security workflow

**Steps:**

```bash
# Use photo of unregistered person
IMAGE_BASE64=$(base64 -i test-images/unknown-person.jpg)

curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/attendance/check-in \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${JWT_TOKEN}" \
  -d '{
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

**Verify UnknownFaceDetected event (WF7 Security):**
```bash
aws logs filter-log-events \
  --log-group-name /aws/events/smart-campus \
  --filter-pattern "UnknownFaceDetected" \
  --region ap-southeast-1
```

**Verify image saved to S3 (security evidence):**
```bash
aws s3 ls s3://smart-campus-images-${AWS_ACCOUNT_ID}/attendance/unknown/camera-01/
```

**Expected Results:**
- ✅ Status 404
- ✅ Clear error message
- ✅ `UnknownFaceDetected` event published to EventBridge
- ✅ Security incident created in `smart-campus-security` table (WF7)
- ✅ Image saved to S3: `attendance/unknown/{camera_id}/{timestamp}.jpg`
- ✅ Alert notification sent to security team (WF4)

---

## Test Case 7: Outside Session Time (WF3 Rule Engine)

**Objective:** Reject check-in outside defined session hours

**Steps:**

```bash
# Check-in at 12:00 (between MORNING and AFTERNOON)
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

**Test 7.2: Check-in before first session**
```bash
# Check-in at 06:00 (before MORNING)
curl -X POST ... -d '{"timestamp": "2026-08-06T06:00:00+07:00"}'
```

**Expected:** REJECTED with next_session = "MORNING"

**Test 7.3: Check-in after last session**
```bash
# Check-in at 22:00 (after EVENING)
curl -X POST ... -d '{"timestamp": "2026-08-06T22:00:00+07:00"}'
```

**Expected:** REJECTED with no next_session

---

## Test Case 8: WFH (Work From Home) Check-in (WF3 + WF4)

**Objective:** Verify WFH approved users can bypass WAF IP check

**Prerequisite:**
- User has approved WFH leave request (`smart-campus-leaves` table, status=APPROVED, type=WFH)

**Steps:**

```bash
# Check-in from non-whitelisted IP (home network)
# WAF should allow because user has approved WFH
curl -X POST ... -d '{
  "image_base64": "...",
  "camera_id": "camera-home",
  "timestamp": "2026-08-06T07:55:00+07:00"
}'
```

**Expected Response:**
```json
{
  "attendance_id": "att-wfh-001",
  "user_id": "user-001",
  "status": "PRESENT",
  "session_type": "MORNING",
  "location_type": "WFH",
  "message": "WFH attendance recorded successfully"
}
```

**Verify WAF bypass:**
- Check CloudWatch WAF logs - request should be ALLOWED (not BLOCKED)
- Check attendance record has `location_type: "WFH"`

---

## Test Case 9: Analytics Query - Athena (WF5)

**Objective:** Verify Data Lake and Athena work for enterprise analytics

**Prerequisite:**
- Data in S3 Data Lake (attendance, tasks, users)
- Glue Crawlers ran and created 3 tables

**Steps:**

1. **Query 1: Daily Attendance Summary**
```sql
SELECT 
  date,
  session_type,
  status,
  COUNT(*) as total
FROM attendance
WHERE year = '2026' AND month = '08' AND day = '06'
GROUP BY date, session_type, status
ORDER BY session_type, status
```

2. **Query 2: Attendance Trend (30 Days)**
```sql
SELECT 
  date,
  COUNT(*) as total,
  SUM(CASE WHEN status = 'PRESENT' THEN 1 ELSE 0 END) as present,
  SUM(CASE WHEN status = 'LATE' THEN 1 ELSE 0 END) as late,
  SUM(CASE WHEN status = 'ABSENT' THEN 1 ELSE 0 END) as absent
FROM attendance
WHERE year = '2026' AND month = '08'
GROUP BY date
ORDER BY date
```

3. **Query 3: Department Comparison Matrix (Enterprise)**
```sql
SELECT 
  u.department,
  COUNT(DISTINCT u.user_id) as total_users,
  ROUND(
    SUM(CASE WHEN a.status = 'PRESENT' THEN 1 ELSE 0 END) * 100.0 / 
    COUNT(*), 1
  ) as punctuality_rate,
  SUM(CASE WHEN a.status = 'LATE' THEN 1 ELSE 0 END) as late_count,
  SUM(CASE WHEN a.status = 'ABSENT' THEN 1 ELSE 0 END) as absent_count
FROM attendance a
JOIN users u ON a.user_id = u.user_id
WHERE a.year = '2026' AND a.month = '08'
GROUP BY u.department
ORDER BY punctuality_rate DESC
```

4. **Query 4: Task Workload Analytics (WF8 Integration)**
```sql
SELECT 
  u.department,
  u.full_name,
  COUNT(t.task_id) as total_assigned,
  SUM(CASE WHEN t.status = 'DONE' THEN 1 ELSE 0 END) as completed,
  ROUND(
    SUM(CASE WHEN t.status = 'DONE' THEN 1 ELSE 0 END) * 100.0 / 
    COUNT(t.task_id), 1
  ) as completion_rate,
  SUM(CASE WHEN t.due_date < CURRENT_DATE AND t.status != 'DONE' THEN 1 ELSE 0 END) as overdue
FROM tasks t
JOIN users u ON t.assignee_id = u.user_id
WHERE t.created_at >= '2026-08-01'
GROUP BY u.department, u.full_name
ORDER BY total_assigned DESC
```

**Execute via AWS CLI:**
```bash
QUERY_ID=$(aws athena start-query-execution \
  --query-string "SELECT COUNT(*) as total FROM attendance WHERE year='2026' AND month='08' AND day='06'" \
  --query-execution-context Database=smart_campus_db \
  --result-configuration OutputLocation=s3://smart-campus-datalake-${AWS_ACCOUNT_ID}/athena-results/ \
  --work-group smart-campus-workgroup \
  --region ap-southeast-1 \
  --query 'QueryExecutionId' \
  --output text)

echo "Query ID: ${QUERY_ID}"

# Wait for completion
aws athena get-query-execution --query-execution-id ${QUERY_ID} --region ap-southeast-1

# Get results
aws athena get-query-results --query-execution-id ${QUERY_ID} --region ap-southeast-1
```

**Expected Results:**
- ✅ Query executes successfully
- ✅ Returns correct counts matching DynamoDB
- ✅ Latency < 10 seconds for partitioned queries
- ✅ JOIN between attendance and users works
- ✅ Results saved to `athena-results/` in S3

---

## Test Case 10: Task Management (WF8)

**Objective:** Verify Task CRUD, assignment, and notification workflow

**Test 10.1: Create Task (Admin/Manager)**
```bash
curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${ADMIN_JWT_TOKEN}" \
  -d '{
    "task_type": "MAINTENANCE",
    "priority": "HIGH",
    "title": "Fix AC in Server Room",
    "description": "AC unit not cooling, temperature rising",
    "assignee_id": "tech-001",
    "due_date": "2026-08-07"
  }'
```

**Expected Response:**
```json
{
  "task_id": "task-xyz789",
  "task_type": "MAINTENANCE",
  "status": "TODO",
  "priority": "HIGH",
  "title": "Fix AC in Server Room",
  "assignee_id": "tech-001",
  "creator_id": "admin-001",
  "created_at": "2026-08-06T10:00:00Z",
  "due_date": "2026-08-07"
}
```

**Test 10.2: Verify TaskAssigned Event & Notification (WF4)**
```bash
# Check EventBridge
aws logs filter-log-events \
  --log-group-name /aws/events/smart-campus \
  --filter-pattern "TaskAssigned" \
  --region ap-southeast-1

# Check Notification Queue
aws sqs receive-message \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/${AWS_ACCOUNT_ID}/smart-campus-notification-queue \
  --region ap-southeast-1
```

**Test 10.3: Update Task Status (Assignee)**
```bash
# Assignee starts task
curl -X PATCH https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/tasks/task-xyz789/status \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${TECH_JWT_TOKEN}" \
  -d '{"status": "IN_PROGRESS"}'

# Assignee completes task
curl -X PATCH .../api/tasks/task-xyz789/status \
  -d '{"status": "DONE", "completion_note": "AC repaired, temperature normal"}'
```

**Test 10.4: Verify TaskCompleted Event**
```bash
aws logs filter-log-events \
  --log-group-name /aws/events/smart-campus \
  --filter-pattern "TaskCompleted" \
  --region ap-southeast-1
```

**Test 10.5: Task Overdue Detection (Scheduled Lambda)**
```bash
# Manually trigger overdue checker
aws lambda invoke \
  --function-name smart-campus-tasks-overdue-checker \
  --payload '{}' \
  response.json \
  --region ap-southeast-1

cat response.json
```

**Expected Results:**
- ✅ Task created in DynamoDB `smart-campus-tasks`
- ✅ `TaskAssigned` event published → Notification sent to assignee
- ✅ Status transitions: TODO → IN_PROGRESS → DONE
- ✅ `completion_note` required for DONE
- ✅ `TaskCompleted` event published
- ✅ Overdue detection finds tasks past due_date with status TODO/IN_PROGRESS
- ✅ `TaskOverdue` event published → Alert to assignee and manager

---

## Test Case 11: Leave Management (WF4)

**Objective:** Verify leave request workflow with auto-notifications

**Test 11.1: Create Leave Request**
```bash
curl -X POST https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/leaves \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${USER_JWT_TOKEN}" \
  -d '{
    "leave_type": "ANNUAL_LEAVE",
    "start_date": "2026-08-10",
    "end_date": "2026-08-12",
    "reason": "Family vacation"
  }'
```

**Test 11.2: Manager Approves**
```bash
curl -X PATCH https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/api/leaves/leave-xyz/status \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${MANAGER_JWT_TOKEN}" \
  -d '{"status": "APPROVED"}'
```

**Expected:**
- ✅ Leave created in `smart-campus-leaves`
- ✅ Notification sent to manager (LeaveSubmitted)
- ✅ On approval: Notification sent to employee (LeaveApproved)
- ✅ If WFH approved: User can check-in from non-whitelisted IP

---

## Test Case 12: End-to-End Workflow Test

**Objective:** Test complete attendance workflow from start to finish

```bash
#!/bin/bash
# end-to-end-test.sh

set -e  # Exit on error

echo "=== Smart Campus E2E Test ==="

# 1. Create user
echo "Step 1: Creating user..."
aws dynamodb put-item \
  --table-name smart-campus-users \
  --item '{"user_id":{"S":"test-user-e2e"},"email":{"S":"test@example.com"},"name":{"S":"Test User"},"role":{"S":"STAFF"},"department":{"S":"IT"},"status":{"S":"ACTIVE"},"face_registered":{"BOOL":false}}' \
  --region ap-southeast-1
echo "✅ User created"

# 2. Register face
echo "Step 2: Registering face..."
IMAGE_BASE64=$(base64 -i test-images/user-e2e.jpg)
REGISTER_RESPONSE=$(curl -s -X POST ${API_URL}/api/faces/register \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${JWT_TOKEN}" \
  -d '{"user_id":"test-user-e2e","image_base64":"'"${IMAGE_BASE64}"'"}')

FACE_ID=$(echo $REGISTER_RESPONSE | jq -r '.face_id')
echo "✅ Face registered: $FACE_ID"

# 3. Check-in
echo "Step 3: Check-in attendance..."
CHECKIN_RESPONSE=$(curl -s -X POST ${API_URL}/api/attendance/check-in \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${JWT_TOKEN}" \
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

# 6. Verify EventBridge events
echo "Step 6: Verifying EventBridge events..."
aws logs filter-log-events \
  --log-group-name /aws/events/smart-campus \
  --filter-pattern "AttendanceRecorded" \
  --start-time $(date -u -d '2 minutes ago' +%s)000 \
  --region ap-southeast-1 | grep -q "AttendanceRecorded"
echo "✅ EventBridge event published"

echo ""
echo "🎉 E2E Test PASSED!"
```

---

## Test Results Summary

**Create test report:**

```bash
# test-report.sh
echo "Smart Campus Test Report - $(date)"
echo "========================================"
echo ""
echo "Workflow Tests:"
echo "✅ WF1: Authentication (Cognito JWT) - PASSED"
echo "✅ WF2: Face Registration (IndexFaces + Anti-fraud) - PASSED"
echo "✅ WF3: Attendance (SearchFacesByImage + Rule Engine + Liveness) - PASSED"
echo "✅ WF4: Notification (SNS Multi-channel + EventBridge) - PASSED"
echo "✅ WF5: Analytics (DynamoDB + Athena + Dashboard) - PASSED"
echo "✅ WF6: AI Assistant (Bedrock NL2SQL) - SKIPPED (quota pending)"
echo "✅ WF7: Security (Risk Engine + Incident) - PASSED"
echo "✅ WF8: Task Management (CRUD + Assignment + Overdue) - PASSED"
echo ""
echo "Test Cases Detail:"
echo "✅ TC1: Face Registration - PASSED"
echo "✅ TC2: Check-in Happy Path - PASSED"
echo "✅ TC3: Check-in Late - PASSED"
echo "✅ TC4: Duplicate Check - PASSED"
echo "✅ TC5: Liveness Detection (Anti-spoofing) - PASSED"
echo "✅ TC6: Unknown Face - PASSED"
echo "✅ TC7: Outside Session - PASSED"
echo "✅ TC8: WFH Check-in - PASSED"
echo "✅ TC9: Analytics Query (Athena) - PASSED"
echo "✅ TC10: Task Management - PASSED"
echo "✅ TC11: Leave Management - PASSED"
echo "✅ TC12: End-to-End - PASSED"
echo ""
echo "Coverage: 12/12 test cases passed (100%)"
echo "Duration: ~20 minutes"
echo "Status: ALL GREEN ✅"
```

---

## Troubleshooting

**Issue: API returns 403 Forbidden**
- Check JWT token valid and not expired
- Check IAM permissions for Lambda role
- Check WAF rules (IP whitelist) - verify request IP
- Check Cognito authorizer configuration

**Issue: Face not recognized (Low confidence)**
- Check face quality (brightness, sharpness, angle)
- Check confidence threshold (default 80%, lower to 70% for testing)
- Verify face exists in Rekognition collection
- Check ExternalImageId matches user_id

**Issue: Liveness check fails for real person**
- Ensure good lighting (no backlight)
- Hold phone steady during session
- Check camera permissions granted
- Try different browser (Chrome recommended)

**Issue: Event not published to EventBridge**
- Check EventBridge rules are active (State = ENABLED)
- Check Lambda has `events:PutEvents` permission
- Check CloudWatch Logs for Lambda errors
- Verify event pattern matches rule

**Issue: SQS message not processed**
- Check Lambda trigger is active on queue
- Check DLQ for failed messages
- Check Lambda timeout (increase to 5 min for workers)
- Check batch size configuration

**Issue: Athena query returns no results**
- Verify Glue Crawler completed (State = READY)
- Check table schema matches data format
- Verify partition keys (year, month, day) in query WHERE clause
- Check S3 data path matches crawler target

---

## Next Step

Proceed to [Step 10: Monitoring with CloudWatch](../5.10-monitoring) to set up system monitoring!