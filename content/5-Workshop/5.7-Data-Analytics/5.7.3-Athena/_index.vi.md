---
title : "Truy vấn với Amazon Athena"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.7.3. </b> "
---

#### 5.7.3. Phân tích dữ liệu với Amazon Athena
Amazon Athena là dịch vụ truy vấn tương tác serverless, giúp bạn dễ dàng phân tích dữ liệu trực tiếp trong S3 bằng SQL tiêu chuẩn. Vì cấu trúc bảng đã được Glue định nghĩa, Athena có thể đọc nó ngay lập tức.

**Bước 1: Truy cập Amazon Athena và Mở Trình soạn thảo**

1. Từ thanh tìm kiếm của AWS Console, gõ **Athena** và chọn dịch vụ.
> ![Tìm kiếm Athena](/aws-image/setupAthena/athena1.png)
2. Ở màn hình chính của Athena, bấm chọn **Launch query editor**.
> ![Launch Query Editor](/aws-image/setupAthena/athena2.png)

**Bước 2: Cấu hình Vị trí lưu kết quả (Query Result Location)**

Athena lưu kết quả của mỗi câu truy vấn thành một file trong S3. Nếu bạn sử dụng tính năng này lần đầu, bạn bắt buộc phải cấu hình vị trí lưu.
1. Trong màn hình Query editor, bạn sẽ thấy một thông báo màu xanh nhắc nhở thiết lập vị trí lưu. Bấm vào nút **Edit settings**.
> ![Edit Settings](/aws-image/setupAthena/athena3.png)
2. Tại màn hình Query settings vừa hiện ra, bấm vào nút **Manage**.
> ![Manage Settings](/aws-image/setupAthena/athena4.png)
3. Ở mục **Location of query result**, bấm nút **Browse S3**.
> ![Browse S3](/aws-image/setupAthena/athena5.png)
4. Chọn Bucket Data Lake của bạn (hoặc một bucket riêng biệt dùng cho Athena) và bấm **Choose**.
> ![Chọn Bucket](/aws-image/setupAthena/athena6.png)
5. Bạn nên gõ thêm một hậu tố thư mục vào cuối đường dẫn S3 (Ví dụ: `/athena-results/`) để kết quả truy vấn được lưu gọn gàng, tránh lẫn lộn với dữ liệu gốc. Sau đó bấm **Save**.
> ![Cấu hình Query Result Location](/aws-image/setupAthena/athena7.png)

**Bước 3: Chạy câu truy vấn SQL**

1. Trở về màn hình **Editor**, đảm bảo Database bên trái đã được chọn là `smart_campus_db`. Ở ô soạn thảo, hãy dán câu lệnh SQL đơn giản sau để xem log điểm danh đã được format trên S3 (nhớ thay đổi tên bảng cho đúng với dữ liệu của bạn, hoặc bạn có thể nhấn đúp vào tên bảng ở cột trái để hệ thống tự điền):
```sql
SELECT * FROM "smart_campus_db"."<tên_bảng_s3_của_bạn>"
```
2. Bấm **Run** (hoặc Run again). Ở tab **Query results** phía dưới, bạn sẽ thấy toàn bộ dữ liệu log điểm danh với đầy đủ các cột như `event_type`, `attendance_id`, `user_id`, `status` được hiển thị rõ ràng và đẹp mắt như một bảng CSDL truyền thống.
> ![Kết quả Truy vấn](/aws-image/setupAthena/athena8.png)



Đến đây, bạn đã làm chủ hoàn toàn luồng **Dữ liệu lớn (Data Pipeline)** trong Smart Campus!
