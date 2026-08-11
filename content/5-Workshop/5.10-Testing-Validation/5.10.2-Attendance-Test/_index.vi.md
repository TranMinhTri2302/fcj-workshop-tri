---
title : "Kiểm thử điểm danh"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.10.2. </b> "
---

#### 5.10.2. Kiểm thử luồng điểm danh nhận diện khuôn mặt qua giao diện Web

Đây là luồng nghiệp vụ cốt lõi của hệ thống: Đăng ký khuôn mặt vào Rekognition, sau đó gửi ảnh để nhận diện và ghi nhận điểm danh vào DynamoDB. Thay vì dùng Postman, chúng ta sẽ trải nghiệm trực tiếp trên giao diện Frontend của ứng dụng.

---

**Bước 1: Đăng nhập vào trang web**

1. Mở trang web Frontend của bạn (URL của CloudFront hoặc chạy local qua `npm run dev`).
2. Đăng nhập bằng tài khoản vừa tạo ở phần trước.
3. Chuyển sang menu **Điểm danh (Attendance)**.

---

**Bước 2: Đăng ký khuôn mặt lần đầu**

Vì đây là lần đầu tiên tài khoản này sử dụng hệ thống, ứng dụng sẽ yêu cầu bạn đăng ký khuôn mặt.
> ![Giao diện đăng ký khuôn mặt](/aws-image/setupTestcheckin/checkin1.png)

1. Tại trang Điểm danh, bạn sẽ thấy thông báo **Tài khoản của bạn chưa có khuôn mặt**.
2. Nhấn nút **Bật Camera** và cho phép trình duyệt truy cập Webcam.
3. Ngồi thẳng, đảm bảo mặt rõ nét trong khung hình và nhấn nút **Chụp ảnh & Đăng ký khuôn mặt**.
4. Chờ hệ thống gọi API `POST /api/faces/register`.

> **Kết quả mong đợi:** Nhận được thông báo "Đăng ký khuôn mặt thành công!". Trạng thái tài khoản của bạn đã được cập nhật thành đã có khuôn mặt.

---

**Bước 3: Xác thực dữ liệu đăng ký khuôn mặt (S3)**

1. Vào AWS Console > **S3** > Bucket `smart-campus-images-{id}`.
2. Mở thư mục `faces/`. Bạn sẽ thấy file ảnh vừa chụp bằng Webcam được lưu.
> ![Danh sách ảnh trên S3](/aws-image/setupTestcheckin/checkin5.png)
> ![Chi tiết ảnh S3](/aws-image/setupTestcheckin/checkin3.png)

> **Kết quả mong đợi:** File ảnh tồn tại trong S3.

---

**Bước 4: Thực hiện Check-in (Điểm danh)**

Sau khi đã đăng ký khuôn mặt, giao diện Điểm danh sẽ chuyển sang chế độ Check-in.

1. Nhấn nút **Bật Camera** (nếu camera đang tắt).
2. Nhấn nút **Chụp ảnh (Check in)**.
3. Hệ thống sẽ gọi API `POST /api/attendance/recognize` để đối chiếu với ảnh gốc trong Rekognition.

> **Kết quả mong đợi:** Màn hình hiển thị tên, độ tin cậy (Confidence) của khuôn mặt. Nếu bạn check-in thành công lần đầu, sẽ có thông báo thành công. Nếu bạn điểm danh lại nhiều lần trong ngày, hệ thống sẽ hiện cảnh báo **"Điểm danh đã được ghi nhận trước đó (bỏ qua trùng lặp)"** như trong ảnh dưới.
> ![Kết quả Check-in](/aws-image/setupTestcheckin/checkin4.png)

---

**Bước 5: Xác thực bản ghi trong DynamoDB**

1. Vào AWS Console > **DynamoDB** > Tables > `smart-campus-faces`.
2. Bấm nút **Explore table items**.
3. Bạn sẽ thấy bản ghi mới xuất hiện chứa metadata của khuôn mặt (bao gồm `faceId`, `confidence`, `s3Key`, `userId`).
> ![Xác thực DynamoDB](/aws-image/setupTestcheckin/checkin6.png)

> **Kết quả mong đợi:** Dữ liệu nhận diện khuôn mặt đã được lưu trữ thành công vào DynamoDB.

---

**Bước 6: Kiểm thử trường hợp nhận diện thất bại (Negative Test)**

Để đảm bảo hệ thống xử lý đúng, hãy thử:
1. Nhờ một người khác (không phải bạn) ngồi vào trước Camera và nhấn Check in.
2. Hoặc đưa một vật dụng (điện thoại, cốc nước) che mặt hoặc che hoàn toàn camera để không có mặt người và nhấn Check in.

> **Kết quả mong đợi:** Hệ thống báo lỗi **"Không nhận diện được - Không phát hiện khuôn mặt trong ảnh"**.
> ![Lỗi không nhận diện được](/aws-image/setupTestcheckin/checkin7.png)


---

**Bước 7: Kiểm thử điểm danh ngoài mạng công ty (WAF IP Whitelisting)**

Để kiểm chứng tính năng chặn IP bằng AWS WAF đã thiết lập ở phần 5.5.4, hãy thử:
1. Ngắt kết nối Wi-Fi hiện tại trên máy tính và phát Wi-Fi từ điện thoại (dùng 4G/5G) để đổi địa chỉ IP.
2. Hoặc sử dụng một VPN để tạo một IP ảo khác mạng nội bộ.
3. Nhấn nút **Chụp ảnh (Check in)** trên giao diện.

> **Kết quả mong đợi:** Yêu cầu bị chặn ngay lập tức ở tầng WAF trước khi chạm đến API Gateway. Hệ thống báo lỗi từ chối truy cập (Ví dụ: 403 Forbidden) do không nằm trong danh sách IP được phép (IP Whitelist).
> ![Lỗi điểm danh sai mạng](/aws-image/setupTestcheckin/checkin8.png)