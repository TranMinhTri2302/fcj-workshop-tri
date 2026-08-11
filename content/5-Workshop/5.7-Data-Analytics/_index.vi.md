---
title : "Data Analytics (S3 & Athena)"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

### Mục tiêu (Goal)

Ở các phần trước, mỗi khi có người checkin, hệ thống lưu kết quả vào DynamoDB (để truy vấn nhanh theo từng User) và bắn sự kiện qua EventBridge sang SQS/SNS. 
Đồng thời, chúng ta cũng có hàm Lambda **Analytics Worker** (phản hồi với hàng đợi SQS) tự động lấy các sự kiện điểm danh này và **ghi trực tiếp** thành các file JSON vào một **S3 Data Lake**.

Sau khi dữ liệu đã nằm an toàn trên Data Lake, ta có thể dùng AWS Glue để tự động nội suy cấu trúc bảng (Schema), và dùng Amazon Athena để truy vấn dữ liệu bằng SQL. Toàn bộ quá trình này hoàn toàn tự động, phi máy chủ (Serverless) và tách biệt khỏi hệ thống API chính (OLTP). Thay vì phải Query trên DynamoDB (rất tốn kém và không phù hợp cho truy vấn thống kê lớn), chúng ta sẽ sử dụng kiến trúc Data Lake Serverless với **AWS Glue** và **Amazon Athena**.

- **AWS Glue (Crawler):** Tự động quét các file Log trong S3 Data Lake để nhận diện cấu trúc dữ liệu (Schema) và tạo ra một bảng ảo (Table).
- **Amazon Athena:** Cho phép bạn viết câu lệnh SQL quen thuộc để truy vấn trực tiếp trên dữ liệu nằm trong S3 thông qua bảng ảo của Glue, mà không cần phải cài đặt bất kỳ Database Server nào. Tính tiền theo lượng dữ liệu quét (Pay per query).

### Các nội dung thực hành chi tiết

Vui lòng bấm chọn từng mục dưới đây ở thanh menu bên trái hoặc click trực tiếp vào các liên kết dưới đây để thực hiện chi tiết từng bước:

{{% children /%}}
