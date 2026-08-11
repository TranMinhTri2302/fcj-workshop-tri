---
title: "Week 6 Worklog"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

# Week 6: Analytics Dashboard & RBAC – Advanced Biometrics – Well-Architected Review

## 1. Weekly objectives

* Redesign the dashboard to visualize data more clearly, make Analytics the main page, and organize the interface by role.
* Upgrade the Cognito authentication flow, add personal face registration, enable face-based login, and support password recovery using biometrics.
* Add the Login screen, protect routes, and increase consistency between user roles and the displayed data.
* Review the system against the AWS Well-Architected Framework and rebalance the design under AWS credit constraints.

## 2. Detailed work log

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Design a **dual data flow**: a daily reporting update flow through EventBridge and a direct query flow through Athena <br> - Apply the 6 pillars of the **AWS Well-Architected Framework** to review the architecture from the perspectives of operations, security, reliability, performance, cost, and sustainability <br> - Reassign team roles: FE, BE, CI/CD, data processing, and monitoring; update the layered architecture diagram at the same time <br> - Prepare a response plan for AWS credit limits as the project moved into the finalization stage | 27/07/2026 | 27/07/2026 | <https://docs.aws.amazon.com/wellarchitected/> |
| 3 | - **Frontend:** Redesign the KPI Summary Cards on the Analytics page; build a Circular Progress Ring (SVG) and Donut Chart (SVG) to show work/attendance ratios <br> - **Backend:** Complete WF1 – Cognito administration: use `admin_create_user` in the Admin endpoint to create accounts so Cognito generates a Temporary Password and sends it by email to new staff | 28/07/2026 | 28/07/2026 | <https://docs.aws.amazon.com/cognito/> |
| 4 | - **Frontend:** Build a gradient Area Chart for attendance trends over time, a list of employees with the highest absence rates using progress bars with color thresholds, and update the data dynamically from the Analytics/Reports endpoint <br> - **Backend:** Complete the biometric flow: the My Profile page allows employees to register their face via Webcam + Upload; prevent duplicates using SearchFacesByImage before IndexFaces | 29/07/2026 | 29/07/2026 | <https://docs.aws.amazon.com/rekognition/> |
| 5 | - **Frontend:** Build the Login screen with Cognito authentication, handle the temporary password change flow, and redirect users after sign-in <br> - Create a Private Route / HOC mechanism to block unauthenticated users from internal pages <br> - **Backend:** Complete face-based login that returns a JWT token; build a face-based password recovery flow that receives an email and a base64 image, verifies it via Rekognition, and resets the password on Cognito; handle cases where a `face_id` is duplicated | 30/07/2026 | 30/07/2026 | <https://docs.aws.amazon.com/cognito/> <br> <https://docs.aws.amazon.com/rekognition/> |
| 6 | - **Frontend:** Fully integrate 3-layer RBAC in the frontend: ADMIN/DIRECTOR can view the entire system, MANAGER is restricted by department, STAFF is redirected to “My Analytics” <br> - Complete the Analytics.jsx page as the home page, optimize the layout and responsiveness <br> - Fix the memory leak caused by the camera when switching tabs with `stopFaceCamera()` <br> - Review the new interface end to end to ensure consistency between JWT roles, route protection, and displayed data | 31/07/2026 | 01/08/2026 | — |

## 3. Personal contributions

* Served as the main contributor to the frontend this week:
  * turned Analytics.jsx into the home page;
  * built KPI cards;
  * implemented custom SVG charts like Donut Chart, Progress Ring, Area Chart, and Progress Bar without external libraries;
  * completed the Login screen and Private Route;
  * implemented 3-level RBAC in the UI;
  * handled the camera lifecycle issue when switching tabs.
* Updated the way data was displayed by role so the UI reflected the appropriate access rights of each user group.
* Reviewed the authentication and biometric flow from the end-user perspective to ensure that sign-in, temporary password changes, and password recovery worked together properly.
* Noted the trade-offs that needed to be balanced between cost, performance, and service scope after reviewing the system under the Well-Architected lens.

## 4. Achievements

* Frontend:
  * The dashboard was redesigned into a more visual Analytics page with role-based layering and no dependency on third-party chart libraries.
  * Login, Private Route, and frontend RBAC worked in sync with JWT.

* Backend and authentication:
  * Completed the Cognito flow for account administration more fully.
  * The personal face registration, face-based login, and biometric password recovery flows were integrated into the system.

* Optimization and review:
  * Resolved the camera lifecycle issue when switching tabs.
  * Completed the architecture review using the Well-Architected Framework and clarified the trade-offs related to cost and operability.
