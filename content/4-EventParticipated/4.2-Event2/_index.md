---
title: "AWS FCAJ Agent Forge - Deepdive"
date: 2026-08-03
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Summary Report: "AWS FCAJ Agent Forge - Deepdive"

### Event Information

- **Event name:** AWS FCAJ Agent Forge - Deepdive (Agent Force Event – L300 Advanced Level)
- **Time:** 9:00 AM, August 3rd, 2026
- **Location:** Offline Workshop with Hands-on Lab, Floor 26, Bitexco Financial Tower
- **Role:** Attendee – listening to in-depth theory sessions and participating directly in hands-on labs

### Event Objectives

AWS FCAJ Agent Forge is an L300-level advanced workshop designed specifically for engineers and enterprises who want to go deep into the world of **Agentic AI**. Unlike your typical trend-sharing sessions, this event had a very clear goal: to guide participants on how to properly design and deploy an AI Agent system — all the way from Proof of Concept to a production-ready state.

The entire content revolved around **Amazon Bedrock Agent Core** and the hardest problems enterprises face when bringing AI into real operations: processing performance, scalability as systems grow, user data security, and how to govern AI permissions without losing control.

### Speakers

- **Nghĩa** – Covered the in-depth theory: system architecture, Bedrock Agent Core, Runtime Environment, Identity & Security, and Gateway
- **Hải Anh** – Led the Hands-on Lab sessions, guiding participants through writing code and setting up a real Agent system

### Key Content

#### What is Agentic AI and Why It's Different

The opening session by Nghĩa gave me a much clearer picture of what **Agentic AI** actually means — it's not just a chatbot that answers questions. It's an autonomous software system capable of reasoning, planning, and executing tasks step by step without constant human intervention.

What I found interesting is that the level of autonomy in an Agent can span a very wide spectrum: from a fully deterministic workflow — where the AI only does exactly what it's been programmed to do — all the way to a fully autonomous multi-agent setup — where multiple Agents collaborate to solve complex problems without being told what to do at every step.

However, the speaker made one thing very clear: **you should never give an AI full autonomy in an enterprise environment**. That's just too risky. Instead, engineers must design a **Human-in-the-loop** mechanism — where humans always play the role of reviewer, approver, or controller over important decisions. I thought this was a very grounded point, because if AI is left to make every decision on its own with no one checking, the risk of errors is high and tracing the root cause becomes extremely difficult.

Another thing brought up was the communication protocols. Unlike the traditional REST API that most developers are used to, Agent systems use two newer standards:
- **MCP (Model Context Protocol):** for the Agent to communicate with Tools
- **A2A (Agent to Agent):** for Agents within a system to communicate with each other

I had never worked much with either of these before, so this was the first time I actually understood why they need to exist alongside REST API rather than just replacing it.

#### Amazon Bedrock Agent Core — A Proper Foundation for Building Agents

The next section went into detail on **Amazon Bedrock Agent Core**, the AWS service that lets engineers avoid building all Agent infrastructure from scratch. Nghĩa explained that to spin up a basic Agent, you only need three things: a **Model (the brain)**, a **System Prompt**, and a set of **Tools**. But if you try to build all of that yourself, meeting industry security and operational standards gets incredibly complex. Bedrock Agent Core exists to package and manage the entire Agent lifecycle in a standardized way.

One thing I found really practical was the section on **model selection strategy**. Within Anthropic's Claude family:
- **Claude Haiku** — best for queries that need fast response times and lower cost
- **Claude Sonnet** — the sweet spot between speed and intelligence, suitable for most use cases
- **Claude Opus** — built for complex technical tasks requiring deep reasoning, but noticeably slower and more expensive

This is a trade-off that engineers need to think through carefully. Choosing a "smarter" model means higher cost and longer response times. You don't always need the most powerful model — what matters is picking the right model for the right problem.

#### Runtime Environment — Infrastructure for Running Agents with Hardware-Level Security

This was honestly the part that made me go "wow" during the morning session. The **Runtime Environment** is the serverless infrastructure AWS provides to run Agents, operating on a pay-as-you-go model with flexible deployment options via Docker/ECR or ZIP files pushed to S3.

But the real standout was the core security technology underneath: **Firecracker MicroVM**. I had never heard of this before today, but after hearing it explained, I finally understood how big tech platforms can serve millions of users simultaneously while still guaranteeing that one person's data never leaks into another person's session.

The idea is simple but powerful: each user session runs inside its own **hardware-isolated MicroVM** with completely separate resources. User A and User B run in two entirely independent MicroVMs — there is no way for data or results from one session to bleed into the other. Before this, when I used OpenAI or Gemini APIs, I never stopped to ask "could my data be mixed with someone else's?" — but now I understand that underneath, this is a real engineering problem that needs a technology like Firecracker to solve.

The session also touched on a very practical UX concern: **if an Agent is processing something complex and forces the user to stare at a blank screen, the experience is terrible**. The solution is to design for asynchronous processing or long-running background jobs — the Agent works in the background and returns results as they become available, rather than making the user wait for the entire pipeline to finish before showing anything.

#### Identity & Security — It's Not Just IAM Roles

The security section was the most complex part of the day, but also the most important. Nghĩa explained that in an Agent system, security needs to be designed as its own dedicated layer called **Identity**, responsible for authentication and authorization in two directions:

- **Inbound:** Is the user allowed to access the Agent?
- **Outbound:** Is the Agent allowed to call out to specific Tools or external services?

The standout concept here was the **Token Vault** mechanism. The system is designed so that the AI never directly sees any API keys or sensitive credentials. Instead, a conversion flow happens:

1. The user logs in and receives a **JWT (JSON Web Token)**
2. That JWT is exchanged for a **WAT (Workload Access Token)** — a combined token representing both the user and the Agent
3. The WAT is then exchanged for the **OAuth Credential** of a third-party service to perform the actual action

This ensures the AI only ever gets the minimum permissions needed to do its job, and can never arbitrarily access services it's not supposed to. The speaker also mentioned **AWS Cognito** as a straightforward solution for issuing JWTs, saving engineers from having to design the entire auth flow from scratch.

My takeaway: authorization in an AI system is not as simple as assigning an IAM Role. It's an intentional chain of token transformations designed to keep secrets hidden from the AI and control its actions at every single step.

#### Gateway — The Middleware You Can't Skip as Systems Grow

The final theory section was on **Gateway**, and this was the part that made me understand why middleware is non-negotiable once an Agent system starts to scale.

Imagine an enterprise with hundreds of Agents and thousands of different tools. If every Agent had to connect directly point-to-point to every Tool, managing and securing all of that would be a nightmare. Gateway solves this by becoming a **central coordination point** for all data flows.

One feature I found genuinely clever was how Gateway leverages **MCP combined with Semantic Search** to help Agents automatically discover and select the right Tool to call, rather than having everything hard-coded. Essentially, each Tool is wrapped with a description (Schema), and when an Agent needs to do something, it searches semantically to find the right Tool — more like a Google search than looking something up in a phone directory.

The Gateway section also covered how to implement **Human-in-the-loop at scale** through Guardrails. A great real-world example was given: an Agent is allowed to automatically issue refunds for transactions under $100. But if a customer asks for a $200 refund, the Agent cannot make that call on its own — Gateway escalates the request to an admin for review and approval before anything happens. This keeps the process flexible while still maintaining control over risk.

Gateway also supports:
- **Interceptors (Hooks):** automatically scanning and masking sensitive personal data (PII) before sending any response back to the user
- **AWS PrivateLink:** secure connectivity to on-premise internal networks without exposing anything to the public internet

### Key Takeaways

#### System Design Mindset

The biggest lesson I'm taking home from this event is that **building AI for an enterprise is not a Prompting problem — it's a System Architecture problem**. I used to think that as long as the prompt was well-written, the AI would work well. But after today, I can see that behind every Agent, there are dozens of technical layers that need to be properly designed: the runtime environment, authentication mechanisms, authorization flows, data security, and how Agents communicate with both Tools and other Agents.

More importantly, **AI should never operate without human oversight** in an enterprise context. Human-in-the-loop and Gateway are not optional features — they are required to ensure the system never operates outside of acceptable boundaries.

#### Technical Knowledge

This event helped me understand a lot more about how to build Agentic AI the right way:

- **Firecracker MicroVM** provides hardware-level session isolation, ensuring user data never leaks between sessions.
- **MCP (Model Context Protocol)** is gradually replacing standard REST APIs for connecting Agents to Tools, and Semantic Search lets Agents find the right Tool without hard-coding.
- The **Token Vault chain (JWT → WAT → OAuth Credential)** is a critical security mechanism for keeping secret keys completely hidden from the AI.
- **Asynchronous processing** is essential to avoid making users wait for complex AI tasks to fully complete.
- **Guardrails in Gateway** are the scalable way to implement Human-in-the-loop across a large system.

#### FinOps and Security

- Serverless Runtime is the most cost-efficient choice for Agents since you only pay when there's actual traffic.
- Security doesn't end at IAM Roles — you need to design the entire token conversion flow so the AI never directly touches sensitive credentials.
- When connecting to on-premise systems, **AWS PrivateLink** is the required approach instead of an Internet Gateway — both more secure and preventing data from leaking to the public internet.

### Applying to Work

After the event, I realized there are a few things I can start applying to real projects right away:

- **Protecting user data:** When building any AI system, I'll design an Interceptor or Guardrail layer in the middleware to automatically scan and mask sensitive PII before the system returns any response to users.
- **Proper API key management:** Instead of embedding API keys directly in code, I'll reference the Token Vault and Workload Access Token model to issue permissions indirectly and more securely.
- **Improving user experience:** Apply asynchronous processing to complex AI tasks so users aren't staring at a blank screen waiting for a result.
- **Designing Agents with boundaries:** When deploying any Agent, always define clear Guardrails to limit the scope of what the AI can act on — especially for anything with real-world impact like financial transactions or data modification.
- **Picking the right model:** Not every task needs the most powerful model. I'll analyze what each task actually requires before deciding between Claude Haiku, Sonnet, or Opus — avoiding unnecessary cost for no real benefit.

### Personal Experience

Attending **AWS FCAJ Agent Forge - Deepdive** was honestly an intense experience in the best possible way. Nearly 100 slides were covered just in the morning session alone, but the content was structured to flow naturally from theory into real-world examples — it never felt like being force-fed information.

The part that stood out to me the most was the explanation of **Firecracker MicroVM**. It was the first time I really understood how large cloud platforms protect user privacy at the deepest layer of infrastructure — not through software logic, but through complete hardware-level virtualization isolation. When I used ChatGPT or AI APIs in the past, I never once stopped to wonder whether my data could get mixed up with someone else's. Now I know that solving that problem requires a dedicated technology like Firecracker, and it doesn't happen by accident.

The Hands-on Lab led by Hải Anh was also genuinely eye-opening. Actually working with the system hands-on made me appreciate the difference between "knowing the theory" and "knowing how to apply it." Some code snippets looked deceptively simple on the slides, but when you're actually typing it out and running it, you start noticing all the small details that are easy to miss if you're just listening to a presentation.

### Lessons Learned

After the event, here are a few lessons I think I'll carry for a long time:

- Building enterprise AI is not just about Prompting or Fine-tuning — it's a full System Design problem that spans Networking, Identity, and Serverless Infrastructure.
- **Human-in-the-loop is not a feature you add later** — it needs to be designed into the architecture from day one as a core component.
- Picking the right model (Haiku / Sonnet / Opus) matters just as much as writing a good Prompt — the wrong choice can make the system slower or more expensive than it needs to be.
- Security in an Agent system is not "assign an IAM Role and call it done" — you need to design the full token chain so the AI never sees a secret key directly.
- **Firecracker MicroVM** showed me that real data isolation is achievable at the hardware level — it's not just a theoretical security concept on paper.
- As systems scale, **Gateway and MCP** become non-negotiable layers for managing hundreds of Agents and thousands of Tools without losing control.

### Event Photos

- **Photos from the event:**

![Event 2-1](/images/4-EventParticipated/EV2-1.png)
![Event 2-2](/images/4-EventParticipated/EV2-2.png)
![Event 2-3](/images/4-EventParticipated/EV2-3.png)
![Event 2-4](/images/4-EventParticipated/EV2-4.png)
![Event 2-5](/images/4-EventParticipated/EV2-5.png)

> Overall, AWS FCAJ Agent Forge helped me understand that building an AI Agent system that actually works well in an enterprise takes a lot more than a smart model. Behind it all is a carefully designed system of security layers, permission flows, data isolation, and risk governance. This event shifted my perspective from "building AI" to "designing AI systems" — and that's a much bigger difference than it sounds.