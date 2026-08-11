---
title : "Trace kiến trúc với X-Ray"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.9.2. </b> "
---

#### 5.9.2. Trace (Dò vết) luồng API bằng AWS X-Ray
Với kiến trúc Serverless, một request từ Frontend có thể đi qua hàng loạt dịch vụ: API Gateway -> Lambda -> Rekognition -> DynamoDB -> SQS. Nếu request chạy chậm, làm sao biết dịch vụ nào đang kéo tụt hiệu năng? Đó là lúc AWS X-Ray tỏa sáng.

**Bật X-Ray cho Lambda**

1. Truy cập dịch vụ **Lambda** trên thanh tìm kiếm của AWS Console.
> ![Tìm kiếm Lambda](/aws-image/setupXRay/xray1.png)
2. Chọn hàm `smart-campus-api` của bạn.
> ![Chọn hàm Lambda](/aws-image/setupXRay/xray2.png)
3. Chuyển sang tab **Configuration**, chọn mục **Permissions** và nhấp vào đường link của **Execution role** để mở IAM Console.
> ![Mở Execution Role](/aws-image/setupXRay/xray3.png)
4. Tại IAM Console, ở tab **Permissions**, bấm **Add permissions** và chọn **Attach policies**.
> ![Bấm Attach policies](/aws-image/setupXRay/xray4.png)
5. Tìm kiếm policy `AWSXRayDaemonWriteAccess`, tích chọn nó và bấm **Add permissions**.
> ![Thêm quyền X-Ray](/aws-image/setupXRay/xray5.png)
6. Quay lại màn hình Lambda của bạn, trong tab **Configuration**, chọn mục **Monitoring and operations tools** và bấm nút **Edit**.
> ![Edit Monitoring](/aws-image/setupXRay/xray6.png)
7. Bật công tắc ở mục **AWS X-Ray (Active tracing)** và bấm **Save** để hoàn tất.
> ![Bật Active Tracing](/aws-image/setupXRay/xray7.png)

**Trải nghiệm bản đồ Service Map và Traces**

1. Dùng Postman hoặc Frontend gọi vài API điểm danh để tạo dữ liệu. Sau đó truy cập dịch vụ **CloudWatch** trên AWS Console, cuộn xuống phần menu bên trái và chọn mục **Trace Map** (hoặc truy cập qua *X-Ray > Service map* nếu dùng giao diện cũ). Tại đây, bạn sẽ thấy bản đồ trực quan minh họa luồng đi của request.
> ![Bản đồ Service Map](/aws-image/setupXRay/xray8.png)
2. Bạn có thể nhấp vào bất kỳ một node (dịch vụ) nào trên bản đồ (Ví dụ: bảng DynamoDB) để xem chi tiết biểu đồ **Metrics** bên dưới (bao gồm Latency, Requests, Faults). Ngoài ra, bạn cũng có thể chuyển sang mục **Traces** ở menu trái để phân tích sâu hơn từng request dưới dạng biểu đồ thác nước (waterfall), nhằm tìm ra chính xác nút thắt cổ chai.
> ![Chi tiết Trace](/aws-image/setupXRay/xray9.png)
