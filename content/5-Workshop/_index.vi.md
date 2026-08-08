---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---


# Smart Campus - Hệ Thống Điểm Danh Serverless với Nhận Diện Khuôn Mặt

## Xây dựng hệ thống điểm danh thông minh với AWS Rekognition, Lambda, DynamoDB và Event-Driven Architecture

#### Tổng quan

Trong workshop này, bạn sẽ học cách xây dựng một hệ thống điểm danh serverless production-grade sử dụng công nghệ nhận diện khuôn mặt AI. Hệ thống không chỉ nhận diện khuôn mặt mà còn tích hợp **Face Liveness Detection** để chống gian lận (phát hiện ảnh in, video replay, mặt nạ 3D), xây dựng kiến trúc **Event-Driven** với EventBridge và SQS để đảm bảo reliability, và tạo **Data Lake** cho analytics với Athena và Glue.

**Bạn sẽ học được:**
- Thiết kế và triển khai serverless architecture trên AWS
- Tích hợp AI/ML services (Rekognition Face Recognition + Liveness Detection)
- Xây dựng event-driven systems với EventBridge, SQS
- Implement reliable messaging với Dead Letter Queues
- Tạo Data Lake analytics pipeline với S3, Glue, Athena
- Setup monitoring và alerting với CloudWatch
- Áp dụng security best practices (WAF, Cognito, IAM)
- Tối ưu chi phí cho serverless applications

**Thời gian ước tính:** 3-4 giờ

**Chi phí ước tính:** ~$10-15 USD (nếu cleanup sau 24h)

#### Workflow chính

Trong workshop này, bạn sẽ triển khai workflow điểm danh hoàn chỉnh:

```
Camera chụp ảnh khuôn mặt
  → API Gateway nhận request
  → Lambda xử lý
  → Rekognition Face Liveness Check (phát hiện ảnh in/video)
  → Rekognition Face Recognition (nhận diện khuôn mặt)
  → Rule Engine validation (check trùng, check session, check time)
  → Lưu attendance record vào DynamoDB
  → Publish event lên EventBridge
  → EventBridge route đến SQS Queues
  → Lambda Workers xử lý (Analytics, Notification)
  → Kết quả: Record trong Data Lake, Email notification
```

**Các loại endpoints bạn sẽ tạo:**
- **Gateway VPC Endpoint**: Để truy cập S3 từ Lambda trong VPC (nếu cần)
- **Interface VPC Endpoint**: Để truy cập Rekognition, DynamoDB từ VPC (optional)
- **API Gateway HTTP Endpoint**: Public endpoint cho attendance check-in

#### Nội dung

1. [Tổng quan Workshop](5.1-overview)
2. [Yêu cầu trước khi bắt đầu](5.2-prerequisite/)
3. [Bước 1: Setup DynamoDB Tables](5.3-dynamodb/)
4. [Bước 2: Tạo Rekognition Face Collection](5.4-rekognition/)
5. [Bước 3: Deploy Lambda Functions](5.5-lambda/)
6. [Bước 4: Configure API Gateway](5.6-apigateway/)
7. [Bước 5: Setup EventBridge & SQS](5.7-eventbridge/)
8. [Bước 6: Tạo Analytics Pipeline](5.8-analytics/)
9. [Bước 7: Testing hệ thống](5.9-testing/)
10. [Bước 8: Monitoring với CloudWatch](5.10-monitoring/)
11. [Bước 9: Tối ưu chi phí và bảo mật](5.11-optimization/)
12. [Dọn dẹp resources](5.12-cleanup/)