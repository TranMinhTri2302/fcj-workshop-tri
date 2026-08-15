---
title : "Xin nghỉ phép & Thông báo"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.10.2.3. </b> "
---

#### Kiểm thử luồng Đơn xin nghỉ phép và Hệ thống thông báo

Bài kiểm thử này tập trung vào tính năng đăng ký nghỉ phép trên **Lịch tương tác (Interactive Calendar)**, kiểm chứng hệ thống chống trùng lặp ngày/lễ và trải nghiệm UX với **Toast Notifications**.

---

**Bước 1: Cấu hình Ngày lễ (Từ góc độ Admin)**

1. Đăng nhập vào hệ thống bằng tài khoản **ADMIN**.
2. Truy cập menu **Nghỉ phép (Leaves)**.
3. Chuyển sang tab **Ngày lễ (Holidays)** và nhấn **Thêm ngày lễ**.
4. Điền các thông tin cần thiết:
   - Ngày
   - Tên ngày lễ
   - Mô tả (tùy chọn)
5. Nhấn **Thêm**.

> ![Thêm Ngày lễ](/aws-image/testendtoend/test17.png)


> ![Lịch hiển thị ngày lễ](/aws-image/testendtoend/test18.png)
> **Kết quả mong đợi:** Ngày lễ được lưu vào hệ thống, hiển thị trên lịch với màu sắc đặc trưng (màu xanh lá).
---

**Bước 2: Tạo Đơn xin nghỉ phép (Từ góc độ Nhân viên)**

1. Đăng xuất và đăng nhập lại bằng tài khoản **STAFF**.
2. Truy cập menu **Nghỉ phép (Leaves)**.
3. Bấm vào một ngày bất kỳ trên Lịch tương tác (Interactive Calendar). Form đăng ký sẽ tự động điền sẵn ngày bạn chọn.
4. Điền các thông tin cần thiết:
   - Loại đăng ký (Nghỉ phép năm hoặc WFH)
   - Ngày
   - Lý do (tùy chọn)
5. Bấm **Gửi đăng ký**.

> ![Đăng ký Nghỉ phép](/aws-image/testendtoend/test19.png)

> **Kết quả mong đợi:** Đơn xin nghỉ phép được khởi tạo ở trạng thái **Chờ duyệt (PENDING)**. Hệ thống gửi Toast Notification màu xanh báo hiệu thành công.

---

**Bước 3: Kiểm thử chặn trùng lịch (Negative Test)**

Để kiểm tra độ chặt chẽ của hệ thống Backend:
1. Thử tạo tiếp một đơn xin nghỉ phép khác **trùng với ngày** bạn vừa xin ở Bước 2.
2. Hoặc thử tạo một đơn xin nghỉ phép vào đúng **Ngày lễ** mà Admin đã cấu hình ở Bước 1.

> ![Lỗi trùng lịch/ngày lễ](/aws-image/testendtoend/test20.png)

> **Kết quả mong đợi:** Hệ thống Backend tự động quét và chặn yêu cầu. Frontend sẽ nhận được lỗi và hiển thị một Toast Notification màu đỏ cảnh báo không thể xin nghỉ vào ngày lễ/ngày đã đăng ký.

---

**Bước 4: Duyệt Đơn và kiểm tra Thông báo (Từ góc độ Quản lý)**

1. Đăng xuất và đăng nhập lại bằng tài khoản **MANAGER** hoặc **ADMIN**.
2. Truy cập menu Nghỉ phép, chuyển sang tab **Chờ duyệt**.
3. Bạn sẽ thấy đơn xin nghỉ của nhân viên ở Bước 2. Bấm nút **Duyệt**.

> ![Duyệt đơn nghỉ phép](/aws-image/testendtoend/test21.png)


> Truy cập vào màn hình **Thông báo (Notifications)**, bạn có thể kiểm chứng luồng hoạt động này: danh sách sẽ hiển thị toàn bộ lịch sử các sự kiện và biến động liên quan đến tài khoản của bạn.
> ![Trung tâm Thông báo](/aws-image/testendtoend/test22.png)
> **Kết quả mong đợi:** Đơn chuyển sang trạng thái Đã duyệt.
> **Kiểm thử Hệ thống Thông báo (Notification Center):** 
> Hệ thống Smart Campus được thiết kế để tự động gửi thông báo cho rất nhiều sự kiện khác nhau (như khi được giao việc, công việc trễ hạn deadline, cập nhật tiến độ, hay các thay đổi về đơn nghỉ phép/WFH). 