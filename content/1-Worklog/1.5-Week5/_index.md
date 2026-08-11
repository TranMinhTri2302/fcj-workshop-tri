---
title: "Week 5 Worklog"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Week 5: Task Management UI – Initial Analytics – PoC Consolidation and Integration Standardization

## 1. Weekly Objectives

* Build the Task Management UI with suitable display modes and integrate direct file uploads to S3 via presigned URL.
* Support consolidation of group PoCs into a shared monorepo and standardize the development environment where needed for frontend integration.
* Prepare the UI and displayed data structures for Notifications, Analytics, and related modules.
* Update the architecture diagram and describe the data flow based on the current implementation.

## 2. Detailed Work Log

| Day | Tasks | Start Date | End Date | References |
| --- | --- | --- | --- | --- |
| 2 | - Helped review PoC source code from team members to ensure the UI could be integrated stably into the shared monorepo <br> - Rechecked dependencies, folder structure, and necessary configurations for the shared development environment <br> - Designed the Task Management interface with an initial List View and component structure | 20/07/2026 | 20/07/2026 | <https://docs.aws.amazon.com/organizations/> |
| 3 | - Expanded the task interface toward a Kanban-style layout with main status columns <br> - Standardized how task states were displayed so that the UI matched the workflow being implemented by the team <br> - Updated the architecture diagram and reviewed the components that had changed after the baseline stage | 21/07/2026 | 21/07/2026 | <https://react.dev/> |
| 4 | - Integrated the file upload component with the flow of requesting a presigned URL and then sending the file directly to S3 via PUT <br> - Added a Progress Bar to show upload progress and handled success/failure states in the UI <br> - Re-aligned the upload payload, field names, and file reference handling to avoid mismatches between the interface and the API <br> - Also reviewed several outputs of the Notification flow in preparation for future UI display | 22/07/2026 | 22/07/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/> |
| 5 | - Started building the Analytics page to display statistical data from the system <br> - Reviewed the response schema for the dashboard, especially attendance metrics, task metrics, and department-level filters <br> - Documented the two-phase Analytics data flow to ensure that the UI accurately reflected the actual data sources | 23/07/2026 | 23/07/2026 | <https://docs.aws.amazon.com/athena/> <br> <https://docs.aws.amazon.com/glue/> |
| 6 | - Optimized state management for the Task Management module, connected new endpoints, and brought the Analytics page to a usable stage <br> - Rechecked responses related to Notifications, Analytics, and Tasks to align naming conventions between the interface and the system <br> - Helped review several fields and serialization/deserialization behaviors to reduce future integration issues <br> - Updated the technical description so that it matched the current implementation more closely | 24/07/2026 | 25/07/2026 | <https://docs.aws.amazon.com/amazondynamodb/> |

## 3. Personal Contributions

* Took charge of the UI work for this week:
  * Task Management module in List View;
  * expanded display toward Kanban;
  * direct file upload to S3 using presigned URL;
  * upload progress bar;
  * initial version of the Analytics page.
* Helped consolidate PoCs into the shared monorepo, reviewing dependencies, folder structure, and naming to reduce environment-related issues during integration.
* Re-standardized several technical inputs between the interface and the APIs, especially for file uploads, Notifications, Analytics schema, and task statuses.
* Updated the architecture diagram and refined the two-phase data flow description to better match the current implementation.

## 4. Outcomes Achieved

* Delivered a usable Task Management interface with direct file upload to S3.
* Initialized the Analytics page and started connecting it to the system’s statistical data.
* Improved the stability of the shared development environment after standardizing dependencies and the monorepo structure.
* Clarified the inputs and outputs of Notifications, Analytics, and Tasks from both the UI and technical documentation perspectives.