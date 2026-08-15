---
title : "Quản lý Công việc & Sự cố"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.10.2.2. </b> "
---

#### Kiểm thử luồng Quản lý Công việc và Báo cáo Sự cố

Bài kiểm thử này tập trung vào tính năng giao việc (Task Management), điều phối sự cố (Incident Reporting) và kiểm chứng tính năng tự động khóa quyền, lọc phòng ban khi giao việc phụ (Subtask). Đồng thời, chúng ta cũng sẽ kiểm thử tính năng tải file an toàn thông qua **S3 Pre-signed URL**.

---

**Bước 1: Giao việc và Tạo công việc con (Từ góc độ Quản lý)**

1. Đăng nhập vào hệ thống bằng tài khoản có Role là **MANAGER** (hoặc ADMIN).
2. Truy cập menu **Công việc (Tasks)**.
3. Nhấn nút **Giao việc** (Chú ý: Các Role như STAFF, TECHNICIAN sẽ không thấy nút này).
4. Điền các thông tin cần thiết:
   - Tiêu đề công việc
   - Loại công việc
   - Người thực hiện
   - Mức độ ưu tiên
5. Nhấn **Xác nhận**.

> ![Giao việc chính](/aws-image/testendtoend/test7.png)

6. Sau khi công việc được tạo, nhấn chọn công việc đó trên bảng, rồi nhấn nút **+ Thêm việc con**.

> ![Quản lý task trên Kanban](/aws-image/testendtoend/test15.png)

7. Điền thông tin cho Subtask và giao cho một nhân viên khác.

> ![Tạo công việc con](/aws-image/testendtoend/test8.png)

8. (Sau khi nhân viên nộp kết quả) Đăng nhập lại bằng tài khoản **MANAGER**, mở chi tiết công việc đang ở trạng thái "Chờ duyệt" và nhấn nút **Duyệt** để nghiệm thu.

> ![Duyệt công việc](/aws-image/testendtoend/test16.png)

> **Kết quả mong đợi:** Công việc chính và công việc con được tạo thành công, thể hiện rõ mối quan hệ cha-con trên bảng giao việc. Hệ thống đẩy Toast Notification thông báo trực tiếp cho nhân viên khi được phân công cũng như khi công việc được duyệt.


---

**Bước 2: Báo cáo sự cố (Từ góc độ Nhân viên)**

1. Đăng nhập vào hệ thống bằng tài khoản có Role là **STAFF**.
2. Truy cập menu **Công việc (Tasks)**.
3. Nhấn nút **Báo cáo sự cố / Yêu cầu bảo trì** (Chú ý: Staff không có quyền tạo công việc chung mà chỉ được báo cáo sự cố).
4. Điền các thông tin cần thiết:
   - Tiêu đề công việc
   - Phân loại sự cố
   - Phòng ban xử lý
   - Đính kèm file (Tải lên hình ảnh mô phỏng hiện trường)
5. Nhấn **Xác nhận**.

> ![Giao diện báo cáo sự cố](/aws-image/testendtoend/test11.png)

> **Kết quả mong đợi:** Sự cố được tạo thành công. Tuy nhân viên không chọn người xử lý (Assignee), hệ thống tự động định tuyến sự cố này về cho **Quản lý phòng Kỹ thuật (TECHNICAL)** tiếp nhận. Nhờ cơ chế **S3 Pre-signed URL**, file đính kèm được Frontend tự động tải thẳng lên S3 Bucket giúp tối ưu hóa băng thông cho Backend Lambda và đảm bảo an toàn.


---

**Bước 3: Phân công xử lý Sự cố (Từ góc độ Quản lý)**

1. Đăng xuất tài khoản STAFF, đăng nhập lại bằng tài khoản **MANAGER** thuộc phòng ban `TECHNICAL`.
2. Truy cập menu **Công việc (Tasks)**, bạn sẽ thấy sự cố vừa được báo cáo đang ở trạng thái Cần làm.

> ![Sự cố hiển thị trên hệ thống](/aws-image/testendtoend/test12.png)

3. Nhấn nút **Phân công** trên sự cố để mở form cập nhật công việc.
4. Tại mục Người thực hiện, chọn nhân viên phù hợp và nhấn **Xác nhận**.

> ![Phân công Kỹ thuật viên xử lý](/aws-image/testendtoend/test13.png)

> **Kết quả mong đợi:** 
> - Trường "Phòng ban xử lý" bị **khóa cứng (disabled)** ở `Bảo trì (Technical)` để đảm bảo đúng quy trình.
> - Danh sách "Người thực hiện" (Assignee) sổ xuống chỉ hiển thị các nhân viên thuộc phòng ban được phân công.

---

**Bước 4: Nộp kết quả xử lý Sự cố**

1. Đăng nhập bằng tài khoản Kỹ thuật viên, cập nhật trạng thái sự cố sang **Đang làm**.
2. Khi xử lý xong, nhấn nút **Gửi duyệt**, tải lên file báo cáo kết quả và nhấn **Gửi duyệt**.

> ![Hoàn thành công việc](/aws-image/testendtoend/test14.png)

> **Kết quả mong đợi:** Hệ thống ghi nhận trạng thái mới. Nếu sự cố đã trễ hạn, hệ thống sẽ tự động hiển thị nhãn màu đỏ cảnh báo.
