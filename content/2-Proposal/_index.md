---

title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
--------------------

# OpsTicket CI/CD — Internal Issue Management System

## Building and Deploying a CI/CD Pipeline on AWS for an Internal Issue Management Web Application

### 1. Executive Summary

OpsTicket CI/CD is a cloud-based internal issue management system designed for offices, schools, labs, and small teams. The system allows users to report internal issues such as unstable Wi-Fi, broken projectors, printer failures, air conditioner problems, power socket issues, or locked internal accounts.

In many real working environments, these issues are often reported through chat messages or direct conversations. This is quick, but it is not easy to track. A message can be missed, the responsible person may be unclear, and there is no reliable way to know whether the issue is new, in progress, or already resolved.

OpsTicket solves this problem by turning informal reports into structured tickets. Each ticket includes a title, description, category, location, priority, status, assignee group, and creation time. Support staff can update ticket status, assign tickets to the correct group, and monitor the overall issue handling process.

From the AWS and DevOps perspective, this project focuses on building, containerizing, deploying, and monitoring the application through a CI/CD pipeline. The backend is built with Java Spring Boot, packaged as a Docker image, stored in Amazon ECR, and deployed to Amazon ECS Fargate. Ticket data is stored in Amazon DynamoDB. Logs, metrics, and alarms are monitored through Amazon CloudWatch. GitHub Actions is used to automate testing, image building, image pushing, and deployment.

The purpose of this project is not only to build a web application, but also to demonstrate a complete cloud deployment workflow on AWS.

---

### 2. Problem Statement

### What’s the Problem?

Internal issues in small organizations are often handled manually or informally. For example, a student may message an admin that the projector in room B203 is not working, or an employee may tell the facility team that the air conditioner is broken. These reports are easy to create but difficult to manage.

The main problems are:

* Reports can be missed in chat groups.
* There is no clear ticket status.
* It is difficult to know who is responsible for the issue.
* Urgent issues are not clearly separated from normal ones.
* There is no simple dashboard to view open, resolved, or overdue issues.
* Manual deployment can cause inconsistent application versions.
* Without monitoring, backend errors may only be noticed after users complain.

### The Proposed Solution

OpsTicket provides a structured ticket workflow for internal issue management. Users can create tickets, and support staff can process them through clear statuses:

```txt
NEW → ASSIGNED → IN_PROGRESS → RESOLVED → CLOSED
```

Tickets are classified by priority:

```txt
P1 - Critical issue
P2 - High priority issue
P3 - Normal issue
```

The backend also supports simple auto-assignment rules. For example:

```txt
WIFI, PROJECTOR, PRINTER, ACCOUNT → IT_SUPPORT
AIR_CONDITIONER, LIGHT, POWER_SOCKET → FACILITY
GENERAL → GENERAL_ADMIN
```

The system is deployed on AWS using a CI/CD workflow:

```txt
GitHub → GitHub Actions → Docker Build → Amazon ECR → Amazon ECS Fargate → CloudWatch
```

This makes OpsTicket a practical AWS use case. AWS is not used only as a hosting platform; it is part of the application delivery, runtime, storage, monitoring, security, and cost management process.

### Benefits

OpsTicket provides two main benefits.

First, it improves internal issue handling. Every issue becomes a trackable record with status, priority, assignee group, and timestamps. This helps support teams avoid missed reports and makes the process more transparent.

Second, it provides a practical DevOps learning project. The project covers Docker, CI/CD, Amazon ECR, Amazon ECS, DynamoDB, CloudWatch, IAM, and cleanup. These are important skills for deploying and operating applications on AWS.

---

### 3. Solution Architecture

OpsTicket uses a container-based AWS architecture with an automated CI/CD pipeline.

At a high level, the workflow is:

```txt
Developer pushes code to GitHub
→ GitHub Actions runs tests and builds Docker image
→ Docker image is pushed to Amazon ECR
→ Amazon ECS Fargate runs the backend container
→ Application Load Balancer exposes the API
→ Backend stores ticket data in DynamoDB
→ CloudWatch collects logs and metrics
→ CloudWatch Alarm and SNS can notify when issues occur
```

![OpsTicket CI/CD Architecture](/images/2-Proposal/opsticket-cicd-architecture.png)

### AWS Services Used

* **Amazon ECR**: Stores Docker images for the Spring Boot backend.
* **Amazon ECS Fargate**: Runs the backend container without managing EC2 servers.
* **Application Load Balancer**: Exposes the backend API and performs health checks.
* **Amazon DynamoDB**: Stores ticket data such as title, category, priority, status, and assignee group.
* **Amazon S3**: Can be used to host the frontend or store optional ticket attachment images.
* **Amazon CloudFront**: Can distribute the frontend if it is deployed as a static web application.
* **Amazon CloudWatch**: Collects application logs, metrics, and alarms.
* **Amazon SNS**: Sends email notifications for system alarms or important ticket events.
* **AWS IAM**: Controls permissions using roles and least-privilege policies.
* **GitHub Actions**: Automates testing, Docker image build, ECR push, and ECS deployment.

### Component Design

* **Backend API**: Built with Java Spring Boot. It provides APIs for health check, ticket creation, ticket listing, ticket detail, status update, assignment, comments, and dashboard summary.
* **Container Layer**: The backend is packaged into a Docker image so that it can run consistently across local, CI, and AWS environments.
* **CI/CD Pipeline**: GitHub Actions runs tests, builds Docker images, pushes images to ECR, and deploys new versions to ECS.
* **Runtime Layer**: ECS Fargate runs the backend container. The Application Load Balancer forwards user requests to the running task.
* **Data Layer**: DynamoDB stores ticket data in a managed and scalable NoSQL database.
* **Monitoring Layer**: CloudWatch provides logs, metrics, and alarms for operation and troubleshooting.

---

### 4. Technical Implementation

### Implementation Phases

#### Phase 1: Build the Backend Application

The project starts with a Spring Boot backend. The first API is `/health`, which is used for local testing and for the Load Balancer health check after deployment.

After that, ticket APIs are developed:

* Create ticket
* List tickets
* Get ticket detail
* Update ticket status
* Assign ticket
* Add comment
* Get dashboard summary

At the beginning, the backend can use in-memory storage for quick testing. In the final AWS version, ticket data must be stored in DynamoDB.

#### Phase 2: Containerize the Backend

The backend is packaged as a Docker image. This ensures that the application can run the same way locally, in GitHub Actions, and on AWS.

The expected result is a Docker image that exposes port `8080` and returns a successful response from:

```txt
GET /health
```

#### Phase 3: Push Image to Amazon ECR

An Amazon ECR repository is created to store backend Docker images. The image is tagged and pushed to ECR. This step prepares the application image for ECS deployment.

#### Phase 4: Deploy to Amazon ECS Fargate

The application is deployed to ECS Fargate using an ECS cluster, task definition, ECS service, target group, Application Load Balancer, and security group. The Load Balancer uses `/health` to check whether the backend is running correctly.

The expected result is that the backend can be accessed through the Load Balancer DNS name.

#### Phase 5: Set Up CI Pipeline

GitHub Actions is configured to run on pull requests. The CI pipeline should:

* Checkout source code
* Set up Java
* Run Maven tests
* Build the application
* Build the Docker image

This helps prevent broken code from being merged.

#### Phase 6: Set Up CD Pipeline

GitHub Actions is configured to run when code is pushed to the `main` branch. The CD pipeline should:

* Build a new Docker image
* Tag the image using the Git commit SHA
* Push the image to Amazon ECR
* Render a new ECS task definition
* Deploy the new task definition to ECS
* Wait until the ECS service is stable

#### Phase 7: Integrate DynamoDB

The backend is updated to store ticket data in DynamoDB. This makes data persistent even when ECS tasks restart or new versions are deployed.

#### Phase 8: Add Monitoring and Alerts

CloudWatch Logs are used to view backend logs. CloudWatch Metrics are used to monitor ECS and Load Balancer behavior. At least one CloudWatch Alarm should be created, such as an alarm for backend 5xx errors or high CPU usage. SNS can be used to send email notifications when an alarm is triggered.

### Technical Requirements

* **Programming Language**: Java
* **Backend Framework**: Spring Boot
* **Build Tool**: Maven
* **Containerization**: Docker
* **CI/CD Tool**: GitHub Actions
* **Container Registry**: Amazon ECR
* **Runtime Platform**: Amazon ECS Fargate
* **Database**: Amazon DynamoDB
* **Monitoring**: Amazon CloudWatch
* **Notification**: Amazon SNS
* **Security**: AWS IAM roles and least-privilege policies
* **Recommended Region**: `ap-southeast-1`

### Main Backend APIs

* `GET /health`
* `POST /tickets`
* `GET /tickets`
* `GET /tickets/{ticketId}`
* `PATCH /tickets/{ticketId}/status`
* `PATCH /tickets/{ticketId}/assign`
* `POST /tickets/{ticketId}/comments`
* `GET /dashboard/summary`
* `POST /admin/sla/check`

### Important Business Rules

OpsTicket should not be only a basic CRUD application. The backend should include simple business rules to make the system more realistic.

#### Ticket Status Workflow

```txt
NEW → ASSIGNED → IN_PROGRESS → RESOLVED → CLOSED
```

#### Priority and SLA

```txt
P1: Critical issue, expected response within 4 hours
P2: High priority issue, expected response within 24 hours
P3: Normal issue, expected response within 72 hours
```

#### Auto Assignment

```txt
WIFI, PROJECTOR, PRINTER, ACCOUNT, SOFTWARE, HARDWARE → IT_SUPPORT
AIR_CONDITIONER, LIGHT, POWER_SOCKET, TABLE_CHAIR → FACILITY
GENERAL → GENERAL_ADMIN
```

---

### 5. Timeline & Milestones

### Project Timeline

The project follows a 12-week internship timeline.

* **Week 1: Study DevOps, CI/CD, Git, Docker, and AWS Deployment Options**
  Understand the software delivery workflow and compare EC2, ECS, and Elastic Beanstalk.

* **Week 2: Build the Initial OpsTicket Backend**
  Create the Spring Boot project, health check API, and basic ticket APIs.

* **Week 3: Dockerize the Application**
  Create Dockerfile, build the image locally, and test the container.

* **Week 4: Push Docker Image to Amazon ECR**
  Create ECR repository, tag the image, and push it to AWS.

* **Week 5: Set Up CI Pipeline**
  Configure GitHub Actions to run Maven tests and build Docker images.

* **Week 6: Deploy Backend to Amazon ECS Fargate**
  Create ECS cluster, task definition, ECS service, target group, Load Balancer, and security group.

* **Week 7: Set Up CD Pipeline**
  Configure GitHub Actions to automatically deploy new versions to ECS when code is pushed to `main`.

* **Week 8: Configure CloudWatch Logs and Metrics**
  Verify backend logs and ECS/Load Balancer metrics in CloudWatch.

* **Week 9: Integrate DynamoDB and Optional SNS Alert**
  Store ticket data in DynamoDB and configure optional alert notifications.

* **Week 10: Improve Security and Cost Optimization**
  Review IAM permissions, avoid hard-coded credentials, reduce unnecessary cost, and plan cleanup.

* **Week 11: Complete Blogs and Workshop Documentation**
  Write blog posts and complete the bilingual workshop website.

* **Week 12: End-to-End Testing, Screenshots, Cleanup, and Final Report**
  Test the complete workflow, collect screenshots, clean up AWS resources, and finalize the report.

---

### 6. Budget Estimation

This project is designed for low-traffic workshop usage. The cost should remain low if the resources are used only during testing and cleaned up afterward.

Main cost components:

* **Amazon ECS Fargate**: Depends on CPU, memory, and running hours.
* **Application Load Balancer**: Depends on running time and traffic.
* **Amazon ECR**: Depends on Docker image storage.
* **Amazon DynamoDB**: On-demand mode is suitable for small and unpredictable usage.
* **Amazon CloudWatch Logs**: Depends on log volume and retention period.
* **Amazon S3 and CloudFront**: Used only if the frontend or attachments are deployed.
* **Amazon SNS**: Low cost for small-volume email notifications.

Cost optimization strategies:

* Use small ECS Fargate task size.
* Stop or delete ECS resources after testing.
* Use DynamoDB on-demand mode.
* Set CloudWatch log retention to 7 or 14 days.
* Delete old ECR images.
* Clean up ALB, ECS, ECR, DynamoDB, S3, CloudWatch alarms, and log groups after the workshop.

The final cost should be estimated using AWS Pricing Calculator based on the selected region, ECS task size, running hours, and expected traffic.

---

### 7. Risk Assessment

### Risk Matrix

| Risk                              | Impact | Probability | Description                                                           |
| --------------------------------- | ------ | ----------- | --------------------------------------------------------------------- |
| IAM permission error              | High   | Medium      | GitHub Actions, ECS, or backend may fail if permissions are missing.  |
| Docker build failure              | Medium | Medium      | Dockerfile or Maven configuration may cause build errors.             |
| ECS task cannot start             | High   | Medium      | Wrong image URI, port mapping, or health check can stop deployment.   |
| ALB health check failure          | High   | Medium      | If `/health` fails, ECS may keep replacing tasks.                     |
| DynamoDB integration issue        | Medium | Medium      | Wrong table name, region, or IAM role can prevent data storage.       |
| CloudWatch alarm misconfiguration | Medium | Low         | Alarm may not trigger if the metric or threshold is wrong.            |
| Cost overrun                      | Medium | Low         | ECS, ALB, and logs may generate cost if resources are not cleaned up. |
| Security exposure                 | High   | Low         | Hard-coded credentials or public resources can create risk.           |

### Mitigation Strategies

* Test the backend and Docker image locally before deploying to AWS.
* Start ECS deployment with a simple `/health` endpoint first.
* Use IAM roles and least-privilege permissions.
* Do not commit AWS access keys or secrets to GitHub.
* Use CloudWatch Logs to troubleshoot ECS task failures.
* Keep the infrastructure small for workshop usage.
* Document all cleanup steps clearly.

### Contingency Plans

* If ECS deployment fails, roll back to the previous working task definition revision.
* If CD pipeline fails, deploy manually once and document the issue.
* If DynamoDB integration is not ready, keep the API testable with temporary storage, then complete DynamoDB before final submission.
* If SNS is not completed, CloudWatch Alarm can still be used as the minimum alerting demonstration.

---

### 8. Expected Outcomes

### Technical Outcomes

By the end of the project, the expected technical outputs are:

* A working Spring Boot backend for OpsTicket.
* REST APIs for ticket management and dashboard summary.
* Dockerized backend image.
* Amazon ECR repository storing backend images.
* Amazon ECS Fargate service running the backend.
* Application Load Balancer exposing the API.
* DynamoDB table storing ticket data.
* GitHub Actions CI workflow.
* GitHub Actions CD workflow.
* CloudWatch Logs, Metrics, and at least one CloudWatch Alarm.
* Cleanup instructions for all AWS resources.

### Business Outcomes

OpsTicket provides a simple but structured way to manage internal issues. Instead of reporting problems through scattered chat messages, each issue becomes a trackable ticket with status, priority, assignee group, and timestamps.

For example, a ticket such as “Projector is not working in room B203” can be created as priority `P2`, assigned to `IT_SUPPORT`, processed through the workflow, and marked as resolved after the issue is fixed.

### Learning Outcomes

This project helps demonstrate practical knowledge in:

* Backend API development with Java Spring Boot.
* Business workflow design for ticket management.
* Docker-based application packaging.
* CI/CD automation using GitHub Actions.
* AWS container deployment with ECR and ECS Fargate.
* Data storage with DynamoDB.
* Monitoring with CloudWatch.
* Basic IAM security and cost optimization.

### Long-term Value

OpsTicket can be extended into a more complete internal operations system. Future improvements may include user authentication with Amazon Cognito, ticket image upload to S3, SLA checking with EventBridge, SNS notification workflow, React dashboard, and infrastructure as code using Terraform, AWS CDK, or CloudFormation.

The project is designed as a practical AWS use case. It shows how AWS services support not only application hosting, but also deployment automation, data storage, monitoring, security, and operational cost control.
