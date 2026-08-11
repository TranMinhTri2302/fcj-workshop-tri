---
title : "Monitoring Testing"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.10.4. </b> "
---

#### 5.10.4. Log, Metric, and Tracing Testing (CloudWatch & X-Ray)

This section guides you to view real-time logs, check metrics, and analyze request flows via X-Ray to confirm the monitoring system is operating correctly.

---

**Step 1: View Lambda Logs on CloudWatch**

1. Go to AWS Console > **CloudWatch** > **Log groups**.
> ![Search CloudWatch](/aws-image/setupTestgiamsat/giamsat1.png)
2. Find and select the log group named `/aws/lambda/smart-campus-api`.
> ![Log groups](/aws-image/setupTestgiamsat/giamsat2.png)
3. From the **Log streams** list, click on the newest stream (usually the top row with the most recent timestamp).
> ![Log streams](/aws-image/setupTestgiamsat/giamsat3.png)
4. You will see detailed event log lines for each API call, including:
   - Request start and end times
   - Result returned from Rekognition
   - Record info written to DynamoDB
   - Event emitted to EventBridge
> ![Log events](/aws-image/setupTestgiamsat/giamsat4.png)

> **Expected Result:** Logs clearly show processing steps, with no `ERROR` or `Exception` lines.

---

**Step 2: Check Lambda Metrics**

1. Go to **CloudWatch**, look at the left menu under **Metrics** > Select **Classic metrics**.
2. At the **Browse** tab below the chart, select the **AWS/Lambda** namespace.
> ![Metrics Interface](/aws-image/setupTestgiamsat/giamsat5.png)
3. Continue selecting the **By Function Name** dimension.
> ![Select Dimension](/aws-image/setupTestgiamsat/giamsat6.png)
4. Find the `smart-campus-api` function and check the metrics to track:
   - **Invocations**: Number of function calls (should increase corresponding to the number of times you tested)
   - **Duration**: Average processing time (ms)
   - **Errors**: Number of errors (expected to be 0)
> ![Select Metric](/aws-image/setupTestgiamsat/giamsat7.png)
5. Adjust the time range on the toolbar (e.g., **1h** or **3h**) and observe the chart.

> **Expected Result:** The chart displays API activity data, not empty.

---

**Step 3: Analyze Traces on X-Ray**

1. Go to **CloudWatch**, on the left menu scroll down to find and select **Trace Map**.
2. Select a time range (e.g., **30m**) on the toolbar. You will see a visual Service Map illustrating the flow of the request.
> ![Trace Map](/aws-image/setupTestgiamsat/giamsat8.png)
3. Click the **smart-campus-api** node on the map to see a detailed analysis panel of Latency, Requests, and Faults in the bottom pane.
> ![Node Details](/aws-image/setupTestgiamsat/giamsat9.png)
4. Looking at the left menu, switch to the **Traces** section to see a list of specific traces, from which you can click on each trace to analyze the processing time chart.
> ![Traces](/aws-image/setupTestgiamsat/giamsat10.png)

---

**Step 4: Trigger CloudWatch Alarm (Alert Testing)**

To verify the alert system operates smoothly without waiting for a real incident, we will perform a simulated test (Simulation Test). By using the AWS CLI to proactively trigger an Alarm, we can evaluate the entire notification flow from CloudWatch to SNS and Email.

1. Open AWS CloudShell (the >_ icon on the top right toolbar) or use your terminal.
2. Run the following command to simulate an alert:
```bash
aws cloudwatch set-alarm-state --alarm-name "Lambda-Error-Alert" --state-value ALARM --state-reason "Test Alert"
```
3. Immediately, the Alarm state will transition from `OK` → `In alarm` (You can see this at **CloudWatch** > **Alarms**).
> ![Alarms list](/aws-image/setupTestgiamsat/giamsat11.png)
4. Simultaneously, you will receive an alert Email from the SNS Topic `smart-campus-notifications`.
> ![Alert Email](/aws-image/setupTestgiamsat/giamsat12.png)

> **Expected Result:** Alarm changes state and an alert Email is sent to your inbox.
