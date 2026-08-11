---
title : "Monitor with CloudWatch"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.9.1. </b> "
---

#### 5.9.1. Centralized Monitoring Configuration with CloudWatch
All Lambda Logs (results printed from `print` or `console.log` statements) are pushed straight into CloudWatch Logs automatically. Our task is to know how to view logs and create alerts to monitor the system.

**Step 1: View Lambda Logs**

1. From the AWS Console, open the **CloudWatch** service. Look at the left menu bar, in the **Logs** section, select **Log groups**. The list of log groups will appear.
> ![Select Log groups](/aws-image/setupCloudWatch/cloudwatch15.png)
2. Find and click on the log group named `/aws/lambda/smart-campus-api`. 
> ![Select Log group name](/aws-image/setupCloudWatch/cloudwatch16.png)
3. Scroll down to the **Log streams** section, click on the latest log stream. You will see all the details of the log lines (INFO, START, END) generated during the run.
> ![Log streams details](/aws-image/setupCloudWatch/cloudwatch17.png)

**Step 2: Create an Alarm (Automated Alert)**
Suppose we want to receive an email alert every time Lambda has an error (Errors > 0).

1. Still in the CloudWatch Console, you can type to search in the top box if needed.
> ![Search CloudWatch](/aws-image/setupCloudWatch/cloudwatch1.png)
2. On the left menu, in the **Alarms** section, select **All alarms** and click **Create alarm**.
> ![Create Alarm](/aws-image/setupCloudWatch/cloudwatch2.png)
3. Click the **Select metric** button.
> ![Select Metric](/aws-image/setupCloudWatch/cloudwatch3.png)
4. In the Browse section, select **Lambda**.
> ![Select Lambda](/aws-image/setupCloudWatch/cloudwatch4.png)
5. Continue to select **By Function Name**.
> ![By Function Name](/aws-image/setupCloudWatch/cloudwatch5.png)
6. Find the `smart-campus-api` function, check the box in the **Errors** column. Then click the **Select metric** button at the bottom corner.
> ![Check Errors](/aws-image/setupCloudWatch/cloudwatch6.png)
7. The Graph configuration screen appears, check the default Statistic which is usually **Sum** or **Average**, then scroll down.
> ![Configure Statistic](/aws-image/setupCloudWatch/cloudwatch7.png)
8. **Conditions**: Select Threshold type as **Static**. In the *Whenever Errors is* section, select **Greater/Equal** and enter `1`. Click **Next**.
> ![Configure Condition](/aws-image/setupCloudWatch/cloudwatch8.png)
9. At the **Configure actions** step, in the **Notification** section, click the **Add notification** button.
> ![Add notification](/aws-image/setupCloudWatch/cloudwatch9.png)
10. For *Alarm state trigger*, select **In alarm**. For *Send a notification to the following SNS topic*, select **Select an existing SNS topic** and choose `smart-campus-notifications` (The Topic you created in the SNS section).
> ![Configure SNS Topic](/aws-image/setupCloudWatch/cloudwatch10.png)
11. Scroll down to the bottom and click **Next**.
> ![Click Next](/aws-image/setupCloudWatch/cloudwatch11.png)
12. At the **Add alarm details** step, name the Alarm in the **Alarm name** box (e.g., `Lambda-Error-Alert`). You can enter an additional description in the box below if needed. Then scroll down and click **Next**.
> ![Name the Alarm](/aws-image/setupCloudWatch/cloudwatch12.png)
13. The Review screen allows you to review the entire configuration. Scroll down to the bottom and click **Create alarm**.
> ![Create Alarm](/aws-image/setupCloudWatch/cloudwatch13.png)
14. The system reports green success, your Alarm has been created and is ready to monitor errors.
> ![Success](/aws-image/setupCloudWatch/cloudwatch14.png)

That's it! Now, whenever the attendance system dies or has an error (Exception), you will receive an Alert Email immediately so you can fix it in time. Below is an example of an ALARM email that you will receive from Amazon SNS notifying that the system is encountering an error:

> ![Alert Email](/aws-image/setupCloudWatch/cloudwatch18.png)
