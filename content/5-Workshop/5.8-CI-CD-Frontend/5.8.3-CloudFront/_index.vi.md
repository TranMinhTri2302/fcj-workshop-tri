---
title : "Tăng tốc với CloudFront"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.8.3. </b> "
---

#### 5.8.3. Khởi tạo Amazon CloudFront (CDN)
Mặc dù S3 có thể dùng để host website, nhưng nó không hỗ trợ chứng chỉ SSL (HTTPS) cho tên miền tuỳ chỉnh và cũng không có bộ đệm (cache) toàn cầu. Amazon CloudFront giải quyết toàn bộ các vấn đề này.

**Bước 1: Tạo CloudFront Distribution**

1. Từ thanh tìm kiếm AWS Console, gõ và chọn dịch vụ **CloudFront**.
> ![Tìm CloudFront](/aws-image/setupCloudfront/cloudfront1.png)
2. Bấm nút **Create distribution**.
> ![Tạo Distribution](/aws-image/setupCloudfront/cloudfront2.png)
3. Ở bước **Get started**: Tại phần **Distribution name**, đặt tên cho Distribution là `smart-campus-frontend`. Cuộn xuống dưới cùng và bấm **Next**.
> ![Tên Distribution](/aws-image/setupCloudfront/cloudfront3.png)
4. Ở bước **Specify origin**: Tại phần **Origin domain**, bấm **Browse S3**.
> ![Chọn S3 Origin](/aws-image/setupCloudfront/cloudfront4.png)
5. Một popup hiện ra, chọn S3 Bucket bạn vừa tạo (Ví dụ: `smart-campus-frontend-2026`) rồi bấm **Choose**. Sau đó quay lại màn hình chính, cuộn xuống cuối trang và bấm **Next**.
> ![Chọn S3 Location](/aws-image/setupCloudfront/cloudfront5.png)
6. Ở bước **Enable security**: Tại phần **Web Application Firewall (WAF)**, chọn **Do not enable security protections**. Sau đó bấm **Next**.
> ![Tắt WAF](/aws-image/setupCloudfront/cloudfront6.png)
7. Ở bước **Review and create**: Cuộn xuống cuối cùng và bấm nút **Create distribution**.
> ![Hoàn tất tạo Distribution](/aws-image/setupCloudfront/cloudfront7.png)

**Bước 2: Cấu hình Error Pages (Cho ứng dụng React/Vue SPA)**

Vì React/Vue là các Single Page Application (SPA), mọi request tới các đường dẫn không tồn tại cần được chuyển hướng về `index.html` để Frontend tự xử lý Routing.

1. Ngay sau khi tạo Distribution thành công, chuyển sang tab **Error pages** và bấm nút **Create custom error response**.
> ![Tạo Error Response](/aws-image/setupCloudfront/cloudfront8.png)
2. Cấu hình chuyển hướng lỗi 404 như sau:
   - **HTTP error code**: `404: Not Found`
   - **Customize error response**: Chọn `Yes`
   - **Response page path**: `/index.html`
   - **HTTP Response code**: `200: OK`
Cuối cùng bấm **Create custom error response** để lưu.
> ![Cấu hình 404](/aws-image/setupCloudfront/cloudfront9.png)

**Bước 3: Truy cập trang web qua CDN**

1. Chuyển sang tab **General** của CloudFront Distribution, tìm và sao chép địa chỉ **Distribution domain name** (Ví dụ: `d...cloudfront.net`).
> ![Domain CloudFront](/aws-image/setupCloudfront/cloudfront10.png)
2. Mở trình duyệt và truy cập vào đường link đó. Bạn sẽ thấy trang web load cực nhanh và được bảo vệ bằng ổ khóa HTTPS an toàn từ Amazon CloudFront!
> ![Giao diện Web HTTPS](/aws-image/setupCloudfront/cloudfront11.png)
