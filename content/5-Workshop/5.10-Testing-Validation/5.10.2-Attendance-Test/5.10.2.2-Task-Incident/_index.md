---
title : "Task Management & Incidents"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.10.2.2. </b> "
---

#### Test Task Management and Incident Reporting Flow

This test focuses on task assignment, incident reporting, and verifying the automatic lock and filtering of departments when assigning subtasks. We will also test the secure file upload feature via **S3 Pre-signed URLs**.

---

**Step 1: Assign Task and Create Subtask (From a Manager's Perspective)**

1. Log in to the system using an account with the **MANAGER** (or ADMIN) role.
2. Access the **Tasks** menu.
3. Click the **Giao việc mới (New Task)** button (Note: Roles like STAFF or TECHNICIAN will not see this button).
4. Fill in the required information:
   - Task Title
   - Task Type
   - Assignee
   - Priority
5. Click **Xác nhận (Confirm)**.

> ![Assign Task](/aws-image/testendtoend/test7.png)

6. After the task is created, click on it on the board, then click the **+ Thêm việc con (Add Subtask)** button.

> ![Task Kanban](/aws-image/testendtoend/test15.png)

7. Fill in the information for the Subtask and assign it to another employee.

> ![Create Subtask](/aws-image/testendtoend/test8.png)

8. (After the employee submits results) Log back in with the **MANAGER** account, open the details of the task which is in the "Chờ duyệt" (Pending Approval) status, and click the **Duyệt (Approve)** button to accept it.

> ![Update Progress](/aws-image/testendtoend/test16.png)

> **Expected Result:** Both the main task and the subtask are created successfully, clearly showing the parent-child relationship on the task board. The system pushes Toast Notifications directly to the assigned employees when assigned and when the task is approved.

---

**Step 2: Report an Incident (From an Employee's Perspective)**

1. Log in to the system using an account with the **STAFF** role.
2. Access the **Tasks** menu.
3. Click the **Báo cáo sự cố / Yêu cầu bảo trì (Report Incident)** button (Note: Staff does not have permission to create general tasks).
4. Fill in the required information:
   - Task Title
   - Incident Category
   - Department
   - Attachment (Upload a photo simulating the scene)
5. Click **Xác nhận (Confirm)**.

> ![Incident Reporting](/aws-image/testendtoend/test11.png)

> **Expected Result:** The incident is created successfully. Although the employee didn't select an Assignee, the system automatically routes this incident to the **TECHNICAL Manager** for handling. Thanks to the **S3 Pre-signed URL** mechanism, the attached file is securely and efficiently uploaded directly to the S3 Bucket by the Frontend, optimizing bandwidth for the Backend Lambda.

---

**Step 3: Assign Incident for Processing (From a Manager's Perspective)**

1. Log out of the STAFF account, log back in with a **MANAGER** account belonging to the `TECHNICAL` department.
2. Access the **Tasks** menu; you will see the incident just reported in the "Cần làm" (TODO) state.

> ![Incident on Board](/aws-image/testendtoend/test12.png)

3. Click the **Phân công (Assign)** button on the incident to open the task update form.
4. In the Assignee section, select a suitable technician and click **Xác nhận (Confirm)**.

> ![Assign Technician](/aws-image/testendtoend/test13.png)

> **Expected Result:** 
> - The "Department" field is **disabled (locked)** at `Bảo trì (Technical)` to ensure adherence to the workflow.
> - The Assignee dropdown list only shows employees belonging to the assigned department.

---

**Step 4: Submit Incident Results**

1. Log in with the assigned Technician account, update the incident status to **Đang làm (IN_PROGRESS)**.
2. When finished, click **Gửi duyệt (Submit for Approval)**, upload the result report file, and click **Gửi duyệt**.

> ![Task Completed](/aws-image/testendtoend/test14.png)

> **Expected Result:** The system records the new status. If the incident is overdue, the system will automatically display a red warning label.
