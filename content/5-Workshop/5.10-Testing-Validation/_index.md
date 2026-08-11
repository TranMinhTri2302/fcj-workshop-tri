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
| **5.10.2** | Facial Recognition Attendance Testing | Rekognition, DynamoDB, S3 |
| **5.10.3** | Event Notification Flow Testing | EventBridge, SNS, SQS |
| **5.10.4** | Monitoring Logs & Metrics Testing | CloudWatch, X-Ray |


> [!IMPORTANT]
> Before starting testing, ensure you have **returned to section 5.5.2** to fully fill in all environment variables for the Lambda function (SNS Topic ARN, SES Email...) from section 5.6.


#### 1. Test sending Requests (Postman / Frontend)
1. Get the API Gateway URL (e.g., `https://xyz.execute-api.ap-southeast-1.amazonaws.com/prod/attendance`).
2. Open Postman, select the **POST** method. Paste the URL.
3. In the **Body** > **raw** > **JSON** tab, enter a payload containing a student's face image (base64) and `camera_id`.
4. Click **Send**.
5. Receive a `200 OK` result with the message: "Attendance successful for student ...".

#### 2. Validate storage in DynamoDB & S3
1. **DynamoDB:** Access AWS Console > DynamoDB > Tables > `smart-campus-attendance`.
   - Click **Explore table items**.
   - You will see a new record appear with `attendance_id`, time, and attendance status.
2. **S3 (Image Storage):** Access S3 bucket `smart-campus-images`.
   - Check if the submitted face image file was saved with the format `YYYY-MM-DD/ID.jpg`.

#### 3. Validate Logs & Metrics (CloudWatch)
1. Access CloudWatch > Log groups > Select the log group for Lambda `smart-campus-api`.
2. Check the latest log stream to see real-time execution `print` statements, results returned from Amazon Rekognition.
3. Switch to the **Metrics** section, select the Lambda function and check the **Invocations** chart to see if the chart bar increments by 1.

#### 4. Validate Event-Driven (SNS / SQS)
1. Open the Mailbox (Gmail) you registered with SNS in Step 5.6.
2. You will receive a new Email from AWS reporting the attendance event.
3. If you have SQS configured, go to SQS > Select `smart-campus-analytics-queue` > Click **Send and receive messages** > **Poll for messages** to see if the event routed from EventBridge was pushed into the queue.

If all the above steps return expected results, congratulations! You have successfully deployed a 100% Serverless Event-Driven architecture on AWS.
