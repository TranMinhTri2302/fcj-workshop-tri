---
title : "Tạo bảng DynamoDB"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

#### 5.4.1. Thiết kế và tạo bảng trên Amazon DynamoDB

Amazon DynamoDB được chọn vì tốc độ phản hồi cực nhanh (chỉ vài mili-giây) và khả năng mở rộng không giới hạn (Serverless NoSQL), rất phù hợp để lưu trữ dữ liệu điểm danh và thông tin người dùng.

Để hệ thống Smart Campus hoạt động hoàn chỉnh theo thiết kế của Backend, kiến trúc thực tế yêu cầu tới **9 bảng dữ liệu (Tables)** trên DynamoDB. Dưới đây là bảng tổng hợp các cấu hình Khóa phân vùng (Partition Key) và Khóa phụ (GSI) tương ứng:

| Tên bảng (Table Name) | Partition Key (Kiểu String) | GSI (Global Secondary Index) | Mục đích sử dụng |
| :--- | :--- | :--- | :--- |
| `smart-campus-attendance` | `record_id` | `date-index` (PK: `date`) | Lưu lịch sử điểm danh hàng ngày |
| `smart-campus-faces` | `face_id` | `userid-index` (PK: `user_id`) | Lưu thông tin nhận diện khuôn mặt |
| `smart-campus-users` | `user_id` | *(Tùy chọn theo Backend)* | Lưu thông tin nhân viên/sinh viên |
| `smart-campus-holidays` | `date` | - | Quản lý ngày nghỉ lễ |
| `smart-campus-leaves` | `request_id` | *(Tùy chọn theo Backend)* | Quản lý đơn xin nghỉ phép |
| `smart-campus-notifications` | `notification_id` | *(Tùy chọn theo Backend)* | Quản lý lịch sử gửi thông báo |
| `smart-campus-settings` | `setting_key` | - | Quản lý cấu hình chung |
| `smart-campus-tasks` | `task_id` | *(Tùy chọn theo Backend)* | Quản lý các công việc tự động |

*(Lưu ý: Các GSI chưa được ghi chi tiết ở trên có thể được cấu hình thêm tùy theo yêu cầu truy vấn của hệ thống Backend trong thực tế).*

**Dưới đây là các bước thao tác trên AWS Console. Chúng ta sẽ lấy bảng `smart-campus-attendance` làm ví dụ minh họa, bạn chỉ cần lặp lại thao tác tương tự cho các bảng còn lại dựa vào thông số ở bảng trên:**

**Bước 1: Truy cập DynamoDB**
1. Tìm kiếm **DynamoDB** trên thanh tìm kiếm của AWS Console.
> ![Search DynamoDB](/aws-image/setupDB/setupdyamodb1.png)
2. Bấm vào nút **Create table** để bắt đầu tạo bảng.
> ![Create Table](/aws-image/setupDB/setupdyamodb2.png)

**Bước 2: Cấu hình Table và Khóa chính (Partition Key)**
3. Tại phần *Table details*:
   - **Table name:** Nhập tên bảng (ví dụ: `smart-campus-attendance`).
   - **Partition key:** Nhập tên Khóa phân vùng tương ứng (ví dụ: `record_id`, Kiểu: *String*).
   - **Table settings:** Chọn `Customize settings`.
> ![Attendance Table](/aws-image/setupDB/setupdyamodb8.png)

**Bước 3: Cấu hình Khóa phụ (GSI) - Nếu có**
4. Mở rộng phần *Secondary indexes*, bấm **Create global index** (đối với bảng `smart-campus-attendance`, ta cần tạo GSI để truy vấn theo ngày).
> ![GSI](/aws-image/setupDB/setupdyamodb4.png)
5. Điền thông tin GSI:
   - **Index name:** Nhập tên GSI (ví dụ: `date-index`).
   - **Partition key:** Nhập Khóa phân vùng của GSI (ví dụ: `date`, Kiểu: *String*).
   - **Attribute projections:** Chọn `All`. Bấm **Create index**.
> ![Attendance GSI](/aws-image/setupDB/setupdyamodb9.png)

**Bước 4: Hoàn tất tạo bảng**
6. Kéo xuống cuối trang và bấm **Create table**.
> ![Create Table Finish](/aws-image/setupDB/setupdyamodb5.png)

Lặp lại quy trình từ Bước 1 đến Bước 4 cho các bảng khác (như `smart-campus-faces`, `smart-campus-users`...) bằng cách tham chiếu với cấu hình trong bảng tổng hợp ở đầu bài.


