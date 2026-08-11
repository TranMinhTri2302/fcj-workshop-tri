---
title: "Week 6 Worklog"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

# Week 6: Analytics Dashboard & RBAC – Advanced Biometrics – Well-Architected Review

## 1. Weekly Objectives

* Redesign the Dashboard into a more effective Analytics homepage with role-based data visibility.
* Complete authentication-related screens such as Login, Private Route, and role-dependent UI behaviors.
* Add UI support for advanced biometric flows such as personal face registration and related verification steps.
* Reassess the system architecture using the AWS Well-Architected Framework and update any points affecting display logic and frontend integration.

## 2. Detailed Work Log

| Day | Tasks | Start Date | End Date | References |
| --- | --- | --- | --- | --- |
| 2 | - Reviewed the system’s dual data flow: scheduled reporting updates and direct user-requested query flow <br> - Updated the layered architecture diagram, role matrix, and the points affecting data visibility in the UI <br> - Took notes on cost, performance, and service-scope trade-offs after the team’s AWS Well-Architected review | 27/07/2026 | 27/07/2026 | <https://docs.aws.amazon.com/wellarchitected/> |
| 3 | - Redesigned KPI Summary Cards for the Analytics homepage <br> - Built a Circular Progress Ring (SVG) and a Donut Chart (pure SVG) to display attendance/task ratios without relying on external chart libraries <br> - Rechecked the data groups and several responses related to account management, roles, and the dashboard | 28/07/2026 | 28/07/2026 | <https://react.dev/> |
| 4 | - Built a gradient Area Chart for attendance trends over time and a Top Absent Employees list with color-threshold progress bars <br> - Connected dynamic data from Analytics/Reports endpoints <br> - Added the `My Profile` interface allowing users to register their own face using Webcam + Upload, and reviewed the end-to-end states of this flow | 29/07/2026 | 29/07/2026 | <https://docs.aws.amazon.com/rekognition/> |
| 5 | - Built the Login screen integrated with Cognito, including temporary password reset flow and post-login redirection <br> - Implemented Private Route/HOC to block unauthenticated users from internal pages <br> - Reviewed the user experience for authentication, face registration, face login, and account recovery from the UI perspective | 30/07/2026 | 30/07/2026 | <https://docs.aws.amazon.com/cognito/> |
| 6 | - Implemented three-layer RBAC entirely on the frontend: ADMIN/DIRECTOR can view the entire system, MANAGER is limited to their department, and STAFF is switched to “My Analytics” <br> - Finalized `Analytics.jsx` as the homepage and improved layout responsiveness <br> - Fixed the camera memory leak issue when switching tabs using `stopFaceCamera()` <br> - Helped recheck role fields, tokens, and data rendering to ensure consistency between JWT, route protection, and the system’s authorization rules | 31/07/2026 | 01/08/2026 | — |

## 3. Personal Contributions

* Took primary responsibility for the UI work of the week:
  * turned `Analytics.jsx` into the homepage;
  * built KPI cards;
  * hand-coded SVG charts such as Donut Chart, Progress Ring, Area Chart, and Progress Bars;
  * completed Login and Private Route;
  * implemented three-layer RBAC on the interface;
  * fixed the camera issue when switching tabs.
* Updated data visibility based on role so that the UI accurately reflected each user group’s access scope.
* Reviewed the authentication and biometric flows from an end-user perspective to ensure that login, temporary password reset, face registration, and related error states formed a consistent experience.
* Documented the cost, performance, and service-scope trade-offs identified during the Well-Architected review.

## 4. Outcomes Achieved

* Redesigned the dashboard into a more visual Analytics homepage with role-based display and no dependency on external chart libraries.
* Achieved consistent integration of Login, Private Route, and frontend RBAC with JWT-based authorization.
* Connected the UI for personal face registration, face login, and related authentication steps into the system flow.
* Resolved the camera lifecycle issue when switching tabs and completed the architecture review from a Well-Architected perspective.