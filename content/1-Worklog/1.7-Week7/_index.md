---
title: "Worklog Week 7"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

# Week 7: Leave Management – RBAC Standardization – Workshop & Blog – S3/CloudFront Deployment

## 1. Weekly Objectives

* Develop the Leave Management module at the UI level and in relation to connected data flows.
* Re-standardize RBAC on the frontend and align it with the system’s permission matrix.
* Complete required personal deliverables such as technical blog posts, GitHub updates, and workshop-related reporting.
* Configure frontend deployment on Amazon S3 with CloudFront.

## 2. Detailed Work Log

| Day | Tasks | Start Date | End Date | References |
| --- | --- | --- | --- | --- |
| 2 | - Reviewed the workshop report folder structure, required illustrations, and overall group progress <br> - Started building the UI for the Leave Management module, identifying the main calendar states and leave request form flows <br> - Re-aligned leave request types, approval statuses, and their corresponding UI representations | 03/08/2026 | 03/08/2026 | <https://docs.aws.amazon.com/amazondynamodb/> |
| 3 | - Built `Leaves.jsx` with an Interactive Calendar color-coded by state (Holiday, Leave, WFH, Weekend) <br> - Completed the leave request form with automatic date filling when a date is selected directly on the calendar <br> - Rechecked the returned data structure for leave types and approval statuses to ensure correct rendering | 04/08/2026 | 04/08/2026 | — |
| 4 | - Finalized `Leaves.jsx`: WFH Check-in button for approved employees, automatic sync of PRESENT into Attendance, Cancel button, and “Cancelled” badge clearly distinguished from “Rejected” <br> - Finalized the styling according to the shared design system and improved the overall experience of the calendar and form <br> - Reviewed permission mapping so that each role would see the correct leave-related functionality <br> - Also helped review changes involving roles, departments, and task statuses affected by the new permission matrix | 05/08/2026 | 05/08/2026 | — |
| 5 | - Built `PermissionGuard` to hide/disable UI elements based on the role encoded in JWT <br> - Re-aligned the interface with the new permission matrix after the system was simplified to five roles <br> - Rechecked the data fields affected by schema migration, especially role, department, leave status, and WFH-related attendance flow <br> - Reviewed additional error cases caused by older data that had not yet been fully synchronized | 06/08/2026 | 06/08/2026 | — |
| 6 | - Configured frontend deployment to Amazon S3 with Static Website Hosting and set up a CloudFront Distribution with HTTPS; rechecked the UI across multiple devices <br> - Drafted, wrote, and published **three technical blog posts** on the AWS Study Group community; updated and redeployed my personal GitHub for supporting evidence <br> - Updated the event reflection reports and synchronized related content into the workshop/final report package | 07/08/2026 | 08/08/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html> <br> <https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/> |

## 3. Personal Contributions

* Took responsibility for the interface work of the week:
  * `Leaves.jsx` with Interactive Calendar;
  * leave request form;
  * WFH Check-in button;
  * status badges;
  * `PermissionGuard` for frontend RBAC;
  * frontend deployment to S3 + CloudFront.
* Followed the Leave Management module closely from the display layer to Attendance-related synchronization logic, especially for approved WFH cases.
* Reviewed the permission matrix between the interface and the system to avoid scattered hard-coded authorization logic.
* Rechecked fields affected by schema changes to reduce integration issues during the final stage.
* Independently prepared and completed three technical blog posts, while also updating my GitHub and workshop/event-related reporting.

## 4. Outcomes Achieved

* Completed the Leave Management module with an interactive calendar, leave request form, and WFH handling.
* Improved clarity of frontend RBAC through `PermissionGuard`.
* Successfully deployed the frontend to S3 + CloudFront with HTTPS.
* Reflected changes in roles, permissions, and data schema more clearly on the interface.
* Completed three technical blog posts for the group and updated the related workshop/event reporting materials.