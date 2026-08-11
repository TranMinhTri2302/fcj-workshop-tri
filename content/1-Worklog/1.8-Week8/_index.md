---
title: "Week 8 Worklog"
date: 2026-08-10
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

# Week 8: Observability & Message Queue – Face Liveness – Workshop Documentation – Final Project Wrap-up

## 1. Weekly objectives

* Add an observability layer to the system using AWS X-Ray, CloudWatch Alarm, and SNS Alerting.
* Increase the reliability of the event pipeline through Amazon SQS.
* Research and build a PoC for fraud-resistant attendance using Amazon Rekognition Face Liveness.
* Complete the workshop documentation, internship report, technical documentation, and final project wrap-up.

## 2. Detailed work log

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Implement the **Observability** layer with AWS X-Ray: add `aws-xray-sdk` to `requirements.txt`, call `patch_all()` in `main.py` to trace boto3 calls to DynamoDB, Rekognition, and S3 <br> - Resolve Segment conflicts on Lambda by checking `AWS_LAMBDA_FUNCTION_NAME` at startup; if running on Lambda, skip attaching `XRayMiddleware` and keep only `patch_all()` for sub-segment tracing <br> - Review the Service Map and Trace Timeline in the X-Ray Console and note that `Rekognition:SearchFacesByImage` had the highest latency | 10/08/2026 | 10/08/2026 | <https://docs.aws.amazon.com/xray/> |
| 3 | - Set up **CloudWatch Alarm + SNS Alerting**: monitor the `Errors` metric of the Lambda `smart-campus-api` and send warning emails to Admin when the system encounters issues <br> - Separate system errors (5xx) from user errors (4xx) to avoid false alarms <br> - Verify the alerting mechanism by creating a controlled error <br> - Add **Amazon SQS** between EventBridge and Lambda as a buffer layer for the event pipeline | 11/08/2026 | 11/08/2026 | <https://docs.aws.amazon.com/AmazonCloudWatch/> <br> <https://docs.aws.amazon.com/sns/> <br> <https://docs.aws.amazon.com/sqs/> |
| 4 | - Research and implement a PoC for **Amazon Rekognition Face Liveness**: the backend uses `CreateFaceLivenessSession` → `GetFaceLivenessSessionResults`, blocks the process if `confidence < 90%`, and only continues to `SearchFacesByImage` using the `ReferenceImage` from the session when the threshold is met <br> - **Frontend:** Integrate a Face Liveness UI using the `@aws-amplify/ui-react-liveness` SDK and gradually replace the static frame-capture flow in Attendance.jsx | 12/08/2026 | 12/08/2026 | <https://docs.aws.amazon.com/rekognition/latest/dg/face-liveness.html> <br> <https://ui.docs.amplify.aws/react/connected-components/liveness> |
| 5 | - Perform End-to-End System Testing for the entire system, review the main flows on the UI and backend before handover <br> - Complete the **Workshop Documentation** according to the FCJ Workshop Template, add illustrations and technical descriptions for each section <br> - Write or refine content for the sections: Workshop Overview, Prerequisite, Auth & Security, Database & Storage, Data Analytics, Monitoring & Tracing, and Testing & Validation | 13/08/2026 | 13/08/2026 | FCJ Workshop Template |
| 6 | - Finalize the workshop documentation, consolidate technical documents such as architecture, API, database schema, and the main system flows <br> - Update the internship report for the 3 events attended and align the content with the workshop/report summary <br> - Review the system architecture one last time, write the final project conclusion, outline future development directions, and prepare the handover documentation and presentation slides | 14/08/2026 | 15/08/2026 | — |

## 3. Personal contributions

* Contributed to the frontend around Face Liveness, end-to-end testing, and the overall user experience review before handover.
* Significantly updated and completed the workshop documentation, especially for the sections:
  * Workshop Overview
  * Prerequisite
  * Auth & Security
  * Database & Storage
  * Data Analytics
  * Monitoring & Tracing
  * Testing & Validation
* Followed the observability work at the integration level and verified the trace and alert outcomes against the real system flows.
* Added technical synthesis, event reports, and final project handover documentation.

## 4. Achievements

* Observability and monitoring:
  * Completed the basic observability layer with X-Ray, CloudWatch Alarm, and SNS Alerting.
  * Identified the main bottleneck at `Rekognition:SearchFacesByImage`.
  * Resolved the Lambda segment conflict by adjusting how tracing was attached for the runtime environment.

* Architecture reliability:
  * Added SQS as a buffering layer for the event pipeline, reducing the risk of data loss during sudden traffic spikes.

* Attendance fraud prevention:
  * Completed a viable Face Liveness PoC that could be extended further in future system development.

* Documentation and delivery:
  * Completed the workshop documentation, technical documentation, event reports, final project summary, and handover documentation.
  * The Smart Campus Platform reached a level of completeness suitable for demonstration and final evaluation.
