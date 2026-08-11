---
title: "Week 7 Worklog"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

# Week 7: Leave Management – Standardizing RBAC – Workshop & Blog – Deploying S3/CloudFront

## 1. Weekly objectives

* Develop the Leave Management module at the data, API, and user interface levels.
* Standardize RBAC across the codebase and handle migration of older DynamoDB data.
* Complete the required personal program deliverables, including technical blogs, GitHub updates, and the workshop-related report.
* Configure deployment of the frontend to Amazon S3 with CloudFront.

## 2. Detailed work log

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Push the team to complete the workshop deliverables on time, while reviewing the report folder structure and the content that needed to be illustrated <br> - Begin developing the **Leave Management** module: design the `smart-campus-leaves` table with GSIs `user_id-index` and `status-index`; define key attributes such as `leave_id`, `user_id`, `leave_type`, `date_from`, `date_to`, `reason`, `status`, `approved_by`, `approved_at`, and `cancel_reason` <br> - Add 4 leave types: WFH, ANNUAL_LEAVE, SICK_LEAVE, and BUSINESS_TRIP | 03/08/2026 | 03/08/2026 | <https://docs.aws.amazon.com/amazondynamodb/> |
| 3 | - **Backend:** Complete the Leave Management APIs: employee submission → manager approval/rejection; Admin management for public holidays using the `smart-campus-holidays` table <br> - **Frontend:** Build the Leaves.jsx page with an interactive calendar that highlights dates by status (holiday, leave, WFH, weekend); the registration form auto-fills dates when selected on the calendar | 04/08/2026 | 04/08/2026 | — |
| 4 | - **Frontend:** Complete Leaves.jsx: add Check-in WFH buttons for approved employees, synchronize PRESENT attendance without Rekognition, add Cancel and “Canceled” badges separate from “Rejected”, and refine the styling to match the shared design system <br> - **Backend:** Standardize permissions by reducing them to 5 roles (ADMIN, DIRECTOR, MANAGER, STAFF, TECHNICIAN), rename department `MAINTENANCE` to `TECHNICAL`, allow DIRECTOR to manage Users and WAF, and restrict STAFF to creating only INCIDENT-type tasks | 05/08/2026 | 05/08/2026 | — |
| 5 | - **Backend:** Consolidate the permission matrix in `permissions.py` <br> - **Frontend:** Build `PermissionGuard` to hide or disable UI elements based on the role in the JWT <br> - Write a migration script to update old DynamoDB data and resolve 500 errors caused by incompatible schemas <br> - Complete the attendance check-out flow and fix the hard-coded WFH check-in time issue | 06/08/2026 | 06/08/2026 | — |
| 6 | - **Deploy Frontend:** Configure the Amazon S3 bucket for Static Website Hosting and Bucket Policy; set up an Amazon CloudFront Distribution as the CDN, configure HTTPS, and verify the UI across multiple devices <br> - Write and publish **3 technical blog posts** in the AWS Study Group; update and redeploy the personal GitHub repository as evidence <br> - Update the report on activities and events attended and sync the related content into the workshop/report summary | 07/08/2026 | 08/08/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html> <br> <https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/> |

## 3. Personal contributions

* Contributed to the frontend this week:
  * Leaves.jsx with an interactive calendar;
  * leave request form;
  * WFH check-in button;
  * distinct status badges;
  * PermissionGuard for UI RBAC;
  * frontend deployment to S3 + CloudFront.
* Followed the Leave Management module from the UI through to the logic that synchronizes with Attendance, especially for approved WFH cases.
* Reviewed the permission matrix between backend and frontend to avoid hard-coded permissions scattered throughout the code.
* Supported the migration of older DynamoDB data, helping reduce errors caused by schema changes.
* Prepared and completed 3 technical blog posts while updating the personal GitHub and the related workshop/report content.

## 4. Achievements

* Frontend:
  * Completed the Leave Management module with an interactive calendar, registration form, and WFH handling.
  * RBAC in the UI became clearer through PermissionGuard.
  * The frontend was successfully deployed on S3 + CloudFront with HTTPS.

* Backend and data:
  * The Leave Management API worked with a clear approval flow.
  * RBAC was standardized into 5 roles and a more centralized permission matrix.
  * Migration of older data resolved several 500 errors caused by schema mismatch.

* Personal evidence and reporting:
  * Completed 3 technical blog posts.
  * Updated the personal GitHub and the report content related to the workshop and events attended.
