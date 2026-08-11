---
title : "Cấu hình Core API"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

### Mục tiêu (Goal)

Đây là "trái tim" của hệ thống Smart Campus. Trong phần này, bạn sẽ lần lượt:
1. Tạo **Rekognition Collection** — kho lưu trữ vector khuôn mặt cho AI nhận diện.
2. Triển khai **AWS Lambda** — hàm xử lý toàn bộ logic nghiệp vụ (nhận ảnh, gọi AI, lưu DynamoDB/S3, bắn Event).
3. Tạo **API Gateway** — điểm tiếp nhận request từ Frontend, chuyển hướng xuống Lambda.
4. Cấu hình **AWS WAF** — bảo vệ API điểm danh, chặn các truy cập từ bên ngoài mạng Campus.


> Thứ tự thực hiện rất quan trọng: **Rekognition → Lambda → API Gateway → WAF**. Mỗi bước phụ thuộc vào kết quả của bước trước.

### Các nội dung thực hành chi tiết

Vui lòng bấm chọn từng mục dưới đây ở thanh menu bên trái hoặc click trực tiếp vào các liên kết dưới đây để thực hiện chi tiết từng bước:

{{% children /%}}
