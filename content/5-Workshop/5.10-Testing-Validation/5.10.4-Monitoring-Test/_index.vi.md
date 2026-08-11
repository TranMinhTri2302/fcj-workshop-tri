---
title : "Kiểm thử giám sát"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.10.4. </b> "
---

#### 5.10.4. Kiểm thử Log, Metric và Tracing (CloudWatch & X-Ray)

Phần này hướng dẫn bạn xem log thời gian thực, kiểm tra metric và phân tích luồng request qua X-Ray để xác nhận hệ thống giám sát đang hoạt động đúng.

---

**Bước 1: Xem Log Lambda trên CloudWatch**

1. Vào AWS Console > **CloudWatch** > **Log groups**.
> ![Tìm CloudWatch](/aws-image/setupTestgiamsat/giamsat1.png)
2. Tìm và chọn log group có tên `/aws/lambda/smart-campus-api`.
> ![Log groups](/aws-image/setupTestgiamsat/giamsat2.png)
3. Từ danh sách **Log streams**, bấm vào stream mới nhất (thường là dòng trên cùng với timestamp gần nhất).
> ![Log streams](/aws-image/setupTestgiamsat/giamsat3.png)
4. Bạn sẽ thấy các dòng log sự kiện chi tiết cho từng lần gọi API, bao gồm:
   - Thời gian bắt đầu và kết thúc request
   - Kết quả trả về từ Rekognition
   - Thông tin bản ghi được ghi vào DynamoDB
   - Sự kiện được phát lên EventBridge
> ![Log events](/aws-image/setupTestgiamsat/giamsat4.png)

> **Kết quả mong đợi:** Log hiển thị rõ ràng các bước xử lý, không có dòng `ERROR` hoặc `Exception`.

---

**Bước 2: Kiểm tra Metric Lambda**

1. Vào **CloudWatch**, nhìn sang menu bên trái mục **Metrics** > Chọn **Classic metrics**.
2. Tại tab **Browse** bên dưới biểu đồ, chọn namespace **AWS/Lambda**.
> ![Giao diện Metrics](/aws-image/setupTestgiamsat/giamsat5.png)
3. Tiếp tục chọn dimension **By Function Name**.
> ![Chọn Dimension](/aws-image/setupTestgiamsat/giamsat6.png)
4. Tìm hàm `smart-campus-api` và tích chọn các metric cần theo dõi:
   - **Invocations**: Số lượt gọi hàm (phải tăng tương ứng số lần bạn test)
   - **Duration**: Thời gian xử lý trung bình (ms)
   - **Errors**: Số lượt lỗi (kỳ vọng là 0)
> ![Chọn Metric](/aws-image/setupTestgiamsat/giamsat7.png)
5. Điều chỉnh khoảng thời gian trên thanh công cụ (ví dụ: **1h** hoặc **3h**) và quan sát biểu đồ.

> **Kết quả mong đợi:** Biểu đồ hiển thị dữ liệu hoạt động của API, không bị trống.

---

**Bước 3: Phân tích Trace trên X-Ray**

1. Vào **CloudWatch**, ở menu bên trái cuộn xuống tìm và chọn **Trace Map**.
2. Chọn khoảng thời gian (ví dụ: **30m**) trên thanh công cụ. Bạn sẽ thấy bản đồ dịch vụ (Service Map) trực quan minh họa luồng đi của request.
> ![Trace Map](/aws-image/setupTestgiamsat/giamsat8.png)
3. Bấm vào node **smart-campus-api** trên bản đồ để xem bảng phân tích chi tiết về Latency, Requests, và Faults ở khung bên dưới.
> ![Chi tiết Node](/aws-image/setupTestgiamsat/giamsat9.png)
4. Nhìn sang menu bên trái, chuyển sang mục **Traces** để xem danh sách các trace cụ thể, từ đó có thể bấm vào từng trace để phân tích biểu đồ thời gian xử lý.
> ![Traces](/aws-image/setupTestgiamsat/giamsat10.png)

---

**Bước 4: Kích hoạt CloudWatch Alarm (Kiểm thử Alert)**

Để xác minh hệ thống cảnh báo hoạt động thông suốt mà không cần chờ phát sinh sự cố thực tế, chúng ta sẽ thực hiện kiểm thử giả lập (Simulation Test). Bằng cách sử dụng AWS CLI để kích hoạt Alarm chủ động, ta có thể đánh giá toàn bộ luồng thông báo từ CloudWatch đến SNS và Email.

1. Mở AWS CloudShell (biểu tượng >_ trên thanh công cụ góc trên bên phải) hoặc dùng terminal của bạn.
2. Chạy lệnh sau để giả lập cảnh báo:
```bash
aws cloudwatch set-alarm-state --alarm-name "Lambda-Error-Alert" --state-value ALARM --state-reason "Test Alert"
```
3. Ngay lập tức, trạng thái Alarm sẽ chuyển từ `OK` → `In alarm` (Bạn có thể xem tại **CloudWatch** > **Alarms**).
> ![Danh sách Alarms](/aws-image/setupTestgiamsat/giamsat11.png)
4. Đồng thời, bạn sẽ nhận được Email cảnh báo từ SNS Topic `smart-campus-notifications`.
> ![Email báo động](/aws-image/setupTestgiamsat/giamsat12.png)

> **Kết quả mong đợi:** Alarm chuyển trạng thái và Email cảnh báo được gửi đến hộp thư của bạn.
