---
title: "Workshop Overview"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Introduction to Smart Campus Platform

**Smart Campus Platform** is a smart attendance system powered by AI face recognition technology, built entirely on AWS serverless architecture. The system addresses the problems of fraudulent and time-consuming attendance processes in schools and enterprises.

#### Real-World Problem

In many organizations, attendance is still recorded manually or through card-based systems:
- **Problem 1**: Easy to cheat (proxy check-ins, printed photos)
- **Problem 2**: Time-consuming (5-10 minutes per session for 50 people)
- **Problem 3**: Difficult to compile reports (requires manual Excel processing)
- **Problem 4**: No real-time alerts when violations occur

#### Workshop Flow

You will build the system in 12 steps:

1. **Setup DynamoDB**: Create 3 tables (users, faces, attendance)
2. **Create Rekognition Collection**: Store face embeddings
3. **Deploy Lambda Functions**: Main API + 2 workers
4. **Configure API Gateway**: Expose Lambda via HTTP endpoint
5. **Setup EventBridge**: Event bus + routing rules
6. **Create SQS Queues**: Analytics Queue + Notification Queue with DLQ
7. **Deploy Lambda Workers**: Process events from SQS
8. **Setup Analytics Pipeline**: S3 Data Lake + Glue + Athena
9. **Testing**: Test the end-to-end attendance flow
10. **Monitoring**: CloudWatch Logs + Metrics + Alarms
11. **Optimization**: Security hardening + cost optimization
12. **Cleanup**: Delete all resources

**Estimated cost:** ~$10-15 USD if you run the full workshop and clean up within 24 hours.

#### Next Step

Start with [Prerequisites](../5.2-prerequisite) to prepare your environment!