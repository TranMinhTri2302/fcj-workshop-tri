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
| **5.10.2** | Kiểm thử điểm danh nhận diện khuôn mặt | Rekognition, DynamoDB, S3 |
| **5.10.3** | Kiểm thử luồng thông báo sự kiện | EventBridge, SNS, SQS |
| **5.10.4** | Kiểm thử Log & Metric giám sát | CloudWatch, X-Ray |




#### 1. Kiểm thử gửi Request (Postman / Frontend)
1. Lấy URL của API Gateway (Ví dụ: `https://xyz.execute-api.ap-southeast-1.amazonaws.com/prod/attendance`).
2. Mở Postman, chọn phương thức **POST**. Dán URL vào.
3. Ở tab **Body** > **raw** > **JSON**, nhập payload chứa ảnh khuôn mặt sinh viên (base64) và `camera_id`.
4. Bấm **Send**.
5. Nhận lại kết quả `200 OK` với thông điệp: "Điểm danh thành công cho sinh viên ...".

#### 2. Xác thực lưu trữ tại DynamoDB & S3
1. **DynamoDB:** Truy cập AWS Console > DynamoDB > Tables > `smart-campus-attendance`.
   - Bấm **Explore table items**.
   - Bạn sẽ thấy một bản ghi mới xuất hiện với `attendance_id`, thời gian và trạng thái đi học.
2. **S3 (Image Storage):** Truy cập S3 bucket `smart-campus-images`.
   - Kiểm tra xem file ảnh khuôn mặt vừa gửi lên có được lưu lại với định dạng tên `YYYY-MM-DD/ID.jpg` hay không.

#### 3. Xác thực Log & Metric (CloudWatch)
1. Truy cập CloudWatch > Log groups > Chọn log group của Lambda `smart-campus-api`.
2. Kiểm tra log stream mới nhất để xem các dòng lệnh `print` thời gian thực thi, kết quả trả về từ Amazon Rekognition.
3. Chuyển sang phần **Metrics**, chọn hàm Lambda và kiểm tra biểu đồ **Invocations** (Số lượt gọi) xem cột biểu đồ có nhích lên 1 đơn vị không.

#### 4. Xác thực Event-Driven (SNS / SQS)
1. Mở Hòm thư (Gmail) mà bạn đã đăng ký với SNS ở Bước 5.6.
2. Bạn sẽ nhận được một Email mới từ AWS báo cáo sự kiện điểm danh.
3. Nếu bạn có cấu hình SQS, hãy truy cập SQS > Chọn `smart-campus-analytics-queue` > Bấm **Send and receive messages** > **Poll for messages** để xem sự kiện định tuyến từ EventBridge đã được đẩy vào hàng đợi chưa.

Nếu tất cả các bước trên đều trả về kết quả đúng như kỳ vọng, xin chúc mừng! Bạn đã triển khai thành công 100% kiến trúc Serverless Event-Driven trên AWS.
