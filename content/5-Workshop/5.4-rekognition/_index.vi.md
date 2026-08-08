---
title: "Tạo Rekognition Face Collection"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Tổng quan

Amazon Rekognition là dịch vụ AI của AWS cung cấp khả năng nhận diện khuôn mặt (face recognition) và phát hiện liveness (chống gian lận). Trong bước này, bạn sẽ:
- Tạo Rekognition Face Collection
- Test IndexFaces (đăng ký khuôn mặt)
- Test SearchFacesByImage (tìm kiếm khuôn mặt)
- Tích hợp Face Liveness Detection

#### Tại sao cần Rekognition?

**Face Recognition** giải quyết bài toán:
- ✅ Điểm danh tự động không cần thẻ từ
- ✅ Accuracy 99.9% (theo AWS)
- ✅ Nhanh: < 1 giây per face
- ✅ Scale: Hỗ trợ millions of faces

**Face Liveness Detection** ngăn chặn:
- ❌ Ảnh in (printed photo)
- ❌ Video replay attack
- ❌ Mặt nạ 3D
- ❌ Deep fake

#### Bước 1: Tạo Face Collection

Face Collection là nơi lưu trữ face embeddings (vector representations) của khuôn mặt.

**Tạo collection:**
```bash
aws rekognition create-collection \
  --collection-id smart-campus-faces \
  --region ap-southeast-1
```

**Expected output:**
```json
{
    "StatusCode": 200,
    "CollectionArn": "arn:aws:rekognition:ap-southeast-1:123456789012:collection/smart-campus-faces",
    "FaceModelVersion": "7.0"
}
```

**Verify collection:**
```bash
aws rekognition describe-collection \
  --collection-id smart-campus-faces \
  --region ap-southeast-1
```

Expected:
```json
{
    "FaceCount": 0,
    "FaceModelVersion": "7.0",
    "CollectionARN": "arn:aws:rekognition:ap-southeast-1:123456789012:collection/smart-campus-faces",
    "CreationTimestamp": "2026-08-06T10:00:00.000000+07:00"
}
```

#### Bước 2: Index Faces (Đăng ký khuôn mặt)

**Chuẩn bị test image:**

Chụp hoặc tìm một ảnh khuôn mặt rõ ràng:
- Format: JPEG hoặc PNG
- Size: < 5MB
- Resolution: Tối thiểu 640x480
- Face phải rõ, nhìn thẳng
- Ánh sáng đầy đủ

Lưu file: `test-images/user1.jpg`

**Upload ảnh lên S3:**
```bash
# Tạo bucket nếu chưa có
aws s3 mb s3://smart-campus-images-${AWS_ACCOUNT_ID} \
  --region ap-southeast-1

# Upload ảnh
aws s3 cp test-images/user1.jpg \
  s3://smart-campus-images-${AWS_ACCOUNT_ID}/faces/raw/user-001/user1.jpg \
  --region ap-southeast-1
```

**Index face vào collection:**
```bash
aws rekognition index-faces \
  --collection-id smart-campus-faces \
  --image '{"S3Object":{"Bucket":"smart-campus-images-'${AWS_ACCOUNT_ID}'","Name":"faces/raw/user-001/user1.jpg"}}' \
  --external-image-id "user-001" \
  --max-faces 1 \
  --quality-filter "AUTO" \
  --detection-attributes "ALL" \
  --region ap-southeast-1
```

**Expected output:**
```json
{
    "FaceRecords": [
        {
            "Face": {
                "FaceId": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
                "BoundingBox": {
                    "Width": 0.45,
                    "Height": 0.67,
                    "Left": 0.28,
                    "Top": 0.15
                },
                "ImageId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
                "ExternalImageId": "user-001",
                "Confidence": 99.99
            },
            "FaceDetail": {
                "BoundingBox": {...},
                "AgeRange": {
                    "Low": 20,
                    "High": 30
                },
                "Gender": {
                    "Value": "Male",
                    "Confidence": 99.5
                },
                "Quality": {
                    "Brightness": 85.2,
                    "Sharpness": 92.1
                }
            }
        }
    ],
    "FaceModelVersion": "7.0"
}
```

**Lưu Face ID** để test sau: `1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p`

**Verify face đã được index:**
```bash
aws rekognition list-faces \
  --collection-id smart-campus-faces \
  --region ap-southeast-1
```

#### Bước 3: Search Faces (Tìm kiếm khuôn mặt)

**Test với ảnh khác của cùng người:**

Chụp hoặc tìm ảnh thứ 2 của cùng người, lưu: `test-images/user1-test.jpg`

**Upload:**
```bash
aws s3 cp test-images/user1-test.jpg \
  s3://smart-campus-images-${AWS_ACCOUNT_ID}/test/user1-test.jpg
```

**Search face:**
```bash
aws rekognition search-faces-by-image \
  --collection-id smart-campus-faces \
  --image '{"S3Object":{"Bucket":"smart-campus-images-'${AWS_ACCOUNT_ID}'","Name":"test/user1-test.jpg"}}' \
  --max-faces 1 \
  --face-match-threshold 80 \
  --region ap-southeast-1
```

**Expected output (nếu match):**
```json
{
    "SearchedFaceBoundingBox": {...},
    "SearchedFaceConfidence": 99.8,
    "FaceMatches": [
        {
            "Similarity": 98.5,
            "Face": {
                "FaceId": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
                "ExternalImageId": "user-001",
                "Confidence": 99.99
            }
        }
    ],
    "FaceModelVersion": "7.0"
}
```

**Interpretation:**
- `Similarity: 98.5%` → Rất cao, đây là cùng người
- Threshold 80% → Accept nếu >= 80%
- `ExternalImageId: "user-001"` → Đây là user nào

**Test với ảnh người khác (expected: không match):**
```bash
aws s3 cp test-images/user2.jpg \
  s3://smart-campus-images-${AWS_ACCOUNT_ID}/test/user2.jpg

aws rekognition search-faces-by-image \
  --collection-id smart-campus-faces \
  --image '{"S3Object":{"Bucket":"smart-campus-images-'${AWS_ACCOUNT_ID}'","Name":"test/user2.jpg"}}' \
  --max-faces 1 \
  --face-match-threshold 80 \
  --region ap-southeast-1
```

Expected output:
```json
{
    "SearchedFaceBoundingBox": {...},
    "SearchedFaceConfidence": 99.5,
    "FaceMatches": [],
    "FaceModelVersion": "7.0"
}
```

`FaceMatches: []` → Không tìm thấy face nào khớp. Unknown face!

#### Bước 4: Face Liveness Detection

Face Liveness phát hiện xem có phải người thật đang đứng trước camera không.

**Create Liveness Session:**
```bash
aws rekognition create-face-liveness-session \
  --region ap-southeast-1 \
  --settings '{"OutputConfig":{"S3Bucket":"smart-campus-images-'${AWS_ACCOUNT_ID}'","S3KeyPrefix":"liveness-sessions/"}}' \
  --client-request-token "$(uuidgen)"
```

Expected:
```json
{
    "SessionId": "abc123-session-id-xyz789"
}
```

**Note:** Để thực sự test Liveness, cần:
1. Frontend web app với camera
2. AWS Amplify Liveness SDK
3. User làm các động tác theo hướng dẫn (nhìn trái, phải, mở miệng...)
4. SDK gửi video frames lên Rekognition

Trong workshop này, chúng ta sẽ giả lập bằng cách gọi API với ảnh tĩnh (sẽ bị reject).

**Get Liveness Session Results (sau khi user hoàn thành):**
```bash
aws rekognition get-face-liveness-session-results \
  --session-id abc123-session-id-xyz789 \
  --region ap-southeast-1
```

Expected (nếu là người thật):
```json
{
    "SessionId": "abc123-session-id-xyz789",
    "Status": "SUCCEEDED",
    "Confidence": 95.8,
    "ReferenceImage": {...},
    "AuditImages": [...]
}
```

Nếu `Confidence >= 80%` → Accept (người thật)
Nếu `Confidence < 80%` → Reject (ảnh in, video, mặt nạ)

#### Bước 5: Integration với Python Code

**File: `rekognition_service.py`**

```python
import boto3
from botocore.exceptions import ClientError

rekognition = boto3.client('rekognition', region_name='ap-southeast-1')
COLLECTION_ID = 'smart-campus-faces'

def index_face(image_bytes: bytes, external_image_id: str) -> dict:
    """Đăng ký khuôn mặt vào collection."""
    try:
        response = rekognition.index_faces(
            CollectionId=COLLECTION_ID,
            Image={'Bytes': image_bytes},
            ExternalImageId=external_image_id,
            MaxFaces=1,
            QualityFilter='AUTO',
            DetectionAttributes=['DEFAULT']
        )
        
        if not response['FaceRecords']:
            raise ValueError("No face detected")
        
        face = response['FaceRecords'][0]['Face']
        return {
            'face_id': face['FaceId'],
            'confidence': face['Confidence'],
            'bounding_box': face['BoundingBox']
        }
    except ClientError as e:
        if e.response['Error']['Code'] == 'InvalidParameterException':
            raise ValueError("No face detected in image")
        raise

def search_face_by_image(image_bytes: bytes, threshold: float = 80.0) -> dict:
    """Tìm kiếm khuôn mặt trong collection."""
    try:
        response = rekognition.search_faces_by_image(
            CollectionId=COLLECTION_ID,
            Image={'Bytes': image_bytes},
            MaxFaces=1,
            FaceMatchThreshold=threshold
        )
        
        if not response['FaceMatches']:
            raise ValueError("No matching face found")
        
        match = response['FaceMatches'][0]
        return {
            'face_id': match['Face']['FaceId'],
            'user_id': match['Face']['ExternalImageId'],
            'similarity': match['Similarity'],
            'confidence': match['Face']['Confidence']
        }
    except ClientError as e:
        if e.response['Error']['Code'] == 'InvalidParameterException':
            raise ValueError("No face detected in image")
        raise

def create_liveness_session() -> str:
    """Tạo liveness session."""
    response = rekognition.create_face_liveness_session(
        Settings={
            'OutputConfig': {
                'S3Bucket': f'smart-campus-images-{AWS_ACCOUNT_ID}',
                'S3KeyPrefix': 'liveness-sessions/'
            }
        }
    )
    return response['SessionId']

def get_liveness_result(session_id: str) -> dict:
    """Lấy kết quả liveness check."""
    response = rekognition.get_face_liveness_session_results(
        SessionId=session_id
    )
    return {
        'status': response['Status'],
        'confidence': response.get('Confidence', 0),
        'is_live': response['Status'] == 'SUCCEEDED' and response.get('Confidence', 0) >= 80
    }
```

**Test code:**
```python
# Test index face
with open('test-images/user1.jpg', 'rb') as f:
    image_bytes = f.read()
    result = index_face(image_bytes, 'user-001')
    print(f"Face indexed: {result['face_id']}, confidence: {result['confidence']}")

# Test search face
with open('test-images/user1-test.jpg', 'rb') as f:
    image_bytes = f.read()
    result = search_face_by_image(image_bytes)
    print(f"Found user: {result['user_id']}, similarity: {result['similarity']}")
```

#### Xử lý các trường hợp đặc biệt

**1. No face detected:**
```python
try:
    result = index_face(image_bytes, user_id)
except ValueError as e:
    if "No face detected" in str(e):
        return {"error": "Không phát hiện khuôn mặt trong ảnh. Vui lòng chụp lại."}
```

**2. Multiple faces:**
```python
# Rekognition trả về error nếu MaxFaces=1 nhưng có nhiều face
# Solution: Set MaxFaces=1 và chỉ lấy face có confidence cao nhất
```

**3. Low quality image:**
```python
# QualityFilter='AUTO' sẽ reject nếu:
# - Too blurry
# - Too dark
# - Face too small
# Client cần chụp lại với chất lượng tốt hơn
```

**4. Face already registered:**
```python
# Search trước khi index
existing = search_face_by_image(image_bytes, threshold=95)
if existing:
    if existing['user_id'] == new_user_id:
        return {"error": "Bạn đã đăng ký khuôn mặt này rồi"}
    else:
        return {"error": "Khuôn mặt này đã được đăng ký cho user khác"}
```

#### Monitoring và Troubleshooting
**Kiểm tra thống kê collection:**
```bash
aws rekognition describe-collection \
  --collection-id smart-campus-faces \
  --region ap-southeast-1 \
  --query '[FaceCount,CollectionARN]'
```

**Liệt kê tất cả khuôn mặt:**
```bash
aws rekognition list-faces \
  --collection-id smart-campus-faces \
  --max-results 100 \
  --region ap-southeast-1
```

**Xóa một khuôn mặt:**
```bash
aws rekognition delete-faces \
  --collection-id smart-campus-faces \
  --face-ids "face-id-to-delete" \
  --region ap-southeast-1
```

**Các lỗi thường gặp:**

| Lỗi | Nguyên nhân | Cách xử lý |
|-------|-------|----------|
| `InvalidParameterException` | Không có khuôn mặt trong ảnh | Chụp lại với khuôn mặt rõ hơn |
| `ResourceNotFoundException` | Collection không tồn tại | Tạo collection trước |
| `InvalidImageFormatException` | Định dạng không được hỗ trợ | Chỉ dùng JPEG/PNG |
| `ImageTooLargeException` | Ảnh > 5MB | Resize ảnh trước khi upload |
| `InvalidS3ObjectException` | S3 key không tồn tại | Kiểm tra đường dẫn S3 |

#### Ước tính chi phí

**Bảng giá Rekognition (ap-southeast-1):**
- IndexFaces: $1.00 cho mỗi 1.000 ảnh
- SearchFacesByImage: $1.00 cho mỗi 1.000 lần tìm kiếm
- Face Liveness: $0.015 cho mỗi session

**Ước tính cho workshop (1.000 khuôn mặt, 5.000 lần tìm kiếm):**
- IndexFaces: $1.00
- SearchFaces: $5.00
- Liveness (100 sessions): $1.50
- **Tổng: $7.50**

**Ước tính production (5.000 người dùng, 10.000 lượt điểm danh/ngày):**
- IndexFaces: 5.000 khuôn mặt = $5 (một lần)
- SearchFaces: 10.000/ngày × 30 ngày = 300.000 = $300/tháng
- Liveness: 10.000/ngày × 30 = 300.000 sessions = $4.500/tháng (rất cao!)

**Tối ưu chi phí:**
- Cache kết quả nhận diện trong 5 phút (tránh tìm kiếm trùng lặp)
- Chỉ dùng Liveness cho lần đăng ký đầu tiên
- Cân nhắc phương án thay thế: chỉ dùng Face Recognition (không dùng Liveness)

#### Xác minh thiết lập

Checklist:
- [ ] Collection đã được tạo và ở trạng thái ACTIVE
- [ ] Ít nhất 1 khuôn mặt đã được index
- [ ] SearchFacesByImage trả về kết quả đúng
- [ ] Khuôn mặt chưa đăng ký trả về FaceMatches rỗng
- [ ] Tích hợp code Python hoạt động
- [ ] Xử lý lỗi đầy đủ

#### Bước tiếp theo

Hãy chuyển sang [Bước 5: Deploy Lambda Functions](../5.5-lambda) để xây dựng backend API!
