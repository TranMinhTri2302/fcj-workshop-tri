---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01
weight : 11
chapter : false
pre : " <b> 5.11. </b> "
---

#### 5.11. Dọn dẹp tài nguyên (Clean-up)

> Sau khi hoàn thành Workshop, hãy xóa tất cả tài nguyên đã tạo để **tránh phát sinh chi phí** không mong muốn trên tài khoản AWS của bạn.

Dưới đây là danh sách tài nguyên cần xóa theo thứ tự an toàn (xóa tài nguyên phụ thuộc trước):

---

**Bước 1: Xóa CloudFront Distributions**

1. Vào **CloudFront** > **Distributions**.
2. Chọn distribution `smart-campus-frontend` (từ bài 5.8.3):
   - Bấm **Disable** và chờ trạng thái chuyển sang `Disabled`.
> ![Disable CloudFront](/aws-image/setupCleanUp/cleanupcloudfront1.png)
   - Sau đó bấm **Delete**.
> ![Delete CloudFront](/aws-image/setupCleanUp/cleanupcloudfront2.png)
3. Làm tương tự với distribution `smart-campus-api-cf` (từ bài 5.5.4).

---

**Bước 2: Xóa CodePipeline và CodeBuild**

1. Vào **CodePipeline** > Chọn `smart-campus-backend-pipeline` > Bấm **Delete pipeline**.
> ![Delete Pipeline](/aws-image/setupCleanUp/cleanupipeline1.png)
2. Xóa tiếp `smart-campus-frontend-pipeline`.
3. Vào **CodeBuild** > Xóa các project `smart-campus-backend-build` và `smart-campus-frontend-build`.
> ![Delete CodeBuild](/aws-image/setupCleanUp/cleanupcodebuild1.png)

---

**Bước 3: Xóa API Gateway**

1. Vào **API Gateway** > Chọn API `SmartCampusHTTPApi` > Bấm **Delete**.
> ![Delete API 1](/aws-image/setupCleanUp/cleanupapi1.png)
> ![Delete API 2](/aws-image/setupCleanUp/cleanupapi2.png)

---

**Bước 4: Xóa Lambda Functions**

1. Vào **Lambda** > **Functions**.
2. Xóa lần lượt:
   - `smart-campus-api`
   - `smart-campus-analytics-worker`
> ![Delete Lambda 1](/aws-image/setupCleanUp/cleanuplambda1.png)
> ![Delete Lambda 2](/aws-image/setupCleanUp/cleanuplambda2.png)

---

**Bước 5: Xóa EventBridge Rules**

1. Vào **Amazon EventBridge** > **Rules**.
2. Xóa `attendance-recorded-to-sns` và `attendance-to-sqs`.
> ![Delete EventBridge](/aws-image/setupCleanUp/cleanupevenbridge1.png)

---

**Bước 6: Xóa SQS Queues**

1. Vào **SQS** > Xóa `smart-campus-analytics-queue`.
2. Xóa tiếp `smart-campus-dlq`.
> ![Delete SQS](/aws-image/setupCleanUp/cleanupsqs1.png)

---

**Bước 7: Xóa SNS Topics**

1. Vào **SNS** > **Topics** > Xóa `smart-campus-notifications`.
> ![Delete SNS 1](/aws-image/setupCleanUp/cleanupsns1.png)
2. Vào **Subscriptions** > Xóa các Subscription còn liên quan.
> ![Delete SNS 2](/aws-image/setupCleanUp/cleanupsns2.png)

---

**Bước 8: Xóa CloudWatch Alarms và Log Groups**

1. Vào **CloudWatch** > **Alarms** > Xóa alarm `Lambda-Error-Alert`.
> ![Delete Alarms 1](/aws-image/setupCleanUp/cleanupcloudwatch1.png)
> ![Delete Alarms 2](/aws-image/setupCleanUp/cleanupcloudwatch5.png)
2. Vào **Log groups** > Xóa `/aws/lambda/smart-campus-api` và `/aws/lambda/smart-campus-analytics-worker`.
> ![Delete Log Groups 1](/aws-image/setupCleanUp/cleanupcloudwatch2.png)
> ![Delete Log Groups 2](/aws-image/setupCleanUp/cleanupcloudwatch3.png)
> ![Delete Log Groups 3](/aws-image/setupCleanUp/cleanupcloudwatch4.png)
3. Vào **WAF Logs** > Xóa log group `aws-waf-logs-smartcampus`.

---

**Bước 9: Xóa WAF Web ACL**

1. Vào **WAF & Shield** > **Web ACLs** (chọn scope Global/CloudFront).
2. Xóa `SmartCampusAPIWebACL`.
> ![Delete WAF 1](/aws-image/setupCleanUp/cleanupwaf1.png)
3. Vào **IP sets** > Xóa `SmartCampusIPSet`.
> ![Delete WAF 2](/aws-image/setupCleanUp/cleanupwaf2.png)

---

**Bước 10: Xóa DynamoDB Tables**

1. Vào **DynamoDB** > **Tables**.
2. Xóa lần lượt tất cả các bảng: `smart-campus-attendance`, `smart-campus-faces`, `smart-campus-users`, `smart-campus-security`, `smart-campus-notifications`, `smart-campus-settings`, `smart-campus-tasks`, `smart-campus-leaves`, `smart-campus-holidays`.
> ![Delete DynamoDB](/aws-image/setupCleanUp/cleanupdynamodb1.png)

---

**Bước 11: Xóa S3 Buckets**

> [!WARNING]
> Phải **xóa toàn bộ object bên trong** trước khi có thể xóa bucket.

1. Vào **S3**, với mỗi bucket sau:
   - `smart-campus-images-{id}`
   - `smart-campus-frontend-2026`
   - `smart-campus-datalake-{id}`
   - Bucket lưu kết quả Athena
2. Vào trong bucket > Chọn tất cả object > **Delete**.
> ![Empty S3 1](/aws-image/setupCleanUp/cleanups31.png)
> ![Empty S3 2](/aws-image/setupCleanUp/cleanups32.png)
3. Sau đó quay ra xóa bucket.
> ![Delete S3 3](/aws-image/setupCleanUp/cleanups33.png)

---

**Bước 12: Xóa Rekognition Collection**

Mở **AWS CloudShell** và chạy lệnh:
```bash
aws rekognition delete-collection --collection-id smart-campus-faces --region ap-southeast-1
```

---

**Bước 13: Xóa Cognito User Pool**

1. Vào **Amazon Cognito** > **User Pools**.
2. Chọn User Pool bạn đã tạo > Bấm **Delete**.
> ![Delete Cognito](/aws-image/setupCleanUp/cleanupcognito1.png)

---

**Bước 14: Xóa IAM Roles**

1. Vào **IAM** > **Roles**.
2. Xóa các role: `smart-campus-api-role-...`, `smart-campus-analytics-worker-role-...`, `AWSGlueServiceRole-SmartCampus`, và các role được tạo tự động bởi CodeBuild/CodePipeline.
> ![Delete IAM](/aws-image/setupCleanUp/cleanupiam1.png)

---

**Bước 15: Xóa Glue Crawler và Database**

1. Vào **AWS Glue** > **Crawlers** > Xóa `smart-campus-attendance-crawler`.
> ![Delete Glue Crawler](/aws-image/setupCleanUp/cleanupglue1.png)
2. Vào **Databases** > Xóa `smart_campus_db`.
> ![Delete Glue Database](/aws-image/setupCleanUp/cleanupglue2.png)

---

**Bước 16: Xóa SES Identities**

1. Vào **Amazon SES** > **Verified identities**.
> ![SES 1](/aws-image/cleanupses/ses_cleaning_1.png)
> ![SES 2](/aws-image/cleanupses/ses_cleaning_2.png)
2. Chọn địa chỉ email đã xác thực > Bấm **Delete**.
> ![SES 3](/aws-image/cleanupses/ses_cleaning_3.png)
> ![SES 4](/aws-image/cleanupses/ses_cleaning_4.png)

---

### ✅ Kiểm tra hoàn tất

Sau khi xóa xong, vào **AWS Cost Explorer** hoặc **Billing Dashboard** để xác nhận không còn tài nguyên nào đang chạy và phát sinh chi phí. Chúc mừng bạn đã hoàn thành toàn bộ Workshop **Smart Campus Platform trên AWS**!
