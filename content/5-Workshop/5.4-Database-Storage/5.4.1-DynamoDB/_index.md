---
title : "Create DynamoDB Table"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

#### 5.4.1. Design and Create Tables on Amazon DynamoDB

Amazon DynamoDB is chosen for its ultra-fast response speed (just a few milliseconds) and unlimited scalability (Serverless NoSQL), making it highly suitable for storing attendance data and user information.

For the Smart Campus system to operate perfectly as designed by the Backend, the actual architecture requires up to **9 data tables (Tables)** on DynamoDB. Below is a summary table of the respective Partition Key and Global Secondary Index (GSI) configurations:

| Table Name | Partition Key (String Type) | GSI (Global Secondary Index) | Usage Purpose |
| :--- | :--- | :--- | :--- |
| `smart-campus-attendance` | `record_id` | `date-index` (PK: `date`) | Store daily attendance history |
| `smart-campus-faces` | `face_id` | `userid-index` (PK: `user_id`) | Store facial recognition info |
| `smart-campus-users` | `user_id` | *(Optional depending on Backend)* | Store employee/student info |
| `smart-campus-holidays` | `date` | - | Manage public holidays |
| `smart-campus-leaves` | `request_id` | *(Optional depending on Backend)* | Manage leave requests |
| `smart-campus-notifications` | `notification_id` | *(Optional depending on Backend)* | Manage notification sending history |
| `smart-campus-settings` | `setting_key` | - | Manage general configurations |
| `smart-campus-tasks` | `task_id` | *(Optional depending on Backend)* | Manage automated tasks |

*(Note: GSIs not detailed above can be configured further depending on the actual Backend system's query requirements).*

**Below are the steps on the AWS Console. We will use the `smart-campus-attendance` table as an illustrative example; you just need to repeat the same operations for the remaining tables based on the parameters in the summary table above:**

**Step 1: Access DynamoDB**
1. Search for **DynamoDB** in the search bar of the AWS Console.
> ![Search DynamoDB](/aws-image/setupDB/setupdyamodb1.png)
2. Click the **Create table** button to start creating a table.
> ![Create Table](/aws-image/setupDB/setupdyamodb2.png)

**Step 2: Configure Table and Primary Key (Partition Key)**
3. In the *Table details* section:
   - **Table name:** Enter the table name (e.g., `smart-campus-attendance`).
   - **Partition key:** Enter the corresponding Partition Key name (e.g., `record_id`, Type: *String*).
   - **Table settings:** Select `Customize settings`.
> ![Attendance Table](/aws-image/setupDB/setupdyamodb8.png)

**Step 3: Configure Secondary Key (GSI) - If any**
4. Expand the *Secondary indexes* section, and click **Create global index** (for the `smart-campus-attendance` table, we need to create a GSI to query by date).
> ![GSI](/aws-image/setupDB/setupdyamodb4.png)
5. Fill in the GSI information:
   - **Index name:** Enter the GSI name (e.g., `date-index`).
   - **Partition key:** Enter the Partition Key of the GSI (e.g., `date`, Type: *String*).
   - **Attribute projections:** Select `All`. Click **Create index**.
> ![Attendance GSI](/aws-image/setupDB/setupdyamodb9.png)

**Step 4: Finalize Table Creation**
6. Scroll down to the bottom of the page and click **Create table**.
> ![Create Table Finish](/aws-image/setupDB/setupdyamodb5.png)

Repeat the process from Step 1 to Step 4 for the other tables (such as `smart-campus-faces`, `smart-campus-users`...) by referring to the configuration in the summary table at the beginning of the lesson.

