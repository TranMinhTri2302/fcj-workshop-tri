---
title: "Week 3 Worklog"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Week 3: Initializing Frontend & Backend – Integrating Cognito, DynamoDB, S3 – Setting Up CI/CD

## 1. Weekly objectives

* Initialize a React project with Vite, build a shared CSS system, create core components, and design the main application layout.
* Build the Python/FastAPI backend foundation in a modular structure following the Repository – Service – Router approach.
* Integrate core AWS services: Amazon Cognito, DynamoDB, and S3 to form the initial system foundation.
* Set up a basic CI/CD pipeline and begin initial testing to maintain code quality.

## 2. Detailed work log

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - **Frontend:** Initialize a React project with Vite (React + JavaScript), clean up the default files, and set up a standard folder structure: components, pages, hooks, services, utils, assets <br> - **Backend:** Align the code organization around modules such as routers, services, models, and schemas; standardize naming and coding conventions across the team | 06/07/2026 | 06/07/2026 | <https://vitejs.dev/guide/> <br> <https://fastapi.tiangolo.com/> |
| 3 | - **Frontend:** Build a foundational styling system using a Glassmorphism approach: configure global CSS variables, color palette, spacing, border radius, shadows, backdrop-filter, and shared utility classes <br> - **Backend:** Create the initial backend foundation following the Repository – Service – Router pattern; create shared configuration files such as settings, dependency injection, and error handling <br> - Begin building the authentication flow through Amazon Cognito: registration, sign-in, and account confirmation | 07/07/2026 | 07/07/2026 | <https://docs.aws.amazon.com/cognito/> |
| 4 | - **Frontend:** Build reusable Core Components: Button, Input, Select/Dropdown, Loading, Alert/Toast, and Empty State <br> - **Backend:** Complete JWT decoding middleware for protected routes; connect DynamoDB by creating the Users, Events, and Checkins tables and set up a GSI for attendance history queries by user | 08/07/2026 | 08/07/2026 | <https://docs.aws.amazon.com/amazondynamodb/> <br> <https://boto3.amazonaws.com/v1/documentation/api/latest/> |
| 5 | - **Frontend:** Design the main application layout: Sidebar Navigation, Header, main content area, breadcrumbs, and child page container <br> - **Backend:** Integrate S3 by generating presigned URLs that allow the client to upload images directly, reducing backend load; build a service layer for DynamoDB operations using boto3, separated from the router layer | 09/07/2026 | 09/07/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/> |
| 6 | - **Frontend:** Build a reusable DataTable component with pagination, sorting, search, and mobile responsiveness <br> - **Backend:** Review all endpoints through Swagger UI (/docs), compare them with the workflow and use cases defined in Week 2, and identify missing APIs <br> - Set up CI/CD with GitHub Actions to run pytest and lint; build initial test cases using moto to simulate DynamoDB and Cognito <br> - Review the overall codebase and evaluate the initial technical output before integration | 10/07/2026 | 11/07/2026 | <https://docs.github.com/en/actions> <br> <https://github.com/getmoto/moto> |

## 3. Personal contributions

* Served as the main contributor to the frontend this week: initialized the Vite project, set up the styling system, built the core component set, the main layout, and the shared DataTable.
* Reviewed the core API specifications to ensure that the frontend and backend aligned with the workflows finalized in Week 2.
* Added Cognito/JWT authentication at the middleware level and protected routes to lay the foundation for future RBAC implementation.
* Supported the organization of the service layer for DynamoDB and kept the data processing logic separate from the router layer for easier testing.
* Connected presigned URLs with the file upload flow from the frontend to prepare for face-processing features in the following week.
* Reviewed the endpoints through Swagger UI and documented the points that still needed alignment in field names, payloads, and responses.

## 4. Achievements

* Frontend:
  * Completed the core React application structure with a shared CSS system, core components, and main layout.
  * The shared DataTable was ready for use in data-management pages.

* Backend:
  * Built the backend foundation using a modular structure with 7 main functional groups.
  * Swagger UI worked stably and was convenient for manual testing and specification review.

* AWS integration and CI/CD:
  * Completed the foundational authentication flow with Cognito/JWT.
  * DynamoDB, S3 presigned URLs, and the GitHub Actions pipeline were all set up, creating a base for the business features in the following week.
