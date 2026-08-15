---
title : "Điểm danh & Nhận diện khuôn mặt"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.10.2.1. </b> "
---

#### Kiểm thử luồng điểm danh và khôi phục mật khẩu qua giao diện Web

Đây là luồng nghiệp vụ cốt lõi của hệ thống: Đăng ký khuôn mặt vào Rekognition, sau đó gửi ảnh để nhận diện và ghi nhận điểm danh vào DynamoDB, cũng như sử dụng AI để khôi phục mật khẩu. Thay vì dùng Postman, chúng ta sẽ trải nghiệm trực tiếp trên giao diện Frontend của ứng dụng.

---

**Bước 1: Khôi phục mật khẩu bằng Face ID**

Trong hệ thống Smart Campus, nếu bạn quên mật khẩu, thay vì dùng Email OTP, hệ thống sử dụng Amazon Rekognition để xác thực khuôn mặt của bạn.

1. Mở trang web Frontend của bạn
2. Tại màn hình Đăng nhập, bấm vào nút **Quên mật khẩu?**.
3. Một cửa sổ (Modal) "Khôi phục bằng Face ID" sẽ hiện ra. Nhập Email tài khoản của bạn và bấm **Tiếp tục**.

> ![Bước nhập Email](/aws-image/testendtoend/test1.png)

4. Hệ thống sẽ mở Camera. Căn chỉnh khuôn mặt và nhấn **Chụp và Xác thực**.

> ![Quét khuôn mặt Face ID](/aws-image/testendtoend/test2.png)

5. Hệ thống gọi API `/api/auth/verify-face-reset` lên Backend.

> **Kết quả mong đợi:** Nếu khuôn mặt khớp với dữ liệu đã đăng ký, hệ thống sẽ cho phép bạn nhập Mật khẩu mới và tự động đăng nhập. Nếu sai người, hệ thống báo lỗi từ sâu AI.

> ![Cập nhật mật khẩu thành công](/aws-image/testendtoend/test3.png)

---

**Bước 2: Đăng ký khuôn mặt lần đầu**

Nếu bạn tạo một tài khoản mới tinh và chưa từng đăng ký khuôn mặt, ứng dụng sẽ yêu cầu bạn đăng ký trước khi điểm danh.

1. Đăng nhập bằng tài khoản mới. Chuyển sang trang **Điểm danh (Attendance)**.
2. Tại trang Điểm danh, bạn sẽ thấy thông báo **Tài khoản của bạn chưa có khuôn mặt**.
3. Nhấn nút **Bật Camera** và cho phép trình duyệt truy cập Webcam.
4. Ngồi thẳng, đảm bảo mặt rõ nét trong khung hình và nhấn nút **Chụp ảnh & Đăng ký khuôn mặt**.

> ![Giao diện đăng ký khuôn mặt](/aws-image/setupTestcheckin/checkin2.jpg)
5. Chờ hệ thống gọi API `POST /api/faces/register`.

> **Kết quả mong đợi:** Nhận được thông báo "Đăng ký khuôn mặt thành công!". Trạng thái tài khoản của bạn đã được cập nhật.

---

**Bước 3: Thực hiện Check-in / Check-out (Điểm danh)**

Sau khi đã đăng ký khuôn mặt, giao diện Điểm danh sẽ chuyển sang chế độ quét khuôn mặt hàng ngày.

1. Nhấn nút **Bật Camera** (nếu camera đang tắt).
2. Nhấn nút **Chụp ảnh (Check in)**.
3. Hệ thống sẽ gọi API `POST /api/attendance/recognize` để đối chiếu với ảnh gốc trong Rekognition.

> **Kết quả mong đợi:** Màn hình hiển thị tên, độ tin cậy (Confidence) của khuôn mặt và thông báo thành công. Nếu bạn điểm danh lại nhiều lần trong cùng một ca học, hệ thống sẽ hiện cảnh báo **"Điểm danh đã được ghi nhận trước đó (bỏ qua trùng lặp)"**.
> ![Kết quả Check-in đúng giờ](/aws-image/testendtoend/test4.png)

---

**Bước 4: Kiểm thử Điểm danh WFH (Làm việc từ xa)**

1. Đăng nhập bằng tài khoản Quản lý, duyệt đơn xin làm việc từ xa (WFH) cho nhân viên hôm nay.
2. Đăng xuất, sau đó đăng nhập lại bằng tài khoản của nhân viên vừa được duyệt.
3. Truy cập menu **Nghỉ phép (Leaves)**. 
4. Phía trên cùng của trang Nghỉ phép, bạn sẽ thấy một thanh màu xanh báo hiệu: *"Hôm nay bạn đang WFH — hãy bấm Điểm danh WFH khi bắt đầu làm việc"*.

> ![Nút Điểm danh WFH trên giao diện Nghỉ phép](/aws-image/testendtoend/test5.png)

5. Bấm vào nút **Điểm danh WFH** (có biểu tượng Ngôi nhà).

> ![Thông báo điểm danh WFH thành công](/aws-image/testendtoend/test6.png)

> **Kết quả mong đợi:** Nhân viên không cần bật Camera quét khuôn mặt mà vẫn check-in thành công. Hệ thống tự động gọi API `/attendance/wfh-checkin` và ghi nhận trạng thái WFH vào CSDL.

---

**Bước 5: Kiểm thử trường hợp nhận diện thất bại (Negative Test)**

Để đảm bảo hệ thống xử lý đúng, hãy thử:
1. Nhờ một người khác (không phải bạn) ngồi vào trước Camera và nhấn Check in.
2. Hoặc đưa một vật dụng (điện thoại, cốc nước) che mặt hoặc che hoàn toàn camera để không có mặt người và nhấn Check in.

> **Kết quả mong đợi:** Hệ thống báo lỗi **"Không nhận diện được - Không phát hiện khuôn mặt trong ảnh"**.
> ![Lỗi không nhận diện được](/aws-image/setupTestcheckin/checkin7.png)

---

**Bước 6: Kiểm thử điểm danh ngoài mạng công ty (WAF IP Whitelisting)**

Để kiểm chứng tính năng chặn IP bằng AWS WAF đã thiết lập ở phần 5.5.4, hãy thử:
1. Ngắt kết nối Wi-Fi hiện tại trên máy tính và phát Wi-Fi từ điện thoại (dùng 4G/5G) để đổi địa chỉ IP.
2. Hoặc sử dụng một VPN để tạo một IP ảo khác mạng nội bộ.
3. Nhấn nút **Chụp ảnh (Check in)** trên giao diện.

> **Kết quả mong đợi:** Yêu cầu bị chặn ngay lập tức ở tầng WAF trước khi chạm đến API Gateway. Hệ thống báo lỗi từ chối truy cập (Ví dụ: 403 Forbidden) do không nằm trong danh sách IP được phép (IP Whitelist).
> ![Lỗi điểm danh sai mạng](/aws-image/setupTestcheckin/checkin8.png)
