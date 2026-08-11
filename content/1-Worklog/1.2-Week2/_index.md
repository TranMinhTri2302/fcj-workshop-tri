---
title: "Week 2 Worklog"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Week 2: Problem Analysis – System Architecture Design – Technical Stack Selection

## 1. Weekly objectives

* Update the project requirements, align the team’s chosen topic, and analyze the problem at the system level.
* Identify actors, use cases, and standardize the core business workflows to serve as input for the development process.
* Design the overall AWS architecture and update it based on mentor feedback.
* Compare backend deployment approaches, select a suitable technical stack, and initialize a monorepo structure.

## 2. Detailed work log

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Update the project requirements and internship guidance from the program <br> - Get familiar with the new team, exchange technical strengths and interests, and discuss the project direction <br> - Compare potential topics such as Smart Campus and Ticket System based on business scope, the ability to demonstrate multiple AWS services, and system scalability | 29/06/2026 | 29/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Align on the **Smart Campus Platform** project, a system for task management, attendance, and personnel management on AWS <br> - Analyze the problem by identifying 5 actor groups (employees, managers, technicians, directors, and admins), and list use cases for each role <br> - Define the relationships between modules: users, attendance, tasks, notifications, analytics, AI assistant, and monitoring | 30/06/2026 | 30/06/2026 | <https://docs.aws.amazon.com/> |
| 4 | - Study the AWS services expected to be used in depth: Cognito, DynamoDB, S3, Rekognition, Lambda, API Gateway, SNS, SES, EventBridge, Kinesis Firehose, Glue, Athena, Bedrock, X-Ray, and CloudWatch <br> - Read official documentation and note the use cases that fit the Smart Campus problem | 01/07/2026 | 01/07/2026 | <https://docs.aws.amazon.com/> |
| 5 | - Design the first version of the system architecture diagram, describing the problem and the main processing flows <br> - Define **8 core business workflows**: <br>&emsp; + WF1: Authentication (Cognito JWT) <br>&emsp; + WF2: Face Registration <br>&emsp; + WF3: Attendance + Rule Engine <br>&emsp; + WF4: Notification <br>&emsp; + WF5: Analytics <br>&emsp; + WF6: AI Assistant (Bedrock NL2SQL) <br>&emsp; + WF7: Security Monitoring <br>&emsp; + WF8: Task & Employee Management | 02/07/2026 | 02/07/2026 | <https://docs.aws.amazon.com/> <br> AWS Architecture Icons |
| 6 | - Share the architecture diagram in the WhatsApp community to receive feedback from mentors: add Region, VPC, Subnet; standardize AWS Architecture Icons; number the processing flows; adjust the FE/BE split and deployment approach <br> - Update the architecture diagram based on the feedback received <br> - Compare backend deployment options based on mentor suggestions: Lambda vs EC2/ECS/Elastic Beanstalk; identify cold-start issues with Java/Spring Boot on Lambda and pivot toward Python/FastAPI <br> - Study private/public architecture design, secret management with AWS Secrets Manager, and avoid hard-coding credentials <br> - Initialize the monorepo folder structure and set up a Python 3.11 environment | 03/07/2026 | 04/07/2026 | <https://docs.aws.amazon.com/secretsmanager/> <br> AWS Well-Architected Framework |

## 3. Personal contributions

* Took responsibility for the problem analysis phase at the technical input level: identifying actors, grouping use cases, and organizing them into 8 workflows so the team had a shared understanding before coding.
* Standardized the workflow description and documented the main flows to make module division easier.
* Updated the architecture diagram based on mentor feedback, especially in the areas of VPC, Subnet, Region, AWS icons, and flow numbering.
* Proposed using **Python/FastAPI** for the backend because it fit better with the goal of rapid PoC development, serverless support, and reduced cold-start risk.
* Helped initialize the monorepo and prepare the architecture diagram and workflow descriptions for the implementation phase.

## 4. Achievements

* Business analysis:
  * Aligned on the Smart Campus Platform project with 5 actor groups and corresponding use case sets.
  * Standardized 8 core workflows as the foundation for team assignment and implementation in the following weeks.

* Architecture design:
  * Completed a system architecture diagram at a level sufficient for mentor review and for the team to begin development.
  * Clarified the network components and the boundaries between frontend, backend, data, and monitoring.

* Technology selection:
  * Finalized the preferred backend direction as Python/FastAPI in line with the serverless strategy.
  * Initialized the monorepo structure and the initial development environment, preparing the project for the coding phase.
