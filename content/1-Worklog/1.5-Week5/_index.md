---
title: "Week 5 Worklog"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Week 5: Tasks UI – Notification & Analytics Pipeline – Consolidating the Team PoC

## 1. Weekly objectives

* Build the Tasks Management module interface with suitable display modes and integrate direct file uploads to S3 using presigned URLs.
* Complete the shared infrastructure, consolidate the team PoC into a unified monorepo, and standardize the development environment.
* Complete WF4 – Notification across multiple channels with a more production-like design.
* Build WF5 – the two-phase Analytics Pipeline and begin implementing the Analytics interface.

## 2. Detailed work log

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - **Team infrastructure:** Complete the AWS Organization, Organizational Unit, and IAM Role configuration so the team can share resources in a controlled way <br> - Consolidate the PoC source code from team members, standardize the development environment with `uv.lock`, `.gitignore`, and declared dependencies <br> - Review the frontend and backend, record risks related to injection and input validation <br> - **Frontend:** Design the Tasks Management module interface with a List View and initial component structure | 20/07/2026 | 20/07/2026 | <https://docs.aws.amazon.com/organizations/> |
| 3 | - Reassign team work based on more specific roles <br> - Complete around 80% of the AWS architecture diagram to send to the mentor <br> - **Frontend:** Expand the task interface into a Kanban view with the main status columns <br> - **Backend:** Start building WF4 – Notifications by defining 5 message templates (AttendanceRecorded, AttendanceRejected, UnknownFaceDetected, SecurityIncidentCreated, Custom) | 21/07/2026 | 21/07/2026 | <https://docs.aws.amazon.com/sns/> <br> <https://docs.aws.amazon.com/ses/> |
| 4 | - **Frontend:** Integrate the Upload File component with logic to request a presigned URL and directly PUT the file to S3; add a Progress Bar to show upload progress <br> - **Backend:** Complete WF4 by publishing through a real Amazon SNS ARN, recording an Audit Trail in DynamoDB (SENT/FAILED), and emitting a `NotificationSent` event to EventBridge <br> - Upgrade the notification flow from broadcast-style messaging to one-to-one email delivery using SES, with SNS fallback when SES fails or the email is missing | 22/07/2026 | 22/07/2026 | <https://docs.aws.amazon.com/ses/> <br> <https://docs.aws.amazon.com/eventbridge/> |
| 5 | - **Backend:** Build WF5 – Analytics Pipeline Phase 1 by querying directly from DynamoDB for the dashboard <br> - Implement Phase 2 – the Data Lake: a Lambda Worker listens to AttendanceRecorded, streams the data through Kinesis Firehose to S3 partitioned by `year/month/day`, uses Glue Crawler to detect schema updates and update the Data Catalog, and enables Athena for SQL queries <br> - **Frontend:** Start building the Analytics page to display statistics | 23/07/2026 | 23/07/2026 | <https://docs.aws.amazon.com/athena/> <br> <https://docs.aws.amazon.com/glue/> <br> <https://docs.aws.amazon.com/firehose/> |
| 6 | - **Backend:** Upgrade the Analytics/Reports module by adding endpoints and schema for the dashboard and supporting department-based filtering <br> - Develop WF8 – Task & Incident Management: create the `smart-campus-tasks` table (13 attributes, 3 GSIs), standardize attribute names to `snake_case`, and add unit tests for serialization/deserialization <br> - **Frontend:** Optimize state management for the Tasks module, connect the new APIs, and complete the Analytics page to a usable state <br> - Write initial tests for the main data pipeline and review the modules developed during the week | 24/07/2026 | 25/07/2026 | <https://docs.aws.amazon.com/amazondynamodb/> |

## 3. Personal contributions

* Contributed to the frontend this week:
  * the Tasks Management interface in List View;
  * expansion toward a Kanban-style display;
  * direct file uploads to S3 using presigned URLs;
  * upload progress bar;
  * initial version of the Analytics page.
* Helped consolidate the team PoC into the shared monorepo, reviewed dependencies, folder structure, and naming conventions to avoid environment breakage during integration.
* Standardized several technical inputs between frontend and backend, especially around Notifications, Analytics schema, and file upload payloads.
* Identified risks related to input and injection and communicated them to the team before expanding the APIs further.
* Updated the architecture diagram and described the two-phase data flow to align it more closely with the implementation.

## 4. Achievements

* Frontend:
  * The Tasks Management module had a usable interface, including direct upload to S3.
  * The Analytics page was initiated and started connecting to statistical data from the backend.

* Team infrastructure and integration:
  * The shared development environment became more stable after standardizing dependencies and the monorepo structure.
  * The architecture diagram reached around 80% completion, which was sufficient to send to the mentor and continue refining.

* Backend and data:
  * WF4 was completed with 5 templates, multi-channel delivery, audit trails, and SES → SNS fallback.
  * WF5 took shape in both phases: fast queries from DynamoDB and long-term analytics through S3/Glue/Athena.
  * WF8 was expanded in terms of data model and testing, creating a foundation for the frontend and integration work in the following week.
