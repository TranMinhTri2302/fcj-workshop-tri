---
title: "Week 8 Worklog"
date: 2026-08-10
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

# Week 8: Face Liveness – Workshop Documentation – System Review and Project Wrap-up

## 1. Weekly Objectives

* Finalize the UI and testing for end-stage flows, especially Face Liveness and attendance-related features.
* Recheck monitoring results, alerts, and evaluation-related components to ensure that the UI reflects the real system flow correctly.
* Complete the workshop documentation, event reflection reports, technical materials, and the final project summary.
* Prepare handover materials and presentation slides for demonstration and evaluation.

## 2. Detailed Work Log

| Day | Tasks | Start Date | End Date | References |
| --- | --- | --- | --- | --- |
| 2 | - Reviewed the system flows after observability components were added, comparing Service Map, traces, and alert points with the actual interface behavior <br> - Documented the items that needed to be included in the workshop documentation and final technical summary, especially for Monitoring/Tracing and attendance-related processing flows <br> - Rechecked error responses and abnormal states that could affect user experience | 10/08/2026 | 10/08/2026 | <https://docs.aws.amazon.com/xray/> |
| 3 | - Rechecked alerting flows and error cases that could affect user experience, especially when the system returned errors or timed out <br> - Reviewed UI notes and descriptions related to the event-processing pipeline to ensure the documentation matched the running system <br> - Helped cross-check several flow changes so that the technical summary would remain consistent with the actual implementation | 11/08/2026 | 11/08/2026 | <https://docs.aws.amazon.com/AmazonCloudWatch/> <br> <https://docs.aws.amazon.com/sns/> |
| 4 | - Integrated the Face Liveness interface using the `@aws-amplify/ui-react-liveness` SDK, gradually replacing the static frame capture flow in `Attendance.jsx` <br> - Rechecked display states, user guidance for face alignment, and the transition flow after receiving Face Liveness results <br> - Cross-checked the `confidence` threshold and returned data so that the UI handled pass/fail cases correctly | 12/08/2026 | 12/08/2026 | <https://docs.aws.amazon.com/rekognition/latest/dg/face-liveness.html> <br> <https://ui.docs.amplify.aws/react/connected-components/liveness> |
| 5 | - Performed system end-to-end testing for the entire application, rechecking the main user flows in the interface before handover <br> - Finalized the **Workshop Documentation** according to the FCJ Workshop Template, adding screenshots and technical explanations for each section <br> - Wrote/edited content for: Workshop Overview, Prerequisite, Auth & Security, Database & Storage, Data Analytics, Monitoring & Tracing, Testing & Validation | 13/08/2026 | 13/08/2026 | FCJ Workshop Template |
| 6 | - Completed the workshop documentation and consolidated technical materials such as architecture, APIs, database schema, and the system’s core flows <br> - Updated the reflection reports for the three attended events and cross-checked them against the workshop/final report content <br> - Performed a final review of the overall system architecture, wrote the project conclusion and future directions, and prepared handover documents and presentation slides | 14/08/2026 | 15/08/2026 | — |

## 3. Personal Contributions

* Took charge of the Face Liveness-related UI, end-to-end testing, and final user experience review before handover.
* Significantly updated and completed the workshop documentation, especially in the following sections in 5-Workshop:
  * Workshop Overview
  * Prerequisite
  * Auth & Security
  * Database & Storage
  * Data Analytics
  * Monitoring & Tracing
  * Testing & Validation
* Followed the observability integration at the practical level and compared trace/alert results against the real system flow to ensure that the technical descriptions matched the product.
* Added the final technical summary, event reflection reports, and end-of-project handover materials.

## 4. Outcomes Achieved

* Finalized the Face Liveness interface and rechecked all important user flows before final evaluation.
* End-to-end testing helped identify and correct remaining issues during the final stage.
* Completed the workshop documentation, technical materials, event reflection reports, project conclusion, and handover package.
* Updated Monitoring/Tracing and related technical sections so that they matched the real system more closely.
* Brought the Smart Campus Platform to a complete enough state for final demonstration and evaluation.