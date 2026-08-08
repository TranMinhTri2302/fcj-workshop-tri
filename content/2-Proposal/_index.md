---
title: "Project Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Smart Campus Platform — Serverless Smart Attendance System

## Building a Facial Recognition Attendance System with Event-Driven Architecture on AWS

### 1. Project Summary

Smart Campus Platform is an intelligent attendance management system designed for educational institutions and enterprises, built entirely on AWS serverless architecture. The system automates the attendance process using AI-powered facial recognition technology, combined with task management, leave management, and data analytics capabilities.

In many real-world working environments, attendance is still conducted manually or via proximity cards. Individuals can ask colleagues to swipe cards on their behalf, use another person's card, or present a printed photograph to bypass rudimentary camera systems. Compiling attendance reports also demands considerable manual processing time.

Smart Campus addresses these issues by transforming fragmented attendance procedures into a structured and automated workflow. The system leverages **Amazon Rekognition Face Recognition** in conjunction with **Face Liveness Detection** to identify faces and prevent fraud through printed photos, video replays, or 3D masks. Each attendance event is recorded with the following details: user_id, timestamp, session_type (MORNING/AFTERNOON/EVENING), status (PRESENT/LATE/ABSENT/REJECTED), confidence score, and camera_id. The system automatically detects violations such as late check-ins, duplicate entries, or out-of-hours attendance.

From an AWS and Cloud Architecture perspective, the project focuses on building, deploying, and operating a production-grade serverless application. The backend is built with Python FastAPI, running on **AWS Lambda** via the **Mangum adapter**, exposing APIs through **Amazon API Gateway**, persisting data in **Amazon DynamoDB**, processing events through **Amazon EventBridge** and **Amazon SQS**, performing data analytics with **Amazon Athena** and **AWS Glue**, monitoring with **Amazon CloudWatch**, and integrating AI capabilities with **Amazon Bedrock (Claude 3)**.

Notably, the system is architected to be **100% Serverless on the AWS platform**, employing an Event-Driven Microservices architecture to ensure high scalability, low operational cost, and resilient operations. The objective of this project extends beyond building a web application — it demonstrates a complete end-to-end cloud deployment process on AWS following industry best practices.

---

### 2. Problem Statement

#### What Is the Current Problem?

In small and medium-sized organizations, educational institutions, or office environments, attendance is frequently performed manually or informally. For example, students may ask classmates to mark attendance on their behalf, employees may use a colleague's card to check in, or individuals may present a printed photograph to circumvent basic camera systems. These practices are difficult to control and highly susceptible to fraud.

The key issues include:

* Attendance fraud is easily committed through proxy check-ins, printed photos, or shared cards.
* Manual attendance is time-consuming, taking approximately 5 to 10 minutes per session for 50 individuals.
* Tracking and aggregating attendance data is difficult.
* No real-time alerts are generated when violations occur.
* Consolidated reports require manual processing, consuming several hours.
* Task and leave management is fragmented and lacks centralization.
* Manual deployments result in version inconsistencies.
* No monitoring is in place — errors are only discovered when users report them.

#### Proposed Solution

Smart Campus delivers a fully structured and automated attendance process. Users simply stand in front of the camera for 2 to 3 seconds, and the system automatically executes the following steps:

1. **Face Liveness Detection**: Verifies whether the subject is a live person, detecting printed photos, video replays, and 3D masks.
2. **Face Recognition**: Identifies the face and matches it against the registered database.
3. **Rule Engine**: Applies business rules including duplicate checking, session validation, and time verification.
4. **Auto Record**: Automatically logs the attendance record into DynamoDB.
5. **Event Publishing**: Publishes events to trigger downstream services such as notifications, analytics, and security.

Attendance workflow:

```text
Camera → Liveness Check (80%+) → Face Recognition (80%+) → Rule Engine → Record → Events
```

Attendance statuses:

```text
PRESENT: On time, within the first 15 minutes of the session
LATE: Late, after 15 minutes but still within the session
ABSENT: Absent, no attendance recorded
REJECTED: Rejected, outside session hours or duplicate entry
```

Sessions:

```text
MORNING: 07:00 - 11:30
AFTERNOON: 13:00 - 17:30
EVENING: 18:00 - 21:00
```


#### Project Benefits

Smart Campus delivers two primary benefits.

First, the system fundamentally improves the attendance process. Each attendance instance becomes a fully traceable record with timestamp, status, and confidence score. This effectively prevents fraud and enhances process transparency. Attendance time is reduced from 5–10 minutes down to 2–3 seconds per person.

Second, the project serves as a hands-on cloud and DevOps exercise grounded in real-world practices. The project encompasses serverless architecture, event-driven patterns, AI/ML integration, data lake analytics, monitoring, security, and cost optimization — all of which are essential skills for deploying and operating applications on AWS.

---

## 3. Objectives
- **Automation & Accuracy:** Leverage AI facial recognition combined with IP Whitelisting to enable rapid, highly accurate attendance with built-in fraud prevention.
- **Data Centralization (Data Lake):** Build an Analytics Pipeline capable of ingesting thousands of event streams to support real-time reporting and analysis.
- **100% Cost Optimization:** Fully embrace Serverless architecture (pay-per-API-call), ensuring zero cost when there are no active users.
- **Cloud-Standard Security:** Enforce strict Role-Based Access Control (RBAC) and protect sensitive data through Firewall and Token-based authentication systems.

## 4. Business Workflows & Solution Architecture

> **[OVERALL ARCHITECTURE DIAGRAM]**
> <!-- TODO: Once the architecture diagram is finalized, insert the image here using the syntax: ![Architecture Diagram](/images/architecture.png) -->

The system is designed around an **Event-Driven Microservices** architecture and leverages more than 15 AWS cloud services. The following sections detail the 6 core business workflows and illustrate how AWS services collaborate to address the problem:

### 4.1. Authentication & Authorization Workflow
- **Business Logic:** Manage the full user account lifecycle and enforce Role-Based Access Control (RBAC) across Admin, Manager, and Staff roles. New users are required to change their password upon first login.
- **AWS Services:** **Amazon Cognito** serves as the Identity Provider for issuing and validating JWT Tokens. The React/Vite Frontend application is hosted on **Amazon S3** and distributed globally via **Amazon CloudFront**.

### 4.2. Face Registration Workflow
- **Business Logic:** Prevent fraud by restricting each employee to registering exactly one verified facial identity in the system.
- **AWS Services:** The `IndexFaces` API of **Amazon Rekognition** is invoked to extract biometric features and store the corresponding FaceID. Original JPEG/PNG images are securely stored in an **Amazon S3 Private Bucket**.

### 4.3. Smart Attendance Workflow
- **Business Logic:** The check-in/check-out process is performed by presenting a face to the camera. The system automatically matches the face, validates the applicable time window, and verifies whether the employee is connecting from an authorized office IP address (preventing fake GPS/VPN bypass).
- **AWS Services:**
  - **AWS WAF (Web Application Firewall):** Blocks attendance requests originating from outside the corporate network (IP Whitelisting).
  - **Amazon Rekognition:** Invokes the `SearchFacesByImage` function to verify facial match confidence (Confidence > 95%).
  - **Amazon API Gateway & AWS Lambda:** API Gateway receives requests from the Frontend and routes them to Lambda (running FastAPI) for business logic processing and status persistence in **Amazon DynamoDB**.

### 4.4. Event Processing & Notification Workflow
- **Business Logic:** When an employee successfully checks in or is assigned a new task, the system automatically dispatches multi-channel notifications to relevant stakeholders without degrading the end-user experience.
- **AWS Services:**
  - **Amazon EventBridge:** Receives events (e.g., `AttendanceRecorded`) and performs intelligent routing.
  - **Amazon SQS:** Serves as a message queue buffering events from EventBridge and forwarding them to Lambda Background Workers.
  - **Amazon SNS & Amazon SES:** Deliver Push Notifications and SMS (via SNS) as well as Emails (via SES) to management personnel.

### 4.5. Task Management Workflow
- **Business Logic:** Assign tasks with strict deadlines and process leave requests. Managers can attach confidential documents to tasks.
- **AWS Services:**
  - **Amazon DynamoDB:** Stores Task and Leave data structures with Global Secondary Indexes (GSI) for high-performance querying.
  - **Amazon S3 Pre-signed URL:** Generates time-limited dynamic links for downloading confidential attachments, preventing data leakage.
  - **Amazon EventBridge (Scheduled Rules):** Executes scheduled jobs every 30 minutes to scan for and alert on overdue tasks.

### 4.6. Data Lake Analytics Workflow
- **Business Logic:** Collect massive volumes of attendance logs across campuses, aggregate and organize data so that directors can view Performance Dashboards comparing metrics across departments.
- **AWS Services:**
  - **Amazon Kinesis Data Firehose:** Ingests attendance log streams, automatically partitions data by date (Dynamic Partitioning), and persists large batched files to the **S3 Data Lake**.
  - **AWS Glue (Data Catalog):** Automatically crawls and catalogs the schema of JSON files stored on S3.
  - **Amazon Athena:** A serverless SQL query engine that reads data directly from S3 via the Glue Catalog, returning high-speed statistical results to the Frontend.


### 4.7. Core AWS Services Inventory
The following table summarizes all AWS services utilized within the architecture:

| No. | AWS SERVICE | ROLE & RESPONSIBILITY IN SMART CAMPUS | RATIONALE & TECHNICAL BENEFITS |
| :---: | :--- | :--- | :--- |
| 1 | **Amazon CloudFront** | Distributes the React Frontend application from the S3 Bucket to end users. Serves as the integration point for AWS WAF. | Accelerates page loading through caching at Edge Locations. Provides automatic HTTPS support and reduces bandwidth consumption. |
| 2 | **AWS WAF** | Application firewall protecting the attendance endpoint by blocking non-office IP addresses. | Prevents remote attendance fraud and defends against web-based attacks and spam requests. |
| 3 | **Amazon S3** | **Bucket 1:** Frontend static hosting. <br> **Bucket 2:** Facial images & confidential document storage. <br> **Bucket 3:** S3 Data Lake for log storage. | Extremely low storage cost with 99.999999999% durability. Supports S3 Pre-signed URLs for secure file access. Seamless integration with Athena. |
| 4 | **Amazon API Gateway** | RESTful/HTTP API gateway receiving requests from the Frontend and invoking AWS Lambda functions. | Built-in Rate Limiting and native JWT authentication via Cognito Authorizer — no custom code required. |
| 5 | **AWS Lambda** | **API Handler:** Processes API business logic. <br> **Workers:** Handles background event processing. | Serverless Pay-As-You-Go model (charges incurred only when code executes). Instant auto-scaling with zero server management. |
| 6 | **Amazon DynamoDB** | Stores all business data (Users, Tasks, Leaves, Attendance records). | Serverless NoSQL database delivering single-digit millisecond response times, highly flexible with Global Secondary Indexes. |
| 7 | **Amazon Cognito** | Manages User Pools, authenticates login credentials, and issues JWT Tokens. | Eliminates the need to build a custom authentication system. Provides high security and supports mandatory password change on first login. |
| 8 | **Amazon EventBridge** | Event Bus that routes domain events (e.g., `AttendanceRecorded`) and executes scheduled rules (Cronjobs). | Decouples system modules following Event-Driven standards, enabling seamless addition of new business capabilities. |
| 9 | **Amazon SQS** | Message queue positioned in front of background Workers. | Guarantees no data loss during processing failures. Integrated Dead Letter Queue (DLQ) enables automatic retries. |
| 10 | **Amazon Rekognition** | Performs facial matching against registered employees during camera-based check-in. | Enterprise-grade AI service ready out-of-the-box — no model training required. Delivers high accuracy (Confidence > 95%). |
| 11 | **Amazon Kinesis Data Firehose, Glue & Athena** | End-to-end pipeline that aggregates attendance logs on S3, catalogs schemas, and enables SQL-based analytics. | Automatic file batching optimizes S3/Athena costs. Cleanly separates OLTP and OLAP concerns. |
| 12 | **AWS CodeBuild & CodePipeline** | Establishes a CI/CD Pipeline to automatically build the Frontend and package the Lambda Backend. | Enables fully automated Continuous Deployment from source code, ensuring safety and consistency across releases. |

### 4.8. Architecture Assessment Against the 5 Pillars of the AWS Well-Architected Framework
The entire Smart Campus Platform architecture is designed in strict adherence to the 5 Pillars of the AWS Well-Architected Framework:

1. **Operational Excellence:** The full application lifecycle is managed automatically through CI/CD pipelines (CodeBuild/CodePipeline). Centralized monitoring of logs and event metrics via Amazon CloudWatch enables early detection of bottlenecks.
2. **Security:** The Least Privilege principle is enforced through granular IAM Roles assigned to each Lambda function. Sensitive attachments are protected using S3 Pre-signed URLs, connections are encrypted via CloudFront HTTPS/TLS, and the API gateway is secured using AWS WAF combined with Cognito JWT Authorizer.
3. **Reliability:** Continuous High Availability is ensured through the default Multi-AZ architecture of the AWS Serverless ecosystem. Automatic retry mechanisms and routing of failed messages to Amazon SQS Dead-Letter Queues (DLQ) prevent the loss of attendance logs.
4. **Performance Efficiency:** Static Frontend assets are distributed seamlessly via CloudFront Edge Locations. Data read/write operations are optimized to single-digit milliseconds with DynamoDB, while heavy analytical queries are offloaded from the primary OLTP system to the Data Lake pipeline (Firehose & Athena).
5. **Cost Optimization:** The 100% Serverless Event-Driven model is applied throughout (charges are incurred only when the system is invoked). S3 Lifecycle Rules are configured to automatically tier storage (transitioning old logs to Glacier), minimizing long-term cold storage costs.

## 5. Proposed Timeline
| Week | Work Items |
| :--- | :--- |
| **Week 1–2** | Requirements analysis, system architecture design, UML and architecture diagramming. Provision AWS networking resources, CloudFront, WAF, and static S3 hosting. |
| **Week 3–4** | Develop Backend API (FastAPI) on AWS Lambda and DynamoDB. Integrate Cognito and Rekognition for facial recognition attendance features. |
| **Week 5** | Design Event-Driven Architecture with EventBridge and SQS. Complete the notification workflow (SNS/SES). |
| **Week 6–7** | Build the Data Lake Pipeline (Firehose → S3 → Glue → Athena) to serve Analytics Reports. Develop the ReactJS Frontend and connect it to the API. |
| **Week 8** | Integrate CI/CD (CodeBuild, CodePipeline), implement automation testing, optimize performance (X-Ray), finalize deliverables, and write the project report. |

## 6. Monthly Budget Estimation
The budget estimate is calculated based on the actual operational scale of a medium-sized campus: **200 employees, each checking in an average of 1 to 4 times per day** (morning arrival, lunch departure, afternoon return, evening departure). In total, the system will process approximately **20,000 attendance events per month** and roughly **150,000 API requests per month** (including task assignments, reporting, and leave requests).

To demonstrate the cost-efficiency of Serverless, the estimate below is calculated **based on standard Pay-As-You-Go pricing** without relying on the AWS 12-month Free Tier.

| AWS SERVICE | EXPECTED MONTHLY USAGE | REFERENCE PRICING (AP-SOUTHEAST-1) | MONTHLY COST (USD) |
| :--- | :--- | :--- | :---: |
| **AWS Lambda** | 150,000 API Requests + 40,000 Worker executions (Memory: 512MB, Avg: 1s) | $0.20 / 1M Requests + Compute time | **$1.62** |
| **Amazon API Gateway** | 150,000 HTTP API calls | $1.00 / 1M Requests | **$0.15** |
| **Amazon SQS** | 50,000 SQS Requests (Send & Receive) | $0.40 / 1M Requests | **$0.02** |
| **Amazon DynamoDB** | 500,000 WCU, 500,000 RCU (On-Demand Mode) + 2GB Storage | $1.25 / 1M WCU, $0.25 / 1M RCU + $0.25/GB | **$1.26** |
| **Amazon S3** | ~5GB Storage (Frontend, Images, Data Lake) + 100k GET/PUT | $0.025 / GB Storage + $0.004 / 1k PUT | **$0.53** |
| **Amazon CloudFront** | 20GB Data Transfer Out + 200k HTTPS Requests | $0.09 / GB | **$1.80** |
| **AWS WAF** | 1 Web ACL + 1 Rule (IP Match) + 150k Requests | $5.00/Web ACL + $1.00/Rule + $0.60/1M Req | **$6.09** |
| **Amazon Cognito** | Under 1,000 MAU (Monthly Active Users) | Free (under 50,000 MAU permanently) | **$0.00** |
| **Amazon Rekognition** | 20,000 facial comparison scans (SearchFacesByImage) | $0.001 / Image scan | **$20.00** |
| **Amazon Firehose & Athena** | ~1GB Data Ingestion & Scanned by Athena queries | $0.03/GB Ingestion + $5.00/TB Scanned | **$0.04** |
| **Amazon CloudWatch** | 1GB Log Ingestion + 3 Custom Metrics | $0.57 / GB Logs | **$1.47** |
| **AWS CodeBuild & CodePipeline** | ~100 build minutes (linux-small) + 1 Active Pipeline | $0.005 / minute + $1.00/Pipeline | **$1.50** |
| **TOTAL** | **Smart Campus Operating Cost (200 Users)** | | **~ $34.48 / month** |

### 6.1. Cost Optimization Strategy
Although the baseline operating cost is already very low, the system employs additional advanced optimization strategies:
1. **Pure Serverless Pay-As-You-Go Model:** Utilizing AWS Lambda with **API Gateway HTTP API** (71% cheaper than REST API) ensures the system incurs zero server maintenance costs during nighttime hours and weekends.
2. **S3 Lifecycle Rules & Firehose Compression:** Automatic compression of attendance logs into Parquet format via Firehose, combined with transitioning logs older than 90 days to **S3 Glacier Flexible Retrieval**, reduces long-term storage costs by up to 85%.
3. **SQS Long Polling:** Configuring `ReceiveMessageWaitTimeSeconds = 20` minimizes empty receive requests to SQS, yielding significant savings on API call costs.
4. **AWS Lambda Power Tuning:** Systematic profiling is performed to identify the optimal memory allocation that balances response latency and execution cost, ensuring Lambda functions are not over-provisioned with excess memory.

## 7. Risk Assessment & Mitigations

| No. | RISK TYPE | DETAILED RISK ANALYSIS | SEVERITY | MITIGATION STRATEGY |
| :---: | :--- | :--- | :---: | :--- |
| 1 | **Performance** | **API Bottleneck or Lambda Cold Start:** When hundreds of employees simultaneously check in at 8:00 AM, Lambda latency may spike due to cold starts. | **HIGH** | - Configure **Provisioned Concurrency** for critical Lambda functions during peak hours.<br>- Employ SQS as a buffer to absorb sudden traffic spikes and enable asynchronous processing. |
| 2 | **Security** | **API Spam Attacks / Fraud:** Malicious actors continuously send junk requests to exhaust the AWS budget (Financial Exhaustion) or submit spoofed images. | **CRITICAL** | - Enable **AWS WAF** with Rate Limiting and block unrecognized IPs.<br>- Require JWT authentication via Cognito Authorizer before any request is processed.<br>- Apply strict Least Privilege permissions to each Lambda execution role. |
| 3 | **Operations** | **Data Loss:** The system is processing attendance when a Lambda Worker times out or crashes unexpectedly. | **MEDIUM** | - Configure an appropriate SQS `VisibilityTimeout`.<br>- Enable **Dead-Letter Queue (DLQ)** to capture messages that fail after 3 attempts, allowing engineers to investigate without losing attendance logs. |
| 4 | **Cost Management** | **Sudden Cost Spike:** An infinite loop bug in Lambda code or excessive error logging to CloudWatch. | **MEDIUM** | - Set up **AWS Budgets Alerts** to automatically notify via Email/Slack when costs exceed $40 USD/month.<br>- Configure CloudWatch Log Retention to a maximum of 14 days instead of indefinite retention. |

## 8. Expected Outcomes

Upon successful deployment, the **Smart Campus** system is expected to achieve the following technical metrics and business objectives:

**Technical KPIs:**
- **Availability SLA:** Achieve a minimum of **99.9%** uptime, supported by AWS Multi-AZ Serverless infrastructure.
- **API Response Latency:** **< 200ms** for standard read/write data operations via API Gateway & DynamoDB.
- **AI Processing Time:** **< 2.0 seconds** from the moment a facial image is submitted to when the attendance result is returned.
- **Concurrent Capacity:** Seamlessly handle a minimum of **500 simultaneous attendance requests** without request drops or system congestion.

**Operational & Business Outcomes:**
- **Cost Optimization:** Achieve over **80%** savings in infrastructure operating costs compared to traditional server provisioning (EC2/VPS), enabled by the serverless Pay-as-you-go model.
- **High Maintainability:** The entire architecture is modularized into independently deployable Microservices (Event-Driven), allowing upgrades or bug fixes to individual features without disrupting the overall system.
- **Superior User Experience:** Complete digitization of manual paperwork, providing a smart, modern, and transparent working environment for all personnel.

#### Technical Deliverables

Upon project completion, the expected technical deliverables include:

* A fully operational FastAPI backend with over 20 API endpoints.
* Face registration and recognition achieving accuracy above 95%.
* Operational Face Liveness Detection effectively preventing fraud.
* 8 DynamoDB tables populated with complete data.
* A fully functional event-driven architecture utilizing EventBridge, SQS, and Lambda workers.
* An S3 Data Lake with data partitioned by date.
* Glue Crawler automatically updating the data schema.
* Athena queries returning accurate analytical results.
* An AI Assistant capable of answering questions in Vietnamese.
* A CloudWatch Dashboard displaying real-time metrics.
* A CI/CD pipeline enabling automated deployments.
* A React Frontend application hosted on CloudFront.
* Security hardened with WAF, Cognito, and IAM.
* Comprehensive instructions for cleaning up all provisioned AWS resources.

#### Business Deliverables

Smart Campus establishes a structured management process. Instead of relying on manual attendance or easily-fraudulent swipe cards, each attendance instance becomes a verifiable record containing timestamp, status, confidence score, and camera_id.

For example, when an employee stands before the camera, the system automatically performs facial recognition, validates liveness, applies the rule engine, and records the result as PRESENT or LATE — all within 2 to 3 seconds.

#### Learning Outcomes

The project demonstrates practical, hands-on knowledge in the following areas:

* Designing serverless architecture on AWS.
* Building event-driven systems.
* Integrating AI/ML services including Rekognition and Bedrock.
* Constructing Data Lake analytics pipelines.
* Monitoring and troubleshooting.
* Security best practices.
* Cost optimization.
* CI/CD automation.

#### Long-Term Value

Smart Campus can be extended into several future development directions, including a multi-tenant SaaS platform, mobile applications for iOS and Android, IoT integration with smart cameras, advanced analytics with SageMaker, geofencing with location services, and integration with ERP or CRM systems.

The project is designed as a practical, real-world AWS use case. It demonstrates that AWS not only supports application hosting, but also enables automated deployment, scalable data storage, AI/ML processing, data analytics, comprehensive monitoring, robust security, and effective operational cost control.