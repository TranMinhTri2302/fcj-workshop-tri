---
title: "Create Rekognition Face Collection"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Overview

Amazon Rekognition is an AWS AI service providing face recognition and liveness detection (anti-spoofing). In this step, you will:
- Create Rekognition Face Collection
- Test IndexFaces (face enrollment)
- Test SearchFacesByImage (face search)
- Integrate Face Liveness Detection

## Why Rekognition?

**Face Recognition** solves:
- ✅ Automated attendance without RFID cards
- ✅ 99.9% accuracy (per AWS)
- ✅ Fast: < 1 second per face
- ✅ Scale: Supports millions of faces

**Face Liveness Detection** prevents:
- ❌ Printed photos
- ❌ Video replay attacks
- ❌ 3D masks
- ❌ Deep fakes

## Step 1: Create Face Collection

Face Collection stores face embeddings (vector representations).

**Create collection:**
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

## Step 2: Index Faces (Enrollment)

**Prepare test image:**

Take or find a clear face photo:
- Format: JPEG or PNG
- Size: < 5MB
- Resolution: Minimum 640x480
- Face must be clear, looking straight
- Good lighting

Save file: `test-images/user1.jpg`

**Upload image to S3:**
```bash
# Create bucket if not exists
aws s3 mb s3://smart-campus-images-${AWS_ACCOUNT_ID} \
  --region ap-southeast-1

# Upload image
aws s3 cp test-images/user1.jpg \
  s3://smart-campus-images-${AWS_ACCOUNT_ID}/faces/raw/user-001/user1.jpg \
  --region ap-southeast-1
```

**Index face into collection:**
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

**Save Face ID** for later testing: `1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p`

**Verify face indexed:**
```bash
aws rekognition list-faces \
  --collection-id smart-campus-faces \
  --region ap-southeast-1
```

## Step 3: Search Faces (Recognition)

**Test with different photo of same person:**

Take or find a second photo of the same person, save: `test-images/user1-test.jpg`

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

**Expected output (if match):**
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
- `Similarity: 98.5%` → Very high, same person
- Threshold 80% → Accept if >= 80%
- `ExternalImageId: "user-001"` → This is which user

**Test with different person (expected: no match):**
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

`FaceMatches: []` → No face in collection matches above 80% threshold. Unknown face!

## Step 4: Face Liveness Detection

Face Liveness detects if a real person is in front of the camera.

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

**Note:** To truly test Liveness, need:
1. Frontend web app with camera
2. AWS Amplify Liveness SDK
3. User performs actions (look left, right, open mouth...)
4. SDK sends video frames to Rekognition

In this workshop, we'll simulate by calling API with static image (will be rejected).

**Get Liveness Session Results (after user completes):**
```bash
aws rekognition get-face-liveness-session-results \
  --session-id abc123-session-id-xyz789 \
  --region ap-southeast-1
```

Expected (if real person):
```json
{
    "SessionId": "abc123-session-id-xyz789",
    "Status": "SUCCEEDED",
    "Confidence": 95.8,
    "ReferenceImage": {...},
    "AuditImages": [...]
}
```

If `Confidence >= 80%` → Accept (real person)
If `Confidence < 80%` → Reject (printed photo, video, mask)

## Step 5: Python Code Integration

**File: `rekognition_service.py`**

```python
import boto3
from botocore.exceptions import ClientError

rekognition = boto3.client('rekognition', region_name='ap-southeast-1')
COLLECTION_ID = 'smart-campus-faces'

def index_face(image_bytes: bytes, external_image_id: str) -> dict:
    """Enroll face into collection."""
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
    """Search for matching face in collection."""
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
    """Create liveness session."""
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
    """Get liveness check result."""
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

## Handling Edge Cases

**1. No face detected:**
```python
try:
    result = index_face(image_bytes, user_id)
except ValueError as e:
    if "No face detected" in str(e):
        return {"error": "No face detected in image. Please retake."}
```

**2. Multiple faces:**
```python
# Rekognition returns error if MaxFaces=1 but multiple faces
# Solution: Set MaxFaces=1 and only take face with highest confidence
```

**3. Low quality image:**
```python
# QualityFilter='AUTO' rejects if:
# - Too blurry
# - Too dark
# - Face too small
# Client needs to retake with better quality
```

**4. Face already registered:**
```python
# Search before indexing
existing = search_face_by_image(image_bytes, threshold=95)
if existing:
    if existing['user_id'] == new_user_id:
        return {"error": "This face is already registered for you"}
    else:
        return {"error": "This face is already registered for another user"}
```

## Monitoring and Troubleshooting

**Check collection stats:**
```bash
aws rekognition describe-collection \
  --collection-id smart-campus-faces \
  --region ap-southeast-1 \
  --query '[FaceCount,CollectionARN]'
```

**List all faces:**
```bash
aws rekognition list-faces \
  --collection-id smart-campus-faces \
  --max-results 100 \
  --region ap-southeast-1
```

**Delete a face:**
```bash
aws rekognition delete-faces \
  --collection-id smart-campus-faces \
  --face-ids "face-id-to-delete" \
  --region ap-southeast-1
```

**Common errors:**

| Error | Cause | Solution |
|-------|-------|----------|
| `InvalidParameterException` | No face in image | Retake with clearer face |
| `ResourceNotFoundException` | Collection doesn't exist | Create collection first |
| `InvalidImageFormatException` | Unsupported format | Use JPEG/PNG only |
| `ImageTooLargeException` | Image > 5MB | Resize before upload |
| `InvalidS3ObjectException` | S3 key doesn't exist | Check S3 path |

## Cost Estimation

**Rekognition pricing (ap-southeast-1):**
- IndexFaces: $1.00 per 1,000 images
- SearchFacesByImage: $1.00 per 1,000 searches
- Face Liveness: $0.015 per session

**Workshop estimate (1000 faces, 5000 searches):**
- IndexFaces: $1.00
- SearchFaces: $5.00
- Liveness (100 sessions): $1.50
- **Total: $7.50**

**Production estimate (5000 users, 10K attendance/day):**
- IndexFaces: 5K faces = $5 (one-time)
- SearchFaces: 10K/day × 30 days = 300K = $300/month
- Liveness: 10K/day × 30 = 300K sessions = $4,500/month (expensive!)

**Cost optimization:**
- Cache recognition results for 5 minutes (avoid duplicate searches)
- Only use Liveness for first-time registration
- Consider alternative: Face Recognition only (no Liveness)

## Verify Setup

Checklist:
- [ ] Collection created and ACTIVE
- [ ] At least 1 face indexed
- [ ] SearchFacesByImage returns correct result
- [ ] Unknown face returns empty FaceMatches
- [ ] Python code integration works
- [ ] Error handling complete

## Next Step

Proceed to [Step 5: Deploy Lambda Functions](../5.5-lambda) to build the backend API!