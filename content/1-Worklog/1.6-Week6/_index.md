---
title: "Week 6 Worklog"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

* Upgrade the Analytics/Reports module and implement an analytics dashboard for the system.
* Improve the production notification flow with Amazon SES/SNS and standardize notification data formatting.
* Fully integrate Amazon Cognito into the authentication flow and build face-based account recovery support.
* Attend the AWS FCAJ Agent Forge – Deepdive workshop to deepen understanding of Agentic AI architecture and multi-layer security.

### Tasks completed this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Upgrade the Analytics/Reports module with new endpoints and schemas to support department-level filtering | 27/07/2026 | 27/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Implement the Analytics page on the frontend to visualize system data | 28/07/2026 | 28/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Improve the notification system for production readiness by switching to 1-to-1 email delivery with Amazon SES and fallback to SNS when needed | 29/07/2026 | 29/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Fully integrate Amazon Cognito into the authentication flow, including handling NEW_PASSWORD_REQUIRED and implementing AuthContext and ProtectedRoute in the frontend | 30/07/2026 | 30/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Develop face-based account recovery support, where the backend verifies a user through Rekognition and resets the password on Cognito | 31/07/2026 | 31/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 7 | - Attend the AWS FCAJ Agent Forge – Deepdive workshop and learn about Agentic AI, Bedrock Agent Core, Firecracker MicroVM, MCP/A2A, Guardrails, and Human-in-the-loop design | 03/08/2026 | 03/08/2026 | <https://cloudjourney.awsstudygroup.com/> <br> [Agent Forge workshop video](https://www.youtube.com/watch?v=F58sam40jxk) |
| 8 | - Consolidate understanding of production-grade Agentic AI system design, human-in-the-loop principles, layered security, and model selection for cost-performance balance | 01/08/2026 | 01/08/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 6 Achievements:

* Upgraded the Analytics/Reports module and implemented a frontend analytics page for clearer data visualization.
* Improved the production notification system by shifting from broadcast messaging to 1-to-1 email delivery through Amazon SES, with SNS fallback for reliability.
* Fully integrated Amazon Cognito into the authentication flow, including handling first-time password change requirements and protecting frontend routes.
* Developed a face-based account recovery flow to improve identity verification and recovery experience.
* Strengthened overall security by reviewing IAM roles, avoiding secret leakage, reviewing AWS costs, and cleaning up unused resources.
* Gained a deeper understanding of Agentic AI architecture, Human-in-the-loop design, multi-layer security, and the selection of suitable AI models for balancing cost and performance.
