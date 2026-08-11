---
title : "Introduction"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### 5.1. Introduction

### 1. Solution Introduction (Use case)
In the context of digital transformation in education and enterprise, manual attendance (swiping magnetic cards, fingerprints) still presents major pain points: congestion during peak hours, forgotten cards, or proxy check-ins. Internal physical server systems often waste resources when no one is using them at night, but become overloaded during the 8:00 AM timeframe.

**Smart Campus Platform** was created to completely solve this problem by combining facial recognition artificial intelligence (**Amazon Rekognition**) and a **100% AWS Serverless** architecture. The system not only processes attendance at lightning speed but also ensures absolute security, automates notification workflows, and provides a Big Data analytics solution at the most optimal cost (Pay-as-you-go).

---

### 2. Architecture Diagram & Workflow

> **Figure 1 - Smart Campus Architecture and Processing Flow**
> ![Architecture Overview](/aws-image/AwsArchitecture.drawio.png)

The Smart Campus system is designed with a 100% Serverless architecture on the AWS platform, applying an Event-Driven Architecture model to ensure high performance, auto-scalability, and cost optimization. The architecture diagram is divided into the following main business flow groups:

#### Group 0: CI/CD Pipeline (Automated Deployment)
The system uses the AWS Developer Tools suite to automate the process of testing and deploying source code whenever there are changes.
- **C1. Push code:** Developer pushes new source code (Frontend/Backend) to the GitHub repository.
- **C2. Trigger pipeline:** AWS CodePipeline listens to events from GitHub and automatically triggers the CI/CD flow.
- **C3. Build & Package:** AWS CodeBuild downloads the source code, compiles (Build React), or packages libraries (Zip Python/FastAPI) into complete builds.
- **C4. Deploy:** 
  - *Frontend:* CodeBuild pushes static files (HTML/CSS/JS) to the S3 Frontend Bucket.
  - *Backend:* CodeBuild pushes the zip file to AWS Lambda and updates the new version.

#### Group 1: Access & Token Retrieval (Access & Auth)
Protects the system from the outside and provides a secure authentication mechanism.
- **1a. Web Access:** Users send access requests from the browser.
- **1b. Secure Access (WAF):** AWS WAF checks IP and security rules before allowing the request to pass.
- **2. Serve SPA:** CloudFront retrieves static web content from the S3 Frontend and distributes it quickly to users via the global CDN network.
- **3. Trigger API:** Business requests from the Frontend are pushed into the API Gateway.
- **4a & 4b. Authenticate:** API Gateway forwards the login request to Lambda. Lambda calls **Amazon Cognito** to authenticate the user and retrieve the JWT Token to return to the Frontend.
- **4c. Validate Token:** Subsequent requests are intercepted by API Gateway to have Cognito check the Token's validity before proceeding.

#### Group 2: HR Management & Face Registration
Processes user data and generates biometric features.
- **5. User Management:** Lambda reads/writes basic HR information into the `Users` table on DynamoDB.
- **6. Registration Request:** The face registration flow for new employees.
- **7. Save raw image:** Lambda uploads the raw image to the S3 Images Bucket to serve as reference material.
- **8. Extract features:** Lambda calls **Amazon Rekognition** (IndexFaces) to extract the biometric matrix.
- **9. Save Metadata:** The FaceID is saved into the `Faces` table on DynamoDB.

#### Group 3: Core Attendance (Face Attendance)
This is the backbone flow of the system, handling low latency (< 1s).
- **10. Attendance Request:** User checks in, the system sends the check-in image to API Gateway.
- **11. Retrieve info:** Lambda queries the `Users` table to cross-reference rules (Shifts, allowed times...).
- **12. Recognize:** Lambda calls Amazon Rekognition (SearchFacesByImage) to match the face with high accuracy.
- **13. Record:** The attendance record is immediately saved into the `Attendance` table on DynamoDB.
- **14. Send Personal Email:** Lambda uses **Amazon SES** to send an attendance receipt (HTML) directly to that person's email.
- **15. Publish Event:** To avoid slowing down the API, Lambda immediately fires an *"AttendanceRecorded"* event to **Amazon EventBridge** and returns HTTP 200 to the Camera.

#### Group 4: Asynchronous (Event-Driven Async Flows)
Processes heavy background tasks using a Fan-out architecture (1 event branching out).
- **16a & 17a. Notification Flow:** EventBridge pushes the event into the SQS queue, triggering the `Notification Worker Lambda`. 
- **18. Broadcast via SNS:** This Worker calls Amazon SNS to "broadcast" messages to multimedia channels (SMS, Mobile Push, Chatbot).
- **16b & 17b. Data Flow (Analytics):** Simultaneously, EventBridge also pushes the event into SQS Analytics, triggering the `Analytics Worker Lambda`.
- **19. Save to Data Lake:** This Worker packages attendance data into JSON files and pushes them to the S3 Data Lake for low-cost, long-term Cold storage.

#### Group 5: Statistical Reporting (Hybrid / Lambda Architecture)
Combines the power of Big Data analytics and real-time data retrieval.
- **20. Catalog Data:** **AWS Glue** periodically crawls the S3 Data Lake to automatically learn and create a Data Schema.
- **21. Report Request:** User accesses the Dashboard screen, API calls down to Dashboard Lambda (Report Lambda).
- **22a. Retrieve Real-time Data (Hot Data):** Dashboard Lambda queries directly into **DynamoDB** (Attendance Table / Task Table) to get the latest attendance and task data of the day.
- **22b, 22c & 22d. Retrieve Historical Data (Cold Data):** Simultaneously, Dashboard Lambda asks **Amazon Athena** to run high-speed SQL queries, combined with the schema from Glue, to scan through historical data on the S3 Data Lake.
- **Aggregate:** Lambda automatically merges data from both flows and returns it to display accurately and optimally on the Dashboard.

#### Group 6: Task & Form Management
- **23. Task Request:** Task assignment or leave requests are pushed to Lambda.
- **24a & 24b. Read/Write DB:** Data is saved into independent `Tasks` and `Leaves` tables.
- **24c. Save Notification:** The notification sending history is recorded in the `Notifications` table.
- **24d & 24e. Presigned URL Upload:** Instead of uploading heavy files through Lambda, Lambda generates a short-term secure link (Presigned URL) and returns it. The User's browser uses this link to upload PDFs/Images directly to S3 Images, optimizing server bandwidth.
- **25. Send Notification:** Sends an email notifying about a new task/form via Amazon SES.

#### Group 7: Cronjob (Overdue Scanning)
- **26. Cron Trigger:** **EventBridge Scheduler** is scheduled to run every X minutes, automatically triggering Lambda.
- **27. Scan overdue:** Lambda scans the `Tasks` table to find tasks nearing their deadline or already overdue.
- **28. Warning Email:** Sends an urging email to the employee via Amazon SES.

#### Group 8: Governance, Security & Monitoring (Cross-cutting)
- **IAM (Identity and Access Management):** All services communicate using the Principle of Least Privilege. Lambda is only allowed to write to specific S3 buckets, not delete buckets.
- **X-Ray & CloudWatch:** 
  - Lambda continuously pushes Logs/Metrics (number of requests, processing time) to CloudWatch.
  - AWS X-Ray draws a Trace Map to track how many milliseconds a request takes through each service.
- **CloudWatch Alarms:** When the Faults rate exceeds the allowed threshold, an Alarm is triggered and calls Amazon SNS to shoot an emergency warning to the engineering team's phones.

---

### 3. In-Scope Services

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

### 4. Expected Outcomes upon Workshop Completion
By the end of this practical series, you will have fully built an enterprise platform:
- **Well-functioning Frontend:** Has an attendance interface and management dashboard.
- **Multi-layer secure authentication:** Anti-spoofing using IAM Least Privilege, WAF, and Cognito JWT.
- **High-load architecture:** Proficient application of Event-Driven (EventBridge + SQS) to eliminate peak hour bottlenecks.
- **Automated Data Pipeline:** Own a Data Lake system completely separating OLTP and OLAP.
- **DevOps CI/CD:** CodePipeline system automatically builds and deploys code without manual intervention.
- **Cleanup:** Ability to quickly clean up resources for complete control over AWS costs.

---

### 5. Future Development Directions

Although the Smart Campus system has completed its core features, the team has identified several potential improvement directions to upgrade the system to a higher level in the future:

#### 5.1. Upgrade AI & Recognition System
- **Liveness Detection (Anti-spoofing):** Integrate an anti-spoofing mechanism using photos or fake videos, ensuring absolute accuracy for the attendance system.
- **Migrate to Amazon Rekognition Video:** Support facial recognition from a live camera stream instead of uploading individual images, significantly increasing processing speed.

#### 5.2. Advanced Analytics
- **Integrate Amazon QuickSight:** Instead of drawing charts manually on the Frontend, integrate **Amazon QuickSight** to create professional BI (Business Intelligence) dashboards, supporting drill-down and multi-dimensional data filtering.
- **Machine Learning Forecasting:** Use **Amazon SageMaker** to train forecasting models for late arrival trends, predict team productivity, and propose automatic shift adjustments.
- **Real-time Streaming with Kinesis:** Replace SQS with **Amazon Kinesis Data Streams** for extremely high-load real-time data analysis flows (millions of events/second).

#### 5.3. Infrastructure & Cost Optimization
- **Infrastructure as Code (IaC):** Migrate the entire AWS resource configuration to **AWS CDK** or **Terraform** to manage infrastructure by version control and easily reuse it.
- **Multi-region Deployment:** Deploy the system across multiple AWS Regions to ensure High Availability and reduce latency for global users.
- **AWS Savings Plans / Reserved Capacity:** When the system reaches a stable traffic threshold, switch from the On-demand to the Reserved model to save an additional 30-60% on operating costs.