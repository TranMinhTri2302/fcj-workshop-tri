---
title : "Triển khai AWS Lambda"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

#### 5.5.2. Triển khai AWS Lambda (Core Logic)
Hàm Lambda này sẽ chịu trách nhiệm toàn bộ logic xử lý API của hệ thống (Nhận diện khuôn mặt, ghi log DynamoDB, lưu ảnh S3...). Vì mã nguồn có sử dụng các thư viện ngoài như `FastAPI`, `mangum`, `boto3`, nên chúng ta không thể code trực tiếp trên AWS Console mà phải đóng gói mã nguồn từ máy tính cá nhân.

**Bước 1: Đóng gói mã nguồn (Packaging)**
1. Mở Terminal (PowerShell) trên máy tính của bạn.
2. Chạy đoạn script sau để cài đặt các thư viện vào thư mục `dist` và nén thành file `lambda_function.zip`:
   ```powershell
   # 1. Di chuyển vào đúng thư mục backend
   cd d:\AWS\smart-campus\backend

   # 2. Xóa thư mục dist cũ (nếu có)
   if (Test-Path "dist") { Remove-Item -Recurse -Force "dist" }

   # 3. Tạo thư mục dist mới
   New-Item -ItemType Directory -Force -Path "dist"

   # 4. Cài đặt các thư viện vào thư mục dist
   pip install -r requirements.txt -t ./dist

   # 5. Copy thư mục app vào dist
   Copy-Item -Path "app" -Destination "dist" -Recurse

   # 6. Nén toàn bộ thư mục dist thành file lambda_function.zip
   Compress-Archive -Path "dist\*" -DestinationPath "lambda_function.zip" -CompressionLevel Optimal
   ```


**Bước 2: Tạo hàm Lambda trên AWS**
1. Tìm kiếm và truy cập dịch vụ **Lambda** trên AWS Console.
> ![Tìm kiếm Lambda](/aws-image/setupLambda/lambda3.png)
2. Tại trang chủ của Lambda, bấm chọn nút màu cam **Create a function**.
> ![Tạo hàm Lambda](/aws-image/setupLambda/lambda4.png)
3. Trong màn hình khởi tạo, chọn **Author from scratch**. Điền **Function name** là `smart-campus-api`. Ở mục **Runtime**, chọn `Python 3.12`. Cuối cùng cuộn xuống dưới và bấm **Create function**.
> ![Điền thông tin Lambda](/aws-image/setupLambda/lambda5.png)

**Bước 3: Upload mã nguồn và cấu hình Handler**
1. Tại màn hình của hàm vừa tạo, ở tab **Code**, bấm **Upload from > .zip file** và chọn file `lambda_function.zip` vừa tạo ở Bước 1.
> ![Upload Zip](/aws-image/setupLambda/lambda6.png)
2. Cuộn xuống phần **Runtime settings**, bấm nút **Edit**.
> ![Edit Runtime settings](/aws-image/setupLambda/lambda9.png)
3. Đổi **Handler** thành `app.main.handler` (vì chúng ta dùng Mangum để bọc FastAPI ứng dụng). Bấm **Save**.
> ![Cấu hình Handler](/aws-image/setupLambda/lambda10.png)

**Bước 4: Cấu hình Biến môi trường (Environment Variables)**
1. Chuyển sang tab **Configuration** > **Environment variables** > **Edit**.
> ![Mở phần Edit Environment variables](/aws-image/setupLambda/lambda15.png)
2. Thêm các biến môi trường theo bảng dưới đây, lấy giá trị từ các service đã tạo ở các bước trước:

| Tên biến (Key) | Giá trị mẫu (Value) | Lấy ở đâu? |
|---|---|---|
| `ENVIRONMENT` | `cloud` | Cố định, ghi trực tiếp |
| `AWS_REGION` | `ap-southeast-1` | Cố định |
| `USERS_TABLE` | `smart-campus-users` | Tên bảng đã tạo ở mục 5.4.1 |
| `FACES_TABLE` | `smart-campus-faces` | Tên bảng đã tạo ở mục 5.4.1 |
| `ATTENDANCE_TABLE` | `smart-campus-attendance` | Tên bảng đã tạo ở mục 5.4.1 |
| `SECURITY_TABLE` | `smart-campus-security` | Tên bảng đã tạo ở mục 5.4.1 |
| `NOTIFICATIONS_TABLE` | `smart-campus-notifications` | Tên bảng đã tạo ở mục 5.4.1 |
| `FACE_COLLECTION_ID` | `smart-campus-faces` | Collection ID đã tạo ở mục 5.5.1 |
| `IMAGE_BUCKET` | `smart-campus-images-{id}` | Tên bucket đã tạo ở mục 5.4.2 |
| `EVENT_BUS_NAME` | `default` *(hoặc tên Event Bus riêng nếu có)* | Mục 5.6.4 (EventBridge) |
| `SES_SENDER_EMAIL` | `your-email@gmail.com` | Email đã Verify ở mục 5.6.1 |
| `COGNITO_USER_POOL_ID` | `ap-southeast-1_XXXXXXXXX` | Sao chép từ User Pool ID ở mục 5.3.1 |
| `COGNITO_CLIENT_ID` | `xxxxxxxxxxxxxxxxxxxx` | Sao chép từ Client ID ở mục 5.3.1 |
| `COGNITO_REGION` | `ap-southeast-1` | Cố định |
| `SECURITY_ALERT_TOPIC_ARN` | `arn:aws:sns:...:smart-campus-security` | ARN của Topic đã tạo ở mục 5.6.2 |
| `NOTIFICATION_TOPIC_ARN` | `arn:aws:sns:...:smart-campus-notifications` | ARN của Topic đã tạo ở mục 5.6.2 |

> [!IMPORTANT]
> Các biến `SECURITY_ALERT_TOPIC_ARN`, `NOTIFICATION_TOPIC_ARN` cần lấy từ **Amazon SNS** (mục 5.6.2), `SES_SENDER_EMAIL` từ **Amazon SES** (mục 5.6.1). Bạn có thể để trống hoặc điền giá trị tạm thời trước, **rồi nhớ quay lại mục này để cập nhật đầy đủ sau khi hoàn thành toàn bộ mục 5.6**, trước khi tiến hành kiểm thử ở mục 5.10.

> ![Biến môi trường](/aws-image/setupLambda/lambda17.png)
*(Bấm **Save** để lưu)*.

**Bước 5: Cấp quyền IAM Role (Bảo mật)**

1. Ở tab **Configuration** > **Permissions**, bấm vào tên Role đang có (Ví dụ: `smart-campus-api-role-...`).
> ![Mở IAM Role](/aws-image/setupLambda/lambda11.png)
2. Ở cửa sổ IAM mới, bấm **Add permissions > Attach policies** để thêm quyền.
> ![Cấu hình Quyền IAM](/aws-image/setupLambda/lambda12.png)
> ![Cấu hình Quyền IAM](/aws-image/setupLambda/lambdanew.jpg)

#### 5.5.3. Tiếp theo: Tạo API Gateway
Sau khi Lambda đã được triển khai và cấp quyền đầy đủ, hãy chuyển sang mục tiếp theo **5.5.3 Khởi tạo API Gateway** để tạo điểm tiếp nhận request và kết nối Lambda vào đường dẫn công khai.

---
Đến đây, Backend Lambda của hệ thống đã sẵn sàng! Tiếp tục sang **5.5.3** để gắn API Gateway vào.
