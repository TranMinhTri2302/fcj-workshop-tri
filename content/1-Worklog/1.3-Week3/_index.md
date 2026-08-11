---
title: "Worklog Week 3"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Week 3: Frontend Initialization – Shared UI Foundation – Core API Alignment

## 1. Weekly Objectives

* Initialize the React project using Vite and set up a standard folder structure for the frontend.
* Build the shared CSS system, core reusable components, and the main application layout.
* Prepare the frontend API service layer and align endpoint specifications with the workflows defined in Week 2.
* Support the review of several foundational technical areas related to authentication, data handling, and file uploads in preparation for feature development.

## 2. Detailed Work Log

| Day | Tasks | Start Date | End Date | References |
| --- | --- | --- | --- | --- |
| 2 | - Initialized the React project with Vite (React + JavaScript), removed default boilerplate files, and set up a standard folder structure: `components`, `pages`, `hooks`, `services`, `utils`, `assets` <br> - Standardized the source code organization to make future module-based expansion easier <br> - Reviewed the workflows finalized in Week 2 to identify the screens and feature groups that should be prioritized first | 06/07/2026 | 06/07/2026 | <https://vitejs.dev/guide/> |
| 3 | - Built the base styling system in a Glassmorphism direction: configured global CSS variables, colors, spacing, border radius, shadows, and shared utility classes <br> - Prepared the router structure, overall layout, and shared application shell <br> - Reviewed the authentication API specification to ensure the frontend would align with the planned login and registration flows | 07/07/2026 | 07/07/2026 | <https://react.dev/> <br> <https://docs.aws.amazon.com/cognito/> |
| 4 | - Built reusable core components: Button, Input, Select/Dropdown, Loading, Alert/Toast, Empty State <br> - Reviewed naming conventions between the UI and the foundational APIs, especially for user and authentication-related data <br> - Helped review route protection and JWT handling at the input/integration level to prepare for RBAC implementation later | 08/07/2026 | 08/07/2026 | <https://docs.aws.amazon.com/amazondynamodb/> |
| 5 | - Designed the main application layout: Sidebar Navigation, Header, main content area, breadcrumb, and page container <br> - Prepared the frontend `services` layer for API calls, token storage, and shared request/response handling <br> - Reviewed the presigned URL upload flow so that the frontend could integrate smoothly in later stages | 09/07/2026 | 09/07/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/> |
| 6 | - Built a reusable DataTable component with pagination, sorting, search, and mobile responsiveness <br> - Rechecked the endpoints through Swagger UI and compared them against the planned interface to identify anything that still needed alignment in payloads, responses, and status codes <br> - Supported a review of core response structures for the main modules to avoid schema mismatches during later integration | 10/07/2026 | 11/07/2026 | <https://fastapi.tiangolo.com/> <br> <https://docs.github.com/en/actions> |

## 3. Personal Contributions

* Took primary responsibility for the frontend setup: initializing the Vite project, defining the folder structure, building the shared CSS system, creating the core reusable components, and implementing the main layout and reusable DataTable.
* Reviewed the core API specifications so that the UI would closely follow the workflows finalized in Week 2.
* Prepared the frontend API layer and data handling approach to support later integration with authentication, file uploads, and business modules.
* Helped review JWT-related handling, payload/response consistency, and naming conventions to reduce mismatches during integration with the server side.

## 4. Outcomes Achieved

* Completed the React application skeleton with a clear folder structure, shared styling system, core reusable components, and the main layout.
* Finished a reusable DataTable component ready for future data management screens.
* Aligned the foundational endpoints with the previously defined workflows, reducing gaps between the UI and the APIs.
* Completed the API service layer setup and prepared the foundation for authentication, image upload, and data display flows in the following weeks.