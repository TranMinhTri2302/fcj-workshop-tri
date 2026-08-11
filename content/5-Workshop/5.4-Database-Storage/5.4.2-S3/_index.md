---
title : "Create S3 Buckets"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

#### 5.4.2. Initialize Amazon S3 Buckets
Our system needs 2 separate buckets with different security policies:

**Bucket 1: S3 Frontend (For static website)**
- **Name:** `smart-campus-frontend-{your-id}`.
- Function: Store the Frontend source code (built to the `dist` folder).


> Instructions for creating this bucket in full (including configuring Public Access and Static Website Hosting) will be provided in section **5.8.1**. 

**Bucket 2: S3 Images (For storing face images)**
- **Name:** `smart-campus-images-{your-id}`.
- Function: Store face images used for registration and AI recognition (Amazon Rekognition).
- Configuration: 
  - Enable **Block all public access**.
  - Enable **SSE-S3** encryption to ensure biometric data safety.
  - Disable **Bucket Versioning**.
  - After creating, go inside the bucket and create a folder named `face`.
> ![Configure S3 Images](/aws-image/setupS3/s3-3_3.png)
![Cấu hình S3 Images](/aws-image/setupS3/setups3-bucket-new.png)
> ![Create face folder](/aws-image/setupS3/s3-5.png)
