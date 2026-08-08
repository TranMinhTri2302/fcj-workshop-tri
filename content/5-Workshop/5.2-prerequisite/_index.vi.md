---
title: "Yêu cầu trước khi bắt đầu"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---


### Mục tiêu

Trong phần này, bạn sẽ chuẩn bị môi trường AWS và môi trường phát triển local để có thể triển khai toàn bộ hệ thống Smart Campus ở các bước tiếp theo.

Sau khi hoàn thành, bạn sẽ có:

- AWS Account đã bật MFA và cảnh báo chi phí
- IAM User để thao tác workshop thay vì dùng root user
- AWS CLI v2 đã cấu hình đúng region `ap-southeast-1`
- IAM Role cho Lambda để truy cập DynamoDB, Rekognition, S3, EventBridge, SQS
- Môi trường Python 3.12 với các dependencies cần thiết
- File cấu hình `.env`, test images và công cụ phát triển cơ bản

---

### Checklist cho các yêu cầu

Trước khi chuyển sang bước tiếp theo, hãy đảm bảo bạn đã hoàn thành các mục sau:

- [ ] Có AWS Account hợp lệ
- [ ] Root user đã bật MFA
- [ ] Đã tạo Billing Alarms, khuyến nghị ở mức $10, $50, $100
- [ ] Đã tạo IAM User `smart-campus-admin`
- [ ] IAM User có đủ quyền để triển khai workshop
- [ ] AWS CLI v2 đã được cài đặt
- [ ] AWS CLI đã cấu hình region `ap-southeast-1`
- [ ] Lệnh `aws sts get-caller-identity` trả về đúng AWS Account
- [ ] Đã tạo IAM Role `SmartCampusLambdaRole` cho Lambda
- [ ] Python 3.12 đã được cài đặt
- [ ] Virtual environment đã được tạo và kích hoạt
- [ ] Đã cài các thư viện: `fastapi`, `mangum`, `boto3`, `pydantic`
- [ ] Đã tạo file `.env`
- [ ] Đã thêm `.env`, `venv/`, `__pycache__/` vào `.gitignore`
- [ ] Đã chuẩn bị ảnh test khuôn mặt
- [ ] Đã kiểm tra hoặc request Service Quotas cần thiết
- [ ] Đã chạy test kết nối AWS thành công

---

**Thời gian ước tính:** 45 - 60 phút

**Region sử dụng trong toàn bộ workshop:** `ap-southeast-1` (Singapore)

Lý do chọn region này:

- Gần Việt Nam, độ trễ thấp
- Hỗ trợ đầy đủ Amazon Rekognition
- Có thể dùng Amazon Bedrock Claude 3 cho phần AI Assistant
- Phù hợp cho workshop về chi phí và tính năng

---

### Các lưu ý quan trọng

- Không sử dụng root user để triển khai workshop.
- Không commit AWS Access Key, Secret Key hoặc file `.env` lên Git.
- Tất cả tài nguyên nên được tạo trong region `ap-southeast-1`.
- Đặt tên resource nhất quán theo format `smart-campus-*`.
- Tag resources với `Project=SmartCampus` để dễ cleanup.
- Nên cleanup toàn bộ tài nguyên sau workshop để tránh phát sinh chi phí.
- Policy trong workshop có thể rộng để dễ thực hành. Trong production cần áp dụng least privilege nghiêm ngặt hơn.

---

## 1. Chuẩn bị AWS Account và bảo mật cơ bản

### 1.1 Tạo AWS Account

Nếu bạn chưa có AWS Account, truy cập:

```text
https://aws.amazon.com/free/
```

Bạn cần:

- Email hợp lệ
- Thẻ thanh toán hợp lệ
- Số điện thoại để xác minh

AWS yêu cầu thẻ để xác minh danh tính. Bạn sẽ không bị tính phí nếu vẫn trong giới hạn Free Tier và cleanup đúng cách.

**Thông tin khuyến nghị:**

| Mục | Giá trị |
|---|---|
| Region chính | `ap-southeast-1` |
| Support Plan | Basic, miễn phí |
| Workshop naming | `smart-campus-*` |

---

### 1.2 Bật MFA cho root user

MFA giúp bảo vệ tài khoản ngay cả khi mật khẩu bị lộ.

Các bước:

1. Đăng nhập AWS Console bằng root user
2. Vào **IAM**
3. Chọn **Dashboard**
4. Trong phần Security recommendations, chọn **Add MFA for root user**
5. Chọn **Virtual MFA device**
6. Dùng Google Authenticator, Authy hoặc ứng dụng tương tự để quét QR code
7. Nhập 2 mã MFA liên tiếp
8. Chọn **Assign MFA**

Kiểm tra bằng cách đăng xuất và đăng nhập lại. AWS phải yêu cầu cả mật khẩu và mã MFA.

---

### 1.3 Tạo Billing Alerts

Billing Alerts giúp tránh phát sinh chi phí ngoài ý muốn do quên cleanup hoặc tạo nhầm tài nguyên.

Các bước:

1. Vào **Billing Dashboard**
2. Chọn **Billing Preferences**
3. Bật **Receive Billing Alerts**
4. Chọn **Manage Billing Alerts**
5. CloudWatch sẽ được mở
6. Vào **Alarms**
7. Tạo alarm cho metric `EstimatedCharges`

Billing metrics nằm ở region `us-east-1`, vì vậy khi kiểm tra billing alarms bằng CLI, hãy dùng region `us-east-1`.

Khuyến nghị tạo 3 alarms:

| Alarm | Ngưỡng | Hành động |
|---|---|---|
| Warning | $10 | Gửi email qua SNS |
| Critical | $50 | Gửi email qua SNS |
| Emergency | $100 | Gửi email và kiểm tra thủ công |

Kiểm tra:

```bash
aws cloudwatch describe-alarms \
  --region us-east-1 | grep EstimatedCharges
```

---

## 2. Tạo IAM User cho workshop

Không nên dùng root user để triển khai tài nguyên. Bạn sẽ tạo một IAM User riêng cho workshop.

### 2.1 Tạo IAM User

Các bước:

1. Vào **IAM** → **Users**
2. Chọn **Create user**
3. Username: `smart-campus-admin`
4. Bật **Provide user access to AWS Management Console** nếu bạn muốn đăng nhập Console bằng user này
5. Chọn mật khẩu tự động hoặc tự đặt mật khẩu mạnh
6. Sang bước phân quyền

---

### 2.2 Gắn IAM Policy cho workshop

Bạn có thể chọn một trong hai cách:

**Cách nhanh cho workshop:**

Gắn AWS managed policy:

```text
AdministratorAccess
```

Cách này đơn giản nhưng quyền rất rộng. Chỉ nên dùng trong môi trường workshop hoặc tài khoản lab.

**Cách khuyến nghị hơn:**

Tạo Customer Managed Policy tên:

```text
SmartCampusWorkshopPolicy
```

Nội dung policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "SmartCampusWorkshopPermissions",
            "Effect": "Allow",
            "Action": [
                "lambda:*",
                "apigateway:*",
                "dynamodb:*",
                "rekognition:*",
                "events:*",
                "sqs:*",
                "sns:*",
                "ses:*",
                "s3:*",
                "glue:*",
                "athena:*",
                "logs:*",
                "cloudwatch:*",
                "cognito-idp:*",
                "wafv2:*",
                "bedrock:*",
                "cloudformation:*",
                "secretsmanager:*",
                "codepipeline:*",
                "codebuild:*",
                "iam:CreateRole",
                "iam:AttachRolePolicy",
                "iam:PutRolePolicy",
                "iam:PassRole",
                "iam:GetRole",
                "iam:DeleteRole",
                "iam:DetachRolePolicy",
                "iam:DeleteRolePolicy",
                "iam:ListRolePolicies",
                "iam:ListAttachedRolePolicies",
                "iam:GetUser"
            ],
            "Resource": "*"
        }
    ]
}
```

Sau đó gắn policy này vào user `smart-campus-admin`.

Kiểm tra:

```bash
aws iam get-user --user-name smart-campus-admin
```

---

### 2.3 Tạo Access Key cho AWS CLI

Các bước:

1. Vào **IAM** → **Users**
2. Chọn user `smart-campus-admin`
3. Chọn tab **Security credentials**
4. Chọn **Create access key**
5. Use case: **Command Line Interface (CLI)**
6. Lưu lại:
   - Access Key ID
   - Secret Access Key

Không chia sẻ hoặc commit các key này lên Git.

---

## 3. Cài đặt và cấu hình AWS CLI v2

### 3.1 Cài đặt AWS CLI

**macOS:**

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

**Linux:**

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

**Windows:**

Tải và chạy installer:

```text
https://awscli.amazonaws.com/AWSCLIV2.msi
```

Kiểm tra:

```bash
aws --version
```

Expected output:

```text
aws-cli/2.x.x Python/3.x.x ...
```

---

### 3.2 Cấu hình AWS CLI

Chạy:

```bash
aws configure
```

Nhập thông tin:

```text
AWS Access Key ID [None]: <your-access-key-id>
AWS Secret Access Key [None]: <your-secret-access-key>
Default region name [None]: ap-southeast-1
Default output format [None]: json
```

AWS CLI sẽ tạo các file:

```text
~/.aws/credentials
~/.aws/config
```

---

### 3.3 Kiểm tra kết nối AWS

Chạy:

```bash
aws sts get-caller-identity
```

Expected output:

```json
{
    "UserId": "AIDAXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/smart-campus-admin"
}
```

Lưu lại AWS Account ID vì bạn sẽ cần dùng trong các bước sau.

Có thể lưu vào biến môi trường:

```bash
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
echo $AWS_ACCOUNT_ID
```

---

## 4. Tạo IAM Role cho Lambda

Lambda functions cần IAM Role để truy cập các dịch vụ AWS như DynamoDB, Rekognition, S3, EventBridge và SQS.

### 4.1 Tạo trust policy

Tạo file `lambda-trust-policy.json`:

```bash
cat > lambda-trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```

---

### 4.2 Tạo IAM Role

```bash
aws iam create-role \
  --role-name SmartCampusLambdaRole \
  --assume-role-policy-document file://lambda-trust-policy.json \
  --description "Execution role for Smart Campus Lambda functions"
```

Expected output:

```json
{
    "Role": {
        "RoleName": "SmartCampusLambdaRole",
        "Arn": "arn:aws:iam::123456789012:role/SmartCampusLambdaRole",
        "CreateDate": "2026-08-06T10:00:00Z"
    }
}
```

---

### 4.3 Attach managed policies

```bash
# CloudWatch Logs, bắt buộc cho Lambda logging
aws iam attach-role-policy \
  --role-name SmartCampusLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# DynamoDB access
aws iam attach-role-policy \
  --role-name SmartCampusLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess

# Rekognition access
aws iam attach-role-policy \
  --role-name SmartCampusLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonRekognitionFullAccess

# S3 access
aws iam attach-role-policy \
  --role-name SmartCampusLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

# EventBridge access
aws iam attach-role-policy \
  --role-name SmartCampusLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEventBridgeFullAccess

# SQS access
aws iam attach-role-policy \
  --role-name SmartCampusLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonSQSFullAccess
```

---

### 4.4 Tạo inline policy bổ sung

Inline policy này dùng cho một số thao tác cụ thể như publish event, đọc SQS, gửi email SES hoặc gọi Bedrock.

```bash
cat > lambda-custom-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PutEventsToSmartCampusBus",
      "Effect": "Allow",
      "Action": [
        "events:PutEvents"
      ],
      "Resource": "arn:aws:events:ap-southeast-1:*:event-bus/smart-campus-events"
    },
    {
      "Sid": "ConsumeSmartCampusQueues",
      "Effect": "Allow",
      "Action": [
        "sqs:ReceiveMessage",
        "sqs:DeleteMessage",
        "sqs:GetQueueAttributes"
      ],
      "Resource": "arn:aws:sqs:ap-southeast-1:*:smart-campus-*"
    },
    {
      "Sid": "SendEmail",
      "Effect": "Allow",
      "Action": [
        "ses:SendEmail",
        "ses:SendRawEmail"
      ],
      "Resource": "*"
    },
    {
      "Sid": "InvokeBedrockModel",
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel"
      ],
      "Resource": "arn:aws:bedrock:ap-southeast-1::foundation-model/*"
    }
  ]
}
EOF

aws iam put-role-policy \
  --role-name SmartCampusLambdaRole \
  --policy-name SmartCampusCustomPolicy \
  --policy-document file://lambda-custom-policy.json
```

---

### 4.5 Kiểm tra IAM Role

```bash
aws iam get-role --role-name SmartCampusLambdaRole

aws iam list-attached-role-policies \
  --role-name SmartCampusLambdaRole

aws iam list-role-policies \
  --role-name SmartCampusLambdaRole
```

Lưu lại Role ARN:

```bash
aws iam get-role \
  --role-name SmartCampusLambdaRole \
  --query 'Role.Arn' \
  --output text
```

Ví dụ:

```text
arn:aws:iam::123456789012:role/SmartCampusLambdaRole
```

---

## 5. Cài đặt Python 3.12 và dependencies

### 5.1 Cài đặt Python 3.12

**macOS:**

```bash
brew install python@3.12
python3.12 --version
```

**Ubuntu/Debian:**

```bash
sudo apt update
sudo apt install python3.12 python3.12-venv python3-pip
python3.12 --version
```

**Windows:**

Tải Python từ:

```text
https://www.python.org/downloads/
```

Khi cài đặt, chọn:

```text
Add Python to PATH
```

Kiểm tra:

```bash
python3.12 --version
```

Expected output:

```text
Python 3.12.x
```

---

### 5.2 Tạo project directory và virtual environment

```bash
mkdir smart-campus-backend
cd smart-campus-backend

python3.12 -m venv venv
```

Kích hoạt virtual environment:

**macOS/Linux:**

```bash
source venv/bin/activate
```

**Windows:**

```bash
venv\Scripts\activate
```

Kiểm tra:

```bash
python --version
```

Expected output:

```text
Python 3.12.x
```

---

### 5.3 Cài đặt dependencies

```bash
pip install --upgrade pip

pip install fastapi==0.104.1
pip install mangum==0.17.0
pip install boto3==1.29.7
pip install pydantic==2.5.0
pip install pydantic-settings==2.1.0
pip install python-multipart==0.0.6
pip install uvicorn==0.24.0
pip install aws-xray-sdk==2.12.1
```

Kiểm tra:

```bash
pip list | grep -E "fastapi|mangum|boto3|pydantic"
```

Expected output:

```text
boto3                 1.29.7
fastapi               0.104.1
mangum                0.17.0
pydantic              2.5.0
pydantic-settings     2.1.0
```

---

### 5.4 Tạo requirements.txt

```bash
cat > requirements.txt <<EOF
fastapi==0.104.1
mangum==0.17.0
boto3==1.29.7
pydantic==2.5.0
pydantic-settings==2.1.0
python-multipart==0.0.6
uvicorn==0.24.0
aws-xray-sdk==2.12.1
EOF
```

Sau này có thể cài lại dependencies bằng:

```bash
pip install -r requirements.txt
```

---

## 6. Cài đặt công cụ phát triển

### 6.1 Git

**macOS:**

```bash
git --version
```

**Ubuntu/Debian:**

```bash
sudo apt install git
```

**Windows:**

Tải từ:

```text
https://git-scm.com/download/win
```

Cấu hình Git:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

---

### 6.2 Visual Studio Code

Khuyến nghị dùng VS Code:

```text
https://code.visualstudio.com/
```

Extensions nên cài:

- Python
- AWS Toolkit
- Pylance
- GitLens
- YAML
- JSON

---

### 6.3 jq và curl

**jq** dùng để xử lý JSON khi làm việc với AWS CLI.

**macOS:**

```bash
brew install jq
```

**Ubuntu/Debian:**

```bash
sudo apt install jq curl
```

Kiểm tra:

```bash
echo '{"status":"ok"}' | jq '.status'
```

Expected output:

```text
"ok"
```

Kiểm tra curl:

```bash
curl --version
```

---

## 7. Clone source code mẫu

Nếu workshop cung cấp source code mẫu, bạn có thể clone repository:

```bash
git clone https://github.com/danhdct122c3/smart_campus.git
cd smart-campus
```

Nếu không có repository mẫu, bạn có thể code từng phần theo hướng dẫn trong các bước tiếp theo.

---

## 8. Cấu hình Environment Variables

Tạo file `.env` trong project root.

Thay `123456789012` bằng AWS Account ID của bạn.

```bash
cat > .env <<EOF
# AWS Configuration
AWS_REGION=ap-southeast-1
AWS_ACCOUNT_ID=123456789012

# DynamoDB Tables
USERS_TABLE=smart-campus-users
FACES_TABLE=smart-campus-faces
ATTENDANCE_TABLE=smart-campus-attendance
TASKS_TABLE=smart-campus-tasks
LEAVES_TABLE=smart-campus-leaves
NOTIFICATIONS_TABLE=smart-campus-notifications
SECURITY_TABLE=smart-campus-security-incidents
HOLIDAYS_TABLE=smart-campus-holidays

# Rekognition
FACE_COLLECTION_ID=smart-campus-faces
LIVENESS_CONFIDENCE_THRESHOLD=80.0
FACE_SIMILARITY_THRESHOLD=80.0

# S3 Buckets
IMAGE_BUCKET=smart-campus-images-123456789012
DATA_LAKE_BUCKET=smart-campus-datalake-123456789012

# EventBridge
EVENT_BUS_NAME=smart-campus-events

# SQS Queues
ANALYTICS_QUEUE_URL=https://sqs.ap-southeast-1.amazonaws.com/123456789012/smart-campus-analytics-queue
NOTIFICATION_QUEUE_URL=https://sqs.ap-southeast-1.amazonaws.com/123456789012/smart-campus-notification-queue

# API Configuration
API_TIMEOUT=30
LOG_LEVEL=INFO
EOF
```

Một số resources như DynamoDB tables, S3 buckets, SQS queues sẽ được tạo ở các bước sau. Ở bước này, bạn chỉ chuẩn bị tên biến để dùng thống nhất.

Thêm các file không được commit vào `.gitignore`:

```bash
echo ".env" >> .gitignore
echo "venv/" >> .gitignore
echo "__pycache__/" >> .gitignore
echo "*.pyc" >> .gitignore
```

---

## 9. Kiểm tra Service Quotas

Một số dịch vụ AWS có quota mặc định. Với workshop nhỏ, hầu hết quota mặc định là đủ, nhưng bạn nên kiểm tra trước.

### 9.1 Amazon Rekognition

Kiểm tra collection hiện tại, nếu chưa tồn tại thì kết quả lỗi là bình thường:

```bash
aws rekognition describe-collection \
  --collection-id smart-campus-faces \
  --region ap-southeast-1 2>&1 || echo "Collection chưa tồn tại, có thể bỏ qua ở bước này"
```

Quota cần chú ý:

| Quota | Mặc định | Ghi chú |
|---|---|---|
| Faces per collection | 20M | Đủ cho workshop |
| SearchFacesByImage TPS | 5 | Có thể cần tăng nếu test tải lớn |

Nếu cần tăng quota:

1. Vào **Service Quotas**
2. Tìm **Rekognition**
3. Chọn **SearchFacesByImage transactions per second**
4. Request quota increase lên khoảng `50 TPS`
5. Justification: `Workshop deployment for Smart Campus attendance system`

Thời gian duyệt có thể từ 24 - 48 giờ.

---

### 9.2 Lambda

Quota mặc định:

```text
1000 concurrent executions per region
```

Quota này đủ cho workshop. Nếu triển khai production có tải lớn, cần request tăng quota.

---

### 9.3 DynamoDB

Quota mặc định:

```text
256 tables per region
```

Đủ cho workshop.

---

### 9.4 API Gateway

Quota mặc định:

```text
10,000 requests/second
```

Đủ cho workshop.

---

### 9.5 Amazon Bedrock

Bedrock là optional, dùng cho phần AI Assistant hoặc analytics nâng cao.

Kiểm tra models:

```bash
aws bedrock list-foundation-models \
  --by-provider anthropic \
  --region ap-southeast-1
```

Nếu chưa có quyền dùng Claude models:

1. Vào **Amazon Bedrock Console**
2. Chọn **Model access**
3. Enable:
   - Claude 3 Sonnet
   - Claude 3 Haiku
4. Chọn **Request model access**

---

## 10. Chuẩn bị ảnh test

Bạn cần một số ảnh để test face registration, face recognition và liveness/spoofing.

Yêu cầu ảnh:

- Format: JPEG hoặc PNG
- Size: nhỏ hơn 5MB
- Resolution: ít nhất 640x480
- Khuôn mặt rõ, nhìn thẳng camera
- Ánh sáng đầy đủ
- Không đeo khẩu trang hoặc kính râm
- Chỉ nên có một khuôn mặt chính trong ảnh

Tạo thư mục:

```bash
mkdir -p test-images/faces
mkdir -p test-images/spoofing
```

Gợi ý chuẩn bị:

```text
test-images/faces/user1.jpg
test-images/spoofing/printed.jpg
test-images/spoofing/video.jpg
```

Trong đó:

1. `user1.jpg`: selfie rõ mặt để đăng ký khuôn mặt
2. `printed.jpg`: ảnh in hoặc ảnh chụp lại từ màn hình để test spoofing
3. `video.jpg`: ảnh chụp từ video replay để test spoofing

---

## 11. Test kết nối AWS bằng Python

Tạo file `test_aws_connection.py`:

```python
# test_aws_connection.py
import boto3

def test_aws_connection():
    print("Testing AWS Connection...")
    print("=" * 50)

    try:
        sts = boto3.client("sts", region_name="ap-southeast-1")
        identity = sts.get_caller_identity()
        print(f"AWS Account: {identity['Account']}")
        print(f"User ARN: {identity['Arn']}")
    except Exception as e:
        print(f"STS Error: {e}")
        return False

    try:
        s3 = boto3.client("s3", region_name="ap-southeast-1")
        buckets = s3.list_buckets()
        print(f"S3 Access: {len(buckets['Buckets'])} buckets found")
    except Exception as e:
        print(f"S3 Error: {e}")
        return False

    try:
        iam = boto3.client("iam", region_name="ap-southeast-1")
        role = iam.get_role(RoleName="SmartCampusLambdaRole")
        print(f"IAM Role: {role['Role']['RoleName']} exists")
    except Exception as e:
        print(f"IAM Error: {e}")
        return False

    print("=" * 50)
    print("All connectivity tests passed.")
    return True

if __name__ == "__main__":
    test_aws_connection()
```

Chạy test:

```bash
python test_aws_connection.py
```

Expected output:

```text
Testing AWS Connection...
==================================================
AWS Account: 123456789012
User ARN: arn:aws:iam::123456789012:user/smart-campus-admin
S3 Access: 0 buckets found
IAM Role: SmartCampusLambdaRole exists
==================================================
All connectivity tests passed.
```

Nếu cả 3 phần STS, S3 và IAM đều thành công, môi trường đã sẵn sàng.

---

## 12. Ước tính chi phí

### 12.1 Chi phí riêng cho module này

| Resource | Chi phí |
|---|---|
| IAM User | $0 |
| IAM Role | $0 |
| AWS CLI | $0 |
| CloudWatch Billing Alarms | khoảng $0.10/alarm/tháng |

Nếu tạo 3 billing alarms:

```text
Tổng chi phí module này: khoảng $0.30/tháng
```

Billing alarms là optional nhưng rất nên bật để kiểm soát chi phí.

---

### 12.2 Ước tính chi phí toàn workshop trong 24 giờ

| Service | Usage | Cost |
|---|---|---|
| Lambda | 1000 invocations, 512MB, trung bình 2s | $0.00 nếu trong Free Tier |
| API Gateway | 1000 requests | $0.004 |
| DynamoDB | 3 tables, 1000 reads, 100 writes | khoảng $0.25 |
| Rekognition | 50 IndexFaces, 200 searches | khoảng $0.60 |
| S3 | 10GB storage, 500 requests | khoảng $0.25 |
| EventBridge | 1000 events | $0.00 nếu trong Free Tier |
| SQS | 2000 requests | $0.00 nếu trong Free Tier |
| Athena | 5 queries, 100MB scanned | $0.00 nếu trong Free Tier |
| CloudWatch | Logs 1GB, 10 metrics | khoảng $0.50 |
| **TOTAL** | | **khoảng $1.64** |

Chi phí có thể tăng nếu:

- Test nhiều lần với số lượng request lớn
- Không cleanup sau workshop
- Sử dụng Bedrock AI Assistant
- Sử dụng SNS SMS
- Lưu nhiều ảnh hoặc log trong thời gian dài

Khuyến nghị:

- Đặt Billing Alarm ở mức $10
- Cleanup ngay sau khi hoàn thành workshop
- Không chạy test tải lớn nếu không cần thiết

---

## 13. Troubleshooting

### Lỗi: AWS CLI not found

Nguyên nhân thường gặp là AWS CLI chưa nằm trong PATH.

```bash
export PATH=$PATH:/usr/local/bin
source ~/.bashrc
```

Với macOS dùng Zsh:

```bash
source ~/.zshrc
```

Kiểm tra lại:

```bash
aws --version
```

---

### Lỗi: Access Denied khi gọi AWS API

Kiểm tra credentials:

```bash
cat ~/.aws/credentials
aws sts get-caller-identity
```

Nếu vẫn lỗi:

- Kiểm tra Access Key có thuộc user `smart-campus-admin` không
- Kiểm tra IAM policy đã được gắn chưa
- Kiểm tra region có đúng `ap-southeast-1` không
- Với một số thao tác IAM, user cần quyền `iam:*` tương ứng

---

### Lỗi: Python 3.12 not found

Kiểm tra Python nằm ở đâu:

```bash
which python3.12
```

Nếu cần, dùng đúng đường dẫn đầy đủ hoặc tạo symlink phù hợp.

---

### Lỗi: pip install bị permission denied

Không nên cài package vào system Python. Hãy kích hoạt virtual environment trước:

```bash
source venv/bin/activate
pip install <package-name>
```

Windows:

```bash
venv\Scripts\activate
pip install <package-name>
```

---

### Lỗi: IAM role policy attachment fails

Nguyên nhân thường là user hiện tại thiếu quyền IAM.

Cách xử lý:

- Kiểm tra IAM User đã có quyền `iam:AttachRolePolicy`, `iam:PutRolePolicy`, `iam:PassRole`
- Với workshop, có thể tạm gắn `IAMFullAccess` hoặc `AdministratorAccess`
- Không nên dùng cấu hình này lâu dài trong production

---

## 14. Một số mẹo để thực hành tốt hơn

- Lưu lại các ARN quan trọng vào một file riêng:
  - Lambda Role ARN
  - Table ARN
  - Queue ARN
  - Event Bus ARN
  - API Gateway Endpoint
- Dùng naming convention thống nhất: `smart-campus-*`
- Tag resources với:
  ```text
  Project=SmartCampus
  ```
- Không dùng Scan operation trong DynamoDB nếu không cần thiết
- Cleanup resources sau workshop
- Có thể bật CloudTrail để audit nếu dùng tài khoản lâu dài

---

## Bước tiếp theo

Bạn đã hoàn tất phần chuẩn bị môi trường cho workshop.

Hãy chuyển sang [Bước 3: Setup DynamoDB Tables](../5.3-dynamodb) để bắt đầu xây dựng tầng dữ liệu cho hệ thống Smart Campus.