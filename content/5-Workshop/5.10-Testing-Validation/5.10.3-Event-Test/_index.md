---
title : "Event Testing"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.10.3. </b> "
---

#### 5.10.3. Test Event Notification Flow (EventBridge → SNS → Email & SQS)

After a successful attendance in step 5.10.2, Lambda will automatically emit an `AttendanceRecorded` event to **EventBridge**. This section confirms that the event has been correctly routed to the Web interface, SNS (send Email), and SQS (save to queue).

> ![Turn on attendance camera](/aws-image/setupTestnoti/noti1.png)
> ![Attendance successful](/aws-image/setupTestnoti/noti2.png)

---

**Step 1: Check notifications on the Web interface (Frontend)**

Our application has a Notifications module that fetches data directly from the Backend.

1. On the Web interface, switch to the **Notifications** menu.
2. Here, you will see a new record with the event type `AttendanceRecorded` (Attendance successful).
3. Click **Details** to see the notification content information.

> **Expected Result:** The Web interface immediately displays the attendance event that just occurred with full information.

---

**Step 2: Validate Email notifications (SNS → Email)**

1. Open the Gmail inbox of the Email address you registered with the SNS Topic `smart-campus-notifications` in step 5.6.2.
2. About 30 seconds to 1 minute after marking attendance, you will receive a new Email from **AWS Notifications** (no-reply@sns.amazonaws.com).
3. The Email content will contain the attendance event info as JSON, including `userId`, `attendanceId`, `status`, `timestamp`.
> ![Notification Email](/aws-image/setupTestnoti/noti3.png)

> **Expected Result:** Receive an automated Email within 1 minute after calling the attendance API.

---

**Step 3: Validate SQS queue received the event**

1. Search for and access AWS Console > **SQS**.
> ![Search SQS](/aws-image/setupTestnoti/noti4.png)
2. Select the queue (e.g., `smart-campus-notification-queue`).
> ![Select queue](/aws-image/setupTestnoti/noti5.png)
3. Click the **Send and receive messages** button (top right corner).
> ![Send and receive](/aws-image/setupTestnoti/noti6.png)
4. Scroll down to the **Receive messages** section, click the **Poll for messages** button.
> ![Poll message](/aws-image/setupTestnoti/noti7.png)
5. Within a few seconds, the system will display the messages currently in the queue.
> ![Has message](/aws-image/setupTestnoti/noti8.png)
6. Click on a message to view the content — you will see the body is the EventBridge payload with `detail-type: AttendanceRecorded`.

> **Expected Result:** The message from the attendance event appears in SQS after polling.

---

**Step 4: Validate EventBridge Rule matched (Monitoring)**

1. Search for and access AWS Console > **Amazon EventBridge**.
> ![Search EventBridge](/aws-image/setupTestnoti/noti9.png)
2. Select **Event buses**.
> ![Select Event buses](/aws-image/setupTestnoti/noti10.png)
3. Select the `smart-campus-events` bus and switch to the **Monitoring** tab, choose the **Last 1 hour** time range.
4. Check the **Matched events** chart — there should be at least 1 event marked as matched with the rule.
> ![Monitoring Chart](/aws-image/setupTestnoti/noti11.png)

> **Expected Result:** The Matched events chart increases, with no Failed invocations.
