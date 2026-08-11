---
title : "Monitoring & Tracing"
date : 2024-01-01
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

### Goal

To ensure the Smart Campus system always operates stably and is ready to respond when an incident occurs, we need tools to monitor the health of the system, view error logs, and measure performance.

- **Amazon CloudWatch:** Acts as a monitoring hub. It will collect Logs (activity journals) from Lambda, API Gateway, and EventBridge. It also provides Metrics (indicators) and Alarms (alerts) when there are abnormalities (e.g., a sudden increase in the number of errors).
- **AWS X-Ray:** An extremely powerful Trace tool for Serverless architectures. X-Ray will draw a visual connection map between services (Client -> API Gateway -> Lambda -> DynamoDB) and point out exactly where the bottleneck is, how many milliseconds it takes at each stage.

### Detailed Practice Content

Please click on each item below in the left menu bar or click directly on the links below to follow the detailed steps:

{{% children /%}}
