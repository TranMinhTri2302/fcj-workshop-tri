---
title : "Cấu hình Amazon SES"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1. </b> "
---

#### 5.6.1. Cấu hình Amazon SES (Xác thực Email)
Amazon SES (Simple Email Service) là dịch vụ gửi email của AWS. Vì tài khoản của chúng ta đang ở chế độ **Sandbox** (môi trường thử nghiệm để chống spam), AWS yêu cầu chúng ta phải xác minh (verify) quyền sở hữu của bất kỳ địa chỉ Email nào trước khi dùng nó làm người gửi (Sender) hoặc người nhận (Receiver).

Trong bài toán này, hệ thống cần gửi Email thông báo điểm danh thành công đến bộ phận nhân sự. Do đó, ta cần xác minh Email người nhận.

**Bước 1: Truy cập Amazon SES**

1. Tìm kiếm và truy cập dịch vụ **Amazon Simple Email Service** trên thanh tìm kiếm của AWS Console.
> ![Tìm kiếm SES](/aws-image/setupSES/ses-1.png)

**Bước 2: Tạo Identity (Xác thực Email)**

1. Ở thanh menu bên trái, dưới mục **Configuration**, chọn **Identities**. Sau đó bấm vào nút màu cam **Create identity** ở góc phải.
> ![Tạo Identity](/aws-image/setupSES/ses-2.png)
2. Ở phần **Identity type**, chọn **Email address**. Nhập địa chỉ Email thật của bạn vào ô trống. Cuộn xuống cuối trang và bấm **Create identity**.
> ![Nhập thông tin Email](/aws-image/setupSES/ses-3.png)

**Bước 3: Xác nhận trong Hòm thư**

1. AWS sẽ báo trạng thái **Verification pending**. Mở hộp thư Gmail của bạn, tìm Email có tiêu đề *"Amazon Web Services – Email Address Verification Request..."* và nhấp vào đường link xác nhận.
> ![Link xác nhận Email](/aws-image/setupSES/ses-4.png)

2. Sau khi click, trình duyệt sẽ báo xác nhận thành công. Quay lại màn hình SES Identity, bạn sẽ thấy trạng thái chuyển sang **Verified** màu xanh lá.
> ![Trạng thái Verified](/aws-image/setupSES/ses-5.png)

Vậy là xong! Chúng ta đã có một địa chỉ Email hợp lệ để hệ thống Smart Campus dùng làm "địa chỉ nhận thông báo" (hoặc gửi thông báo). Bước tiếp theo, chúng ta sẽ cấu hình **Amazon SNS** để thiết lập kênh phát sóng gửi nội dung vào Email này.
