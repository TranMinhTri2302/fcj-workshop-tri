---
title : "Triển khai & Tự động hóa CI/CD"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

### Mục tiêu (Goal)

Đến thời điểm hiện tại, Back-end API (Serverless) của Smart Campus đã hoạt động trơn tru. Bây giờ là lúc chúng ta đưa Giao diện người dùng (Front-end) lên Cloud, đồng thời thiết lập luồng tích hợp và triển khai liên tục (CI/CD) cho cả Frontend và Backend để tự động hóa hoàn toàn vòng đời phát triển phần mềm.

Trong chương này chúng ta sẽ thực hiện:
1. **Tự động hóa CI/CD cho Backend với AWS CodePipeline:** Thiết lập luồng tự động build và deploy hạ tầng serverless mỗi khi có code mới.
2. **Host trang web tĩnh trên Amazon S3:** Rẻ, bền bỉ và không cần quản lý máy chủ.
3. **Tăng tốc với Amazon CloudFront (CDN):** Cache nội dung ở các Edge Location trên toàn cầu, giúp tải trang web cực nhanh và tăng cường bảo mật.
4. **Tự động hóa luồng CI/CD cho Frontend với AWS CodePipeline:** Kéo code từ GitHub, build và đẩy thẳng lên S3 hoàn toàn tự động.

### Các nội dung thực hành chi tiết

Vui lòng bấm chọn từng mục dưới đây ở thanh menu bên trái hoặc click trực tiếp vào các liên kết dưới đây để thực hiện chi tiết từng bước:

{{% children /%}}
