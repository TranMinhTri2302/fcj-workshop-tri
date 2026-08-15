---
title: "AWS FCAJ Agent Forge - Agent Core in Production (Deepdive day 2)"
date: 2026-08-08
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Event Summary Report: "AWS FCAJ Agent Forge – Agent Core in Production"

### Event Information

- **Event Name:** AWS FCAJ Agent Forge – Agent Core in Production (Continuation of Deepdive Series)
- **Date:** 08/08/2026 
- **Format:** Offline Workshop with Hands-on Lab, Floor 26, Bitexco Financial Tower
- **Role:** Attendee who reviewed and replicated the entire Hands-on Lab on personal AWS account

### Event Objectives

While the previous Deepdive session addressed the question **"What is Agentic AI and how is the foundational architecture designed?"**, this session tackles a much harder question: **"How do we move an AI Agent from a local demo to Production without losing control?"**

The entire content revolves around four operational pillars of **Amazon Bedrock Agent Core**:

1. **Memory** – What the Agent remembers, how it remembers, and for how long
2. **Observability** – How to see inside the "black box" when the Agent runs live
3. **Evaluation** – Measuring response quality with metrics instead of subjective judgment
4. **Policy & Security** – Locking down Agent permissions before it does something foolish

Additionally, the opening section covers unexpected but valuable content: **career development guidance for young Cloud/AI engineers**. The session concludes with a **nearly one-hour Hands-on Lab** deploying a complete Refund Assistant.

---

### Main Content

#### 1. Career Development Guidance for Cloud/AI Professionals

This section opens the session and deserves careful documentation, as it reflects wisdom from someone with years of hands-on experience rather than generic online advice.

**T-Shaped Skills Model**

The speaker emphasizes that a Cloud/AI engineer's career path shouldn't develop linearly, but rather in a **T-shape** — with both depth and breadth:

- **First Year (Vertical – Depth):** This phase demands going deep into a core specialty. Don't learn broadly. Choose one domain (e.g., serverless infrastructure, data pipelines, or model deployment) and master it thoroughly.
- **Years 2–3 (Horizontal – Breadth):** After establishing a foundation, begin expanding into real **production environments** — where everything differs completely from local setups. Simultaneously, build **domain knowledge** (business-specific expertise) in concrete fields like **EdTech, HealthTech, FinTech**.

One statement particularly resonated: **AWS certifications are necessary but not sufficient conditions**. Certifications help you pass the resume screening, but what keeps you in the industry is the ability to solve real problems for real companies.

**Soft Skills and AI Ethics**

This part made me reconsider my time investment strategy. The speaker emphasizes that **communication skills are a survival skill** — specifically, the ability to explain complex technical concepts to **non-technical people** (customers, managers, business teams) in ways they understand.

I immediately related this to reality: I often understand a problem clearly in my head, but when explaining to non-technical colleagues, I get stuck. In business environments, if you can't convince decision-makers, even brilliant solutions don't get approved.

Two other emphasized factors:

- **Responsibility in AI Usage (Responsible AI):** Develop mindfulness about whether your AI system is safe, unbiased, and won't harm users.
- **Ownership Mentality:** Don't just complete a task and move on; treat the systems you build as your own — responsible from code-writing through production deployment and even when it breaks at 2 AM.

**Real-World Practice Opportunities**

The most concrete advice: participate in **Hackathons** organized by companies and startups. The reasoning is very practical — Hackathons force you to transform "book knowledge" into working products within strict time limits. This process cannot be replaced by online learning.

> **Personal Insight:** I used to think accumulating certifications was enough. After this section, I realized certificates only prove I *know* something; actual products and projects prove I can *do* it. My upcoming plan is to choose one specific domain to deepen rather than learn broadly across multiple areas.

---

#### 2. Core Architecture of Agent Core

**The Boundary Between Chatbot and AI Agent**

This section redefines something I thought I already understood. The speaker makes clear distinctions:

| Criterion | Regular Chatbot | AI Agent |
|---|---|---|
| Output | Text only | Can execute real actions |
| Capability | Question-answering | Self-directed planning and execution |
| Tools | None | Can invoke external tools |
| State | Usually stateless | Has short-term and long-term memory |

Simply put: **Chatbots tell you what to do. Agents go do it.**

**The Perception Loop: Thinking – Reasoning – Tooling**

The speaker explains the loop an Agent executes upon receiving a request:

1. **Reasoning (Analysis):** The Agent analyzes context — what does the user really want? What information exists, and what's missing?
2. **Thinking (Planning):** The Agent maps out steps to resolve the request. What comes first, what follows, and what data is needed at each step?
3. **Tool Use (Execution):** The Agent automatically invokes external tools — querying databases, calling APIs, searching the internet — to gather data or execute actions.

This loop repeats until the task completes. The critical point is **the Agent decides which Tools to call and when**, not the programmer hard-coding each step.

> **Personal Insight:** What changed my thinking: when building Agents, my job isn't writing "if A then do B" logic anymore; it's **describing available Tools clearly and letting the Agent choose**. This represents a significant shift from imperative to declarative programming paradigms.

---

#### 3. Memory – Managing Agent Memory

This section provided the most valuable architectural insights of the entire session, explaining concretely how an Agent "remembers" — something I previously only vaguely understood as "stuffing chat history into prompts."

**Short-term Memory (Conversation State)**

Short-term Memory manages **immediate state within a session** to maintain coherent conversation flow.

The provided example is easy to grasp — a running shoe purchase scenario:

- User: "I want to buy running shoes"
- Agent asks: Which brand? → **Nike**
- Agent continues: What color? → **Black**
- Agent continues: What size? → **42**

All these messages are stored as **raw text** (unprocessed) and handled **synchronously** — the Agent must read the entire context before responding. Synchronous processing provides instant response speed, but **trades off against volume** (context window limits and token costs).

**Long-term Memory (Persistent Knowledge)**

My favorite aspect here: **developers don't need to build this from scratch**. Bedrock Agent Core provides this functionality out-of-the-box.

The mechanism works like this:

- A module called **Memory Extraction** runs **asynchronously in the background** parallel to the main conversation.
- It automatically reads the short-term chat flow and **extracts important information** (key insights/knowledge) — for example: "this customer prefers Nike," "shoe size is 42," "prefers dark colors."
- These condensed insights flow into long-term storage.

The beauty of **asynchronous processing** is that extraction **doesn't slow down user responses**. Users still get immediate replies while "remembering" happens in the background.

**Memory Strategy – The Trade-off Problem**

The system provides **multiple strategies** for converting short-term to long-term data. Each strategy has different trade-offs:

- Detailed storage strategy → Agent remembers better but **costs more in storage and tokens**
- Concise summary strategy → Cheaper and faster but **may lose important details**

The speaker emphasizes: **no strategy is universally correct**. Engineers must evaluate specific problems to **trade off** between contextual richness and operational cost.

> **Personal Insight:** I previously thought Memory meant just dumping entire chat history into prompts. Now I understand that's both expensive and inefficient. The two-tier architecture (short-term synchronous – long-term asynchronous extraction) is a pattern worth adopting immediately in personal projects.

---

#### 4. Observability – System Monitoring

**Solving the "Black Box" Problem**

The speaker opens with a crucial point: **in Production environments, we cannot deploy AI blind**. Unlike standard APIs with clear input-output, AI Agents make independent Tool decisions, skip certain Tools, and reason in unpredictable directions. Without visibility inside, we cannot diagnose failures.

The solution comprises two mechanisms:

- **Logging:** Record all interaction content — what the user asked, what the Agent replied, which Tools were called with which parameters.
- **Tracing:** Track **the complete lifecycle of a request** — from user submission through each reasoning step, each Tool invocation, to final response. This enables answering "why did the Agent respond like that?"

**Metrics & Alerting**

Beyond logging, the system monitors operational indicators:

- **Resource consumption:** GPU / CPU / memory
- **Latency:** How long Agent responses take
- **Traffic:** Request volume over time

From these metrics, engineers establish **Alerting** — automatic notifications when anomalies occur. More importantly, connect to **auto-scaling mechanisms** so the system automatically expands when traffic spikes, rather than users experiencing timeouts.

> **Personal Insight:** This section made me realize something: **building an Agent is only 30% of the work; 70% is operating it**. An Agent running well on my machine but lacking logs and traces becomes a debugging nightmare in production.

---

#### 5. Evaluation – Assessing Agent Quality

This section answers: **how do we know if our Agent answers well or poorly?**

**Measurement Using Metrics Rather Than Intuition**

The approach is straightforward: compare directly between two things:

- **Predicted Response:** The answer generated by AI
- **Ground Truth:** The reference answer (pre-written by humans, considered correct)

This comparison produces **quantitative metrics** rather than subjective assessment like "seems okay to me." The speaker emphasizes that without a Ground Truth dataset, there's no way to know whether today's prompt change improved or degraded the system.

**Human-in-the-Loop and SME Role**

The most critical point here: **evaluation cannot be 100% automated**.

Participation from **SMEs (Subject Matter Experts)** — domain specialists — is mandatory to validate:

- Are responses **factually correct within the domain**?
- Do responses make **sense in real context**?
- Do they comply with industry regulations?

Example: a medical advisory Agent might produce fluent responses matching Ground Truth linguistically, but only doctors know if the advice is safe for patients.

> **Personal Insight:** I previously evaluated AI quality purely subjectively — run a few test cases, if answers seemed reasonable, call it done. Now I understand the need for **a Ground Truth test suite built from day one**, like unit tests for code. Without it, every prompt modification is gambling blind.

---

#### 6. Policy & Security – Authorization and Access Control

**Cedar Language – Expressing Authorization Policies**

The system uses a specialized language to establish **Authorization** — defining what Agents can access in the system. Rather than scattering permission logic throughout code, all policies are declared centrally in auditable form.

**Strict vs. Permissive Mode**

This part proved exceptionally practical:

- **Permissive Mode:** Used during **development**. Agents operate with minimal restrictions, enabling rapid experimentation without configuring permissions for each operation.
- **Strict Mode:** **Mandatory in Production.** All permissions are tightly locked; Agents only perform explicitly authorized actions.

This is critical because it enforces the **Least Privilege principle**. Forgetting to enable Strict in production could mean:

- Agents accidentally call destructive APIs (deleting data, mass-canceling orders)
- Agents access sensitive data they have no business reading → **data leaks**
- Exploitation via prompt injection to perform unintended actions

> **Personal Insight:** This is an error I could easily make — developing comfortably in permissive mode, then forgetting to tighten permissions before deployment. I'll add "verify Strict Mode enabled" to my mandatory pre-release checklist.

---

#### 7. Expanding Agent Capabilities with Specialized Tools

This section introduces three built-in tools extending Agent capabilities beyond a pure language model.

**Browser Tool**

Grants Agents **internet access** for **real-time data** retrieval. This solves an inherent LLM limitation: knowledge frozen at training time. With Browser Tool, Agents can check current prices, latest news, inventory status — continuously changing information.

**Code Interpreter**

A **sandbox (isolated safe environment)** where Agents can:

- Write code themselves
- Execute it for complex calculations
- Generate graphs and process data
- Return results (numerical or visual) to users

Running in sandbox is critical: if an Agent generates buggy or dangerous code, it only affects the isolated environment, not the main system.

**Payment Integration**

Direct **payment gateway API calls**, transforming Agents from consultants into **automated sales assistants** — completing orders and processing transactions within a single conversation.

> **Personal Insight:** Seeing these three tools together clarified the Chatbot-Agent boundary. With Payment Integration, AI no longer just "talks" but **touches real money** — explaining exactly why Policy & Security above matters so much.

---

#### 8. Hands-on Lab: Deploying Refund Assistant

This longest practical section, lasting nearly an hour, builds a complete order refund processing Agent.

**Step 1 – Project Setup**

Tools used:

- **Agent CLI:** Command-line tool for Agent project scaffolding
- **Node.js:** Runtime environment
- **AWS CDK (Cloud Development Kit):** Infrastructure as Code

After initializing source code, CDK generates templates and deploys **Serverless infrastructure** via **CloudFormation**. Critical cost point: **infrastructure bills only when invoked** (when actually called). No traffic = no charges — huge advantage for proof-of-concept phases and variable-traffic workloads.

**Step 2 – Integrating Refund Tool**

Building an Agent handling **Refund requests**. Difference from regular chatbot:

- Chatbot: "Please contact support for refund assistance."
- Agent: **Automatically calls functions** to check order status, verify refund eligibility, then responds with real data.

**Step 3 – CLI Invocation & Mock Data**

Running commands directly from **Terminal** simulates Client-to-API calls. Most interesting: the system displays **Agent's Thinking and Reasoning in real-time on terminal**. You watch its thought process — "user wants refund → need order ID → call lookup_order tool → receive data → check eligibility → respond."

Regarding data, this lab uses **Mock Data embedded directly in System Prompt** instead of connecting actual databases like **DynamoDB**. Practical reasoning: **lab goal is quickly testing Agent logic**, not building complete systems. Separating Agent logic from data layers enables much faster iteration early on.

**Step 4 – Log and Trace Streaming**

A small but valuable architectural detail:

| Type | Storage | Purpose |
|---|---|---|
| **Logs** | Local disk | Development debugging |
| **Traces** | Cloud Observability Service | Production monitoring |

Separating both streams prevents cloud storage bloat from detailed dev-only logs while maintaining end-to-end production visibility.

> **Personal Insight:** This lab taught me the stark difference between "watching someone else demo" and "running commands yourself." Particularly the moment I saw reasoning appear on the terminal — previously I viewed Agents as black boxes taking input and returning output; now I saw step-by-step thinking. Using Mock Data is also a neat trick I'll apply: **don't connect real databases until Agent logic stabilizes**.

---

### Key Takeaways

#### System Operations Mindset

The biggest lesson from this session: **the gap between Agent demo and production Agent is far larger than expected**.

An Agent running locally needs just Model + Prompt + some Tools. A production Agent additionally requires: proper Memory strategy, comprehensive Observability systems, Evaluation with Ground Truth, Strict-mode Security policies, and auto-scaling mechanisms. That's four to five technical layers completely unrelated to "writing good prompts."

#### Technical Knowledge

- **Two-tier Memory:** Short-term handles **synchronously** for instant responses, Long-term uses **asynchronous Memory Extraction** to avoid slowing user experience. This architectural pattern deserves wider adoption.
- **Memory Strategy is a trade-off problem** — no universal answer. Must balance contextual richness against token/storage costs per use case.
- **Logging and Tracing are different things:** Logs show *what happened*, Traces show *in what sequence and how long each step took*.
- **Predicted Response vs Ground Truth** foundation for quantitative evaluation — but still **requires SME review**, cannot be fully automated.
- **Cedar + Strict Mode** is the final protective mechanism ensuring Least Privilege for production Agents.
- **Sandbox for Code Interpreter is mandatory** — never let AI-generated code execute in environments with real system access.

#### FinOps Considerations

- **Serverless infrastructure bills only on invocation** — ideal for PoC phases and variable-traffic workloads.
- **Memory Strategy directly impacts your bill:** more context in prompts = more tokens per call. Optimizing Memory optimizes cost.
- **Mock Data during development** saves both money and time versus setting up real databases immediately.

#### Career Development

- Develop following the **T-model**: first year goes deep in specialty, years 2–3 expand to production and domain knowledge.
- **Certifications are necessary, products are sufficient.**
- **Non-technical communication skills** matter as much as technical skills.
- **Hackathons** provide the fastest path from theory to hands-on experience.

---

### Practical Application

After this session, here's what I can implement immediately:

- **Design two-tier Memory for personal projects:** Instead of dumping entire chat history into prompts, separate into short-term (keep recent turns) and long-term (extract core user insights, running asynchronously).
- **Build Ground Truth dataset first:** Prepare 20–30 question-answer pairs establishing standards before writing prompts, enabling measurable improvement tracking instead of guessing.
- **Enable Tracing from day one:** Configure end-to-end traces from project initialization. Keep logs local, push traces to cloud.
- **Add "Strict Mode check" to release checklist:** Before every production deployment, audit all policies ensuring no Permissive mode leaks from development.
- **Use Mock Data for fast iteration:** When starting new Agents, embed fake data in System Prompt to stabilize logic first, then replace with real DynamoDB connections.
- **Apply Serverless for low-traffic AI workloads:** Internal tools used tens of times daily cost far less on serverless versus 24/7 instance running.
- **Choose one domain for specialization:** Rather than learning broadly, select one specific business domain to develop domain knowledge alongside technical skills.

---

### Personal Experience

This session runs nearly 2 hours 40 minutes; I rewatched several sections three times before full comprehension, especially Memory Strategy and Strict/Permissive transitions. But this is "appropriately difficult" content — each rewatch reveals something new.

Most impressive is **session structure**: starts with career guidance (soft topics), gradually moves to architecture, then four operational pillars, ending with hands-on lab. This progression prevents overwhelm — each theory component reappears in the lab with concrete meaning.

**Evaluation** surprised me most. Previously I skipped this almost entirely — run a few tests, reasonable answers, call it done. After learning about Predicted Response vs Ground Truth and mandatory SME review, I recognized my old approach **lacked quality controls entirely**. Deploying would mean every prompt tweak gambles blind.

The **Hands-on Lab** truly drives understanding home. Small details like separating local Logs from cloud Traces seem obvious until you actually run and see outputs in two different places. The moment reasoning appeared on the black terminal screen was special — transforming abstract concepts into observable reality.

Although the career guidance segment at the beginning of the session was brief, it left me with the most food for thought afterward. It forced me to re-evaluate how I’m investing my time: am I learning broadly, or am I diving deep? Am I genuinely building domain knowledge, or just hoarding certificates?

After Day 2 of the event, which involved both the group project and the hands-on practice, I realized that Kiro's 50-token allowance ran out pretty quickly, meaning I couldn't continue building alongside the team.

The organizing team was incredibly thoughtful. Understanding that many attendees at the Day 2 Deepdive had missed Day 1, they made sure the hands-on session walked us through setting up the environment, configuring Kiro, and prompting exactly as they had done on the first day. They were also very easygoing and proactive in creating a fun, welcoming vibe. One of the organizers even joked about wanting to win the Canon photography contest so he could treat everyone to Haidilao.

---

### Lessons Learned

- **Building an Agent is 30%; operating it is 70%.** This determines whether Agents survive production or remain impressive demos.
- **Asynchronous processing is the key to good UX.** Everything not immediately needed by users (like Memory Extraction) belongs in background.
- **You cannot improve what you don't measure.** Without Ground Truth, prompt refinement is just subjective guessing.
- **Humans remain irreplaceable in evaluation.** SMEs are mandatory gates, especially in high-domain-complexity fields.
- **Permissive is for development, Strict is for living.** Forgetting to tighten permissions before production is among the costliest mistakes.
- **Memory Strategy is a financial question, not just technical.** Every stored token costs money; Kiro’s 50 tokens disappeared fast when not used optimally.
- **Serverless is a friend to experimentation** — no traffic means no cost; enables comfortable trial and error.
- **Careers need architecture like systems:** deepen first, broaden after, never forget domain knowledge and communication.

---

### Participation Proof (Images)

![Event 3-1](/images/4-EventParticipated/EV3-1.png)
![Event 3-2](/images/4-EventParticipated/EV3-2.png)
![Event 3-3](/images/4-EventParticipated/EV3-3.png)
![Event 3-4](/images/4-EventParticipated/EV3-4.png)
![Event 3-5](/images/4-EventParticipated/EV3-5.png)

> Overall, this session filled the missing puzzle piece from the previous Deepdive: if that session explained **what Agents are made of**, this one showed **how to make them survive in the real world**. Memory determines how intelligent Agents become; Observability determines whether you maintain control; Evaluation determines if you know quality or just assume it; Security determines whether Agents cause harm. Missing any single piece, and Agents remain impressive demos rather than actual products.