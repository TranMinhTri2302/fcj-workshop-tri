---
title : "Cấu hình AWS Glue"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.7.2. </b> "
---

#### 5.7.2. Khởi tạo Data Catalog với AWS Glue
AWS Glue là dịch vụ Data Integration serverless. Chúng ta sẽ sử dụng tính năng **Glue Crawler** của nó để tự động đọc các file log điểm danh trong S3 bucket và suy luận ra cấu trúc bảng (Table Schema) lưu vào Data Catalog.

**Bước 1: Khởi tạo Database trong AWS Glue**

1. Từ thanh tìm kiếm của AWS Console, gõ **Glue** và chọn dịch vụ **AWS Glue**.
> ![Tìm kiếm Glue](/aws-image/setupGlue/glue1.png)
2. Ở menu bên trái, chọn **Databases** dưới mục Data Catalog và bấm **Add database**.
> ![Vào Databases](/aws-image/setupGlue/glue2.png)
3. Đặt tên database là `smart_campus_db` và bấm **Create database**.
> ![Tạo Database](/aws-image/setupGlue/glue3.png)

**Bước 2: Tạo IAM Role cho Glue Crawler**

1. Từ thanh tìm kiếm của AWS Console, gõ **IAM** và chọn dịch vụ **IAM**.
> ![Tìm kiếm IAM](/aws-image/setupGlue/glue4.png)
2. Truy cập vào **Roles** ở menu bên trái và bấm **Create role**.
> ![Vào IAM Roles](/aws-image/setupGlue/glue5.png)
3. Chọn Trusted entity type là **AWS service** và Use case là **Glue**. Bấm **Next**.
> ![Chọn Trusted entity](/aws-image/setupGlue/glue6.png)
4. Tại bước Add permissions, tìm kiếm và tick chọn policy `AmazonS3ReadOnlyAccess` để cấp quyền đọc Data Lake trên S3.
> ![Chọn S3 Policy](/aws-image/setupGlue/glue7_1.png)
5. Tiếp tục tìm kiếm và tick chọn policy `AWSGlueServiceRole`. Sau đó bấm **Next**.
> ![Chọn Glue Policy](/aws-image/setupGlue/glue7_2.png)
6. Tại bước đặt tên, nhập Role name là `AWSGlueServiceRole-SmartCampus`.
> ![Đặt tên Role](/aws-image/setupGlue/glue8_1.png)
7. Cuộn xuống kiểm tra lại danh sách các policy đã chọn và bấm **Create role**.
> ![Review and Create Role](/aws-image/setupGlue/glue8_2.png)
8. Đảm bảo Role đã được tạo thành công với thông báo hiển thị màu xanh.
> ![Role created](/aws-image/setupGlue/glue9.png)

**Bước 3: Tạo và Cấu hình Glue Crawler**

1. Quay lại trang AWS Glue, ở menu bên trái chọn **Crawlers** dưới mục Data Catalog và bấm **Create crawler**.
> ![Chọn Crawlers](/aws-image/setupGlue/glue10.png)
2. Tại phần Set crawler properties, đặt tên cho crawler (ví dụ: `smart-campus-attendance-crawler`) và bấm **Next**.
> ![Đặt tên Crawler](/aws-image/setupGlue/glue11.png)
3. Ở bước Choose data sources and classifiers, bấm **Add a data source**.
> ![Add data source](/aws-image/setupGlue/glue12.png)
4. Chọn Data source là **S3** và dán đường dẫn thư mục S3 chứa dữ liệu Data Lake của bạn vào ô **S3 path**. Bấm **Add an S3 data source**.
> ![Điền thông tin S3](/aws-image/setupGlue/glue13.png)
5. Kiểm tra lại thông tin cấu hình data source vừa thêm.
> ![Review Data Source](/aws-image/setupGlue/glue14.png)
6. Bấm **Next** để tiếp tục.
> ![Bấm Next](/aws-image/setupGlue/glue15.png)
7. Tại bước Configure security settings, chọn Existing IAM role mà bạn vừa tạo là `AWSGlueServiceRole-SmartCampus` và bấm **Next**.
> ![Chọn IAM Role](/aws-image/setupGlue/glue16.png)
8. Ở bước Set output and scheduling, chọn Target database là `smart_campus_db` và Crawler schedule chọn **On demand**. Bấm **Next**.
> ![Set output](/aws-image/setupGlue/glue17.png)
9. Xem lại toàn bộ thông tin cấu hình ở bước cuối cùng và bấm **Create crawler**.
> ![Create crawler](/aws-image/setupGlue/glue18.png)

**Bước 4: Chạy Crawler**

1. Sau khi crawler được tạo xong, bấm nút **Run crawler** để tiến hành cào dữ liệu.
> ![Run crawler](/aws-image/setupGlue/glue19.png)
2. Chờ khoảng vài chục giây (tuỳ lượng file log). Nếu trạng thái của crawler chuyển sang **Completed**, điều đó có nghĩa là quá trình tự động nội suy cấu trúc (Schema) từ các file log trên S3 đã thành công và một Table mới tương ứng đã được tạo trong Catalog!
> ![Crawler completed](/aws-image/setupGlue/glue20.png)
