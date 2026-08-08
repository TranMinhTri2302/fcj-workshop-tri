---
title: "Thiết lập DynamoDB Tables"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Tổng quan

Trong bước này, bạn sẽ tạo 3 DynamoDB tables cho hệ thống Smart Campus:
1. **smart-campus-users**: Lưu thông tin người dùng
2. **smart-campus-faces**: Lưu metadata khuôn mặt (faceId mapping)
3. **smart-campus-attendance**: Lưu records điểm danh

#### Tại sao chọn DynamoDB?

**Ưu điểm:**
- ✅ **Serverless**: Không cần quản lý database server
- ✅ **Tự động mở rộng**: Scale tự động theo traffic
- ✅ **Độ trễ thấp**: Thời gian phản hồi dưới 10 millisecond
- ✅ **Tính sẵn sàng cao**: SLA 99.99%, sao chép multi-AZ tự động
- ✅ **Trả theo sử dụng**: Giá theo yêu cầu, chỉ trả tiền khi dùng
- ✅ **Tích hợp tốt**: Kết nối chặt chẽ với Lambda, EventBridge

**Khi KHÔNG nên dùng:**
- ❌ Truy vấn phức tạp với JOIN (→ sử dụng RDS/Aurora)
- ❌ Phân tích OLAP (→ sử dụng Redshift/Athena)
- ❌ Tìm kiếm full-text (→ sử dụng OpenSearch)

#### Bảng 1: smart-campus-users

**Mục đích:** Lưu trữ thông tin người dùng (quản trị viên, quản lý, nhân viên, sinh viên)

**Cấu trúc dữ liệu:**
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

**Chỉ mục:**
- **Primary Key**: `user_id` (Partition Key)
- **GSI**: `email-index` (email → user_id) - Để đăng nhập bằng email

**Tạo bảng:**
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

**Xác minh:**
```bash
aws dynamodb describe-table \
  --table-name smart-campus-users \
  --region ap-southeast-1 \
  --query 'Table.[TableName,TableStatus,ItemCount]'
```

Kết quả mong đợi:
```
[
    "smart-campus-users",
    "ACTIVE",
    0
]
```

**Thêm user test:**
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

**Truy vấn theo email (test GSI):**
```bash
aws dynamodb query \
  --table-name smart-campus-users \
  --index-name email-index \
  --key-condition-expression "email = :email" \
  --expression-attribute-values '{":email":{"S":"admin@smartcampus.edu.vn"}}' \
  --region ap-southeast-1
```

#### Bảng 2: smart-campus-faces

**Mục đích:** Lưu metadata khuôn mặt (mapping Rekognition faceId → userId)

**Cấu trúc dữ liệu:**
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

**Chỉ mục:**
- **Primary Key**: `face_id` (Partition Key)
- **GSI**: `user_id-index` (user_id → face_id) - Để truy vấn face theo user

**Tạo bảng:**
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

**Xác minh:**
```bash
aws dynamodb describe-table \
  --table-name smart-campus-faces \
  --region ap-southeast-1 \
  --query 'Table.TableName'
```

#### Bảng 3: smart-campus-attendance

**Mục đích:** Lưu bản ghi điểm danh

**Cấu trúc dữ liệu:**
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
  "timestamp": "string",       // ISO-8601 với timezone
  "date": "string"             // YYYY-MM-DD (GSI sort key)
}
```

**Chỉ mục:**
- **Primary Key**: `attendance_id` (Partition Key)
- **GSI1**: `user_id-index` (user_id, date) - Truy vấn điểm danh theo user
- **GSI2**: `date-index` (date, timestamp) - Truy vấn tất cả điểm danh theo ngày

**Tạo bảng:**
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

**Xác minh:**
```bash
aws dynamodb describe-table \
  --table-name smart-campus-attendance \
  --region ap-southeast-1 \
  --query 'Table.[TableName,GlobalSecondaryIndexes[*].IndexName]'
```

Kết quả mong đợi:
```
[
    "smart-campus-attendance",
    [
        "user_id-index",
        "date-index"
    ]
]
```

**Thêm bản ghi test:**
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

**Truy vấn điểm danh theo ngày:**
```bash
aws dynamodb query \
  --table-name smart-campus-attendance \
  --index-name date-index \
  --key-condition-expression "#d = :date" \
  --expression-attribute-names '{"#d":"date"}' \
  --expression-attribute-values '{":date":{"S":"2026-08-06"}}' \
  --region ap-southeast-1
```

#### Bật Point-in-Time Recovery (PITR)

Để bảo vệ dữ liệu, enable PITR cho tất cả tables:

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

PITR cho phép restore table về bất kỳ thời điểm nào trong 35 ngày qua.

#### Mẫu Truy vấn & Hiệu năng

**Truy vấn phổ biến và cách tối ưu:**

**1. Lấy user theo email (đăng nhập):**
```python
# Use GSI email-index
response = dynamodb.query(
    TableName='smart-campus-users',
    IndexName='email-index',
    KeyConditionExpression='email = :email',
    ExpressionAttributeValues={':email': 'user@example.com'}
)
```
**Performance:** <10ms với GSI

**2. Lấy lịch sử điểm danh của user:**
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
**Performance:** <50ms cho 1 tháng data

**3. Lấy tất cả điểm danh trong một ngày:**
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
**Performance:** <100ms cho 500 records

**4. Kiểm tra trùng lặp (đã điểm danh session này chưa):**
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

#### Ước tính Chi phí

**Giá On-Demand (ap-southeast-1):**
- Read: $0.25 per million requests
- Write: $1.25 per million requests
- Storage: $0.25 per GB-month

**Ước tính chi phí hàng tháng (5000 users):**

| Table | Reads/month | Writes/month | Storage | Cost |
|-------|-------------|--------------|---------|------|
| users | 1M | 10K | 5MB | $0.25 |
| faces | 500K | 5K | 1MB | $0.13 |
| attendance | 5M | 1M | 500MB | $2.50 |
| **TOTAL** | | | | **$2.88** |

**Mẹo tối ưu chi phí:**
- Use Provisioned Capacity nếu traffic predictable (save ~30%)
- Enable TTL để auto-delete old records
- Archive old attendance to S3 after 6 months

#### Xử lý Sự cố

**Lỗi: Table đã tồn tại**
```
An error occurred (ResourceInUseException) when calling the CreateTable operation
```
**Solution:** Table name conflict, xóa table cũ hoặc dùng tên khác

**Lỗi: Vượt quá throughput**
```
ProvisionedThroughputExceededException
```
**Solution:** Đang dùng On-Demand mode nên không nên xảy ra. Nếu xảy ra, check có virus scan tool đang scan DynamoDB không.

**Lỗi: Tạo GSI thất bại**
```
GlobalSecondaryIndexes format incorrect
```
**Solution:** Check JSON format trong --global-secondary-indexes, đảm bảo escape quotes đúng

#### Xác minh Thiết lập

Chạy script sau để xác minh tất cả bảng đã tạo thành công:

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

#### Bước tiếp theo

Hãy chuyển sang [Bước 4: Tạo Rekognition Face Collection](../5.4-rekognition) để setup AI face recognition!

---

**Thực hành Tốt nhất:**
- ✅ Sử dụng On-Demand billing mode cho workshop (unpredictable traffic)
- ✅ Enable PITR cho tất cả production tables
- ✅ Tag resources với `Project: SmartCampus` để dễ cleanup
- ✅ Design GSI cẩn thận dựa trên access patterns
- ✅ Avoid Scan operations (slow và expensive)
- ✅ Use consistent naming: `smart-campus-*`
