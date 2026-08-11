---
title : "Configure Amazon SNS"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2. </b> "
---

#### 5.6.2. Configure Amazon SNS (Send notifications)
Amazon SNS (Simple Notification Service) is a Pub/Sub service. We will create a "broadcast channel" (called a **Topic**) and allow Email addresses (like the SES just created) to Subscribe to this channel. When a message is pushed to the Topic, everyone who is subscribed will receive it.

**Step 1: Create an SNS Topic**

1. Search for and access the **Simple Notification Service** on the AWS Console.
> ![Search SNS](/aws-image/setupSNS/sns-1.png)
2. On the SNS homepage (Dashboard), in the **Create topic** frame, enter the Topic name as `smart-campus-notifications` and click **Next step**.
> ![Enter Name](/aws-image/setupSNS/sns-2.png)
3. On the detailed configuration page, under the **Type** section, select **Standard** (to support sending Emails, SMS). The Topic name is auto-filled.
> ![Select Standard](/aws-image/setupSNS/sns-3.png)
4. Scroll down to the bottom of the page and click **Create topic**.
> ![Create Topic](/aws-image/setupSNS/sns-4.png)

**Step 2: Create a Subscription**
After successfully creating the Topic, the system will report success and redirect you to the details page of that Topic. Now we will add the HR Email to the mailing list.

1. Right on the Topic details page, in the **Subscriptions** tab, click the orange **Create subscription** button.
> ![Create Subscription](/aws-image/setupSNS/sns-5.png)
2. On the registration screen:
   - **Topic ARN**: The system automatically selects the Topic you just created.
   - **Protocol**: Select **Email**.
   - **Endpoint**: Enter the Email address you verified in the SES section (e.g., `danhbattu2049@gmail.com`).
   - Scroll down and click **Create subscription**.
> ![Configure Subscription](/aws-image/setupSNS/sns-6.png)

**Step 3: Confirm Subscription**
Similar to SES, the Email owner must agree to receive messages from SNS.

1. The Subscription status will now be **Pending confirmation**. Open your Gmail inbox, find the email titled *"AWS Notification - Subscription Confirmation"*. Click the **Confirm subscription** link in the email.
> ![Confirm Subscription](/aws-image/setupSNS/sns-7.png)
2. A web page will open stating **Subscription confirmed!**
> ![Subscription confirmed](/aws-image/setupSNS/sns-8.png)
3. Return to the SNS screen, reload the page, and the status will change to **Confirmed**.

Now, the `smart-campus-notifications` notification channel is completely ready. Any event sent to this Topic will instantly turn into an Email sent straight to HR.
