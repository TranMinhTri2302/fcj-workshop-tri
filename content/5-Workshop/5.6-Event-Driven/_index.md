---
title : "Event-Driven Architecture"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

# Part 4: Event-Driven Architecture

Up to this point, our attendance API operates on a **Synchronous** mechanism. This means that when a user uploads an image, the API Gateway sends it to Lambda. Lambda calls AI, saves to DB, saves to S3, and only then returns a successful result to the Frontend.

This approach has a drawback: If we want to add features like "Send a successful attendance Email" or "Security alert if a stranger is detected", stuffing all the code into the initial Lambda function will drastically increase the Response Time, leading to poor user experience (long wait times) or even Timeouts.

To solve this problem, we will apply the **Event-Driven Architecture** with AWS's classic trio of tools:
- **Amazon EventBridge**: A transit hub (Event Bus) that receives and routes events.
- **Amazon SNS (Simple Notification Service)**: A Pub/Sub system used to broadcast notifications to multiple endpoints (like Email, SMS, etc.).
- **Amazon SQS (Simple Queue Service)**: A queue that temporarily stores messages for asynchronous processing, preventing system overload.
- **Amazon SES (Simple Email Service)**: A dedicated email sending service.

### New Processing Flow:
1. After saving to the DB, instead of independently sending an Email, the main Lambda function (in section 5.5) does only one thing: **Emits an Event** containing `CHECKIN_SUCCESS` (Attendance successful) or `SECURITY_ALERT` (Security alert) to **EventBridge**.
2. Immediately after emitting the Event, Lambda responds with the result to the user (very fast!).
3. In the background, **EventBridge** will use Rules to catch these Events and route them:
   - `SECURITY_ALERT` events will be pushed straight to an **SNS Topic** to instantly send security alerts.
   - `CHECKIN_SUCCESS` events will be pushed into an **SQS Queue** acting as a buffer, then another Worker (or direct processing) will pull the info, combine it with **Amazon SES** to send attendance report Emails to HR.

In the following modules, we will sequentially set up SES, SNS, SQS, and finally configure EventBridge to link them all together.

{{% children /%}}
