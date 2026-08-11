---
title : "Cấu hình Amazon SQS"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3. </b> "
---

#### 5.6.3. Cấu hình Amazon SQS (Hàng đợi)
Amazon SQS (Simple Queue Service) đóng vai trò là một "vùng đệm" (Buffer). Bằng cách cho các sự kiện đi vào SQS trước khi gọi Lambda xử lý (gửi email, ghi log Analytics), hệ thống sẽ không bao giờ bị quá tải hoặc mất dữ liệu ngay cả khi có hàng nghìn sinh viên điểm danh cùng lúc.


**Bước 1: Tạo Dead-Letter Queue (DLQ)**

1. Tìm kiếm và truy cập dịch vụ **SQS** trên AWS Console.
> ![Tìm kiếm SQS](/aws-image/setupSQS/sqs1.png)
2. Bấm **Create queue**.
> ![Tạo Queue](/aws-image/setupSQS/sqs2.png)
3. **Type**: Chọn **Standard**.
> ![Chọn Standard](/aws-image/setupSQS/sqs3.png)
4. **Name**: Nhập `smart-campus-dlq`.
> ![Nhập tên DLQ](/aws-image/setupSQS/sqs4.png)
5. Cuộn xuống cuối và bấm **Create queue**.
> ![Tạo DLQ](/aws-image/setupSQS/sqs5.png)

**Bước 2: Tạo Main Queue và gắn DLQ**

1. Quay lại trang danh sách SQS, tiếp tục bấm **Create queue** lần thứ hai.
> ![Tạo Main Queue](/aws-image/setupSQS/sqs6.png)
2. **Type**: Chọn **Standard**.
> ![Chọn Standard](/aws-image/setupSQS/sqs7.png)
3. **Name**: Nhập `smart-campus-analytics-queue` (hoặc `smart-campus-notification-queue` tuỳ luồng bạn muốn thiết lập).
> ![Tên Main Queue](/aws-image/setupSQS/sqs8.png)
4. Cuộn xuống phần **Dead-letter queue**:
   - Chọn **Enabled**.
> ![Bật DLQ](/aws-image/setupSQS/sqs9.png)
   - **Choose queue**: Trỏ vào cái `smart-campus-dlq` vừa tạo ở Bước 1.
> ![Chọn DLQ](/aws-image/setupSQS/sqs10.png)
   - **Maximum receives**: Đặt là `3` (Nghĩa là nếu hệ thống thử xử lý 3 lần vẫn lỗi, tin nhắn sẽ bị đẩy sang DLQ).
> ![Số lần nhận max](/aws-image/setupSQS/sqs11.png)
5. Mở phần **Access policy**. Để EventBridge có thể gửi tin nhắn vào SQS, chúng ta cần cấp quyền `sqs:SendMessage` cho dịch vụ `events.amazonaws.com`.
> ![Mở Access Policy](/aws-image/setupSQS/sqs12.png)
6. Ở ô JSON, bạn thay thế nội dung cũ bằng đoạn JSON sau (nhớ thay `{AccountID}` và `{Region}` bằng thông tin của bạn):
```json
{
  "Version": "2012-10-17",
  "Id": "Queue1_Policy_UUID",
  "Statement": [
    {
      "Sid": "EventBridgePublishToSQS",
      "Effect": "Allow",
      "Principal": {
        "Service": "events.amazonaws.com"
      },
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:ap-southeast-1:123456789012:smart-campus-analytics-queue"
    }
  ]
}
```
> ![Nhập JSON Policy](/aws-image/setupSQS/sqs13.png)
7. Cuộn xuống bấm **Create queue**.
> ![Bấm Tạo](/aws-image/setupSQS/sqs14.png)

**Bước 3: Kích hoạt Lambda Trigger từ SQS (Tuỳ chọn)**

Nếu bạn muốn SQS tự động gọi một hàm Lambda khác để xử lý dữ liệu:
1. Mở Main Queue vừa tạo, chuyển sang tab **Lambda triggers**.
> ![Tab Lambda Trigger](/aws-image/setupSQS/sqs21.png)
2. Bấm **Configure Lambda function trigger**.
> ![Bấm Configure](/aws-image/setupSQS/sqs22.png)
3. Chọn hàm Lambda (Ví dụ: `smart-campus-analytics-processor`).
> ![Chọn Lambda](/aws-image/setupSQS/sqs24.png)
4. Bấm **Save**.
> ![Bấm Save](/aws-image/setupSQS/sqs25.png)
5. Hệ thống hiển thị Lambda đã được kích hoạt thành công.
> ![Thành công](/aws-image/setupSQS/sqs26.png)

Vậy là hàng đợi SQS đã sẵn sàng. Bước cuối cùng để ghép nối bức tranh này lại với nhau chính là **Amazon EventBridge**.
