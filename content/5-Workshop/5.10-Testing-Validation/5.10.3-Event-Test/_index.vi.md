---
title : "Kiểm thử sự kiện"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.10.3. </b> "
---

#### 5.10.3. Kiểm thử luồng thông báo sự kiện (EventBridge → SNS → Email & SQS)

Sau khi điểm danh thành công ở bước 5.10.2, Lambda sẽ tự động phát một sự kiện `AttendanceRecorded` lên **EventBridge**. Phần này xác nhận rằng sự kiện đó đã được định tuyến đúng sang giao diện Web, SNS (gửi Email) và SQS (lưu hàng đợi).

> ![Bật camera điểm danh](/aws-image/setupTestnoti/noti1.png)
> ![Điểm danh thành công](/aws-image/setupTestnoti/noti2.png)

---

**Bước 1: Kiểm tra thông báo trên giao diện Web (Frontend)**

Ứng dụng của chúng ta có một phân hệ Notifications lấy dữ liệu trực tiếp từ Backend.

1. Trên giao diện Web, chuyển sang menu **Thông báo (Notifications)**.
2. Tại đây, bạn sẽ thấy một bản ghi mới với loại sự kiện là `AttendanceRecorded` (Điểm danh thành công).
3. Bấm **Chi tiết** để xem thông tin nội dung thông báo.

> **Kết quả mong đợi:** Giao diện Web hiển thị ngay lập tức sự kiện điểm danh vừa diễn ra với đầy đủ thông tin.

---

**Bước 2: Xác thực Email thông báo (SNS → Email)**

1. Mở hộp thư Gmail của địa chỉ Email bạn đã đăng ký với SNS Topic `smart-campus-notifications` ở bước 5.6.2.
2. Sau khoảng 30 giây đến 1 phút kể từ khi điểm danh, bạn sẽ nhận được một Email mới từ **AWS Notifications** (no-reply@sns.amazonaws.com).
3. Nội dung Email sẽ chứa thông tin sự kiện điểm danh dưới dạng JSON, bao gồm `userId`, `attendanceId`, `status`, `timestamp`.
> ![Email thông báo](/aws-image/setupTestnoti/noti3.png)

> **Kết quả mong đợi:** Nhận được Email tự động trong vòng 1 phút sau khi gọi API điểm danh.

---

**Bước 3: Xác thực hàng đợi SQS đã nhận sự kiện**

1. Tìm kiếm và truy cập AWS Console > **SQS**.
> ![Tìm SQS](/aws-image/setupTestnoti/noti4.png)
2. Chọn hàng đợi (ví dụ: `smart-campus-notification-queue`).
> ![Chọn hàng đợi](/aws-image/setupTestnoti/noti5.png)
3. Bấm nút **Send and receive messages** (góc trên bên phải).
> ![Send and receive](/aws-image/setupTestnoti/noti6.png)
4. Cuộn xuống phần **Receive messages**, bấm nút **Poll for messages**.
> ![Poll message](/aws-image/setupTestnoti/noti7.png)
5. Trong vòng vài giây, hệ thống sẽ hiển thị các message đang nằm trong hàng đợi.
> ![Có message](/aws-image/setupTestnoti/noti8.png)
6. Bấm vào một message để xem nội dung — bạn sẽ thấy body là payload EventBridge với `detail-type: AttendanceRecorded`.

> **Kết quả mong đợi:** Message từ sự kiện điểm danh xuất hiện trong SQS sau khi poll.

---

**Bước 4: Xác thực EventBridge Rule đã khớp (Monitoring)**

1. Tìm kiếm và truy cập AWS Console > **Amazon EventBridge**.
> ![Tìm EventBridge](/aws-image/setupTestnoti/noti9.png)
2. Chọn **Event buses**.
> ![Chọn Event buses](/aws-image/setupTestnoti/noti10.png)
3. Chọn bus `smart-campus-events` và chuyển sang tab **Monitoring**, chọn khoảng thời gian **Last 1 hour**.
4. Kiểm tra biểu đồ **Matched events** — sẽ có ít nhất 1 sự kiện được đánh dấu là khớp (matched) với rule.
> ![Biểu đồ Monitoring](/aws-image/setupTestnoti/noti11.png)

> **Kết quả mong đợi:** Biểu đồ Matched events tăng lên, không có Failed invocations.

