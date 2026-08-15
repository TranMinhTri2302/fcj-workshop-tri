---
title: "2. Proposal"
weight: 2
chapter: false
---

# Project Proposal

## 1. Project Overview
**Smart Campus Platform** is a comprehensive software system designed to modernize and digitize management task and check-in, check-out, leave requests for employees in at company. The project including automated facial recognition attendance (AI), Task & Attendance Management and Analytics Report, check in check out.
> ![Project Overview](/aws-image/overview/overview1.png)
Notably, the system is designed **100% Serverless on the AWS platform**, applying an Event-Driven architecture to ensure high scalability, low costs, and resilient operations.

## 2. Problem Statement
The system solves painful issues in traditional management:
- **Manual Attendance & Fraud:** Using magnetic cards or fingerprints is easily bypassed, prone to forgotten cards, or causes delays during peak hours.
- **Fragmented Data:** HR, task, and attendance data are scattered, making it difficult for Directors/Managers to view a comprehensive report.
- **High Server Operations Cost:** Internal systems usually do not have users 24/7 (evenings and weekends are often empty), yet businesses still have to pay to maintain physical servers.
- **Lack of Proactive Alerts:** Management is not promptly notified when employees are late, take leave, or when tasks become overdue.

## 3. Objectives
- **Automation & Accuracy:** Apply facial recognition AI combined with IP Whitelisting for quick, absolutely accurate, and fraud-proof attendance tracking.
- **Data Centralization (Data Lake):** Build a data pipeline (Analytics Pipeline) to collect thousands of event streams to serve real-time reporting analysis.
- **100% Cost Optimization:** Radically apply Serverless architecture (pay per API call), ensuring zero cost when no one is using the system.
- **Cloud-Standard Security:** Strict Role-Based Access Control (RBAC) and protection of sensitive data using Firewall.

## 4. Workflows & Solution Architecture

> **[OVERALL ARCHITECTURE DIAGRAM]**
> ![Architecture Diagram](/aws-image/AwsArchitecture.drawio.png)

The system is designed based on an **Event-Driven** architecture and utilizes over 15 AWS cloud services. Below are the details of the 6 core business workflows and how AWS services coordinate to solve the problems:

### 4.1. Auth & Users Workflow
- **Business Logic:** Manage the user account lifecycle, assign Role-Based Access Control (RBAC) for Admin, Manager, Staff. Force new users to change their password on first login.
> ![Workflow 1](/aws-image/overview/overview2.png)
- **AWS Services:** Use **Amazon Cognito** as the Identity Provider to issue and verify JWT Tokens. The React/Vite Frontend interface is hosted on **Amazon S3** and distributed via **Amazon CloudFront**.

### 4.2. Face Registration Workflow
- **Business Logic:** Prevent fraud by ensuring each employee is only allowed to register a single authentic face into the system.
> ![Workflow 2](/aws-image/overview/overview3.png)
- **AWS Services:** Call the `IndexFaces` API of **Amazon Rekognition** to extract biometric features and save the FaceID. Original JPEG/PNG images are strictly secured in an **Amazon S3 Private Bucket**.

### 4.3. Smart Attendance Workflow
- **Business Logic:** The check-in process is done by showing a face to the camera. The system automatically matches, checks valid timeframes, and verifies if the employee is using the correct office IP (preventing fake GPS/VPN).
> ![Workflow 3](/aws-image/overview/overview4.png)
> ![Workflow 3](/aws-image/overview/overview5.png)
- **AWS Services:** 
  - **AWS WAF (Web Application Firewall):** Blocks attendance requests originating outside the company network (IP Whitelisting).
  - **Amazon Rekognition:** Calls the `SearchFacesByImage` function to check for a match (Confidence > 95%).
  - **Amazon API Gateway & AWS Lambda:** API Gateway receives requests from the Frontend, passes them to Lambda (running FastAPI) to process logic and save the state into **Amazon DynamoDB**.

### 4.4. Event-driven Notifications Workflow
- **Business Logic:** When an employee successfully checks in or is assigned a new task, the system automatically pushes multi-channel notifications to relevant personnel without slowing down the user's experience.
> ![Workflow 4](/aws-image/overview/overview6.png)
> ![Workflow 4](/aws-image/overview/overview7.png)
- **AWS Services:** 
  - **Amazon EventBridge:** Receives events (e.g., `AttendanceRecorded`) and routes them.
  - **Amazon SQS:** Acts as a queue catching events from EventBridge, sending them to the Lambda Background Worker.
  - **Amazon SNS & Amazon SES:** Sends Push Notifications, SMS (SNS), and Email (SES) to management.

### 4.5. Task Management Workflow
- **Business Logic:** Assign tasks with strict deadlines and process Leave Requests. Management can attach confidential documents to tasks.
> ![Workflow 5](/aws-image/overview/overview8.png)
> ![Workflow 5](/aws-image/overview/overview9.png)
> ![Workflow 5](/aws-image/overview/overview10.png)
- **AWS Services:**
  - **Amazon DynamoDB:** Stores Tasks and Leaves data structures with Global Secondary Indexes (GSI) for fast querying.
  - **Amazon S3 Pre-signed URL:** Generates dynamic, time-limited links to download confidential attachments, preventing data leaks.
  - **Amazon EventBridge (Cronjob):** Runs scheduled tasks every 30 minutes to scan and alert about overdue Tasks.

### 4.6. Data Lake Analytics Workflow
- **Business Logic:** Collect massive attendance logs from campuses, aggregate data so Directors can view Performance Reports (Dashboards) comparing departments.
> ![Workflow 6](/aws-image/overview/overview1.png)
> ![Workflow 6](/aws-image/overview/overview11.png)
- **AWS Services:**
  
  - **AWS Glue (Data Catalog):** Automatically crawls the schema of JSON files on S3.
  - **Amazon Athena:** Serverless SQL query engine, reading data directly from S3 via Glue Catalog to return high-speed statistical results to the Frontend.

### 4.7. Core AWS Services Breakdown
Below is a summary table of AWS services applied in the architecture diagram:

| No. | AWS SERVICE | ROLE & TASK IN SMART CAMPUS | REASON FOR CHOICE & TECHNICAL BENEFITS |
| :---: | :--- | :--- | :--- |
| 1 | **Amazon CloudFront** | Distributes React Frontend app from S3 Bucket to users. Acts as an anchor for AWS WAF. | Accelerates page load via Edge Location caching. Automatic HTTPS support, reduces bandwidth load. |
| 2 | **AWS WAF** | Firewall protecting attendance, blocking non-office IPs. | Prevents remote attendance fraud, combats Web attacks and Spam requests. |
| 3 | **Amazon S3** | **Bucket 1:** Hosts Frontend. <br> **Bucket 2:** Stores face images & secure docs. <br> **Bucket 3:** S3 Data Lake for logs. | Cheap storage, 99.999999999% reliability. Supports S3 Presigned URL for hiding secure files. Integrates well with Athena. |
| 4 | **Amazon API Gateway** | RESTful/HTTP API gateway receiving requests from Frontend and calling AWS Lambda. | Supports Rate Limiting, built-in JWT authentication via Cognito Authorizer with zero code. |
| 5 | **AWS Lambda** | **API Handler:** Processes API logic. <br> **Workers:** Processes background Events. | Pay-As-You-Go Serverless model (only pay when code runs). Instant auto-scaling, no server management. |
| 6 | **Amazon DynamoDB** | Stores all business data (Users, Tasks, Leaves, Attendance). | Serverless NoSQL Database, millisecond response times, flexible with Global Secondary Indexes. |
| 7 | **Amazon Cognito** | Manages User Pool, login authentication, and JWT Token issuance. | No need to build custom Auth system. High security, forces password change on first login. |
| 8 | **Amazon EventBridge** | Event Bus routing events (e.g., `AttendanceRecorded`) and running Cronjobs. | Decouples modules following Event-Driven standards, making it easy to add new features. |
| 9 | **Amazon SQS** | Message Queue placed before Workers. | Ensures no data loss when errors occur. Integrates Dead Letter Queue (DLQ) for retries. |
| 10 | **Amazon Rekognition** | Matches employee faces via camera upon check-in. | Extremely powerful built-in AI, no model training time required. High accuracy (Confidence > 95%). |
| 11 | **Amazon Glue & Athena** | The pipeline of Glue & Athena for aggregating S3 attendance logs, analyzing, and querying with SQL. | Automated file batching to save S3/Athena costs. Separates OLTP and OLAP systems. |
| 12 | **AWS CodeBuild & CodePipeline** | Sets up CI/CD Pipeline to auto-build Frontend and package Lambda Backend. | Fully automated Continuous Deployment from source code. Ensures safety and consistency between releases. |

### 4.8. Architecture Assessment according to the 5 Pillars of AWS Well-Architected Framework
The entire Smart Campus Platform architecture is designed to strictly adhere to the 5 Pillars of the AWS Well-Architected Framework:

1. **Operational Excellence:** Manages the entire application lifecycle automatically using CI/CD scripts (CodeBuild/CodePipeline). Centralized monitoring of logs and event metrics via Amazon CloudWatch to detect bottlenecks early.
2. **Security:** Enforces the Principle of Least Privilege via specific IAM Roles for each Lambda function. Hides sensitive attachments using S3 Pre-signed URLs, encrypts connections using CloudFront's HTTPS/TLS, and protects the API gateway with AWS WAF combined with Cognito JWT Authorizer.
3. **Reliability:** Ensures continuous High Availability thanks to the default Multi-AZ architecture of the Serverless ecosystem. Automatic Retry mechanisms and pushing error messages into Amazon SQS's Dead-Letter Queue (DLQ) prevent attendance log loss.
4. **Performance Efficiency:** Smoothly distributes the static Frontend app via CloudFront Edge locations. Optimizes read/write data times to milliseconds with DynamoDB, while offloading the main OLTP system by pushing large queries to the Data Lake pipeline (Athena).
5. **Cost Optimization:** Radically applies the 100% Serverless Event-Driven model (only pay when the system is called). Sets S3 Lifecycle Rules to automatically downgrade storage (moving old logs to Glacier), minimizing cold storage costs.

## 5. Expected Timeline
| Week | Task Items |
| :--- | :--- |
| **Week 1-2** | Requirements analysis, system architecture design. Setup AWS network resources, CloudFront, WAF, static S3. Build basic ReactJS Frontend. |
| **Week 3-4** | Build Backend API (FastAPI) on AWS Lambda and DynamoDB. Integrate Cognito and Rekognition for facial attendance features. |
| **Week 5-6** | Design Event-Driven Architecture with EventBridge and SQS. Build Data Lake Pipeline for Analytics Reporting. |
| **Week 7-8** | Integrate CI/CD (CodeBuild, CodePipeline), Automation Testing, finalize Notification flows (SNS/SES), summary and report writing. |

## 6. Monthly Budget Estimation
The budget estimation is calculated based on the actual operating scale of a medium-sized campus: **200 employees, each checking in an average of 1 to 3 times/day**. In total, the system will process approximately **15,000 attendance checks/month** and about **150,000 API requests/month** (including task assignment, reporting, leaves).

To prove the optimization of Serverless, the estimate below is calculated **based on raw pricing (Pay-As-You-Go)** and does not rely on the AWS 12-month Free Tier package.

| AWS SERVICE | EXPECTED MONTHLY USAGE | REFERENCE PRICE (AP-SOUTHEAST-1) | MONTHLY COST (USD) |
| :--- | :--- | :--- | :---: |
| **AWS Lambda** | 150,000 API Requests + 40,000 Worker executions (Memory: 512MB, Avg: 1s) | $0.20 / 1M Requests + $0.0000166667 | **$1.37** |
| **Amazon API Gateway** | 150,000 HTTP API calls | $1.25 / 1M Requests | **$0.19** |
| **Amazon SQS** | 50,000 SQS Requests (Send & Receive) | $0.40 / 1M Requests | **$0.02** |
| **Amazon DynamoDB** | 500,000 WCU, 500,000 RCU (On-Demand Mode) + 2GB Storage | $1.25 / 1M WCU, $0.25 / 1M RCU + $0.25/GB | **$1.26** |
| **Amazon S3** | ~5GB Storage (Frontend, Images, Data Lake) + 100k GET/PUT | $0.025 / GB Storage + $0.005 / 1k PUT | **$0.625** |
| **Amazon CloudFront** | 20GB Data Transfer Out + 200k HTTPS Requests | $0.120 / GB + $0.012 / 10k Req | **$2.64** |
| **AWS WAF** | 1 Web ACL + 1 Rule (IP Match) + 150k Requests | $5.00/Web ACL + $1.00/Rule + $0.60/1M Req | **$6.09** |
| **Amazon Cognito** | Under 1,000 MAU (Monthly Active Users) | Free (Forever under 50,000 MAU) | **$0.00** |
| **Amazon Rekognition** | 15,000 face matching scans (SearchFacesByImage) | $0.001 / Scan | **$15.00** |
| **Amazon Athena** | ~1GB Data Scanned by Athena query | $5.00/TB Scanned | **$0.05** |
| **Amazon CloudWatch** | 1GB Ingestion Logs + 3 Custom Metrics | $0.57 / GB Logs | **$1.47** |
| **AWS CodeBuild & CodePipeline** | ~100 build minutes (linux-small) + 1 Active Pipeline | $0.005 / min + $1.00/Pipeline | **$1.50** |
| **TOTAL** | **Smart Campus Operation Cost (200 Users)** | | **~ $30.22 / month** |

### 6.1. Cost Optimization Strategies
Although the baseline operational cost is already very low, the system employs additional in-depth optimization strategies:
1. **Pure Serverless Pay-As-You-Go Model:** Using AWS Lambda and **API Gateway HTTP API** (71% cheaper than REST API) ensures the system incurs zero server maintenance costs during nights or weekends.

2. **Using SQS Long Polling:** Configuring `ReceiveMessageWaitTimeSeconds = 20` minimizes Empty Receive Requests to SQS, significantly saving API call costs.
3. **AWS Lambda Power Tuning:** Performing probing for the most optimal RAM level to balance response speed (Latency) and execution cost, ensuring Lambda is not over-provisioned with memory causing waste.

## 7. Risk Assessment & Mitigations

| No. | RISK TYPE | DETAILED RISK ANALYSIS | LEVEL | MITIGATION STRATEGY |
| :---: | :--- | :--- | :---: | :--- |
| 1 | **Performance** | **API Bottleneck or Lambda Cold Start:** When hundreds of employees rush to check-in at 8:00 AM, Lambda latency can spike (Cold Start). | **HIGH** | - Setup **Provisioned Concurrency** for critical Lambda functions during peak hours.<br>- Use SQS as a Buffer to absorb traffic spikes, processing asynchronously. |
| 2 | **Security** | **API Spam / Fraud Attacks:** Malicious actors continuously send spam requests exhausting the AWS budget (Financial Exhaustion) or use fake images. | **CRITICAL** | - Enable **AWS WAF** combined with Rate Limiting and blocking unknown IPs.<br>- Require JWT verification via Cognito Authorizer before processing.<br>- Extreme Least Privilege configuration for each Lambda Role. |
| 3 | **Operations** | **Data Loss:** While the system is processing attendance, a Lambda Worker times out or crashes unexpectedly. | **MEDIUM** | - Configure appropriate SQS `VisibilityTimeout`.<br>- Enable **Dead-Letter Queue (DLQ)** to catch messages failing more than 3 times, allowing engineers to review without losing attendance logs. |
| 4 | **Cost Management** | **Spike Cost:** Infinite loop errors in Lambda code or logging too many errors to CloudWatch. | **MEDIUM** | - Setup **AWS Budgets Alert** to automatically warn via Email/Slack when costs exceed $40 USD/month.<br>- Configure CloudWatch Log Retention to a maximum of 14 days instead of indefinite. |

## 8. Expected Outcomes

Upon full deployment, the **Smart Campus** system is expected to achieve the following technical indicators and business objectives:

**Technical KPIs:**
- **Availability SLA:** Achieve at least **99.9%** stable uptime thanks to AWS's Serverless Multi-AZ infrastructure.
- **API Latency:** **< 200ms** for standard data read/write tasks via API Gateway & DynamoDB.
- **AI Processing Time:** **< 2.0 seconds** from sending a facial image until receiving attendance results.
- **Concurrent Capacity:** Smoothly process a minimum of **500 concurrent attendance requests** without dropped requests or system congestion.

**Business Outcomes:**
- **Cost Optimization:** Save over **80%** of infrastructure operation costs compared to renting traditional servers (EC2/VPS), thanks to the serverless (Pay-as-you-go) model.

- **Superior User Experience:** Fully digitizes paperwork, providing a smart, modern, and transparent work environment for all personnel.