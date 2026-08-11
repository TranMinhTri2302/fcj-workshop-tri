---
title : "Tạo S3 Buckets"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

#### 5.4.2. Khởi tạo Amazon S3 Buckets
Hệ thống của chúng ta cần 2 bucket riêng biệt với các chính sách bảo mật khác nhau:

**Bucket 1: S3 Frontend (Dành cho web tĩnh)**
- **Tên:** `smart-campus-frontend-{your-id}`.
- Chức năng: Lưu trữ mã nguồn Frontend (đã build ra thư mục `dist`).


Bucket này sẽ được hướng dẫn tạo đầy đủ (bao gồm cấu hình Public Access và Static Website Hosting) tại mục **5.8.1**.

**Bucket 2: S3 Images (Lưu ảnh khuôn mặt)**
- **Tên:** `smart-campus-images-{your-id}`.
- Chức năng: Lưu trữ hình ảnh khuôn mặt dùng để đăng ký và nhận diện AI (Amazon Rekognition).
- Cấu hình: 
  - Kích hoạt **Block all public access** (Chặn toàn bộ truy cập công cộng). 
  - Kích hoạt tính năng mã hóa **SSE-S3** để đảm bảo an toàn dữ liệu sinh trắc học.
  - Tắt **Bucket Versioning**.
  - Sau khi tạo xong, vào trong bucket tạo một thư mục (folder) tên là `face`.
> ![Cấu hình S3 Images](/aws-image/setupS3/s3-3_3.png)
![Cấu hình S3 Images](/aws-image/setupS3/setups3-bucket-new.png)
> ![Tạo folder face](/aws-image/setupS3/s3-5.png)
