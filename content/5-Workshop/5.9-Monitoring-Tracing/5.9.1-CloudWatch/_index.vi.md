---
title : "Giám sát với CloudWatch"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.9.1. </b> "
---

#### 5.9.1. Cấu hình giám sát tập trung bằng CloudWatch
Mọi Log của Lambda (kết quả in ra từ câu lệnh `print` hoặc `console.log`) đều được đẩy thẳng vào CloudWatch Logs một cách tự động. Nhiệm vụ của chúng ta là biết cách xem log và tạo các cảnh báo để giám sát hệ thống.

**Bước 1: Xem Log của Lambda**

1. Từ AWS Console, mở dịch vụ **CloudWatch**. Nhìn sang thanh menu bên trái, phần **Logs**, chọn **Log groups**. Danh sách các log group sẽ hiện ra.
> ![Chọn Log groups](/aws-image/setupCloudWatch/cloudwatch15.png)
2. Tìm và bấm vào log group có tên `/aws/lambda/smart-campus-api`. 
> ![Chọn tên Log group](/aws-image/setupCloudWatch/cloudwatch16.png)
3. Cuộn xuống phần **Log streams**, bấm vào luồng log mới nhất. Bạn sẽ xem được toàn bộ chi tiết các dòng log (INFO, START, END) phát sinh trong quá trình chạy.
> ![Chi tiết Log streams](/aws-image/setupCloudWatch/cloudwatch17.png)

**Bước 2: Tạo Alarm (Cảnh báo tự động)**
Giả sử chúng ta muốn nhận email báo động mỗi khi Lambda có lỗi (Errors > 0).

1. Vẫn ở Console CloudWatch, bạn có thể nhập tìm kiếm ở ô phía trên cùng màn hình nếu cần.
> ![Tìm kiếm CloudWatch](/aws-image/setupCloudWatch/cloudwatch1.png)
2. Ở menu bên trái, mục **Alarms**, chọn **All alarms** và bấm **Create alarm**.
> ![Tạo Alarm](/aws-image/setupCloudWatch/cloudwatch2.png)
3. Bấm nút **Select metric**.
> ![Select Metric](/aws-image/setupCloudWatch/cloudwatch3.png)
4. Trong phần Browse, chọn **Lambda**.
> ![Chọn Lambda](/aws-image/setupCloudWatch/cloudwatch4.png)
5. Tiếp tục chọn **By Function Name**.
> ![By Function Name](/aws-image/setupCloudWatch/cloudwatch5.png)
6. Tìm hàm `smart-campus-api`, tick vào checkbox ở cột **Errors**. Sau đó bấm nút **Select metric** ở góc dưới.
> ![Tick Errors](/aws-image/setupCloudWatch/cloudwatch6.png)
7. Màn hình cấu hình Graph xuất hiện, kiểm tra phần Statistic mặc định thường là **Sum** hoặc **Average**, rồi kéo xuống dưới.
> ![Cấu hình Statistic](/aws-image/setupCloudWatch/cloudwatch7.png)
8. **Conditions**: Chọn Threshold type là **Static**. Phần *Whenever Errors is*, chọn **Greater/Equal** và nhập `1`. Bấm **Next**.
> ![Cấu hình Condition](/aws-image/setupCloudWatch/cloudwatch8.png)
9. Ở bước **Configure actions**, phần **Notification**, bấm nút **Add notification**.
> ![Add notification](/aws-image/setupCloudWatch/cloudwatch9.png)
10. Mục *Alarm state trigger*, chọn **In alarm**. Mục *Send a notification to the following SNS topic*, chọn **Select an existing SNS topic** và chọn `smart-campus-notifications` (Topic bạn đã tạo ở phần SNS).
> ![Cấu hình SNS Topic](/aws-image/setupCloudWatch/cloudwatch10.png)
11. Cuộn xuống dưới cùng và bấm **Next**.
> ![Bấm Next](/aws-image/setupCloudWatch/cloudwatch11.png)
12. Ở bước **Add alarm details**, đặt tên cho Alarm ở ô **Alarm name** (Ví dụ: `Lambda-Error-Alert`). Bạn có thể điền thêm mô tả ở ô bên dưới nếu cần. Sau đó cuộn xuống và bấm **Next**.
> ![Đặt tên Alarm](/aws-image/setupCloudWatch/cloudwatch12.png)
13. Màn hình Review cho phép bạn xem lại toàn bộ cấu hình. Cuộn xuống dưới cùng và bấm **Create alarm**.
> ![Create Alarm](/aws-image/setupCloudWatch/cloudwatch13.png)
14. Hệ thống báo xanh thành công, Alarm của bạn đã được tạo và sẵn sàng theo dõi lỗi.
> ![Thành công](/aws-image/setupCloudWatch/cloudwatch14.png)

Vậy là xong! Bây giờ, hễ hệ thống điểm danh chết hoặc có lỗi (Exception), bạn sẽ nhận được Email cảnh báo ngay lập tức để kịp thời khắc phục. Dưới đây là ví dụ về một email báo động (ALARM) mà bạn sẽ nhận được từ Amazon SNS thông báo rằng hệ thống đang gặp lỗi:

> ![Email Cảnh Báo](/aws-image/setupCloudWatch/cloudwatch18.png)
