---
title : "Leave Management & Notifications"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.10.2.3. </b> "
---

#### Test Leave Application Flow and Notification System

This test focuses on the leave registration feature on the **Interactive Calendar**, verifying the system's ability to prevent overlapping dates/holidays, and experiencing the UX with **Toast Notifications**.

---

**Step 1: Configure Holidays (From an Admin's Perspective)**

1. Log in to the system using the **ADMIN** account.
2. Access the **Leaves** menu.
3. Switch to the **Ngày lễ (Holidays)** tab and click **Thêm ngày lễ (Add Holiday)**.
4. Fill in the required information:
   - Ngày (Date)
   - Tên ngày lễ (Holiday Name)
   - Mô tả (Description - optional)
5. Click **Thêm (Add)**.

> ![Add Holiday](/aws-image/testendtoend/test17.png)


> ![Holiday Calendar](/aws-image/testendtoend/test18.png)
> **Expected Result:** The holiday is saved into the system and displayed on the calendar with a distinct color (green).
---

**Step 2: Create a Leave Request (From an Employee's Perspective)**

1. Log out and log back in with a **STAFF** account.
2. Access the **Leaves** menu.
3. Click on any date on the Interactive Calendar. The registration form will automatically pre-fill the date you selected.
4. Fill in the required information:
   - Loại đăng ký (Leave Type: Annual Leave or WFH)
   - Ngày (Date)
   - Lý do (Reason - optional)
5. Click **Gửi đăng ký (Submit)**.

> ![Leave Registration](/aws-image/testendtoend/test19.png)

> **Expected Result:** The leave request is initialized in the **Chờ duyệt (PENDING)** state. The system sends a green Toast Notification indicating success.

---

**Step 3: Test overlapping schedule prevention (Negative Test)**

To check the strictness of the Backend system:
1. Try to create another leave request **overlapping the date** you just requested in Step 2.
2. Or try to create a leave request on the exact **Holiday** that the Admin configured in Step 1.

> ![Overlapping Error](/aws-image/testendtoend/test20.png)

> **Expected Result:** The Backend system automatically scans and blocks the request. The Frontend will receive an error and display a red Toast Notification warning that you cannot request leave on a holiday or already registered date.

---

**Step 4: Approve Request and Check Notifications (From a Manager's Perspective)**

1. Log out and log back in with a **MANAGER** or **ADMIN** account.
2. Access the **Leaves** menu, switch to the **Chờ duyệt (Pending Approval)** tab.
3. You will see the employee's leave request from Step 2. Click the **Duyệt (Approve)** button.

> ![Approve Leave](/aws-image/testendtoend/test21.png)


> **System-wide Notification Test:** 
> The Smart Campus system is designed to automatically generate notifications for various events (e.g., new task assignments, missed deadlines, task progress updates, and leave/WFH request changes).
> By accessing the **Thông báo (Notifications)** screen, you can verify this flow: the list will display all updates and activities related to your account.
> ![Notification Center](/aws-image/testendtoend/test22.png)
> **Expected Result:** The request changes to the Approved state.