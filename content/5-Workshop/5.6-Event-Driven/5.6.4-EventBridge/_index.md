---
title : "Configure Amazon EventBridge"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.6.4. </b> "
---

#### 5.6.4. Configure Amazon EventBridge (Event orchestration)
EventBridge is the "heart" of the event-driven architecture. Whenever Lambda finishes processing a facial recognition flow, instead of calling the email-sending API itself, it "broadcasts" an event to EventBridge.
EventBridge will listen to this event and automatically route it to suitable Targets like SNS (to send Emails) or SQS (to save Logs).

**Step 1: Create a Rule (Routing rule) to SNS**
We will create a rule: Whenever there is an attendance event, instantly push it to SNS.

1. Search for the **Amazon EventBridge** service on the AWS Console.
> ![Search EventBridge](/aws-image/setupEvenBridge/event-1.png)
2. On the left menu, select **Rules**.
> ![Select Rules](/aws-image/setupEvenBridge/event-2.png)
3. Click **Create rule**.
> ![Click Create Rule](/aws-image/setupEvenBridge/event-3.png)
4. **Name**: Enter `attendance-recorded-to-sns`. **Event bus**: Select `default` (or the Event Bus you are using).
> ![Name Rule](/aws-image/setupEvenBridge/event-4.png)
5. **Rule type**: Select **Rule with an event pattern**. Click **Next**.
> ![Select Rule Type](/aws-image/setupEvenBridge/event-5.png)
6. In the **Event pattern** section:
   - Select **Custom pattern (JSON editor)**.
   - Enter the following JSON snippet to accurately catch the attendance event:
```json
{
  "source": ["smart-campus.api"],
  "detail-type": ["AttendanceRecorded"]
}
```
> ![Configure Event Pattern](/aws-image/setupEvenBridge/event-6.png)
7. Click **Next**. In the **Select target(s)** section:
   - **Target types**: Select **AWS service**.
   - **Select a target**: Select **SNS topic**.
> ![Select Target Type](/aws-image/setupEvenBridge/event-7.png)
8. **Topic**: Select `smart-campus-notifications` (the Topic created in the SNS section).
> ![Select SNS Topic](/aws-image/setupEvenBridge/event-8.png)
9. Click **Next** past the Add tags step.
> ![Click Next Tags](/aws-image/setupEvenBridge/event-9.png)
10. Review the entire configuration on the Review screen and click **Create rule**.
> ![Create Rule](/aws-image/setupEvenBridge/event-10.png)

**Step 2: Create Routing Rule to SQS (Optional)**
Similarly, we will create a 2nd Rule to push events to SQS to serve Data Analytics.

1. At the Rules screen, click **Create rule** again.
> ![Create Rule 2](/aws-image/setupEvenBridge2/event1.png)
2. **Name**: Enter `attendance-to-sqs`.
> ![Rule 2 Name](/aws-image/setupEvenBridge2/event2.png)
3. Select **Rule with an event pattern** and then click **Next**.
> ![Rule Type 2](/aws-image/setupEvenBridge2/event3_1.png)
4. Similarly, under **Event pattern**, select **Custom pattern (JSON editor)**.
> ![Custom Pattern](/aws-image/setupEvenBridge2/event4_1.png)
5. Enter the identical JSON as in Step 1.
> ![Pattern JSON](/aws-image/setupEvenBridge2/event4_2.png)
6. Click **Next**.
> ![Next Pattern](/aws-image/setupEvenBridge2/event5.png)
7. In the **Target** section, select **AWS service**, but this time for **Select a target**, you choose **SQS queue**.
> ![Select SQS Target](/aws-image/setupEvenBridge2/event6_1.png)
8. Select the `smart-campus-analytics-queue` Queue that you created.
> ![Select Queue](/aws-image/setupEvenBridge2/event6_2.png)
9. Click **Next** until the Review screen and click **Create rule**.
> ![Create SQS Rule](/aws-image/setupEvenBridge2/event7.png)

At this point, the entire Event-Driven architecture flow is complete!
- Lambda emits events to **EventBridge**.
- EventBridge pushes the event to 2 independent branches: SNS (to send Emails) and SQS (to store in the queue).
