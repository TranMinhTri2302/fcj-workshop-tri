---
title : "Configure Amazon SQS"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3. </b> "
---

#### 5.6.3. Configure Amazon SQS (Queue)
Amazon SQS (Simple Queue Service) acts as a "buffer". By routing events into SQS before calling Lambda to process them (sending emails, writing Analytics logs), the system will never be overloaded or lose data even when thousands of students mark attendance simultaneously.



**Step 1: Create a Dead-Letter Queue (DLQ)**

1. Search for and access the **SQS** service on the AWS Console.
> ![Search SQS](/aws-image/setupSQS/sqs1.png)
2. Click **Create queue**.
> ![Create Queue](/aws-image/setupSQS/sqs2.png)
3. **Type**: Select **Standard**.
> ![Select Standard](/aws-image/setupSQS/sqs3.png)
4. **Name**: Enter `smart-campus-dlq`.
> ![Enter DLQ Name](/aws-image/setupSQS/sqs4.png)
5. Scroll down to the end and click **Create queue**.
> ![Create DLQ](/aws-image/setupSQS/sqs5.png)

**Step 2: Create Main Queue and attach DLQ**

1. Return to the SQS list page, continue to click **Create queue** a second time.
> ![Create Main Queue](/aws-image/setupSQS/sqs6.png)
2. **Type**: Select **Standard**.
> ![Select Standard](/aws-image/setupSQS/sqs7.png)
3. **Name**: Enter `smart-campus-analytics-queue` (or `smart-campus-notification-queue` depending on the flow you want to set up).
> ![Main Queue Name](/aws-image/setupSQS/sqs8.png)
4. Scroll down to the **Dead-letter queue** section:
   - Select **Enabled**.
> ![Enable DLQ](/aws-image/setupSQS/sqs9.png)
   - **Choose queue**: Point to the `smart-campus-dlq` just created in Step 1.
> ![Select DLQ](/aws-image/setupSQS/sqs10.png)
   - **Maximum receives**: Set to `3` (Meaning if the system tries to process 3 times and still fails, the message will be pushed to the DLQ).
> ![Max receives](/aws-image/setupSQS/sqs11.png)
5. Open the **Access policy** section. For EventBridge to be able to send messages to SQS, we need to grant `sqs:SendMessage` permission to the `events.amazonaws.com` service.
> ![Open Access Policy](/aws-image/setupSQS/sqs12.png)
6. In the JSON box, replace the old content with the following JSON snippet (remember to replace `{AccountID}` and `{Region}` with your information):
```json
{
  "Version": "2012-10-17",
  "Id": "Queue1_Policy_UUID",
  "Statement": [
    {
      "Sid": "EventBridgePublishToSQS",
      "Effect": "Allow",
      "Principal": {
        "Service": "events.amazonaws.com"
      },
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:ap-southeast-1:123456789012:smart-campus-analytics-queue"
    }
  ]
}
```
> ![Enter JSON Policy](/aws-image/setupSQS/sqs13.png)
7. Scroll down and click **Create queue**.
> ![Click Create](/aws-image/setupSQS/sqs14.png)

**Step 3: Enable Lambda Trigger from SQS (Optional)**

If you want SQS to automatically call another Lambda function to process data:
1. Open the newly created Main Queue, switch to the **Lambda triggers** tab.
> ![Lambda Trigger Tab](/aws-image/setupSQS/sqs21.png)
2. Click **Configure Lambda function trigger**.
> ![Click Configure](/aws-image/setupSQS/sqs22.png)
3. Select the Lambda function (e.g., `smart-campus-analytics-processor`).
> ![Select Lambda](/aws-image/setupSQS/sqs24.png)
4. Click **Save**.
> ![Click Save](/aws-image/setupSQS/sqs25.png)
5. The system shows that Lambda has been successfully triggered.
> ![Success](/aws-image/setupSQS/sqs26.png)

Now the SQS queue is ready. The final step to piece this picture together is **Amazon EventBridge**.
