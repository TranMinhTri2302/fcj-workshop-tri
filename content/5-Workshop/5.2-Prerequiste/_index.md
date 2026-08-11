---
title : "Prerequisites"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 5.2. Prerequisites

To begin deploying the **Smart Campus Platform**, you need to prepare some basic tools and resources on the AWS environment.

### 1. AWS Account
- You need an AWS account with administrator privileges (`AdministratorAccess`).
- If you are using a newly created account (Free Tier), this Serverless system is designed to fall entirely within the AWS free tier limits, ensuring you incur no costs during practice.
- **Recommended Region:** Select the `ap-southeast-1` (Singapore) region for the lowest latency to Vietnam.

### 2. Prepare Basic IAM Roles
In this system, AWS services need to communicate with each other (e.g., Lambda calls Rekognition, API Gateway calls Lambda). To ensure the **Least Privilege** principle, we will create specific IAM Roles at each practical step. However, for now, you need to understand the principle:
- Do not use hard-coded Access Keys / Secret Keys in your code.
- All communication permissions are granted via **IAM Roles**.

Below is a comprehensive JSON Policy (used for the Role deploying the project via CloudFormation/SAM). This policy has been supplemented with management permissions for WAF, CloudWatch, X-Ray, and CI/CD (CodeBuild/CodePipeline) to ensure no "Access Denied" errors occur during deployment:

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

### 3. Install Local Tools
Although you can configure the entire system using the UI (AWS Console), installing the tools below will help you test APIs and manage source code more easily:
- **Visual Studio Code (VSCode):** To read and edit Frontend (React) and Backend (Python/FastAPI) source code.
- **Postman**: Used to test the API Endpoints we are about to create on Amazon API Gateway.
- **Git:** Necessary to push source code to the repository and integrate with AWS CodePipeline later.

### 4. Project Source Code
Please clone (download) the standard source code of the Smart Campus project to your personal computer to use for the next steps:

```bash
git clone https://github.com/danhdct122c3/WorkShopAWS.git
cd smart-campus
```
*(The source code directory structure will consist of 2 main parts: `/frontend` containing ReactJS code and `/backend`)*

