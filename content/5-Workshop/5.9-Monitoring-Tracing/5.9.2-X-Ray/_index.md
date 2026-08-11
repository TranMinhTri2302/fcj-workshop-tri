---
title : "Trace Architecture with X-Ray"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.9.2. </b> "
---

#### 5.9.2. Trace API flow with AWS X-Ray
With a Serverless architecture, a request from the Frontend can go through a series of services: API Gateway -> Lambda -> Rekognition -> DynamoDB -> SQS. If a request is running slow, how do we know which service is dragging down performance? That's when AWS X-Ray shines.

**Enable X-Ray for Lambda**

1. Access the **Lambda** service on the AWS Console search bar.
> ![Search Lambda](/aws-image/setupXRay/xray1.png)
2. Select your `smart-campus-api` function.
> ![Select Lambda function](/aws-image/setupXRay/xray2.png)
3. Switch to the **Configuration** tab, select the **Permissions** section and click on the **Execution role** link to open the IAM Console.
> ![Open Execution Role](/aws-image/setupXRay/xray3.png)
4. In the IAM Console, on the **Permissions** tab, click **Add permissions** and select **Attach policies**.
> ![Click Attach policies](/aws-image/setupXRay/xray4.png)
5. Search for the `AWSXRayDaemonWriteAccess` policy, check it and click **Add permissions**.
> ![Add X-Ray permissions](/aws-image/setupXRay/xray5.png)
6. Return to your Lambda screen, in the **Configuration** tab, select the **Monitoring and operations tools** section and click the **Edit** button.
> ![Edit Monitoring](/aws-image/setupXRay/xray6.png)
7. Toggle the switch in the **AWS X-Ray (Active tracing)** section and click **Save** to finish.
> ![Enable Active Tracing](/aws-image/setupXRay/xray7.png)

**Experience the Service Map and Traces**

1. Use Postman or Frontend to call a few attendance APIs to generate data. Then access the **CloudWatch** service on the AWS Console, scroll down the left menu and select the **Trace Map** section (or access via *X-Ray > Service map* if using the old interface). Here, you will see a visual map illustrating the flow of the request.
> ![Service Map](/aws-image/setupXRay/xray8.png)
2. You can click on any node (service) on the map (e.g., DynamoDB table) to view detailed **Metrics** charts below (including Latency, Requests, Faults). In addition, you can also switch to the **Traces** section on the left menu for deeper analysis of each request in a waterfall chart, to precisely find the bottleneck.
> ![Trace Details](/aws-image/setupXRay/xray9.png)
