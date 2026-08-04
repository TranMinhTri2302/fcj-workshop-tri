---
title: "Week 4 Worklog"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Begin implementing the Smart Campus project with Python/FastAPI and organize the codebase into maintainable modules.
* Build user authentication using Amazon Cognito and protect secured routes through JWT-based middleware.
* Integrate DynamoDB, S3, CloudWatch, and GitHub Actions to support application development and monitoring.

### Tasks completed this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Start implementing the Smart Campus project with Python/FastAPI and organize code into modules such as routers, services, models, and schemas | 13/07/2026 | 13/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Build authentication with Amazon Cognito, including the register–login–confirm flow and JWT-decoding middleware | 14/07/2026 | 14/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Connect DynamoDB and create tables for Users, Events, and Checkins, while setting up a Global Secondary Index for attendance history queries | 15/07/2026 | 15/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Develop core APIs for event management, user profile retrieval, and check-in, including duplicate prevention using ConditionExpression | 16/07/2026 | 16/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Integrate S3 through presigned URLs for direct image uploads <br> - Configure CloudWatch logs and alarms <br> - Set up GitHub Actions workflows for pytest and linting | 17/07/2026 | 17/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 7 | - Consolidate understanding of JWT/OIDC in Cognito, effective NoSQL data modeling, and a practical CI/CD pipeline | 18/07/2026 | 18/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 4 Achievements:

* Started implementing the Smart Campus project with a modular Python/FastAPI structure that improves maintainability and scalability.
* Built a working authentication flow through Amazon Cognito and added middleware to automatically protect secured routes.
* Connected DynamoDB and implemented tables for Users, Events, and Checkins, with a GSI to support attendance history lookups.
* Built the main APIs for event management, profile retrieval, and check-in while preventing duplicate submissions.
* Integrated S3 using presigned URLs so clients can upload images directly, which reduces server-side load.
* Configured CloudWatch monitoring and alarm notifications and set up CI/CD automation with GitHub Actions.
* Deepened understanding of JWT/OIDC, NoSQL design, and basic DevOps practices for real-world project delivery.
