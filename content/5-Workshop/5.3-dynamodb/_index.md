---
title: "Set Up DynamoDB Tables"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Overview

In this step, you will create 3 DynamoDB tables for the Smart Campus system:
1. **smart-campus-users**: Stores user information
2. **smart-campus-faces**: Stores face metadata (faceId mapping)
3. **smart-campus-attendance**: Stores attendance records

#### Why choose DynamoDB?

**Advantages:**
- ✅ **Serverless**: No need to manage database servers
- ✅ **Auto scaling**: Automatically scales based on traffic
- ✅ **Low latency**: Response time under 10 milliseconds
- ✅ **High availability**: 99.99% SLA, automatic multi-AZ replication
- ✅ **Pay-per-use**: On-demand pricing, only pay when you use it
- ✅ **Strong integration**: Tight integration with Lambda and EventBridge

**When NOT to use it:**
- ❌ Complex queries with JOINs (→ use RDS/Aurora)
- ❌ OLAP analytics (→ use Redshift/Athena)
- ❌ Full-text search (→ use OpenSearch)

#### Table 1: smart-campus-users

**Purpose:** Store user information (administrators, managers, staff, students)

**Data structure:**
```json
{
  "user_id": "uuid",           // Partition Key
  "email": "string",           // GSI
  "name": "string",
  "role": "string",            // ADMIN | MANAGER | STAFF | STUDENT
  "department": "string",
  "status": "string",          // ACTIVE | INACTIVE | SUSPENDED
  "face_registered": "boolean",
  "phone": "string",
  "created_at": "string",      // ISO-8601
  "updated_at": "string"
}
```

**Indexes:**
- **Primary Key**: `user_id` (Partition Key)
- **GSI**: `email-index` (email → user_id) - For logging in with email

**Create table:**
```bash
aws dynamodb create-table \
  --table-name smart-campus-users \
  --attribute-definitions \
    AttributeName=user_id,AttributeType=S \
    AttributeName=email,AttributeType=S \
  --key-schema \
    AttributeName=user_id,KeyType=HASH \
  --global-secondary-indexes \
    "[
      {
        \"IndexName\": \"email-index\",
        \"KeySchema\": [{\"AttributeName\":\"email\",\"KeyType\":\"HASH\"}],
        \"Projection\": {\"ProjectionType\":\"ALL\"},
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\":5,\"WriteCapacityUnits\":5}
      }
    ]" \
  --billing-mode PAY_PER_REQUEST \
  --region ap-southeast-1 \
  --tags Key=Project,Value=SmartCampus
```

**Verify:**
```bash
aws dynamodb describe-table \
  --table-name smart-campus-users \
  --region ap-southeast-1 \
  --query 'Table.[TableName,TableStatus,ItemCount]'
```

Expected result:
```
[
    "smart-campus-users",
    "ACTIVE",
    0
]
```

**Add a test user:**
```bash
aws dynamodb put-item \
  --table-name smart-campus-users \
  --item '{
    "user_id": {"S": "user-001"},
    "email": {"S": "admin@smartcampus.edu.vn"},
    "name": {"S": "Admin User"},
    "role": {"S": "ADMIN"},
    "department": {"S": "IT"},
    "status": {"S": "ACTIVE"},
    "face_registered": {"BOOL": false},
    "created_at": {"S": "2026-08-06T10:00:00Z"}
  }' \
  --region ap-southeast-1
```

**Query by email (test GSI):**
```bash
aws dynamodb query \
  --table-name smart-campus-users \
  --index-name email-index \
  --key-condition-expression "email = :email" \
  --expression-attribute-values '{":email":{"S":"admin@smartcampus.edu.vn"}}' \
  --region ap-southeast-1
```

#### Table 2: smart-campus-faces

**Purpose:** Store face metadata (mapping Rekognition faceId → userId)

**Data structure:**
```json
{
  "face_id": "string",         // Partition Key (Rekognition Face ID)
  "user_id": "string",         // GSI
  "s3_key": "string",          // S3 path to original image
  "confidence": "number",      // 0-100
  "bounding_box": {
    "width": "number",
    "height": "number",
    "left": "number",
    "top": "number"
  },
  "created_at": "string"
}
```

**Indexes:**
- **Primary Key**: `face_id` (Partition Key)
- **GSI**: `user_id-index` (user_id → face_id) - To query faces by user

**Create table:**
```bash
aws dynamodb create-table \
  --table-name smart-campus-faces \
  --attribute-definitions \
    AttributeName=face_id,AttributeType=S \
    AttributeName=user_id,AttributeType=S \
  --key-schema \
    AttributeName=face_id,KeyType=HASH \
  --global-secondary-indexes \
    "[
      {
        \"IndexName\": \"user_id-index\",
        \"KeySchema\": [{\"AttributeName\":\"user_id\",\"KeyType\":\"HASH\"}],
        \"Projection\": {\"ProjectionType\":\"ALL\"},
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\":5,\"WriteCapacityUnits\":5}
      }
    ]" \
  --billing-mode PAY_PER_REQUEST \
  --region ap-southeast-1 \
  --tags Key=Project,Value=SmartCampus
```

**Verify:**
```bash
aws dynamodb describe-table \
  --table-name smart-campus-faces \
  --region ap-southeast-1 \
  --query 'Table.TableName'
```

#### Table 3: smart-campus-attendance

**Purpose:** Store attendance records

**Data structure:**
```json
{
  "attendance_id": "string",   // Partition Key (UUID)
  "user_id": "string",         // GSI
  "face_id": "string",
  "camera_id": "string",
  "room_id": "string",
  "session_type": "string",    // MORNING | AFTERNOON | EVENING
  "status": "string",          // PRESENT | LATE | ABSENT | REJECTED
  "confidence": "number",      // Face similarity score 0-100
  "timestamp": "string",       // ISO-8601 with timezone
  "date": "string"             // YYYY-MM-DD (GSI sort key)
}
```

**Indexes:**
- **Primary Key**: `attendance_id` (Partition Key)
- **GSI1**: `user_id-index` (user_id, date) - Query attendance by user
- **GSI2**: `date-index` (date, timestamp) - Query all attendance records by date

**Create table:**
```bash
aws dynamodb create-table \
  --table-name smart-campus-attendance \
  --attribute-definitions \
    AttributeName=attendance_id,AttributeType=S \
    AttributeName=user_id,AttributeType=S \
    AttributeName=date,AttributeType=S \
    AttributeName=timestamp,AttributeType=S \
  --key-schema \
    AttributeName=attendance_id,KeyType=HASH \
  --global-secondary-indexes \
    "[
      {
        \"IndexName\": \"user_id-index\",
        \"KeySchema\": [
          {\"AttributeName\":\"user_id\",\"KeyType\":\"HASH\"},
          {\"AttributeName\":\"date\",\"KeyType\":\"RANGE\"}
        ],
        \"Projection\": {\"ProjectionType\":\"ALL\"},
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\":5,\"WriteCapacityUnits\":5}
      },
      {
        \"IndexName\": \"date-index\",
        \"KeySchema\": [
          {\"AttributeName\":\"date\",\"KeyType\":\"HASH\"},
          {\"AttributeName\":\"timestamp\",\"KeyType\":\"RANGE\"}
        ],
        \"Projection\": {\"ProjectionType\":\"ALL\"},
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\":5,\"WriteCapacityUnits\":5}
      }
    ]" \
  --billing-mode PAY_PER_REQUEST \
  --region ap-southeast-1 \
  --tags Key=Project,Value=SmartCampus
```

**Verify:**
```bash
aws dynamodb describe-table \
  --table-name smart-campus-attendance \
  --region ap-southeast-1 \
  --query 'Table.[TableName,GlobalSecondaryIndexes[*].IndexName]'
```

Expected result:
```
[
    "smart-campus-attendance",
    [
        "user_id-index",
        "date-index"
    ]
]
```

**Add a test record:**
```bash
aws dynamodb put-item \
  --table-name smart-campus-attendance \
  --item '{
    "attendance_id": {"S": "att-001"},
    "user_id": {"S": "user-001"},
    "face_id": {"S": "face-12345"},
    "camera_id": {"S": "camera-01"},
    "room_id": {"S": "A101"},
    "session_type": {"S": "MORNING"},
    "status": {"S": "PRESENT"},
    "confidence": {"N": "98.5"},
    "timestamp": {"S": "2026-08-06T07:55:00+07:00"},
    "date": {"S": "2026-08-06"}
  }' \
  --region ap-southeast-1
```

**Query attendance by date:**
```bash
aws dynamodb query \
  --table-name smart-campus-attendance \
  --index-name date-index \
  --key-condition-expression "#d = :date" \
  --expression-attribute-names '{"#d":"date"}' \
  --expression-attribute-values '{":date":{"S":"2026-08-06"}}' \
  --region ap-southeast-1
```

#### Enable Point-in-Time Recovery (PITR)

To protect data, enable PITR for all tables:

```bash
# Users table
aws dynamodb update-continuous-backups \
  --table-name smart-campus-users \
  --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true \
  --region ap-southeast-1

# Faces table
aws dynamodb update-continuous-backups \
  --table-name smart-campus-faces \
  --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true \
  --region ap-southeast-1

# Attendance table
aws dynamodb update-continuous-backups \
  --table-name smart-campus-attendance \
  --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true \
  --region ap-southeast-1
```

PITR allows you to restore a table to any point in time within the past 35 days.

#### Query Patterns & Performance

**Common queries and optimization methods:**

**1. Get user by email (login):**
```python
# Use GSI email-index
response = dynamodb.query(
    TableName='smart-campus-users',
    IndexName='email-index',
    KeyConditionExpression='email = :email',
    ExpressionAttributeValues={':email': 'user@example.com'}
)
```
**Performance:** <10ms with GSI

**2. Get a user’s attendance history:**
```python
# Use GSI user_id-index
response = dynamodb.query(
    TableName='smart-campus-attendance',
    IndexName='user_id-index',
    KeyConditionExpression='user_id = :uid AND #d BETWEEN :from AND :to',
    ExpressionAttributeNames={'#d': 'date'},
    ExpressionAttributeValues={
        ':uid': 'user-001',
        ':from': '2026-08-01',
        ':to': '2026-08-31'
    }
)
```
**Performance:** <50ms for 1 month of data

**3. Get all attendance records for a day:**
```python
# Use GSI date-index
response = dynamodb.query(
    TableName='smart-campus-attendance',
    IndexName='date-index',
    KeyConditionExpression='#d = :date',
    ExpressionAttributeNames={'#d': 'date'},
    ExpressionAttributeValues={':date': '2026-08-06'}
)
```
**Performance:** <100ms for 500 records

**4. Check for duplicates (has the user already checked in for this session?):**
```python
# Query by user_id and date, then filter by session_type in application code
response = dynamodb.query(
    TableName='smart-campus-attendance',
    IndexName='user_id-index',
    KeyConditionExpression='user_id = :uid AND #d = :date',
    ExpressionAttributeNames={'#d': 'date'},
    ExpressionAttributeValues={
        ':uid': 'user-001',
        ':date': '2026-08-06'
    }
)
# Then filter: [r for r in records if r['session_type'] == 'MORNING']
```
**Performance:** <20ms

#### Cost Estimation

**On-Demand pricing (ap-southeast-1):**
- Read: $0.25 per million requests
- Write: $1.25 per million requests
- Storage: $0.25 per GB-month

**Monthly cost estimate (5000 users):**

| Table | Reads/month | Writes/month | Storage | Cost |
|-------|-------------|--------------|---------|------|
| users | 1M | 10K | 5MB | $0.25 |
| faces | 500K | 5K | 1MB | $0.13 |
| attendance | 5M | 1M | 500MB | $2.50 |
| **TOTAL** | | | | **$2.88** |

**Cost optimization tips:**
- Use Provisioned Capacity if traffic is predictable (save ~30%)
- Enable TTL to auto-delete old records
- Archive old attendance to S3 after 6 months

#### Troubleshooting

**Error: Table already exists**
```
An error occurred (ResourceInUseException) when calling the CreateTable operation
```
**Solution:** Table name conflict, delete the old table or use a different name

**Error: Throughput exceeded**
```
ProvisionedThroughputExceededException
```
**Solution:** You are using On-Demand mode, so this should not happen. If it does, check whether there is any virus scan tool scanning DynamoDB.

**Error: Failed to create GSI**
```
GlobalSecondaryIndexes format incorrect
```
**Solution:** Check the JSON format in --global-secondary-indexes, and make sure quotes are escaped correctly

#### Verify Setup

Run the following script to verify that all tables have been created successfully:

```bash
#!/bin/bash
TABLES=("smart-campus-users" "smart-campus-faces" "smart-campus-attendance")

for table in "${TABLES[@]}"; do
  echo "Checking $table..."
  aws dynamodb describe-table \
    --table-name $table \
    --region ap-southeast-1 \
    --query 'Table.[TableName,TableStatus,ItemCount,GlobalSecondaryIndexes[*].IndexName]' \
    || echo "❌ $table NOT FOUND"
done

echo "✅ All tables verified!"
```

Expected output:
```
Checking smart-campus-users...
["smart-campus-users", "ACTIVE", 1, ["email-index"]]
Checking smart-campus-faces...
["smart-campus-faces", "ACTIVE", 0, ["user_id-index"]]
Checking smart-campus-attendance...
["smart-campus-attendance", "ACTIVE", 1, ["user_id-index", "date-index"]]
✅ All tables verified!
```

#### Next Step

Proceed to [Step 4: Create a Rekognition Face Collection](../5.4-rekognition) to set up AI face recognition!

---

**Best Practices:**
- ✅ Use On-Demand billing mode for the workshop (unpredictable traffic)
- ✅ Enable PITR for all production tables
- ✅ Tag resources with `Project: SmartCampus` for easier cleanup
- ✅ Design GSIs carefully based on access patterns
- ✅ Avoid Scan operations (slow and expensive)
- ✅ Use consistent naming: `smart-campus-*`