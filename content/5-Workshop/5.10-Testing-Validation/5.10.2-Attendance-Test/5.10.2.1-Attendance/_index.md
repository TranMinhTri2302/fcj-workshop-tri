---
title : "Attendance & Biometrics"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.10.2.1. </b> "
---

#### Test Facial Recognition Attendance and Password Reset Flow

This is the core business flow of the system: Registering a face to Rekognition, sending an image to recognize and record attendance into DynamoDB, and using AI for password recovery. Instead of using Postman, we will experience it directly on the application's Frontend interface.

---

**Step 1: Face ID Password Reset**

In the Smart Campus system, if you forget your password, instead of using Email OTP, the system uses Amazon Rekognition to authenticate your face.

1. Open your Frontend website 
2. On the Login screen, click the **Quên mật khẩu? (Forgot Password?)** button.
3. A "Khôi phục bằng Face ID (Reset with Face ID)" modal will appear. Enter your account's Email and click **Tiếp tục (Continue)**.

> ![Enter Email step](/aws-image/testendtoend/test1.png)

4. The system will open the Camera. Align your face and click **Chụp và Xác thực (Capture & Verify)**.

> ![Face ID Verification](/aws-image/testendtoend/test2.png)

5. The system calls the `/api/auth/verify-face-reset` API to the Backend.

> **Expected Result:** If the face matches the registered data, the system will allow you to enter a New Password and automatically log you in. If it's the wrong person, the system returns an access denied error.

> ![Password update success](/aws-image/testendtoend/test3.png)

---

**Step 2: First-time face registration**

If you create a brand new account and have never registered a face, the application will prompt you to register before marking attendance.

1. Log in with the new account. Switch to the **Attendance** page.
2. On the Attendance page, you will see the message **Your account does not have a face yet**.
3. Click the **Turn on Camera** button and allow the browser to access the Webcam.
4. Sit straight, ensure your face is clear in the frame, and click the **Capture & Register Face** button.

> ![Face registration interface](/aws-image/setupTestcheckin/checkin2.jpg)
5. Wait for the system to call the `POST /api/faces/register` API.

> **Expected Result:** Receive the message "Face registration successful!". Your account status has been updated.

---

**Step 3: Perform Check-in / Check-out (Attendance)**

After registering a face, the Attendance interface will switch to the daily facial scanning mode.

1. Click the **Turn on Camera** button (if the camera is off).
2. Click the **Capture Image (Check in)** button.
3. The system will call the `POST /api/attendance/recognize` API to match against the original image in Rekognition.

> **Expected Result:** The screen displays the name and Confidence of the face with a success message. If you mark attendance multiple times in the same session, the system will show a warning **"Attendance was previously recorded (skipping duplicate)"**.
> ![On-time Check-in result](/aws-image/testendtoend/test4.png)

---

**Step 4: Test WFH Attendance (Remote Work)**

1. Log in with a Manager account, and approve an employee to Work from Home (WFH) today.
2. Log out, then log back in with the approved employee's account.
3. Access the **Nghỉ phép (Leaves)** menu.
4. At the top of the Leaves page, you will see a blue banner: *"You are WFH today — please click WFH Check-in when you start working"*.

> ![WFH Check-in button on Leaves page](/aws-image/testendtoend/test5.png)

5. Click the **Điểm danh WFH (WFH Check-in)** button (which has a Home icon).

> ![WFH Check-in success notification](/aws-image/testendtoend/test6.png)

> **Expected Result:** The employee does not need to turn on the Camera to scan their face but can still check in successfully. The system automatically calls the `/attendance/wfh-checkin` API and records the WFH status into the Database.

---

**Step 5: Test recognition failure (Negative Test)**

To ensure the system handles correctly, try:
1. Have someone else (not you) sit in front of the Camera and click Check in.
2. Or use an object (phone, water cup) to cover your face or completely cover the camera so there is no human face, and click Check in.

> **Expected Result:** The system reports an error **"Cannot recognize - No face detected in image"**.
> ![Cannot recognize error](/aws-image/setupTestcheckin/checkin7.png)

---

**Step 6: Test off-network attendance (WAF IP Whitelisting)**

To verify the IP blocking feature using AWS WAF configured in section 5.5.4, try:
1. Disconnect your computer's current Wi-Fi and use a Mobile Hotspot (4G/5G) to change your IP address.
2. Alternatively, use a VPN to create a virtual IP outside the internal network.
3. Click the **Check in** button on the interface.

> **Expected Result:** The request is blocked immediately at the WAF layer before reaching the API Gateway. The system returns an access denied error (e.g., 403 Forbidden) because the IP is not in the allowed list (IP Whitelist).
> ![Off-network attendance error](/aws-image/setupTestcheckin/checkin8.png)
