---
title : "Testing & Validation"
date : 2024-01-01
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

### Goal

After completing all deployment steps from sections **5.3 to 5.9**, this section will guide you through **End-to-End testing** of the entire Smart Campus system — from calling APIs, validating storage results, monitoring logs/metrics, to cleaning up resources after completion.

Each testing step has a **specific expected result**, helping you self-verify that the system is operating as designed.

---

### Testing Steps

| Section | Content | Related Services |
|---|---|---|
| **5.10.1** | API Testing (Swagger UI & Postman) | API Gateway, Lambda, Cognito |
| **5.10.2** | **End-to-End Business Workflows Testing** | *Multiple Services* |
| ↳ 5.10.2.1 | Attendance & Biometric Login | Rekognition, DynamoDB, S3 |
| ↳ 5.10.2.2 | Task & Incident Management | S3 Pre-signed URL, DynamoDB |
| ↳ 5.10.2.3 | Leave Management & Notifications | API Gateway, Cognito |
| **5.10.3** | Event Notification Flow Testing | EventBridge, SNS, SQS |
| **5.10.4** | Monitoring Logs & Metrics Testing | CloudWatch, X-Ray |


---
You can click on each item in the left navigation bar to view detailed instructions for each testing flow!
