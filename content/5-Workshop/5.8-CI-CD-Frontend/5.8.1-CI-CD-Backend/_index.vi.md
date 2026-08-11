---
title : "Tự động hóa CI/CD Backend"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.8.1. </b> "
---

#### 5.8.1. Thiết lập luồng CI/CD cho Backend với AWS CodePipeline
Việc triển khai các hàm Lambda (Backend) mỗi khi có code mới cũng cần được tự động hóa. Vì kiến trúc Backend sử dụng SAM (Serverless Application Model), nên quá trình CI/CD sẽ có chút khác biệt so với Frontend: Chúng ta sẽ dùng CodeBuild để chạy trực tiếp lệnh `sam build` và `sam deploy` lên AWS, và **bỏ qua bước Deploy** của Pipeline.

**Điều kiện tiên quyết:** Code Backend đã nằm trên kho lưu trữ GitHub và trong thư mục gốc của dự án **đã có file `buildspec-backend.yml`** với nội dung như bên dưới. Nếu chưa có, hãy tạo file này trước khi tiến hành:

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.12
    commands:
      - pip install --upgrade pip
      - pip install aws-lambda-powertools boto3

  build:
    commands:
      # Đóng gói thư mục dist
      - cd backend
      - pip install -r requirements.txt -t ./dist
      - cp -r app ./dist/
      # Tạo file zip để upload lên Lambda
      - cd dist && zip -r ../lambda_function.zip . && cd ..
      # Cập nhật Lambda function trực tiếp bằng AWS CLI
      - aws lambda update-function-code
          --function-name smart-campus-api
          --zip-file fileb://lambda_function.zip
          --region ap-southeast-1

artifacts:
  files:
    - backend/lambda_function.zip
```



**Bước 1: Khởi tạo CodePipeline**
1. Tìm kiếm và truy cập dịch vụ **CodePipeline** trên AWS Console.
> ![Tìm kiếm CodePipeline](/aws-image/setupCodePipeline/pipeline1.png)
2. Bấm **Create pipeline**. 
> ![Tạo Pipeline](/aws-image/setupCodePipeline/pipeline2.png)
3. Chọn **Build custom pipeline** rồi bấm **Next**.
> ![Chọn Build Custom](/aws-image/setupCodePipeline/pipeline3.png)
4. **Pipeline name**: Đặt tên, ví dụ: `smart-campus-backend-pipeline`. Chọn **New service role** và bấm **Next**.
> ![Tên Pipeline và Role](/aws-image/setupCodePipeline/pipeline4.png)

**Bước 2: Cấu hình Source (Nguồn code)**
1. **Source provider**: Chọn **GitHub (via GitHub App)**. Bấm nút **Connect to GitHub**.
> ![Connect GitHub](/aws-image/setupCodePipeline/pipeline5.png)
2. Nhập tên kết nối (VD: `github-smart-campus`) và bấm **Connect to GitHub**.
> ![Tên kết nối](/aws-image/setupCodePipeline/pipeline6.png)
3. Hệ thống sẽ mở pop-up cấp quyền, bấm **Authorize**.
> ![Cấp quyền](/aws-image/setupCodePipeline/pipeline7.png)
4. Chọn đúng Repository chứa code Backend của bạn và bấm **Install & Authorize**.
> ![Chọn Repo](/aws-image/setupCodePipeline/pipeline8.png)
5. Màn hình CodePipeline sẽ hiển thị App Installation, bấm **Connect**.
> ![Hoàn tất kết nối](/aws-image/setupCodePipeline/pipeline9.png)
6. Trở lại trang cấu hình Source, đảm bảo Repository, nhánh `main`, **CodePipeline default** và **Webhook** được cấu hình, sau đó bấm **Next**.
> ![Xác nhận Repo và Branch](/aws-image/setupCodePipeline/pipeline10.png)

**Bước 3: Cấu hình Build (CodeBuild)**
1. **Build provider**: Chọn **AWS CodeBuild**. Region chọn `ap-southeast-1`. Bấm nút **Create project**.
> ![Chọn CodeBuild](/aws-image/setupCodePipeline/pipeline11.png)
2. Trong cửa sổ mới, đặt tên Project là `smart-campus-backend-build`.
> ![Tên CodeBuild](/aws-image/setupCodePipeline/pipeline12.png)
3. **Environment**: Chọn **Amazon Linux**, Runtime **Standard**, Image `aws/codebuild/amazonlinux-x86_64-standard:6.0` (hoặc bản mới nhất). Chọn **New service role**.
> ![Cấu hình Môi trường](/aws-image/setupCodePipeline/pipeline13.png)
4. Kéo xuống phần **Buildspec**, chọn **Use a buildspec file** và nhập tên file là `buildspec-backend.yml`.
> ![File buildspec](/aws-image/setupCodePipeline/pipeline14.png)
5. Kéo xuống phần Logs, bỏ qua hoặc giữ mặc định rồi bấm **Continue to CodePipeline**.
> ![Tiếp tục](/aws-image/setupCodePipeline/pipeline15.png)
6. Khi có thông báo màu xanh tạo project thành công, cuộn xuống bấm **Next**.
> ![Xác nhận Build Stage](/aws-image/setupCodePipeline/pipeline16.png)

**Bước 4: Bỏ qua bước Test và Deploy**
Vì lệnh `sam deploy` trong CodeBuild đã tự động triển khai hạ tầng Backend (CloudFormation, Lambda, API Gateway) lên AWS rồi, nên chúng ta không cần bước Deploy của CodePipeline nữa.
1. Tại bước **Add test stage**, bấm **Skip test stage**.
> ![Bỏ qua Test](/aws-image/setupCodePipeline/pipeline17.png)
2. Tại bước **Add deploy stage**, bấm **Skip deploy stage**.
> ![Bỏ qua Deploy](/aws-image/setupCodePipeline/pipeline18.png)
3. Màn hình Review hiện ra, cuộn xuống dưới cùng và bấm **Create pipeline**.
> ![Review Stage](/aws-image/setupCodePipeline/pipeline19.png)

**Bước 5: Cấp quyền bổ sung cho Role của CodeBuild**
Lúc này Pipeline sẽ bắt đầu chạy nhưng bước Build sẽ báo lỗi đỏ vì Role của CodeBuild chưa có quyền tạo Lambda, S3, IAM, CloudFormation... Bạn cần gán thêm quyền cho nó.
1. Từ thanh tìm kiếm, gõ **IAM** và chọn dịch vụ.
> ![Tìm IAM](/aws-image/setupCodePipeline/pipeline20.png)
2. Chọn mục **Roles** bên trái, tìm Role có tên chứa `codebuild-smart-campus-backend-build` và bấm vào tên Role đó.
> ![Tìm Role CodeBuild](/aws-image/setupCodePipeline/pipeline21.png)
3. Bấm nút **Add permissions** > **Attach policies**.
> ![Attach policies](/aws-image/setupCodePipeline/pipeline22.png)
4. Tìm và tick chọn các Policy như: **AWSLambda_FullAccess**, **AmazonS3FullAccess**... (và các quyền khác tùy theo yêu cầu của ứng dụng). Sau đó bấm **Add permissions**.
> ![AWSLambda_FullAccess](/aws-image/setupCodePipeline/pipeline23.png)
> ![AmazonS3FullAccess](/aws-image/setupCodePipeline/pipeline24.png)

**Hoàn tất**
Quay lại trang CodePipeline, bấm nút **Release change** (hoặc Retry) để chạy lại. Nếu tất cả các bước đều hiện màu xanh là Backend của bạn đã tự động lên mây thành công!
> ![Hoàn tất](/aws-image/setupCodePipeline/pipelinenew.png)

