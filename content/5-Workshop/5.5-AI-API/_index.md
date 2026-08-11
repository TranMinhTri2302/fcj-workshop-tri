---
title : "Core API Config"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

### Goal

This is the "heart" of the Smart Campus system. In this section, you will sequentially:
1. Create a **Rekognition Collection** — a facial vector repository for AI recognition.
2. Deploy **AWS Lambda** — the function that handles all business logic (receiving images, calling AI, saving to DynamoDB/S3, emitting Events).
3. Create an **API Gateway** — the entry point receiving requests from the Frontend and routing them down to Lambda.
4. Configure **AWS WAF** — to protect the attendance API, blocking access from outside the Campus network.


> The execution order is very important: **Rekognition → Lambda → API Gateway → WAF**. Each step depends on the results of the previous one.

### Detailed Practice Content

Please click on each item below in the left menu bar or click directly on the links below to follow the detailed steps:

{{% children /%}}
