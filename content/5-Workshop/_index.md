---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---


# Smart Campus - Serverless Attendance System with Face Recognition

## Building a Smart Attendance System with AWS Rekognition, Lambda, DynamoDB, and Event-Driven Architecture

#### Overview

In this workshop, you will learn how to build a production-grade serverless attendance system using AI-powered face recognition technology. The system not only recognizes faces but also integrates **Face Liveness Detection** to prevent spoofing (detecting printed photos, video replays, and 3D masks), builds an **Event-Driven** architecture with EventBridge and SQS to ensure reliability, and creates a **Data Lake** for analytics using Athena and Glue.

**What you will learn:**
- Designing and deploying serverless architecture on AWS
- Integrating AI/ML services (Rekognition Face Recognition + Liveness Detection)
- Building event-driven systems with EventBridge and SQS
- Implementing reliable messaging with Dead Letter Queues
- Creating a Data Lake analytics pipeline with S3, Glue, and Athena
- Setting up monitoring and alerting with CloudWatch
- Applying security best practices (WAF, Cognito, IAM)
- Optimizing costs for serverless applications

**Estimated duration:** 3-4 hours

**Estimated cost:** ~$10-15 USD (if cleaned up after 24 hours)

#### Core Workflow

In this workshop, you will deploy a complete attendance workflow:

```
Camera captures face image
  → API Gateway receives request
  → Lambda processes
  → Rekognition Face Liveness Check (detecting printed photo/video)
  → Rekognition Face Recognition (face identification)
  → Rule Engine validation (duplicate check, session check, time check)
  → Save attendance record to DynamoDB
  → Publish event to EventBridge
  → EventBridge routes to SQS Queues
  → Lambda Workers process (Analytics, Notification)
  → Result: Record in Data Lake, Email notification
```

**Types of endpoints you will create:**
- **Gateway VPC Endpoint**: To access S3 from Lambda within a VPC (if needed)
- **Interface VPC Endpoint**: To access Rekognition and DynamoDB from a VPC (optional)
- **API Gateway HTTP Endpoint**: Public endpoint for attendance check-in

#### Table of Contents

1. [Workshop Overview](5.1-overview)
2. [Prerequisites](5.2-prerequisite/)
3. [Step 1: Setup DynamoDB Tables](5.3-dynamodb/)
4. [Step 2: Create Rekognition Face Collection](5.4-rekognition/)
5. [Step 3: Deploy Lambda Functions](5.5-lambda/)
6. [Step 4: Configure API Gateway](5.6-apigateway/)
7. [Step 5: Setup EventBridge & SQS](5.7-eventbridge/)
8. [Step 6: Create Analytics Pipeline](5.8-analytics/)
9. [Step 7: System Testing](5.9-testing/)
10. [Step 8: Monitoring with CloudWatch](5.10-monitoring/)
11. [Step 9: Cost and Security Optimization](5.11-optimization/)
12. [Cleanup Resources](5.12-cleanup/)