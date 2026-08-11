---
title : "Giám sát & Theo dõi"
date : 2024-01-01
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

### Mục tiêu (Goal)

Để đảm bảo hệ thống Smart Campus luôn hoạt động ổn định và sẵn sàng đáp ứng khi có sự cố, chúng ta cần các công cụ để theo dõi sức khỏe của hệ thống, xem log lỗi và đo lường hiệu năng.

- **Amazon CloudWatch:** Đóng vai trò là trung tâm giám sát. Nó sẽ thu thập Log (nhật ký hoạt động) từ Lambda, API Gateway và EventBridge. Đồng thời cung cấp các Metric (chỉ số) và Alarm (cảnh báo) khi có bất thường (Ví dụ: số lượng lỗi tăng vọt).
- **AWS X-Ray:** Một công cụ Trace cực kỳ mạnh mẽ cho kiến trúc Serverless. X-Ray sẽ vẽ ra một bản đồ kết nối trực quan giữa các dịch vụ (Client -> API Gateway -> Lambda -> DynamoDB) và chỉ cho bạn thấy chính xác nút thắt cổ chai (bottleneck) đang nằm ở đâu, mất bao nhiêu mili-giây ở từng chặng.

### Các nội dung thực hành chi tiết

Vui lòng bấm chọn từng mục dưới đây ở thanh menu bên trái hoặc click trực tiếp vào các liên kết dưới đây để thực hiện chi tiết từng bước:

{{% children /%}}
