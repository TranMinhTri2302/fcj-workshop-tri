---
title : "Tự động hóa CI/CD Frontend"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.8.4. </b> "
---

#### 5.8.4. Thiết lập luồng CI/CD cho Frontend với AWS CodePipeline
Thay vì mỗi lần cập nhật giao diện, bạn phải chạy lệnh build thủ công rồi kéo thả lên S3, chúng ta sẽ nhờ AWS làm việc này hoàn toàn tự động mỗi khi có code mới đẩy lên GitHub.

**Điều kiện tiên quyết:** Bạn đã đẩy thư mục mã nguồn Frontend lên một kho lưu trữ trên GitHub và trong thư mục gốc có chứa file cấu hình `buildspec.yml` dành cho Frontend.

**Bước 1: Khởi tạo CodePipeline**
1. Tìm kiếm và truy cập dịch vụ **CodePipeline** trên AWS Console.
2. Bấm **Create pipeline**.
3. **Pipeline name**: Đặt tên, ví dụ: `smart-campus-frontend-pipeline`.
4. **Service role**: Chọn **New service role** (để AWS tự động tạo quyền cho Pipeline). Bấm **Next**.
> ![Pipeline Settings](/aws-image/setupCodePipeline/pipeline26.png)

**Bước 2: Cấu hình Source (Nguồn code)**
1. **Source provider**: Chọn **GitHub (via GitHub App)**. Bấm nút **Connect to GitHub**.
> ![Connect GitHub](/aws-image/setupCodePipeline/pipeline5.png)
2. Nhập tên kết nối (Ví dụ: `github-smart-campus`) và làm theo hướng dẫn trên màn hình để cấp quyền (Authorize) cho AWS truy cập vào tài khoản GitHub của bạn.
> ![Tên kết nối](/aws-image/setupCodePipeline/pipeline6.png)
> ![Cấp quyền](/aws-image/setupCodePipeline/pipeline7.png)
3. Chọn Repository chứa code Frontend của bạn và bấm **Install & Authorize**. Sau đó bấm **Connect**.
> ![Chọn Repo](/aws-image/setupCodePipeline/pipeline8.png)
> ![Hoàn tất kết nối](/aws-image/setupCodePipeline/pipeline9.png)
4. Trở lại trang cấu hình Source, đảm bảo Repository, Branch (Ví dụ: `main`) và **CodePipeline default** được chọn. Bấm **Next**.
> ![Xác nhận Repo và Branch](/aws-image/setupCodePipeline/pipeline11.png)

**Bước 3: Cấu hình Build (Trình biên dịch)**
1. **Build provider**: Chọn **AWS CodeBuild**. Region chọn `ap-southeast-1`.
2. Bấm nút **Create project**, một cửa sổ mới sẽ hiện ra. Đặt tên project (Ví dụ: `smart-campus-frontend-build`).
> ![Tên project Build](/aws-image/setupCodePipeline/pipeline27.png)
3. **Environment**: Chọn hệ điều hành **Amazon Linux**, Runtime **Standard**, Image phiên bản mới nhất (`aws/codebuild/amazonlinux2-x86_64-standard...`).
> ![Cấu hình Environment](/aws-image/setupCodePipeline/pipeline28.png)
4. Cuộn xuống phần Buildspec, chọn **Use a buildspec file** (trỏ tới file `buildspec-frontend.yml` của frontend).
> ![File Buildspec](/aws-image/setupCodePipeline/pipeline29.png)
5. Cuộn xuống cuối bấm **Continue to CodePipeline**.
> ![Continue CodePipeline](/aws-image/setupCodePipeline/pipeline30.png)
6. Quay lại cửa sổ CodePipeline, bấm **Next**.
> ![Xác nhận bước Build](/aws-image/setupCodePipeline/pipeline31.png)

**Bước 4: Cấu hình Deploy (Triển khai)**
1. **Deploy provider**: Chọn **Amazon S3**.
2. **Region**: Trùng với Region của Bucket S3.
3. **Bucket**: Chọn tên S3 Bucket tĩnh bạn đã tạo ở bài 5.8.2 (ví dụ: `smart-campus-frontend-2026`). 
4. Đánh dấu tick vào ô **Extract file before deploy** (Rất quan trọng, để bung file nén ZIP chứa code build ra).
5. Bấm **Next**, xem lại cấu hình rồi bấm **Create pipeline**.
> ![Deploy to S3](/aws-image/setupCodePipeline/pipeline32.png)

**Bước 5: Trải nghiệm thực tế**
Mỗi lần bạn thay đổi giao diện, sửa chữ, đổi màu, rồi commit và `git push` lên nhánh `main`. Bạn vào lại CodePipeline sẽ thấy đường ống (Pipeline) tự động chạy từ Source -> Build -> Deploy.
Khi hoàn thành, tải lại trang web là giao diện mới đã lên sóng!
> ![Hoàn tất triển khai Frontend](/aws-image/setupCodePipeline/pipeline33.png)
