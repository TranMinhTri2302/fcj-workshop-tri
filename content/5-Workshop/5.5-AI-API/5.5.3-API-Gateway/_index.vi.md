---
title : "Tạo API Gateway"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.5.3. </b> "
---

#### 5.5.3. Khởi tạo và cấu hình API Gateway
API Gateway là cổng duy nhất tiếp nhận toàn bộ request từ Frontend và chuyển hướng xuống hàm Lambda đã tạo ở bước trước. Vì Lambda đã sẵn sàng, chúng ta có thể cấu hình integration hoàn chỉnh ngay trong bước này.

1. Tìm kiếm và chọn dịch vụ **API Gateway** trên thanh tìm kiếm của AWS Console.
> ![Tìm kiếm API Gateway](/aws-image/setupAPI/api1.png)
2. Tại giao diện chính của API Gateway, cuộn xuống và chọn **Create an API**.
> ![Tạo API](/aws-image/setupAPI/api2.png)
3. Tìm đến phần **HTTP API** (loại API hiện đại, chi phí thấp của AWS), nhấn nút **Build**.
> ![Chọn HTTP API](/aws-image/setupAPI/api3.png)
4. Tại bước *Configure API*, thiết lập các thông số và kết nối với Lambda:
   - **API name**: Đặt tên cho API (ví dụ: `SmartCampusHTTPApi`).
   - Bấm nút **Add integration** và thiết lập:
     - **Integrations**: Chọn `Lambda`.
     - **AWS Region**: `ap-southeast-1`.
     - **Lambda function**: Chọn hàm `smart-campus-api` vừa tạo ở mục 5.5.2.
   - Nhấn **Next**.
> ![Cấu hình API](/aws-image/setupAPI/api4.png)
5. Tại bước *Configure routes*, thiết lập Route để chuyển hướng toàn bộ Request xuống Lambda (Mô hình Serverless Proxy):
   - **Method**: Chọn `ANY`.
   - **Resource path**: Nhập `/{proxy+}`.
   - **Integration target**: Chọn hàm Lambda `smart-campus-api`.
   - Nhấn **Next**.
> ![Cấu hình Route](/aws-image/setupAPI/api5.png)
6. Tại bước *Define stages*, hệ thống mặc định tạo Stage `$default` với Auto-deploy. Giữ nguyên và nhấn **Next**.
> ![Cấu hình Stages](/aws-image/setupAPI/api6.png)
7. Ở bước *Review and create*, kiểm tra lại thông tin rồi nhấn **Create**.
> ![Kiểm tra và tạo](/aws-image/setupAPI/api7.png)
8. Sau khi tạo thành công, hệ thống sinh ra một **Invoke URL**. **Hãy sao chép và lưu lại đường dẫn này** — cần dùng ngay ở bước tiếp theo (5.5.4 WAF).
> ![Copy Invoke URL](/aws-image/setupAPI/api8.png)
9. Kiểm tra bằng cách mở tab mới, dán Invoke URL và thêm `/docs` vào cuối. Nếu giao diện **Swagger UI** hiện ra là API Gateway đã kết nối thành công với Lambda!
> ![Kiểm tra Swagger UI](/aws-image/setupAPI/api9.png)




> 📋 **Checklist:** Lưu lại **Invoke URL**  Bước tiếp theo (5.5.4 WAF) sẽ dùng URL này để cấu hình CloudFront bảo vệ API.
