---
title : "Configure Amazon SES"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1. </b> "
---

#### 5.6.1. Configure Amazon SES (Email Verification)
Amazon SES (Simple Email Service) is AWS's email sending service. Because our account is in **Sandbox** mode (a testing environment to prevent spam), AWS requires us to verify ownership of any Email address before using it as a Sender or Receiver.

In this scenario, the system needs to send a successful attendance notification Email to the HR department. Therefore, we need to verify the recipient's Email.

**Step 1: Access Amazon SES**

1. Search for and access the **Amazon Simple Email Service** on the AWS Console search bar.
> ![Search SES](/aws-image/setupSES/ses-1.png)

**Step 2: Create Identity (Email Verification)**

1. On the left menu bar, under **Configuration**, select **Identities**. Then click the orange **Create identity** button in the top right corner.
> ![Create Identity](/aws-image/setupSES/ses-2.png)
2. In the **Identity type** section, select **Email address**. Enter your real Email address in the blank field. Scroll down to the bottom of the page and click **Create identity**.
> ![Enter Email info](/aws-image/setupSES/ses-3.png)

**Step 3: Confirm in Mailbox**

1. AWS will report a **Verification pending** status. Open your Gmail inbox, look for the email titled *"Amazon Web Services – Email Address Verification Request..."* and click the confirmation link.
> ![Email confirmation link](/aws-image/setupSES/ses-4.png)

2. After clicking, the browser will report successful confirmation. Returning to the SES Identity screen, you will see the status change to a green **Verified**.
> ![Verified Status](/aws-image/setupSES/ses-5.png)

That's it! We have a valid Email address for the Smart Campus system to use as a "notification receiving address" (or sending address). Next, we will configure **Amazon SNS** to set up a broadcast channel to push content to this Email.
