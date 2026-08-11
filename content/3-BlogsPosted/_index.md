---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---


This section will list and introduce the blogs you have posted to [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj). For example:

###  [Blog 1 - PAGINATION STRATEGY IN AMAZON DYNAMODB](3.1-Blog1/)
This blog covers the essential pagination strategy in Amazon DynamoDB using **LastEvaluatedKey** and **ExclusiveStartKey** to split query results into pages instead of loading all data at once. It demonstrates how pagination can reduce Read Capacity Units (RCU) costs by up to 10,000x, improve response time from minutes to hundreds of milliseconds, and ensure applications scale effectively with growing data.

**Facebook Link:** [https://www.facebook.com/share/p/1BxRgPHRBn/](https://www.facebook.com/share/p/1BxRgPHRBn/)

###  [Blog 2 - PREVENTING FACE RECOGNITION SPOOFING WITH AMAZON REKOGNITION FACE LIVENESS](3.2-Blog2/)
This blog addresses a critical security vulnerability in face recognition systems: spoofing attacks using photos, pre-recorded videos, or 3D masks. It explains how **Amazon Rekognition Face Liveness** solves this by verifying a live person is present before identity matching, using randomized color challenges and real-time video analysis. The post details the complete integration flow: CreateFaceLivenessSession → Frontend Amplify UI Liveness Detector → GetFaceLivenessSessionResults → Confidence-based decision → SearchFacesByImage.

**Facebook Link:** [https://www.facebook.com/groups/awsstudygroupfcj/permalink/2240573343374292/](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2240573343374292/)

###  [Blog 3 - IMPROVING MONTHLY CLOUD COST VARIANCE ANALYSIS WITH A WEEKLY FINOPS CHECKPOINT](3.3-Blog3/)
This blog presents a proactive **Weekly FinOps Checkpoint** process to improve monthly cloud cost variance analysis. Instead of waiting until month-end (too late) or monitoring daily (too noisy), a weekly review cycle helps detect spending trends early, provides clearer context for variance reports, and fosters collaboration between Finance/FinOps and technical teams. It covers three implementation approaches: AWS FinOps Agent (AI-powered), manual Cost Explorer workflows, and structured engagement with account owners using WoW% and Annualized Impact metrics.

**Facebook Link:** [https://www.facebook.com/groups/awsstudygroupfcj/permalink/2239975663434060/](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2239975663434060/)