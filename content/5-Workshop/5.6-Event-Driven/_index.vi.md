---
title : "Kiến trúc Event-Driven"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

# Phần 4: Kiến trúc Hướng sự kiện (Event-Driven)

Đến thời điểm hiện tại, API điểm danh của chúng ta hoạt động theo cơ chế **Đồng bộ (Synchronous)**. Có nghĩa là khi người dùng gửi ảnh lên, API Gateway sẽ chuyển cho Lambda. Lambda gọi AI, lưu DB, lưu S3, và sau đó mới trả về kết quả thành công cho Frontend.

Cách làm này có một nhược điểm: Nếu chúng ta muốn bổ sung thêm các tính năng như "Gửi Email báo điểm danh thành công", hay "Cảnh báo bảo mật nếu phát hiện người lạ", nếu nhét hết code vào hàm Lambda ban đầu, thời gian chờ phản hồi (Response Time) sẽ tăng lên rất cao, dẫn tới trải nghiệm người dùng kém (đợi lâu) hoặc thậm chí là Timeout.

Để giải quyết vấn đề này, chúng ta sẽ áp dụng **Kiến trúc Hướng sự kiện (Event-Driven Architecture)** với bộ 3 công cụ kinh điển của AWS:
- **Amazon EventBridge**: Trạm trung chuyển (Event Bus) nhận và định tuyến các sự kiện.
- **Amazon SNS (Simple Notification Service)**: Hệ thống Pub/Sub dùng để phát (broadcast) thông báo tới nhiều điểm cuối (như Email, SMS, v.v.).
- **Amazon SQS (Simple Queue Service)**: Hàng đợi (Queue) giúp lưu trữ các bản tin tạm thời để xử lý bất đồng bộ, tránh quá tải hệ thống.
- **Amazon SES (Simple Email Service)**: Dịch vụ gửi Email chuyên dụng.

### Luồng xử lý mới:
1. Hàm Lambda chính (ở phần 5.5) sau khi lưu DB xong, thay vì tự đi gửi Email, nó chỉ làm một thao tác duy nhất: **Bắn một Event (Sự kiện)** mang nội dung `CHECKIN_SUCCESS` (Điểm danh thành công) hoặc `SECURITY_ALERT` (Cảnh báo an ninh) vào **EventBridge**.
2. Sau khi bắn xong Event, Lambda lập tức phản hồi kết quả về cho người dùng (rất nhanh!).
3. Ở đằng sau, **EventBridge** sẽ dùng các quy tắc (Rules) để bắt các Event này và định tuyến chúng:
   - Các Event `SECURITY_ALERT` sẽ được đẩy thẳng sang **SNS Topic** để gửi cảnh báo an ninh ngay lập tức.
   - Các Event `CHECKIN_SUCCESS` sẽ được đẩy vào **SQS Queue** làm bộ đệm, sau đó một Worker khác (hoặc xử lý trực tiếp) sẽ rút thông tin ra, kết hợp với **Amazon SES** để gửi Email báo cáo điểm danh cho nhân sự.

Trong các module tiếp theo, chúng ta sẽ lần lượt thiết lập SES, SNS, SQS và cuối cùng là cấu hình EventBridge để liên kết toàn bộ chúng lại.

{{% children /%}}
