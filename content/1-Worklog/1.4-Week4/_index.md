---
title: "Week 4 Worklog"
date: 2026-07-13
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Week 4: Employee Management & Attendance UI – Webcam Integration – Face Recognition Flow Alignment

## 1. Weekly Objectives

* Build the Employee Management screen and the Attendance page with webcam support in the browser.
* Complete the face registration and attendance flows on the frontend in an end-to-end manner.
* Recheck payloads, naming conventions, and business states between the UI and the face recognition/attendance APIs.
* Support reviews of data-related changes, business rules, and the Task module during the baseline stage.

## 2. Detailed Work Log

| Day | Tasks | Start Date | End Date | References |
| --- | --- | --- | --- | --- |
| 2 | - Held a group meeting to finalize the weekly baseline target, prioritizing workflows that could be clearly demonstrated <br> - Built the Employee Management page (`Users Page`), called the employee list API, and displayed the data using the DataTable prepared in Week 3 <br> - Reviewed the face registration flow from the UI side to ensure that input fields matched the image-processing pipeline | 13/07/2026 | 13/07/2026 | <https://docs.aws.amazon.com/rekognition/> |
| 3 | - Developed the add/edit employee form with input validation <br> - Built the “Face Registration” modal in `Users.jsx`, supporting both file upload and direct webcam capture via `navigator.mediaDevices` <br> - Rechecked the image payload structure and response fields such as `face_id`, `confidence`, and `BoundingBox` to ensure the UI consumed them correctly | 14/07/2026 | 14/07/2026 | <https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia> |
| 4 | - Studied the WebRTC API (`getUserMedia`) and built a custom `useCamera` hook to manage camera permission, video stream, and lifecycle <br> - Implemented webcam image capture logic by drawing frames to `<canvas>`, converting them to Base64, and sending them to the API <br> - Re-aligned the attendance status display for PRESENT / LATE / REJECTED so that the UI accurately reflected the business rules | 15/07/2026 | 15/07/2026 | <https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API> |
| 5 | - Designed `Attendance.jsx`: webcam recognition view, employee name + confidence display, scanning effect, attendance history table with date/shift filters, and color-coded status badges <br> - Added UX states such as camera loading, permission denial, unavailable camera, and recognition success/failure <br> - Helped review how task-related data and business states should later be represented on the UI | 16/07/2026 | 16/07/2026 | <https://react.dev/> |
| 6 | - Performed end-to-end testing of the face registration and attendance flows from the UI to the API, identifying issues related to naming, data types, and CORS during integration <br> - Adjusted the UI according to changes involving `faceId`/`face_id`, attendance states, and error messages <br> - Coordinated a review of edge cases related to image uploads, recognition responses, attendance rules, and attendance history display | 17/07/2026 | 18/07/2026 | <https://fastapi.tiangolo.com/> |

## 3. Personal Contributions

* Took primary responsibility for the frontend work of the week:
  * `Users Page` displaying employee data;
  * add/edit employee form;
  * face registration modal with Upload + Webcam;
  * `useCamera` custom hook;
  * webcam-to-canvas capture flow;
  * `Attendance` page with webcam recognition, scanning effect, filters, and status badges.
* Reviewed the face registration flow from the client side to ensure field names, payloads, and response parsing were consistent.
* Clarified how attendance business states should be displayed, especially for PRESENT, LATE, REJECTED, and duplicate attendance cases.
* Helped review data-related changes and naming conventions to avoid mismatches between the UI, APIs, and persistence layer.

## 4. Outcomes Achieved

* Completed the employee management UI and webcam-based attendance interface.
* Stabilized the WebRTC hook and the image capture flow, directly supporting WF2 and WF3.
* Connected the face registration and attendance flows end-to-end from the UI to the APIs.
* Identified and resolved major integration issues such as inconsistent naming, recognition response handling, and lost CORS headers when the API threw exceptions.