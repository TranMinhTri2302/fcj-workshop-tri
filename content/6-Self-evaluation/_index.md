---
title: "Self-Assessment"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

During my internship at **First Cloud AI Journey (FCAJ)** from **June 2026** to **August 2026**, I had the opportunity to learn, experiment, and apply academic foundations in a real-world engineering environment.

Throughout the program, I participated in building the **Smart Campus Platform**—a serverless attendance and workflow management system on AWS integrated with AI-powered face recognition. Through this project, I significantly sharpened my skills across **cloud architecture, serverless development, DevOps, AI/ML integration, data engineering, and event-driven systems**.

In terms of attitude, I consistently aimed to deliver on tasks, respected workplace norms, and proactively communicated with my mentor and teammates when stuck. At the same time, I made a habit of researching issues independently before asking questions to build real self-sufficiency.

To reflect objectively on the internship, I assess myself against the following criteria:

| No. | Criteria | Description | Good | Fair | Average |
| --- | --- | --- | --- | --- | --- |
| 1 | **Domain Knowledge & Technical Skills** | Understanding & applying AWS, serverless, backend, frontend, and tooling to production-style projects | ✅ | ☐ | ☐ |
| 2 | **Learning Agility** | Quick pickup of new concepts, especially AWS services and cloud architecture patterns | ✅ | ☐ | ☐ |
| 3 | **Proactiveness** | Independent research, self-directed experimentation, and ownership without micro-guidance | ✅ | ☐ | ☐ |
| 4 | **Sense of Responsibility** | Meeting deadlines, tracking progress, and maintaining output quality | ✅ | ☐ | ☐ |
| 5 | **Discipline** | Respecting schedules, rules, workflows, and project security | ✅ | ☐ | ☐ |
| 6 | **Growth Mindset** | Welcoming feedback, fixing mistakes, and iterating through reviews | ✅ | ☐ | ☐ |
| 7 | **Communication** | Articulating ideas, reporting progress, and discussing technical tradeoffs | ☐ | ✅ | ☐ |
| 8 | **Teamwork** | Cooperating smoothly with mentors, team admins, and peers | ✅ | ☐ | ☐ |
| 9 | **Professional Conduct** | Treating colleagues with respect and maintaining proper composure | ✅ | ☐ | ☐ |
| 10 | **Problem-Solving Mindset** | Diagnosing root causes, proposing solutions, and verifying outcomes | ✅ | ☐ | ☐ |
| 11 | **Project / Organization Contribution** | Tangible contributions to workflows, dashboards, pipelines, and docs | ✅ | ☐ | ☐ |
| 12 | **Overall Evaluation** | Comprehensive verdict on the entire internship | ✅ | ☐ | ☐ |

---

## Detailed Self-Assessment by Technical Domain

### 1. Cloud Architecture & Serverless Development  
**Rating: ⭐⭐⭐⭐⭐**

I participated in designing and implementing a **Serverless Event-Driven** architecture on AWS. The stack leveraged services including Lambda, API Gateway, DynamoDB, EventBridge, SQS, SNS, S3, Rekognition, Cognito, Athena, Glue, CloudFront, WAF, CloudWatch, X-Ray, CodePipeline, and CodeBuild.

**What I accomplished:**
- Built serverless backends using **API Gateway + Lambda**.
- Deployed Python APIs on Lambda using **FastAPI + Mangum**.
- Designed asynchronous pipelines using **EventBridge, SQS, and DLQ**.
- Combined **DynamoDB for transactional data** with **Athena/S3 for analytics**.
- Applied **IAM Least Privilege** principles when wiring service permissions.

**What I learned:**
- How to architect serverless systems for scalable elasticity.
- How event-driven boundaries decouple services.
- How retries, dead-letter queues, and asynchronous error handling interact in reality.
- The chasm between "demo working" and "operable in production".

**Challenges overcome:**
- Local **Windows** vs. **AWS Lambda Linux** wheel/dependency discrepancies.
- Debugging subtle **CORS, event loop, and Mangum** quirks on Python 3.12.
- Tightening IAM policies until they were lean without breaking execution.

---

### 2. AI/ML Integration  
**Rating: ⭐⭐⭐⭐⭐**

A core pillar of the attendance system was integrating AI. I worked with **Amazon Rekognition** for face identification and liveliness verification.

**What I accomplished:**
- Integrated **Face Recognition** via Rekognition (`SearchFacesByImage` against collections).
- Implemented **Face Liveness Detection** to mitigate spoofing via printed photos, replay videos, or masks.
- Channeled secure image uploads via **S3 pre-signed URLs**.
- Persisted and modeled recognition outputs to power attendance logic.

**What I learned:**
- Tuning confidence thresholds for real-world accuracy vs. false rejections.
- Normalizing Rekognition payloads before writing to DynamoDB.
- Practical failure modes of AI in the wild: false positives, spoofing vectors, edge-case lighting.

**Challenges overcome:**
- Boto3 Float serialization quirks when writing to DynamoDB.
- Base64 image payload validation.
- Ensuring raw face images never leak publicly.

---

### 3. Data Engineering & Analytics  
**Rating: ⭐⭐⭐⭐**

I contributed to building the analytics pipeline—helping bridge the gap between transactional day-to-day records and reporting views.

**What I accomplished:**
- Built the flow: **Kinesis Firehose → S3 Data Lake → Glue Crawler → Athena**.
- Partitioned S3 objects by `year/month/day` for faster query pruning.
- Connected analytical queries to a React + Recharts dashboard.
- Supported role-filtered visibility (Director, Manager, Staff).

**What I learned:**
- How a pragmatic Data Lake looks on AWS.
- How Glue Crawlers infer schemas to materialize Athena tables.
- Designing dashboards around operational questions rather than raw tables.
- Hybrid querying: DynamoDB for hot writes/reads (<14d), Athena for historical depth.

**Challenges overcome:**
- Glue schema discovery hiccups across schema evolution.
- Partition projection and Athena workgroup configuration.
- Field naming synchronization (`camelCase` vs. `snake_case`) across layers.

---

### 4. DevOps & CI/CD  
**Rating: ⭐⭐⭐⭐**

I gained hands-on experience with automated build and deployment for both backend and frontend—moving away from "it works on my machine".

**What I accomplished:**
- Configured automated pipelines via **CodePipeline + CodeBuild**.
- Built Python 3.12 packages targeting Linux Lambda runtimes.
- Built Vite-powered frontends, synced to S3, and invalidated CloudFront caches.
- Authored and tuned `buildspec.yml` stages.

**What I learned:**
- Pipeline anatomy on AWS.
- Cross-platform wheel building (`manylinux2014_x86_64`).
- Scoping CodeBuild service roles properly.
- Fighting SPA caching issues over CloudFront.

**Challenges overcome:**
- Packaging Python C-extensions so Lambda didn't throw import errors.
- Fixing S3/CloudFront 404s on deep client-side routes.
- Reading pipeline logs to diagnose post-deploy regressions.

---

### 5. Security & Observability  
**Rating: ⭐⭐⭐⭐**

Security and observability taught me that a working system is unfinished until it is accountable and diagnosable.

**What I accomplished:**
- Applied **Defense in Depth** across boundaries.
- Restricted perimeter access with **AWS WAF**.
- Authenticated with **Cognito JWTs**.
- Enforced code-level **RBAC**.
- Monitored behavior with **CloudWatch** metrics, logs, and alarms.
- Traced execution across hops with **X-Ray**.

**What I learned:**
- Managing WAFv2 IP sets.
- JWT verification mechanics at API entry points.
- Structured JSON logging conventions.
- Catching async failures via DLQ alarms.

**Challenges overcome:**
- Camera APIs needing HTTPS via CloudFront.
- Scoping WAF rules to CloudFront distributions.
- Striking a line between paranoia and usability.

---

## 6. Frontend Development  
**Rating: ⭐⭐⭐⭐**

Beyond backend and cloud plumbing, I built user interfaces—connecting state to screens.

**What I accomplished:**
- Built UI with **React + Vite**.
- Styled with Glassmorphism aesthetics + Dark Mode support.
- Visualized metrics with **Recharts**.
- Created dashboards, Kanban boards, notifications, and dropzone uploaders.
- Dynamically rendered controls based on RBAC claims.

**What I learned:**
- Project layout patterns in Vite.
- Local CORS bypassing via Vite proxy.
- Wiring UI state to S3 pre-signed upload flows.
- Responsive layout strategy across device classes.

---

## Completed Deliverables

| Deliverable | Description | Tech Stack | Status |
|:---|:---|:---|:---|
| **Smart Campus Platform** | Serverless full-stack attendance & task system | AWS, FastAPI, React, Rekognition, EventBridge, Athena | ✅ Complete |
| **WF1–WF8 Workflows** | Core business process workflows | AWS Serverless, EventBridge, SQS, DynamoDB, Bedrock | ✅ 7/8 Done |
| **Analytics Dashboard** | Role-based reporting + departmental comparison | Athena, Glue, Recharts, DynamoDB | ✅ Complete |
| **CI/CD Pipeline** | Automated build & deploy for backend/frontend | CodePipeline, CodeBuild, S3, CloudFront | ✅ Complete |

---

## Areas for Improvement

Even with positive outcomes, I see clear edges to blunt:

1. **Technical Communication**  
   I need to simplify how I explain architecture. Sometimes I still dump too many mechanics on non-specialists instead of leading with business/system outcomes.

2. **Technical Writing Discipline**  
   My docs (ADRs, runbooks, handovers) need tighter prose and more predictable headings. Less wall-of-text, more scannability.

3. **Shift-Left Testing**  
   I tended to wire things until they breathed, *then* tested. I need more proactive unit/integration tests (`pytest`, `moto`) earlier in the loop.

4. **Effort Estimation on Multi-Service Tasks**  
   Wiring 5 AWS services always has hidden tax (IAM + propagation + timing). I need wider safety margins when estimating integration stories.

---

## Most Valuable Learnings

1. **End-to-End Cloud Ownership**  
   Sketch → IaC → Ship → Alert → Refactor. Seeing the full loop changes how you write the first line of code.

2. **Production Serverless Realism**  
   Demos use happy paths; production uses retries, idempotency keys, DLQs, cold starts, and cost guardrails.

3. **AI is an Integration Component, Not Magic**  
   Rekognition is just another dependency with latency, quotas, formatting traps, and confidence cliffs.

4. **Event-Driven Empathy**  
   Decoupling via EventBridge buys freedom today but requires tracing sanity tomorrow. Eventual consistency forces async UI thinking.

5. **Mentorship by Socratic Friction**  
   Good seniors don’t spoon-feed; they ask annoying questions until you invent the right pattern yourself.

---

## Self-Assessment Verdict

Overall, I evaluate my internship as **Good**. I delivered most scope, shipped real value into Smart Campus, and absorbed hard-won cloud instincts.

FCAJ taught me that engineering in the cloud isn't about collecting badges—it’s about writing code you aren't afraid to wake up to at 3 AM.

Moving forward, I feel much sharper targeting **Cloud Engineering, AI Engineering, and Solutions Architecture**. I know the gaps; now I know how to fill them.