---
title : "Cấu hình Xác thực & Bảo mật"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

### Mục tiêu (Goal)

Trong phần này, chúng ta sẽ xây dựng tuyến phòng thủ xác thực người dùng đầu tiên cho toàn bộ hệ thống Smart Campus. Thay vì phải tự code logic mã hóa mật khẩu và tạo Token phức tạp, chúng ta sẽ ủy thác hoàn toàn cho **Amazon Cognito**.


> **AWS WAF** sẽ được cấu hình ở **mục 5.5.4** — sau khi API Gateway và Lambda đã được tạo xong, vì WAF cần Invoke URL của API Gateway để hoạt động.

### Các nội dung thực hành chi tiết

#### 5.3.1. Khởi tạo Amazon Cognito User Pool
1. Truy cập vào AWS Console, tìm kiếm dịch vụ **Cognito**.
> ![Tìm kiếm dịch vụ Cognito](/aws-image/setupCognito/cognito1.png)
2. Tại giao diện Amazon Cognito, chọn **Create user pool**.
> ![Khởi tạo Cognito](/aws-image/setupCognito/cognito2.png)
3. Ở màn hình thiết lập, trong mục *Application type*, chọn **Single-page application (SPA)**. Nhập tên ứng dụng của bạn (ví dụ: `smart-campus-client`).
> ![Thiết lập ứng dụng SPA](/aws-image/setupCognito/cognito3.png)
4. Cuộn xuống phần *Configure options*, đánh dấu chọn **Email** làm phương thức đăng nhập chính. Sau đó nhấn nút **Create user directory**.
> ![Cấu hình Email](/aws-image/setupCognito/cognito4.png)
5. Hệ thống sẽ hiển thị thông báo màu xanh báo hiệu User Pool và App Client đã được tạo thành công.
> ![Tạo thành công](/aws-image/setupCognito/cognito5.png)
6. Tại trang thông tin (User pool information), hãy sao chép và lưu lại **User pool ID**. Mã này sẽ được đưa vào file `.env` của mã nguồn ReactJS.
> ![Copy User Pool ID](/aws-image/setupCognito/cognito6.png)
7. Ở menu bên trái, chuyển sang tab **App clients** dưới mục *Applications* và bấm vào App client bạn vừa tạo.
> ![Chọn App Client](/aws-image/setupCognito/cognito7.png)
8. Tại trang chi tiết App client, bạn có thể sao chép **Client ID**. Tuy nhiên, chúng ta cần cấu hình thêm một chút.
> ![Xem Client ID](/aws-image/setupCognito/cognito8.png)
9. Nhấn nút **Edit** ở góc phải phần *App client information* để chỉnh sửa quyền xác thực.
> ![Sửa App Client](/aws-image/setupCognito/cognito9.png)
10. Trong mục *Authentication flows*, hãy đánh dấu tick vào **ALLOW_USER_PASSWORD_AUTH**. Bước này cực kỳ quan trọng để Frontend có thể gọi API đăng nhập bằng Username và Password truyền thống.
> ![Cấu hình Auth Flows](/aws-image/setupCognito/cognito10.png)
11. Cuộn xuống cuối trang và nhấn nút **Save changes** để lưu lại.
> ![Lưu thay đổi](/aws-image/setupCognito/cognito11.png)
12. Sau khi lưu thành công, bạn có thể sao chép lại **Client ID** một lần nữa để chắc chắn rằng cấu hình đã hoàn tất.
> ![Hoàn tất cấu hình](/aws-image/setupCognito/cognito12.png)
