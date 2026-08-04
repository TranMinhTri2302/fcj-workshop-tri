---
title: "Week 5 Worklog"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Implement the Tasks module using a layered architecture and optimize DynamoDB access with GSI and cursor-based pagination.
* Add business logic for auto-assignment, role-based access control, and status-driven notifications.
* Integrate face recognition with Amazon Rekognition into the check-in workflow and strengthen automated testing.

### Tasks completed this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Implement the Tasks module and design a DynamoDB table with the `assignee_id-status-index` GSI for efficient querying | 20/07/2026 | 20/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Complete cursor-based pagination for DynamoDB <br> - Add business rules such as auto-assignment, role-based update permissions, and notification triggers at each status milestone | 21/07/2026 | 21/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Integrate Amazon Rekognition to build face registration and check-in APIs using SearchFacesByImage | 22/07/2026 | 22/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Enforce business-hour restrictions for face-based check-in <br> - Improve automated tests with moto for Rekognition, DynamoDB, and SES | 23/07/2026 | 23/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Update GitHub Actions CI/CD pipelines to run the full test suite | 24/07/2026 | 24/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 7 | - Consolidate the week’s learning around GSI design, cursor pagination, AI integration, and access control | 25/07/2026 | 25/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 5 Achievements:

* Implemented the Tasks module using a layered architecture, with standardized DynamoDB table naming configured through settings variables.
* Designed the Tasks table with a GSI to support efficient queries by assignee and status while avoiding unnecessary scan operations.
* Completed cursor-based pagination for DynamoDB, which is more appropriate for NoSQL systems than offset-based pagination.
* Added business logic for auto-assignment, role-based permissions, and milestone notifications that align with the project workflow.
* Integrated Amazon Rekognition into the face check-in flow, including face registration, matching, and handling cases such as no face detected or low image quality.
* Enforced time-based restrictions for face check-in and expanded automated testing to cover the new features.
* Improved the CI/CD pipeline and gained a deeper understanding of GSI design, pagination patterns, AI integration in real workflows, and the importance of role-based access control.
