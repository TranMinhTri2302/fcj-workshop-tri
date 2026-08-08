---
title: "Tổng quan Workshop"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Giới thiệu về Smart Campus Platform

**Smart Campus Platform** là hệ thống điểm danh thông minh sử dụng công nghệ nhận diện khuôn mặt AI, được xây dựng hoàn toàn trên kiến trúc serverless AWS. Hệ thống giải quyết bài toán điểm danh gian lận và tốn thời gian trong các trường học, doanh nghiệp.

#### Bài toán thực tế

Trong nhiều tổ chức, điểm danh vẫn được thực hiện thủ công hoặc bằng thẻ từ:
- **Vấn đề 1**: Dễ gian lận (nhờ người khác điểm danh, dùng ảnh in)
- **Vấn đề 2**: Tốn thời gian (5-10 phút/buổi cho 50 người)
- **Vấn đề 3**: Khó tổng hợp báo cáo (phải xử lý thủ công Excel)
- **Vấn đề 4**: Không có cảnh báo realtime khi vi phạm


#### Workshop Flow

Bạn sẽ xây dựng hệ thống theo 12 bước:

1. **Setup DynamoDB**: Tạo 3 tables (users, faces, attendance)
2. **Tạo Rekognition Collection**: Nơi lưu face embeddings
3. **Deploy Lambda Functions**: Main API + 2 workers
4. **Configure API Gateway**: Expose Lambda qua HTTP endpoint
5. **Setup EventBridge**: Event bus + routing rules
6. **Create SQS Queues**: Analytics Queue + Notification Queue với DLQ
7. **Deploy Lambda Workers**: Process events từ SQS
8. **Setup Analytics Pipeline**: S3 Data Lake + Glue + Athena
9. **Testing**: Test end-to-end attendance flow
10. **Monitoring**: CloudWatch Logs + Metrics + Alarms
11. **Optimization**: Security hardening + cost optimization
12. **Cleanup**: Xóa tất cả resources



**Estimated cost:** ~$10-15 USD nếu chạy đầy đủ workshop và cleanup trong 24h.

#### Bước tiếp theo

Hãy bắt đầu với [Yêu cầu trước khi bắt đầu](../5.2-prerequisite) để chuẩn bị môi trường!
