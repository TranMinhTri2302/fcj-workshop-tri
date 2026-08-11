---
title : "Attendance Testing"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.10.2. </b> "
---

#### 5.10.2. Test Facial Recognition Attendance Flow via Web Interface

This is the core business flow of the system: Registering a face to Rekognition, then sending an image to recognize and record attendance into DynamoDB. Instead of using Postman, we will experience it directly on the application's Frontend interface.

---

**Step 1: Log in to the website**

1. Open your Frontend website (CloudFront URL or run locally via `npm run dev`).
2. Log in with the account just created in the previous section.
3. Switch to the **Attendance** menu.

---

**Step 2: First-time face registration**

Since this is the first time this account is using the system, the application will prompt you to register a face.
> ![Face registration interface](/aws-image/setupTestcheckin/checkin1.png)

1. On the Attendance page, you will see the message **Your account does not have a face yet**.
2. Click the **Turn on Camera** button and allow the browser to access the Webcam.
3. Sit straight, ensure your face is clear in the frame, and click the **Capture & Register Face** button.
4. Wait for the system to call the `POST /api/faces/register` API.

> **Expected Result:** Receive the message "Face registration successful!". Your account status has been updated to having a face.

---

**Step 3: Validate face registration data (S3)**

1. Go to AWS Console > **S3** > Bucket `smart-campus-images-{id}`.
2. Open the `faces/` folder. You will see the image file just taken with the Webcam saved.
> ![Image list on S3](/aws-image/setupTestcheckin/checkin5.png)
> ![S3 image details](/aws-image/setupTestcheckin/checkin3.png)

> **Expected Result:** The image file exists in S3.

---

**Step 4: Perform Check-in (Attendance)**

After registering a face, the Attendance interface will switch to Check-in mode.

1. Click the **Turn on Camera** button (if the camera is off).
2. Click the **Capture Image (Check in)** button.
3. The system will call the `POST /api/attendance/recognize` API to match against the original image in Rekognition.

> **Expected Result:** The screen displays the name and Confidence of the face. If you check in successfully for the first time, there will be a success message. If you mark attendance multiple times a day, the system will show a warning **"Attendance was previously recorded (skipping duplicate)"** as shown in the image below.
> ![Check-in result](/aws-image/setupTestcheckin/checkin4.png)

---

**Step 5: Validate records in DynamoDB**

1. Go to AWS Console > **DynamoDB** > Tables > `smart-campus-faces`.
2. Click the **Explore table items** button.
3. You will see a new record appear containing the face's metadata (including `faceId`, `confidence`, `s3Key`, `userId`).
> ![Validate DynamoDB](/aws-image/setupTestcheckin/checkin6.png)

> **Expected Result:** Facial recognition data was successfully saved into DynamoDB.

---

**Step 6: Test recognition failure (Negative Test)**

To ensure the system handles correctly, try:
1. Have someone else (not you) sit in front of the Camera and click Check in.
2. Or use an object (phone, water cup) to cover your face or completely cover the camera so there is no human face, and click Check in.

> **Expected Result:** The system reports an error **"Cannot recognize - No face detected in image"**.
> ![Cannot recognize error](/aws-image/setupTestcheckin/checkin7.png)

---

**Step 7: Test off-network attendance (WAF IP Whitelisting)**

To verify the IP blocking feature using AWS WAF configured in section 5.5.4, try:
1. Disconnect your computer's current Wi-Fi and use a Mobile Hotspot (4G/5G) to change your IP address.
2. Alternatively, use a VPN to create a virtual IP outside the internal network.
3. Click the **Check in** button on the interface.

> **Expected Result:** The request is blocked immediately at the WAF layer before reaching the API Gateway. The system returns an access denied error (e.g., 403 Forbidden) because the IP is not in the allowed list (IP Whitelist).
> ![Off-network attendance error](/aws-image/setupTestcheckin/checkin8.png)