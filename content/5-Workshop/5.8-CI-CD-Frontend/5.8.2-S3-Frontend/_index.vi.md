---
title : "Tạo S3 Bucket Hosting"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.8.2. </b> "
---

#### 5.8.2. Hosting Frontend trên Amazon S3
Amazon S3 có một tính năng vô cùng lợi hại là **Static Website Hosting**. Bạn chỉ việc upload các file HTML, CSS, JS (sau khi đã build) lên S3, nó sẽ biến thành một máy chủ web thực thụ mà không cần bạn phải duy trì hệ điều hành hay phần mềm web server (như Nginx/Apache).

**Bước 1: Tạo S3 Bucket cho Website**

1. Từ thanh tìm kiếm AWS Console, gõ và chọn dịch vụ **S3**.
> ![Tìm S3](/aws-image/setupS3frontend/s31.png)
2. Bấm nút **Create bucket**.
> ![Nút Create bucket](/aws-image/setupS3frontend/s32.png)
3. **Bucket name**: Đặt tên có ý nghĩa, ví dụ `smart-campus-frontend-2026` (Lưu ý: Tên bucket phải là duy nhất trên toàn cầu AWS).
> ![Tên Bucket](/aws-image/setupS3frontend/s33_1.png)
4. Ở phần **Object Ownership** chọn **ACLs disabled**. Ở phần **Block Public Access settings for this bucket**, bỏ chọn **Block all public access** và đánh dấu xác nhận (Acknowledge) cảnh báo của AWS.
> ![Block Public Access](/aws-image/setupS3frontend/s33_2.png)
5. Cuộn xuống cuối và bấm **Create bucket**.
> ![Tạo bucket thành công](/aws-image/setupS3frontend/s33_3.png)

**Bước 2: Bật Static Website Hosting**

1. Vào bucket vừa tạo, chuyển sang tab **Properties**.
> ![Tab Properties](/aws-image/setupS3frontend/s34_1.png)
2. Cuộn xuống cuối cùng đến phần **Static website hosting**, bấm **Edit**. Chọn **Enable**. Ở mục **Index document**, điền `index.html`.
> ![Cấu hình Index Document](/aws-image/setupS3frontend/s34_2.png)
3. Cuộn xuống dưới cùng trang và bấm **Save changes**.
> ![Save changes](/aws-image/setupS3frontend/s34_3.png)

**Bước 3: Cấu hình Policy và Upload source code**

1. Chuyển sang tab **Permissions**. Cuộn đến phần **Bucket policy** và bấm **Edit**.
> ![Tab Permissions](/aws-image/setupS3frontend/s35.png)
2. Điền cấu hình Policy dạng JSON để cho phép `s3:GetObject` công khai cho tất cả object.
> ![Cấu hình Policy JSON](/aws-image/setupS3frontend/s36.png)
3. Cuộn xuống dưới cùng và bấm **Save changes**.
> ![Save Policy](/aws-image/setupS3frontend/s36_2.png)
4. Chuyển sang tab **Objects** và bấm nút **Upload**.
> ![Nút Upload](/aws-image/setupS3frontend/s37.png)
5. Kéo thả các file trong thư mục build (ví dụ thư mục `dist` của Frontend) vào khu vực Upload.
> ![Kéo thả file](/aws-image/setupS3frontend/s38_1.png)
6. Cuộn xuống dưới cùng và bấm nút **Upload** để hoàn tất việc tải code lên S3.
> ![Hoàn tất Upload](/aws-image/setupS3frontend/s38_2.png)

**Bước 4: Xem kết quả**

Quay lại tab **Properties**, cuộn xuống phần **Static website hosting** ở dưới cùng. Click vào đường link **Bucket website endpoint**.
> ![Static website endpoint](/aws-image/setupS3frontend/s39.png)

Chờ trình duyệt tải, bạn sẽ thấy giao diện Frontend Smart Campus hiện ra thành công! Ứng dụng của bạn đã chính thức chạy tĩnh trên Amazon S3.
> ![Giao diện Smart Campus](/aws-image/setupS3frontend/s40.png)
