---
title: "Prerequisites Setup"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---


### Objective

In this section, you will prepare the AWS environment and local development environment to deploy the entire Smart Campus system in the subsequent steps.

After completing this section, you will have:

- An AWS Account with MFA enabled and billing alerts configured
- An IAM User to perform workshop operations instead of using the root user
- AWS CLI v2 configured with the correct region `ap-southeast-1`
- An IAM Role for Lambda to access DynamoDB, Rekognition, S3, EventBridge, and SQS
- A Python 3.12 environment with the necessary dependencies
- A `.env` configuration file, test images, and basic development tools

---

### Prerequisites Checklist

Before proceeding to the next step, ensure you have completed the following items:

- [ ] Valid AWS Account
- [ ] MFA enabled for the root user
- [ ] Billing Alarms created (recommended at $10, $50, $100)
- [ ] IAM User `smart-campus-admin` created
- [ ] IAM User has sufficient permissions to deploy the workshop
- [ ] AWS CLI v2 installed
- [ ] AWS CLI configured with region `ap-southeast-1`
- [ ] The `aws sts get-caller-identity` command returns the correct AWS Account
- [ ] IAM Role `SmartCampusLambdaRole` created for Lambda
- [ ] Python 3.12 installed
- [ ] Virtual environment created and activated
- [ ] Required libraries installed: `fastapi`, `mangum`, `boto3`, `pydantic`
- [ ] `.env` file created
- [ ] `.env`, `venv/`, `__pycache__/` added to `.gitignore`
- [ ] Test face images prepared
- [ ] Required Service Quotas checked or requested
- [ ] AWS connectivity test run successfully

---

**Estimated time:** 45 - 60 minutes

**Region used throughout the workshop:** `ap-southeast-1` (Singapore)

Reasons for choosing this region:

- Close to Vietnam, low latency
- Full support for Amazon Rekognition
- Availability of Amazon Bedrock Claude 3 for the AI Assistant component
- Suitable for the workshop in terms of cost and features

---

### Important Notes

- Do not use the root user to deploy workshop resources.
- Do not commit AWS Access Keys, Secret Keys, or the `.env` file to Git.
- All resources should be created in the `ap-southeast-1` region.
- Use a consistent naming convention for resources: `smart-campus-*`.
- Tag resources with `Project=SmartCampus` for easier cleanup.
- Clean up all resources after the workshop to avoid incurring costs.
- Policies in the workshop may be broad for ease of practice. In production, apply the principle of least privilege more strictly.

---

## 1. AWS Account Setup and Basic Security

### 1.1 Create an AWS Account

If you do not already have an AWS Account, visit:

```text
https://aws.amazon.com/free/
```

You will need:

- A valid email address
- A valid credit card
- A phone number for verification

AWS requires a credit card for identity verification. You will not be charged as long as you stay within the Free Tier limits and clean up resources properly.

**Recommended settings:**

| Item | Value |
|---|---|
| Primary Region | `ap-southeast-1` |
| Support Plan | Basic (Free) |
| Workshop naming | `smart-campus-*` |

---

### 1.2 Enable MFA for the Root User

MFA helps protect your account even if your password is compromised.

Steps:

1. Sign in to the AWS Console as the root user
2. Go to **IAM**
3. Select **Dashboard**
4. Under Security recommendations, select **Add MFA for root user**
5. Choose **Virtual MFA device**
6. Use Google Authenticator, Authy, or a similar app to scan the QR code
7. Enter two consecutive MFA codes
8. Select **Assign MFA**

Verify by signing out and signing back in. AWS should now require both your password and an MFA code.

---

### 1.3 Create Billing Alerts

Billing Alerts help prevent unexpected charges caused by forgotten cleanup or misconfigured resources.

Steps:

1. Go to the **Billing Dashboard**
2. Select **Billing Preferences**
3. Enable **Receive Billing Alerts**
4. Select **Manage Billing Alerts**
5. CloudWatch will open
6. Go to **Alarms**
7. Create an alarm for the `EstimatedCharges` metric

Billing metrics are located in the `us-east-1` region, so when checking billing alarms via the CLI, use `us-east-1`.

It is recommended to create 3 alarms:

| Alarm | Threshold | Action |
|---|---|---|
| Warning | $10 | Send email via SNS |
| Critical | $50 | Send email via SNS |
| Emergency | $100 | Send email and require manual investigation |

Verification:

```bash
aws cloudwatch describe-alarms \
  --region us-east-1 | grep EstimatedCharges
```

---

## 2. Create an IAM User for the Workshop

It is not recommended to use the root user to deploy resources. You will create a dedicated IAM User for this workshop.

### 2.1 Create an IAM User

Steps:

1. Go to **IAM** → **Users**
2. Select **Create user**
3. Username: `smart-campus-admin`
4. Enable **Provide user access to AWS Management Console** if you want to log in to the Console with this user
5. Choose an auto-generated password or set a strong password manually
6. Proceed to the permissions step

---

### 2.2 Attach IAM Policy for the Workshop

You can choose one of two options:

**Quick setup for the workshop:**

Attach the AWS managed policy:

```text
AdministratorAccess
```

This is simple but grants very broad permissions. Only use this in a workshop or lab account environment.

**Recommended approach:**

Create a Customer Managed Policy named:

```text
SmartCampusWorkshopPolicy
```

Policy content:

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

Then attach this policy to the `smart-campus-admin` user.

Verification:

```bash
aws iam get-user --user-name smart-campus-admin
```

---

### 2.3 Create an Access Key for AWS CLI

Steps:

1. Go to **IAM** → **Users**
2. Select the user `smart-campus-admin`
3. Select the **Security credentials** tab
4. Select **Create access key**
5. Use case: **Command Line Interface (CLI)**
6. Save the following:
   - Access Key ID
   - Secret Access Key

Do not share or commit these keys to Git.

---

## 3. Install and Configure AWS CLI v2

### 3.1 Install AWS CLI

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

Download and run the installer:

```text
https://awscli.amazonaws.com/AWSCLIV2.msi
```

Verification:

```bash
aws --version
```

Expected output:

```text
aws-cli/2.x.x Python/3.x.x ...
```

---

### 3.2 Configure AWS CLI

Run:

```bash
aws configure
```

Enter the following information:

```text
AWS Access Key ID [None]: <your-access-key-id>
AWS Secret Access Key [None]: <your-secret-access-key>
Default region name [None]: ap-southeast-1
Default output format [None]: json
```

AWS CLI will create the following files:

```text
~/.aws/credentials
~/.aws/config
```

---

### 3.3 Verify AWS Connection

Run:

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

Save your AWS Account ID, as you will need it in subsequent steps.

You can also save it as an environment variable:

```bash
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
echo $AWS_ACCOUNT_ID
```

---

## 4. Create an IAM Role for Lambda

Lambda functions require an IAM Role to access AWS services such as DynamoDB, Rekognition, S3, EventBridge, and SQS.

### 4.1 Create a Trust Policy

Create a file named `lambda-trust-policy.json`:

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

### 4.2 Create the IAM Role

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

### 4.3 Attach Managed Policies

```bash
# CloudWatch Logs, required for Lambda logging
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

### 4.4 Create an Additional Inline Policy

This inline policy is used for specific actions such as publishing events, reading from SQS, sending emails via SES, or invoking Bedrock.

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

### 4.5 Verify the IAM Role

```bash
aws iam get-role --role-name SmartCampusLambdaRole

aws iam list-attached-role-policies \
  --role-name SmartCampusLambdaRole

aws iam list-role-policies \
  --role-name SmartCampusLambdaRole
```

Save the Role ARN:

```bash
aws iam get-role \
  --role-name SmartCampusLambdaRole \
  --query 'Role.Arn' \
  --output text
```

Example:

```text
arn:aws:iam::123456789012:role/SmartCampusLambdaRole
```

---

## 5. Install Python 3.12 and Dependencies

### 5.1 Install Python 3.12

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

Download Python from:

```text
https://www.python.org/downloads/
```

During installation, select:

```text
Add Python to PATH
```

Verification:

```bash
python3.12 --version
```

Expected output:

```text
Python 3.12.x
```

---

### 5.2 Create a Project Directory and Virtual Environment

```bash
mkdir smart-campus-backend
cd smart-campus-backend

python3.12 -m venv venv
```

Activate the virtual environment:

**macOS/Linux:**

```bash
source venv/bin/activate
```

**Windows:**

```bash
venv\Scripts\activate
```

Verification:

```bash
python --version
```

Expected output:

```text
Python 3.12.x
```

---

### 5.3 Install Dependencies

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

Verification:

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

### 5.4 Create requirements.txt

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

Later, you can reinstall dependencies using:

```bash
pip install -r requirements.txt
```

---

## 6. Install Development Tools

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

Download from:

```text
https://git-scm.com/download/win
```

Configure Git:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

---

### 6.2 Visual Studio Code

VS Code is recommended:

```text
https://code.visualstudio.com/
```

Recommended extensions to install:

- Python
- AWS Toolkit
- Pylance
- GitLens
- YAML
- JSON

---

### 6.3 jq and curl

**jq** is used to process JSON when working with the AWS CLI.

**macOS:**

```bash
brew install jq
```

**Ubuntu/Debian:**

```bash
sudo apt install jq curl
```

Verification:

```bash
echo '{"status":"ok"}' | jq '.status'
```

Expected output:

```text
"ok"
```

Verify curl:

```bash
curl --version
```

---

## 7. Clone the Sample Source Code

If the workshop provides a sample source code repository, you can clone it:

```bash
git clone https://github.com/danhdct122c3/smart_campus.git
cd smart-campus
```

If a sample repository is not available, you can code each component following the instructions in the subsequent steps.

---

## 8. Configure Environment Variables

Create a `.env` file in the project root.

Replace `123456789012` with your AWS Account ID.

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

Some resources like DynamoDB tables, S3 buckets, and SQS queues will be created in later steps. In this step, you are just preparing the variable names to be used consistently.

Add files that should not be committed to `.gitignore`:

```bash
echo ".env" >> .gitignore
echo "venv/" >> .gitignore
echo "__pycache__/" >> .gitignore
echo "*.pyc" >> .gitignore
```

---

## 9. Check Service Quotas

Some AWS services have default quotas. For a small workshop, the default quotas are generally sufficient, but you should check them beforehand.

### 9.1 Amazon Rekognition

Check the current collection. If it does not exist, an error result is normal:

```bash
aws rekognition describe-collection \
  --collection-id smart-campus-faces \
  --region ap-southeast-1 2>&1 || echo "Collection does not exist yet, can be skipped at this step"
```

Quotas to note:

| Quota | Default | Note |
|---|---|---|
| Faces per collection | 20M | Sufficient for the workshop |
| SearchFacesByImage TPS | 5 | May need an increase for heavy load testing |

If you need to increase the quota:

1. Go to **Service Quotas**
2. Search for **Rekognition**
3. Select **SearchFacesByImage transactions per second**
4. Request a quota increase to approximately `50 TPS`
5. Justification: `Workshop deployment for Smart Campus attendance system`

Approval may take 24 - 48 hours.

---

### 9.2 Lambda

Default quota:

```text
1000 concurrent executions per region
```

This quota is sufficient for the workshop. If deploying to production with heavy load, you will need to request a quota increase.

---

### 9.3 DynamoDB

Default quota:

```text
256 tables per region
```

Sufficient for the workshop.

---

### 9.4 API Gateway

Default quota:

```text
10,000 requests/second
```

Sufficient for the workshop.

---

### 9.5 Amazon Bedrock

Bedrock is optional, used for the AI Assistant or advanced analytics component.

Check models:

```bash
aws bedrock list-foundation-models \
  --by-provider anthropic \
  --region ap-southeast-1
```

If you do not have access to Claude models:

1. Go to the **Amazon Bedrock Console**
2. Select **Model access**
3. Enable:
   - Claude 3 Sonnet
   - Claude 3 Haiku
4. Select **Request model access**

---

## 10. Prepare Test Images

You will need a few images to test face registration, face recognition, and liveness/spoofing detection.

Image requirements:

- Format: JPEG or PNG
- Size: less than 5MB
- Resolution: at least 640x480
- Clear face, looking straight at the camera
- Adequate lighting
- No face masks or sunglasses
- Preferably only one main face in the image

Create directories:

```bash
mkdir -p test-images/faces
mkdir -p test-images/spoofing
```

Preparation guide:

```text
test-images/faces/user1.jpg
test-images/spoofing/printed.jpg
test-images/spoofing/video.jpg
```

Where:

1. `user1.jpg`: a clear selfie for face registration
2. `printed.jpg`: a printed photo or a photo of a screen to test spoofing
3. `video.jpg`: a photo captured from a video replay to test spoofing

---

## 11. Test AWS Connectivity Using Python

Create a file named `test_aws_connection.py`:

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

Run the test:

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

If all three parts (STS, S3, and IAM) succeed, your environment is ready.

---

## 12. Cost Estimation

### 12.1 Cost Specific to This Module

| Resource | Cost |
|---|---|
| IAM User | $0 |
| IAM Role | $0 |
| AWS CLI | $0 |
| CloudWatch Billing Alarms | Approx. $0.10/alarm/month |

If you create 3 billing alarms:

```text
Total cost for this module: Approx. $0.30/month
```

Billing alarms are optional but highly recommended for cost control.

---

### 12.2 Estimated Total Workshop Cost Over 24 Hours

| Service | Usage | Cost |
|---|---|---|
| Lambda | 1000 invocations, 512MB, average 2s | $0.00 if within Free Tier |
| API Gateway | 1000 requests | $0.004 |
| DynamoDB | 3 tables, 1000 reads, 100 writes | Approx. $0.25 |
| Rekognition | 50 IndexFaces, 200 searches | Approx. $0.60 |
| S3 | 10GB storage, 500 requests | Approx. $0.25 |
| EventBridge | 1000 events | $0.00 if within Free Tier |
| SQS | 2000 requests | $0.00 if within Free Tier |
| Athena | 5 queries, 100MB scanned | $0.00 if within Free Tier |
| CloudWatch | Logs 1GB, 10 metrics | Approx. $0.50 |
| **TOTAL** | | **Approx. $1.64** |

Costs may increase if:

- You test multiple times with a large number of requests
- You do not clean up after the workshop
- You use the Bedrock AI Assistant
- You use SNS SMS
- You store many images or logs for an extended period

Recommendations:

- Set a Billing Alarm at the $10 level
- Clean up immediately after completing the workshop
- Do not run large load tests unless necessary

---

## 13. Troubleshooting

### Error: AWS CLI not found

A common cause is that the AWS CLI is not in your PATH.

```bash
export PATH=$PATH:/usr/local/bin
source ~/.bashrc
```

For macOS using Zsh:

```bash
source ~/.zshrc
```

Verify again:

```bash
aws --version
```

---

### Error: Access Denied when calling AWS APIs

Check your credentials:

```bash
cat ~/.aws/credentials
aws sts get-caller-identity
```

If the error persists:

- Check if the Access Key belongs to the user `smart-campus-admin`
- Check if the IAM policy has been attached
- Check if the region is correctly set to `ap-southeast-1`
- For some IAM operations, the user needs the corresponding `iam:*` permissions

---

### Error: Python 3.12 not found

Check where Python is installed:

```bash
which python3.12
```

If necessary, use the correct full path or create an appropriate symlink.

---

### Error: pip install permission denied

You should not install packages into the system Python. Activate the virtual environment first:

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

### Error: IAM role policy attachment fails

The cause is often that the current user lacks IAM permissions.

Resolution:

- Check that the IAM User has the `iam:AttachRolePolicy`, `iam:PutRolePolicy`, and `iam:PassRole` permissions
- For the workshop, you can temporarily attach `IAMFullAccess` or `AdministratorAccess`
- Do not use this configuration long-term in production

---

## 14. Best Practice Tips

- Save important ARNs in a separate file:
  - Lambda Role ARN
  - Table ARN
  - Queue ARN
  - Event Bus ARN
  - API Gateway Endpoint
- Use a consistent naming convention: `smart-campus-*`
- Tag resources with:
  ```text
  Project=SmartCampus
  ```
- Avoid using the Scan operation in DynamoDB unless necessary
- Clean up resources after the workshop
- You can enable CloudTrail for auditing if using the account long-term

---

## Next Step

You have completed the environment preparation for the workshop.

Proceed to [Step 3: Setup DynamoDB Tables](../5.3-dynamodb) to begin building the data layer for the Smart Campus system.