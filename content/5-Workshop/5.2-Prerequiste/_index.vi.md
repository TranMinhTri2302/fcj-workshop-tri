---
title : "Chuẩn bị tài nguyên"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 5.2. Chuẩn bị tài nguyên

Để bắt đầu triển khai hệ thống **Smart Campus Platform**, bạn cần chuẩn bị các công cụ và tài nguyên cơ bản trên môi trường AWS.

### 1. Tài khoản AWS (AWS Account)
- Bạn cần một tài khoản AWS với quyền quản trị viên (`AdministratorAccess`).
- Nếu bạn sử dụng tài khoản mới tạo (Free Tier), hệ thống Serverless này được thiết kế để hoàn toàn nằm trong giới hạn miễn phí của AWS, đảm bảo bạn không phát sinh chi phí trong quá trình thực hành.
- **Region khuyên dùng:** Chọn khu vực `ap-southeast-1` (Singapore) để có độ trễ thấp nhất về Việt Nam.

### 2. Chuẩn bị IAM Role cơ bản
Trong hệ thống này, các dịch vụ AWS cần giao tiếp với nhau (Ví dụ: Lambda gọi Rekognition, API Gateway gọi Lambda). Để đảm bảo nguyên tắc **Đặc quyền tối thiểu (Least Privilege)**, chúng ta sẽ tạo các IAM Role cụ thể ở từng bước thực hành. Tuy nhiên, trước mắt bạn cần hiểu nguyên tắc:
- Không sử dụng Access Key / Secret Key nhúng cứng (hard-code) vào code.
- Tất cả quyền giao tiếp đều được cấp phát qua **IAM Role**.

Dưới đây là một JSON Policy tổng hợp (dùng cho Role triển khai dự án qua CloudFormation/SAM). Policy này đã được bổ sung thêm các quyền quản lý WAF, CloudWatch, X-Ray và CI/CD (CodeBuild/CodePipeline) để đảm bảo không gặp lỗi "Access Denied" khi triển khai:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CloudFormationAndIAM",
            "Effect": "Allow",
            "Action": [
                "cloudformation:*",
                "iam:CreateRole",
                "iam:DeleteRole",
                "iam:GetRole",
                "iam:PassRole",
                "iam:AttachRolePolicy",
                "iam:DetachRolePolicy",
                "iam:PutRolePolicy",
                "iam:DeleteRolePolicy",
                "iam:GetRolePolicy",
                "iam:ListRolePolicies",
                "iam:ListAttachedRolePolicies",
                "iam:ListRoles",
                "iam:CreatePolicy",
                "iam:DeletePolicy",
                "iam:GetPolicy",
                "iam:GetPolicyVersion",
                "iam:ListPolicyVersions",
                "iam:CreatePolicyVersion",
                "iam:DeletePolicyVersion"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "ap-southeast-1"
                }
            }
        },
        {
            "Sid": "LambdaAndAPIGateway",
            "Effect": "Allow",
            "Action": [
                "lambda:*",
                "apigateway:*"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "ap-southeast-1"
                }
            }
        },
        {
            "Sid": "S3FrontendOnly",
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket",
                "s3:DeleteBucket",
                "s3:PutBucketPolicy",
                "s3:PutBucketWebsite",
                "s3:PutBucketCORS",
                "s3:PutBucketVersioning",
                "s3:PutBucketPublicAccessBlock",
                "s3:DeleteBucketPolicy",
                "s3:GetBucketLocation",
                "s3:GetBucketPolicy",
                "s3:GetBucketWebsite",
                "s3:ListBucket",
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject",
                "s3:ListBucketVersions",
                "s3:DeleteObjectVersion"
            ],
            "Resource": [
                "arn:aws:s3:::smart-campus-*",
                "arn:aws:s3:::smart-campus-*/*",
                "arn:aws:s3:::aws-sam-cli-managed-default-samclisourcebucket-*",
                "arn:aws:s3:::aws-sam-cli-managed-default-samclisourcebucket-*/*"
            ]
        },
        {
            "Sid": "DynamoDBTablesOnly",
            "Effect": "Allow",
            "Action": [
                "dynamodb:CreateTable",
                "dynamodb:DeleteTable",
                "dynamodb:DescribeTable",
                "dynamodb:UpdateTable",
                "dynamodb:GetItem",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem",
                "dynamodb:DeleteItem",
                "dynamodb:Query",
                "dynamodb:Scan",
                "dynamodb:BatchGetItem",
                "dynamodb:BatchWriteItem",
                "dynamodb:DescribeTimeToLive",
                "dynamodb:UpdateTimeToLive",
                "dynamodb:TagResource",
                "dynamodb:UntagResource",
                "dynamodb:ListTagsOfResource"
            ],
            "Resource": "arn:aws:dynamodb:ap-southeast-1:*:table/smart-campus-*"
        },
        {
            "Sid": "SQSQueuesOnly",
            "Effect": "Allow",
            "Action": [
                "sqs:CreateQueue",
                "sqs:DeleteQueue",
                "sqs:GetQueueAttributes",
                "sqs:SetQueueAttributes",
                "sqs:GetQueueUrl",
                "sqs:ListQueues",
                "sqs:TagQueue",
                "sqs:UntagQueue",
                "sqs:ListQueueTags",
                "sqs:PurgeQueue"
            ],
            "Resource": "arn:aws:sqs:ap-southeast-1:*:smart-campus-*"
        },
        {
            "Sid": "EventBridgeOnly",
            "Effect": "Allow",
            "Action": [
                "events:PutRule",
                "events:DeleteRule",
                "events:DescribeRule",
                "events:EnableRule",
                "events:DisableRule",
                "events:PutTargets",
                "events:RemoveTargets",
                "events:ListTargetsByRule",
                "events:TagResource",
                "events:UntagResource",
                "events:CreateEventBus",
                "events:DeleteEventBus",
                "events:DescribeEventBus",
                "events:PutEvents"
            ],
            "Resource": [
                "arn:aws:events:ap-southeast-1:*:rule/smart-campus-*",
                "arn:aws:events:ap-southeast-1:*:event-bus/smart-campus-*"
            ]
        },
        {
            "Sid": "CloudFrontOnly",
            "Effect": "Allow",
            "Action": [
                "cloudfront:CreateDistribution",
                "cloudfront:DeleteDistribution",
                "cloudfront:GetDistribution",
                "cloudfront:UpdateDistribution",
                "cloudfront:CreateOriginAccessControl",
                "cloudfront:DeleteOriginAccessControl",
                "cloudfront:GetOriginAccessControl",
                "cloudfront:CreateResponseHeadersPolicy",
                "cloudfront:DeleteResponseHeadersPolicy",
                "cloudfront:GetResponseHeadersPolicy",
                "cloudfront:CreateInvalidation",
                "cloudfront:GetInvalidation",
                "cloudfront:ListDistributions",
                "cloudfront:ListOriginAccessControls",
                "cloudfront:ListResponseHeadersPolicies",
                "cloudfront:TagResource",
                "cloudfront:UntagResource"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "ap-southeast-1"
                }
            }
        },
        {
            "Sid": "LogsAndECR",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:DeleteLogGroup",
                "logs:DescribeLogGroups",
                "logs:PutRetentionPolicy",
                "logs:TagResource",
                "logs:UntagResource",
                "logs:ListTagsForResource",
                "ecr:CreateRepository",
                "ecr:DeleteRepository",
                "ecr:DescribeRepositories",
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage",
                "ecr:PutImage",
                "ecr:InitiateLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:CompleteLayerUpload"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "ap-southeast-1"
                }
            }
        },
        {
            "Sid": "ManagedServicesReadWrite",
            "Effect": "Allow",
            "Action": [
                "rekognition:*",
                "bedrock:InvokeModel",
                "bedrock:ListFoundationModels",
                "athena:StartQueryExecution",
                "athena:GetQueryExecution",
                "athena:GetQueryResults",
                "athena:ListQueryExecutions",
                "firehose:PutRecord",
                "firehose:PutRecordBatch",
                "firehose:CreateDeliveryStream",
                "firehose:DeleteDeliveryStream",
                "firehose:DescribeDeliveryStream",
                "firehose:ListDeliveryStreams",
                "sns:CreateTopic",
                "sns:DeleteTopic",
                "sns:GetTopicAttributes",
                "sns:SetTopicAttributes",
                "sns:Publish",
                "sns:Subscribe",
                "sns:Unsubscribe",
                "sns:ListSubscriptionsByTopic",
                "sns:TagResource",
                "sns:UntagResource",
                "cognito-idp:CreateUserPool",
                "cognito-idp:DeleteUserPool",
                "cognito-idp:DescribeUserPool",
                "cognito-idp:UpdateUserPool",
                "cognito-idp:CreateUserPoolClient",
                "cognito-idp:DeleteUserPoolClient",
                "cognito-idp:DescribeUserPoolClient",
                "cognito-idp:AdminCreateUser",
                "cognito-idp:AdminDeleteUser",
                "cognito-idp:AdminSetUserPassword",
                "cognito-idp:AdminUpdateUserAttributes",
                "cognito-idp:ListUsers",
                "cognito-idp:InitiateAuth",
                "cognito-idp:RespondToAuthChallenge",
                "cognito-idp:SignUp",
                "cognito-idp:ConfirmSignUp",
                "ses:VerifyEmailIdentity",
                "ses:DeleteIdentity",
                "ses:GetIdentityVerificationAttributes",
                "ses:SendEmail",
                "ses:SendTemplatedEmail",
                "ses:ListIdentities",
                "glue:GetTable",
                "glue:GetDatabase",
                "glue:CreateTable",
                "glue:DeleteTable",
                "glue:GetPartitions",
                "quicksight:*"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "ap-southeast-1"
                }
            }
        },
        {
            "Sid": "CIAndGlobalServices",
            "Effect": "Allow",
            "Action": [
                "codebuild:*",
                "codepipeline:*",
                "cloudwatch:*",
                "xray:*",
                "wafv2:*",
                "acm:*",
                "ssm:*"
            ],
            "Resource": "*"
        }
    ]
}
```

### 3. Cài đặt các công cụ (Tools) tại máy tính nội bộ
Mặc dù bạn có thể cấu hình toàn bộ hệ thống bằng giao diện (AWS Console), việc cài đặt các công cụ dưới đây sẽ giúp bạn test API và quản lý source code dễ dàng hơn:
- **Visual Studio Code (VSCode):** Để đọc và chỉnh sửa mã nguồn Frontend (React) và Backend (Python/FastAPI).
- **Postman** : Dùng để test các API Endpoint mà chúng ta sắp tạo trên Amazon API Gateway.
- **Git:** Cần thiết để push source code lên kho lưu trữ và tích hợp với AWS CodePipeline sau này.

### 4. Source Code dự án
Vui lòng clone (tải về) mã nguồn chuẩn của dự án Smart Campus về máy tính cá nhân của bạn để sử dụng cho các bước tiếp theo:

```bash
git clone https://github.com/danhdct122c3/WorkShopAWS.git
cd smart-campus
```
*(Cấu trúc thư mục source code sẽ bao gồm 2 phần chính: `/frontend` chứa code ReactJS và `/backend`)*

