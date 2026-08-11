---
title: "Week 4 Worklog"
date: 2026-07-13
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Week 4: HR & Attendance UI – Integrating Rekognition – Tasks Module – Handling Integration Errors

## 1. Weekly objectives

* Build the Employee Management and Attendance interfaces with a webcam in the browser.
* Integrate Amazon Rekognition and S3 to complete the face registration (WF2) and attendance (WF3) flows end to end.
* Expand the Tasks Management module with DynamoDB access patterns and clear business rules.
* Strengthen automated testing and resolve integration issues that arose between the frontend, Rekognition, DynamoDB, and backend.

## 2. Detailed work log

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Hold a team meeting to finalize the week’s baseline goals, prioritizing the workflows that could be demonstrated clearly <br> - **Frontend:** Build the Employee Management page (Users Page), call the API to retrieve the employee list, and display it using the DataTable prepared in Week 3 <br> - **Backend:** Set up the S3 bucket `smart-campus-images` with Block Public Access, configure CORS to allow direct uploads from the frontend, create a Rekognition Collection, and build the `rekognition.py` wrapper for `IndexFaces` and `SearchFacesByImage` | 13/07/2026 | 13/07/2026 | <https://docs.aws.amazon.com/rekognition/> <br> <https://docs.aws.amazon.com/AmazonS3/latest/userguide/> |
| 3 | - **Frontend:** Develop a modal form for adding or editing employees with input validation <br> - Build the “Face Registration” modal in Users.jsx, supporting both Upload file and Webcam capture via `navigator.mediaDevices` <br> - **Backend:** Complete WF2 – Face Registration end to end: receive base64 image data, decode and validate JPEG/PNG (up to 5MB), save the original image to S3, and call IndexFaces to generate `faceId`, confidence, and `BoundingBox` | 14/07/2026 | 14/07/2026 | <https://docs.aws.amazon.com/rekognition/latest/dg/API_IndexFaces.html> |
| 4 | - **Frontend:** Study the WebRTC API (`getUserMedia`), write a custom `useCamera` hook to manage camera permissions, video stream, and lifecycle <br> - Build the logic to capture images from the webcam, draw them onto a `<canvas>`, convert them to Base64, and send them to the backend <br> - **Backend:** Complete WF3 – Attendance: build a Rule Engine with three shifts (MORNING 07:00–12:00, AFTERNOON 13:00–17:30, EVENING 17:30–21:00), and automatically classify attendance as PRESENT, LATE, or REJECTED | 15/07/2026 | 15/07/2026 | <https://docs.aws.amazon.com/rekognition/latest/dg/API_SearchFacesByImage.html> |
| 5 | - **Frontend:** Design the Attendance.jsx page: webcam recognition, display of employee name and confidence %, scanning effect, history table with date/shift filters, and colored badges by status <br> - Add UX states for camera loading, permission errors, unavailable camera, and successful or failed recognition <br> - **Backend:** Design the Tasks table in DynamoDB with partition key `task_id` and create the GSI `assignee_id-status-index`; implement cursor-based pagination using `ExclusiveStartKey` / `LastEvaluatedKey` | 16/07/2026 | 16/07/2026 | <https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html> |
| 6 | - **Backend:** Add business rules for Tasks: auto-assign incidents by department, control update permissions by role, and automatically send notifications at status milestones <br> - Strengthen automated testing by writing unit and integration tests for WF2, WF3, and Tasks; use moto to simulate Rekognition, DynamoDB, and SES; update the CI/CD pipeline <br> - Resolve technical issues: parse `BoundingBox` to a String before writing to DynamoDB because boto3 does not directly support Rekognition’s Float type; sync `faceId` → `face_id`; add a global exception handler to preserve CORS when the backend throws errors | 17/07/2026 | 18/07/2026 | <https://github.com/getmoto/moto> |

## 3. Personal contributions

* Served as the main contributor to the frontend this week:
  * Users Page for displaying the employee list;
  * add/edit employee forms;
  * face registration modal with Upload + Webcam;
  * custom `useCamera` hook;
  * webcam capture logic through canvas;
  * Attendance page with webcam recognition, scanning effect, filtering, and status badges.
* Reviewed the face registration flow from the client to the backend to ensure consistent field names and payload structure.
* Clarified the attendance business rules further, especially for PRESENT, LATE, REJECTED, and duplicate check-in cases.
* Supported the Tasks module from the data query perspective, naming conventions, and future UI display needs.
* Documented and resolved integration issues related to Rekognition float values, `face_id` naming, and CORS loss when exceptions were thrown.

## 4. Achievements

* Frontend:
  * Completed the employee management interface and webcam-based attendance interface.
  * The WebRTC hook and image-capture flow worked stably and directly supported WF2 and WF3.

* Backend:
  * WF2 (Face Registration) and WF3 (Attendance) worked end to end.
  * The Tasks module had a suitable data structure, GSI, and pagination mechanism.

* Testing and integration:
  * Added tests for the new workflows and updated the CI/CD pipeline.
  * Resolved important integration issues such as Rekognition data types, inconsistent naming, and the loss of CORS when the backend threw exceptions.
