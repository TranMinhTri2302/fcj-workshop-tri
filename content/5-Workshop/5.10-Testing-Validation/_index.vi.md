---
title : "Kiểm thử (Testing & Validation)"
date : 2024-01-01
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

### Mục tiêu

Sau khi đã hoàn thành toàn bộ các bước triển khai từ mục **5.3 đến 5.9**, phần này sẽ hướng dẫn bạn **kiểm thử End-to-End** toàn hệ thống Smart Campus — từ việc gọi API, xác thực kết quả lưu trữ, theo dõi log/metric, đến việc dọn dẹp tài nguyên sau khi hoàn thành.

Mỗi bước kiểm thử đều có **kết quả mong đợi cụ thể**, giúp bạn tự xác nhận hệ thống đang hoạt động đúng thiết kế.

---

### Các bước kiểm thử

| Mục | Nội dung | Dịch vụ liên quan |
|---|---|---|
| **5.10.1** | Kiểm thử API (Swagger UI & Postman) | API Gateway, Lambda, Cognito |
| **5.10.2** | **Kiểm thử End-to-End Nghiệp vụ** | *Nhiều dịch vụ* |
| ↳ 5.10.2.1 | Điểm danh & Nhận diện khuôn mặt | Rekognition, DynamoDB, S3 |
| ↳ 5.10.2.2 | Quản lý Công việc & Sự cố | S3 Pre-signed URL, DynamoDB |
| ↳ 5.10.2.3 | Xin nghỉ phép & Thông báo | API Gateway, Cognito |
| **5.10.3** | Kiểm thử luồng thông báo sự kiện | EventBridge, SNS, SQS |
| **5.10.4** | Kiểm thử Log & Metric giám sát | CloudWatch, X-Ray |

---
Bạn có thể bấm vào từng mục trên thanh điều hướng bên trái để xem hướng dẫn chi tiết cho từng luồng kiểm thử nhé!
