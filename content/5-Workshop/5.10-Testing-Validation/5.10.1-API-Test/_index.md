---
title : "API Testing"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.10.1. </b> "
---

#### 5.10.1. API Testing via Swagger UI and Postman

The first testing step is to confirm that the API Gateway has connected correctly to Lambda and can receive/process requests. We will use **Swagger UI** (the auto-generated interface from FastAPI) and **Postman**.

---

**Step 1: Check Swagger UI**

1. Get the **Invoke URL** of the API Gateway (saved in step 5.5.3), for example:
   `https://abc123.execute-api.ap-southeast-1.amazonaws.com/`
2. Open a browser, paste the URL and append `/docs` at the end:
   `https://abc123.execute-api.ap-southeast-1.amazonaws.com/docs`
3. If the FastAPI **Swagger UI** interface appears with a full list of API Endpoints, the API Gateway has successfully connected to Lambda.

> **Expected Result:** The Swagger UI page displays all API groups: `/api/auth`, `/api/users`, `/api/attendance`, `/api/faces`...
> ![Swagger UI Interface](/aws-image/setupTestapi/testapi1.png)

---

**Step 2: Create a new user (POST /api/users)**

1. Open **Postman** (or use Swagger UI directly), create a new Request with:
   - **Method:** `POST`
   - **URL:** `https://<invoke-url>/api/users`
   - **Headers:** `Content-Type: application/json`
   - **Body (raw JSON):**
   ```json
   {
     "email": "hohane8316@mrworlds.com",
     "name": "Nguyen Van Test",
     "role": "STAFF",
     "department": "IT",
     "phone": "0901234567",
     "employee_id": "EMP-12345"
   }
   ```
   > ![Create user body](/aws-image/setupTestapi/testapi2.png)
2. Click **Send** (or **Execute** on Swagger).
3. (Note: A temporary password will be automatically sent by AWS Cognito to the email you provided above).

> **Expected Result:** HTTP `201 Created` with the response body containing the newly created user info and `user_id`.
> ![Create user successful](/aws-image/setupTestapi/testapi3.png)

---

**Step 3: Login and get Access Token (POST /api/auth/login)**

1. Check your Email to get the temporary password sent by Cognito.
   > ![Temporary password email](/aws-image/setupTestapi/testapi4.png)
2. Create a new Request:
   - **Method:** `POST`
   - **URL:** `https://<invoke-url>/api/auth/login`
   - **Body (raw JSON):**
   ```json
   {
     "email": "test.user@example.com",
     "password": "<Temporary_password_in_email>"
   }
   ```
   > ![Login body](/aws-image/setupTestapi/testapi5.png)
3. Click **Send** (or **Execute**).
4. (If the system requires a password change for the first time, you can call the `POST /api/auth/respond-challenge` API with `new_password`).
5. After a successful login, copy the `access_token` value from the response body. This token will be used in subsequent steps.

> **Expected Result:** HTTP `200 OK` with the response containing `access_token` (JWT token).
> ![Login successful](/aws-image/setupTestapi/testapi6.png)

---

**Step 4: Call authenticated API (GET /api/users)**

1. Create a new Request:
   - **Method:** `GET`
   - **URL:** `https://<invoke-url>/api/users`
   - **Headers:** Add `Authorization: Bearer <access_token_just_retrieved>`
   > ![Call user list API](/aws-image/setupTestapi/testapi7.png)
2. Click **Send** (or **Execute**).

> **Expected Result:** HTTP `200 OK` returning a list of users in the system (including the user you just created).
> ![User list result](/aws-image/setupTestapi/testapi8.png)
