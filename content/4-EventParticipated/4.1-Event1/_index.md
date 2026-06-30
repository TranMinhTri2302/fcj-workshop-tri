---
title: "FCAJ Community Day - June 2026"
date: 2026-06-01
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
----------------------

# Summary Report: “FCAJ Community Day - June 2026”

### Event Information

* **Event name:** FCAJ Community Day - June 2026
* **Time:** 09:00, June 27, 2026
* **Location:** Offline event, floor 26/36
* **Role:** Attendee / Participant

### Event Objectives

* Share practical enterprise experience in **Cloud Infrastructure**, **AI**, and **DevOps**.
* Discuss how AI impacts the roles of Cloud Engineers, DevOps Engineers, and Solution Architects.
* Introduce real-world use cases of **Voice AI**, **DevOps Agent**, and **AI for HR operations**.
* Provide insights into **AWS architecture**, **security**, **private connectivity**, and **cloud cost estimation**.
* Help participants understand how to design systems with **security, scalability, and cost optimization** in mind.

### Speakers

* **Steve Trần** – Cloud Thinker
* **Nghị, Kiệt, Trung** – R AI, Voice AI session
* **Bảo and Nguyên** – Cloud Kinetic, DevOps Agent session
* **Trường and Minh Anh** – Noventis, AI for HR session
* **Toàn** – Cloud / Solutions speaker, AWS architecture and cost breakdown

### Key Highlights

#### Cloud Engineering in the AI Era

The event started with discussions about the development of cloud adoption in enterprises and how engineers should prepare themselves in the AI era. One important message was that AI can speed up coding and implementation, but it cannot fully replace human decision-making in complex production systems.

* AI can support repetitive and technical tasks, but engineers are still needed for **incident response**, **architecture decisions**, and **system trade-offs**.
* As enterprises scale globally, system complexity increases rapidly.
* A single engineer or AI tool cannot fully understand all layers of a large system, from source code to business logic.
* Cloud Engineers and Solution Architects still play a critical role in designing reliable and scalable infrastructure.

#### Voice AI for Banking Use Case

The Voice AI session was one of the most interesting parts of the event. The speakers shared how Voice AI can be applied in a banking context, where users interact with the system through voice and the system processes the request using AI models.

A key challenge mentioned was that Vietnamese is still considered a **low-resource language** in many AI systems. There is no simple and reliable direct pipeline where Vietnamese voice can be sent straight into an LLM for full understanding. Because of that, the system needs multiple processing steps:

* Convert **voice input** into **Vietnamese text**.
* Process or translate the Vietnamese text into **English**.
* Send the English text into the **LLM** for reasoning and response generation.
* Use **tool calling** to perform banking-related tasks or controlled actions.

During the Q&A, an attendee asked whether the system could support different Vietnamese regional accents. The speaker explained that through real usage and training, the system could improve its ability to understand different accents from Vietnamese speakers. Besides recognizing speech content, the system could also identify additional voice features, such as whether the speaker is **male or female**.

This session helped me understand that building Voice AI is not only about connecting speech recognition with an LLM. It also requires solving real-world problems such as language support, accent variation, data quality, and safe task execution.

#### DevOps Agent and Incident Automation

The DevOps Agent session showed how AI agents can support operations teams by automating parts of the incident investigation process.

* The agent can help analyze logs and system behavior.
* It can reduce the time engineers spend manually investigating issues.
* The main goal is to reduce **MTTR** and improve system reliability.
* AI can support DevOps teams by suggesting possible causes, affected services, and next steps.

This session showed that AI can be useful not only for coding, but also for production monitoring and system operations.

#### AI in Human Resources

The session by Noventis focused on applying AI in HR workflows, especially recruitment and candidate management.

* AI can help screen CVs faster.
* It can support candidate data management.
* It can reduce manual effort in repetitive HR tasks.
* The demo showed how tools such as **Amazon Q** can support internal business processes.

This topic helped me see how AI can be applied outside technical teams and bring value to business departments such as HR.

#### Security, MCP Server, and Private AWS Connectivity

The final sessions focused on security and AWS infrastructure. The speakers discussed how to connect applications such as an MCP Server securely using private AWS architecture.

Key points included:

* Security should be considered from the architecture design stage, not after deployment.
* Sensitive credentials, such as third-party API keys, should not be hard-coded.
* **AWS Secrets Manager** should be used to manage credentials securely.
* Private connectivity can improve security, but it also increases infrastructure cost.

A practical AWS cost breakdown was also discussed:

* **Route 53 Resolver:** around **$180/month**
* **Application Load Balancer:** around **$32/month**
* Basic private setup cost: around **$250–350/month**, not including data transfer

The speaker also emphasized that cloud cost estimation cannot be based only on the number of users. Engineers must work with business or product teams to estimate actual data volume, such as GB/month, before calculating infrastructure cost accurately.

### Key Takeaways

#### Design Mindset

* **AI is an accelerator, not a full replacement** for engineers.
* Engineers still need strong judgment for troubleshooting, architecture design, and critical incidents.
* System complexity is unavoidable as businesses scale, so systems should be designed with modularity in mind.
* Security and cost should be part of the initial design, not treated as afterthoughts.

#### Technical Architecture

* Voice AI systems require multiple layers: speech recognition, language processing, LLM reasoning, and tool calling.
* For Vietnamese Voice AI, regional accents and low-resource data are important challenges.
* DevOps Agents can support log analysis, incident investigation, and MTTR reduction.
* AWS private architecture improves security but introduces additional networking costs.
* Secrets and credentials should be managed through secure services such as AWS Secrets Manager.

#### Cloud Cost and FinOps

* FinOps is not only about reducing cost, but also about understanding the relationship between infrastructure and business usage.
* Private networking components can cost more than expected.
* Cost estimation should be based on real workload assumptions, especially data transfer volume.
* AI can support FinOps by analyzing usage patterns and suggesting optimization opportunities.

### Applying to Work

* Review current cloud architecture to identify unnecessary private endpoints or expensive networking components.
* Apply better credential management by using tools such as **AWS Secrets Manager**.
* Consider AI tools for log analysis, security assessment, and cost optimization.
* When estimating cloud cost, clarify data volume with business teams instead of relying only on user count.
* For AI applications, especially Voice AI, evaluate language support, data quality, and user diversity such as regional accents.

### Event Experience

Attending **“FCAJ Community Day - June 2026”** was valuable because it provided practical knowledge from real enterprise use cases. The event was not only about AI trends, but also about how AI, cloud, DevOps, and security are actually applied in production systems.

#### Learning from experienced speakers

The speakers shared realistic challenges from enterprise environments, especially around cloud complexity, AI adoption, security, and cost. I learned that engineers need to understand both technical design and business impact when building cloud systems.

#### Practical technical exposure

The Voice AI session helped me understand the complexity behind building a Vietnamese voice-based AI system. The AWS architecture session also gave me a clearer view of hidden costs in private networking, such as Route 53 Resolver and ALB.

#### Networking and discussion

The open Q&A format allowed participants to ask practical questions. My question about Vietnamese regional accents in Voice AI helped me understand how the system handles real user diversity and how voice data can be improved through usage.

#### Lessons learned

* AI should be used to improve productivity, not blindly replace engineering judgment.
* Voice AI for Vietnamese requires careful handling of speech, language, accent, and LLM integration.
* Cloud architecture must balance security, performance, and cost.
* Cost estimation needs collaboration between engineering and business teams.
* Security and FinOps should be included from the beginning of system design.

#### Some event photos
![Event 1-1](/images/4-EventParticipated/EV1-1.png)
![Event 1-2](/images/4-EventParticipated/EV1-2.png)
![Event 1-3](/images/4-EventParticipated/EV1-3.png)
![Event 1-4](/images/4-EventParticipated/EV1-4.png)
![Event 1-5](/images/4-EventParticipated/EV1-5.png)

> Overall, FCAJ Community Day helped me gain a more practical understanding of how AI and cloud technologies are used in real enterprise systems. The event also strengthened my awareness of system complexity, secure architecture, FinOps, and the importance of human decision-making in the AI era.
