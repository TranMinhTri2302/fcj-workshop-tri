---
title : "Cấu hình Amazon EventBridge"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.6.4. </b> "
---

#### 5.6.4. Cấu hình Amazon EventBridge (Điều phối sự kiện)
EventBridge là "trái tim" của kiến trúc hướng sự kiện. Mỗi khi Lambda xử lý xong luồng nhận diện khuôn mặt, thay vì tự nó đi gọi API gửi email, nó sẽ "phát loa" một sự kiện lên EventBridge.
EventBridge sẽ nghe sự kiện này và tự động chuyển hướng nó tới các đích đến (Targets) phù hợp như SNS (để gửi Email) hoặc SQS (để lưu Log).

**Bước 1: Tạo Rule (Quy tắc định tuyến) sang SNS**
Chúng ta sẽ tạo một quy tắc: Hễ có sự kiện điểm danh, lập tức đẩy sang SNS.

1. Tìm kiếm dịch vụ **Amazon EventBridge** trên AWS Console.
> ![Tìm kiếm EventBridge](/aws-image/setupEvenBridge/event-1.png)
2. Ở menu bên trái, chọn **Rules**.
> ![Chọn Rules](/aws-image/setupEvenBridge/event-2.png)
3. Bấm **Create rule**.
> ![Bấm Create Rule](/aws-image/setupEvenBridge/event-3.png)
4. **Name**: Nhập `attendance-recorded-to-sns`. **Event bus**: Chọn `default` (hoặc Event Bus bạn đang dùng).
> ![Đặt tên Rule](/aws-image/setupEvenBridge/event-4.png)
5. **Rule type**: Chọn **Rule with an event pattern**. Bấm **Next**.
> ![Chọn loại Rule](/aws-image/setupEvenBridge/event-5.png)
6. Ở phần **Event pattern**:
   - Chọn **Custom pattern (JSON editor)**.
   - Nhập đoạn JSON sau để bắt chính xác sự kiện điểm danh:
```json
{
  "source": ["smart-campus.api"],
  "detail-type": ["AttendanceRecorded"]
}
```
> ![Cấu hình Event Pattern](/aws-image/setupEvenBridge/event-6.png)
7. Bấm **Next**. Ở phần **Select target(s)**:
   - **Target types**: Chọn **AWS service**.
   - **Select a target**: Chọn **SNS topic**.
> ![Chọn Target Type](/aws-image/setupEvenBridge/event-7.png)
8. **Topic**: Chọn `smart-campus-notifications` (Topic đã tạo ở phần SNS).
> ![Chọn Topic SNS](/aws-image/setupEvenBridge/event-8.png)
9. Bấm **Next** qua bước Add tags.
> ![Bấm Next Tags](/aws-image/setupEvenBridge/event-9.png)
10. Xem lại toàn bộ cấu hình ở màn hình Review và bấm **Create rule**.
> ![Tạo Rule](/aws-image/setupEvenBridge/event-10.png)

**Bước 2: Tạo Rule định tuyến sang SQS (Tuỳ chọn)**
Tương tự, ta sẽ tạo thêm Rule thứ 2 để đẩy sự kiện sang SQS nhằm phục vụ Data Analytics.

1. Tại màn hình Rules, bấm **Create rule** lần nữa.
> ![Tạo Rule 2](/aws-image/setupEvenBridge2/event1.png)
2. **Name**: Nhập `attendance-to-sqs`.
> ![Tên Rule 2](/aws-image/setupEvenBridge2/event2.png)
3. Chọn **Rule with an event pattern** rồi bấm **Next**.
> ![Rule Type 2](/aws-image/setupEvenBridge2/event3_1.png)
4. Tương tự, ở **Event pattern**, chọn **Custom pattern (JSON editor)**.
> ![Custom Pattern](/aws-image/setupEvenBridge2/event4_1.png)
5. Nhập JSON giống hệt Bước 1.
> ![Pattern JSON](/aws-image/setupEvenBridge2/event4_2.png)
6. Bấm **Next**.
> ![Next Pattern](/aws-image/setupEvenBridge2/event5.png)
7. Ở phần **Target**, chọn **AWS service**, nhưng lần này **Select a target** bạn chọn **SQS queue**.
> ![Chọn SQS Target](/aws-image/setupEvenBridge2/event6_1.png)
8. Chọn Queue `smart-campus-analytics-queue` mà bạn đã tạo.
> ![Chọn Queue](/aws-image/setupEvenBridge2/event6_2.png)
9. Bấm **Next** cho đến màn hình Review và bấm **Create rule**.
> ![Tạo Rule SQS](/aws-image/setupEvenBridge2/event7.png)

Đến đây, toàn bộ luồng kiến trúc hướng sự kiện (Event-Driven) đã hoàn tất!
- Lambda phát sự kiện lên **EventBridge**.
- EventBridge đẩy sự kiện ra 2 ngả độc lập: SNS (gửi Email) và SQS (lưu hàng đợi).
