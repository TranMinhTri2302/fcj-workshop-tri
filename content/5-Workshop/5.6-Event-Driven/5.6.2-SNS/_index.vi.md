---
title : "Cấu hình Amazon SNS"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2. </b> "
---

#### 5.6.2. Cấu hình Amazon SNS (Gửi thông báo)
Amazon SNS (Simple Notification Service) là dịch vụ Pub/Sub. Chúng ta sẽ tạo một "kênh phát sóng" (gọi là **Topic**) và cho phép các địa chỉ Email (như SES vừa tạo) đăng ký theo dõi (Subscribe) kênh này. Khi có tin nhắn đẩy vào Topic, tất cả những ai đang theo dõi đều sẽ nhận được.

**Bước 1: Tạo SNS Topic**

1. Tìm kiếm và truy cập dịch vụ **Simple Notification Service** trên AWS Console.
> ![Tìm kiếm SNS](/aws-image/setupSNS/sns-1.png)
2. Tại trang chủ (Dashboard) của SNS, trong khung **Create topic**, nhập tên Topic là `smart-campus-notifications` và bấm **Next step**.
> ![Nhập Tên](/aws-image/setupSNS/sns-2.png)
3. Ở trang cấu hình chi tiết, tại phần **Type**, chọn **Standard** (để hỗ trợ gửi Email, SMS). Tên Topic đã được tự động điền sẵn.
> ![Chọn Standard](/aws-image/setupSNS/sns-3.png)
4. Cuộn xuống cuối trang và bấm **Create topic**.
> ![Tạo Topic](/aws-image/setupSNS/sns-4.png)

**Bước 2: Tạo Subscription (Đăng ký nhận tin)**
Sau khi tạo xong Topic, hệ thống sẽ báo thành công và chuyển bạn đến trang chi tiết của Topic đó. Bây giờ chúng ta sẽ thêm Email nhân sự vào danh sách nhận tin.

1. Ngay tại trang chi tiết Topic, ở tab **Subscriptions**, bấm nút màu cam **Create subscription**.
> ![Tạo Subscription](/aws-image/setupSNS/sns-5.png)
2. Tại màn hình đăng ký:
   - **Topic ARN**: Hệ thống tự động chọn đúng Topic bạn vừa tạo.
   - **Protocol**: Chọn **Email**.
   - **Endpoint**: Nhập địa chỉ Email mà bạn đã xác thực ở phần SES (Ví dụ: `danhbattu2049@gmail.com`).
   - Cuộn xuống bấm **Create subscription**.
> ![Cấu hình Subscription](/aws-image/setupSNS/sns-6.png)

**Bước 3: Xác nhận Subscription**
Tương tự như SES, chủ sở hữu Email phải đồng ý nhận tin nhắn từ SNS.

1. Trạng thái Subscription lúc này sẽ là **Pending confirmation**. Mở hộp thư Gmail, tìm email có tiêu đề *"AWS Notification - Subscription Confirmation"*. Bấm vào nút **Xác nhận đăng ký** (hoặc Confirm subscription) trong email.
> ![Xác nhận Subscription](/aws-image/setupSNS/sns-7.png)
2. Một trang web sẽ mở ra báo **Subscription confirmed!**
> ![Subscription confirmed](/aws-image/setupSNS/sns-8.png)
3. Quay lại màn hình SNS, tải lại trang, trạng thái sẽ chuyển thành **Confirmed**.

Bây giờ, kênh thông báo `smart-campus-notifications` đã hoàn toàn sẵn sàng. Bất kỳ sự kiện nào được gửi vào Topic này sẽ lập tức biến thành một Email gửi thẳng đến nhân sự.
