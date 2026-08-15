---
title: "AWS FCAJ Agent Forge - Hands-on AgentCore (Deepdive day 3)"
date: 2026-08-15
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

# Summary Report: "AWS FCAJ Agent Forge - Hands-on AgentCore (Deepdive day 3)"

### Event Information

- **Event Name:** AWS FCAJ Agent Forge - Session 3 (Hands-on Lab & Case Study)
- **Format:** Workshop combined with direct hands-on practice on the system (hands-on lab)
- **Key Focus:** Practical deployment of the core components of **AWS Bedrock AgentCore**
- **Role:** Participant, engaging in case study analysis and coding directly on the system following step-by-step guidance

### Event Objectives

While the previous sessions of the **AWS FCAJ Agent Forge** series leaned heavily toward architectural theory and the design mindset of Agentic AI, this third session shifted gears entirely into **hands-on execution**. The clear objective of the workshop was to help participants transition from "understanding" to "doing" — specifically, to manually deploy each key component in the **AgentCore** ecosystem, from Memory and Gateways to interactive User Interfaces and Observability systems.

The workshop delivered a consistent, powerful message: building AI Agents is not merely about stitching tools together. Rather, it requires a deep understanding of how various components coordinate within a complete, production-ready architecture, and knowing how to apply them effectively to solve real-world business challenges.

### Main Content

#### Overview of Agentic AI Architecture with AgentCore

The opening segment helped reconstruct the big picture of a modern Agent system. The crux of the architecture lies in the concept of the **AgentCore Harness** — which I visualize as a "skeleton" or "physical body" wrapping around the AI "brain." While the Large Language Model (LLM) acts as the brain responsible for reasoning and planning, the Harness serves as the body that enables the brain to interact with the physical and digital world. It connects the model with a Code Interpreter, web browsers, and various custom integration tools.

What resonated with me the most in this section was the **"Everything is a plugin"** design philosophy. Instead of building a rigid, monolithic system, the AgentCore architecture is completely modularized. This means every single capability of the Agent — whether it is executing code, browsing the web, or calling an external corporate API — is designed as a plug-and-play module. This approach makes extending, customizing, and maintaining the system incredibly seamless, especially as business requirements inevitably evolve over time.

I realized that this plugin-centric mindset is not just a technical choice; it is a sustainable design philosophy. A system that is easy to extend and dismantle at a modular level will naturally enjoy a longer lifecycle and adapt far better to future technological shifts.

#### Case Study 1 — Multi-Agent System for QA and Automated Bug-Fixing

This was one of the two case studies that excited me the most. The speaker introduced a **Multi-Agent** (collaborative agents) architecture applied to a challenge that every software engineer is deeply familiar with: testing and bug fixing.

The core idea is to pair two specialized Agents with distinct roles:
- **QA Agent** — responsible for running test scripts, analyzing source code, and detecting bugs or potential vulnerabilities.
- **Bug-fixing Agent** — automatically proposing code changes and generating patches to fix the bugs identified by the QA Agent.

What makes this system highly practical is that **it does not completely eliminate humans** from the loop. While the two agents can autonomously handle the repetitive, time-consuming tasks of scanning and patching, the final quality gate remains firmly in human hands through a **Senior Review (Human-in-the-loop)** process. A senior engineer reviews and approves the draft Pull Request before any changes are merged into the main codebase.

This case study vividly demonstrated the real-world value of Multi-Agent systems: they do not seek to replace human intelligence, but rather liberate professionals from tedious chores so they can focus on high-impact decisions requiring deep intuition and experience. This is precisely the "human + AI" partnership model that I believe will become the industry standard in the near future.

#### Case Study 2 — IoT & AI Application: Vital Signs Monitoring via Wi-Fi Signals

The second case study was mind-opening and truly pushed the boundaries of my imagination. The presenter introduced an innovative solution that uses **ambient Wi-Fi signals** to track human vital signs, such as **heart rate and respiration rate**, completely **without using cameras**.

At first glance, this concept sounded like science fiction. However, as the underlying physics was explained, it became clear: micro-movements of the body (such as the chest rising and falling during breathing, or the subtle vibrations of a beating heart) cause minute disturbances in the Wi-Fi waves propagating through a room. By utilizing AI to analyze these micro-disturbances and filter out background noise, the system can accurately calculate vital signs.

The most profound value of this solution lies in **privacy preservation**. Monitoring health metrics without cameras opens up massive opportunities in smart home development and elderly care, where continuous camera surveillance is often perceived as intrusive, uncomfortable, and a breach of personal privacy. This was an outstanding example of how AI, when combined with IoT, can solve real-world problems in ways I had never previously considered.

#### Hands-on Lab — Deploying Core Components of AgentCore

This was the most intense and rewarding part of the session, as we rolled up our sleeves to code. The lab guided us through deploying each core pillar of the AgentCore ecosystem:

**Configuring AgentCore Memory**
I learned how to configure the memory subsystem to store and manage the Agent's context. This is highly critical; without an explicit memory layer, an Agent suffers from "amnesia" after every single API call. It cannot maintain a coherent multi-turn conversation or recall information discussed earlier in the session. Effective context management is the defining factor that separates a truly intelligent agent from a basic, single-turn chatbot.

**Using AgentCore Gateway**
This segment focused on utilizing the Gateway to manage and secure connections to external APIs, tools, and specifically the **Model Context Protocol (MCP)**. I came to understand the Gateway as a central traffic cop, allowing the Agent to communicate with the outside world in an organized, audited, and secure manner, rather than letting individual agents connect to disparate external tools in a chaotic, point-to-point fashion.

**Developing UI with Streamlit**
To transform an abstract Agent into a tangible product that users can interact with, a user interface is mandatory. The lab utilized **Streamlit** — a library that enables developers to rapidly spin up interactive web interfaces entirely in Python, without needing deep frontend expertise (like React or Vue). I found this to be an excellent tool for rapid prototyping and creating Proof of Concepts (PoC), allowing us to validate ideas with stakeholders without sinking weeks of development time into UI design.

**Integrating Observability**
The final, yet perhaps most critical component, is **Observability**. I learned to configure tools to monitor the Agent's performance, trace its internal execution flow (identifying exactly which tool the Agent called, its raw reasoning process, and execution time), and keep a close eye on token consumption costs. This is an area often neglected by beginners, yet it is absolutely mandatory when transitioning a system to production. Without deep visibility into what the Agent is doing behind the scenes, debugging anomalies or optimizing operational costs becomes virtually impossible.

### Key Takeaways

#### System Design Mindset

The single biggest lesson I took home from this session is that **modular, plugin-based architecture is the key to building sustainable AI systems**. The "Everything is a plugin" philosophy not only makes the codebase easier to maintain, but also ensures the system remains agile enough to adapt to the rapidly changing AI landscape.

Furthermore, the **Multi-Agent combined with Human-in-the-loop** paradigm proves that the future of enterprise AI is not about replacing human talent, but augmenting it. AI excels at handling repetitive, execution-heavy tasks, while humans retain the final decision-making authority.

#### Technical Knowledge

- **AgentCore Harness** serves as the operational "body" for the AI "brain," seamlessly bridging the LLM with the Code Interpreter, web browser, and custom enterprise tools.
- **AgentCore Memory** is the critical component that dictates an Agent's ability to maintain rich context across multiple interactions over time.
- **AgentCore Gateway + MCP** enables secure, centralized management of connections to external APIs and microservices.
- **Streamlit** is the ultimate tool for backend and AI engineers to rapidly visualize ideas and build interactive demos in minutes.
- **Observability** is a non-negotiable requirement for running Agents in production, allowing developers to trace performance, logical steps, and financial costs.

#### Practical Application

- The Multi-Agent QA + Bug-fixing architecture demonstrates how AI can automate a massive portion of the software development lifecycle, freeing engineers from repetitive debugging cycles.
- Non-invasive vital signs tracking via Wi-Fi sensing highlights the immense potential of combining AI with IoT to create secure, privacy-centric health monitoring solutions.

### Applying to Work

Following this session, I have identified several concrete initiatives that I can immediately apply to my current projects:

- **Adopt a Plugin-First Architecture:** When building any new AI feature, I will prioritize modular design to ensure we can easily swap out underlying LLM models or API tools without refactoring the entire system.
- **Implement Robust Context Management:** For applications requiring ongoing customer interaction, I will carefully design the Vector Database and memory caching mechanisms right from Day 1 to ensure a seamless conversational history.
- **Incorporate Observability Early:** Instead of treating logging as an afterthought, I will integrate tracing and token-monitoring tools early in the development cycle to prevent runaway API costs.
- **Use Streamlit for Rapid Prototyping:** For any upcoming AI ideas that need internal approval, I will use Streamlit to build quick, functional web interfaces to pitch to leadership and clients.
- **Ensure Human-in-the-Loop Safeguards:** For any high-impact actions (such as code modifications, database writes, or financial transactions), I will design mandatory human-approval gates into the Agent's workflow.

### Core Message

The most valuable insight I took away from the workshop is that **technical mastery of AI tools is only half the battle; the real differentiator is Domain Knowledge**. Technology is merely an enabler. To apply AI effectively and solve real business problems, we must deeply understand the business processes we are trying to automate. An engineer who is an expert in prompting but lacks an understanding of business workflows will struggle to deliver tangible enterprise value.

### Personal Experience

Session 3 of **AWS FCAJ Agent Forge** offered a highly satisfying, grounded experience compared to the purely theoretical lectures of the past. Actually writing code and executing the agents on AWS made me appreciate the gap between "knowing the theory" and "executing the practice." Many architectural patterns that sounded straightforward during the slides revealed complex edge cases once I started implementing them.

The two case studies — particularly the Wi-Fi sensing application — were deeply inspiring. They served as a powerful reminder that we should not restrict our view of AI to just text processing or image generation. The physical world is full of rich sensor data waiting to be unlocked by creative AI system design.

### Lessons Learned

- Building an AI Agent is an integration challenge involving Memory, Gateways, UI, and Observability; it is far more complex than just making a simple model API call.
- A plugin-based architecture ensures system durability and flexibility in a fast-changing tech landscape.
- Human-in-the-loop controls are essential for safety and reliability, even within highly autonomous Multi-Agent systems.
- Observability is a core requirement that must be designed into the architecture from the start, not bolted on at the end.
- Domain knowledge is just as critical as software engineering skills — understanding the core business problem is the ultimate key to unlocking AI's true value.

### Proof of Attendance

- **Event Participation Photos:**

![Event 4-1](/images/4-EventParticipated/EV4-1.png)
![Event 4-2](/images/4-EventParticipated/EV4-2.png)
![Event 4-3](/images/4-EventParticipated/EV4-3.png)
![Event 4-4](/images/4-EventParticipated/EV4-4.png)
![Event 4-5](/images/4-EventParticipated/EV4-5.png)

> In conclusion, Session 3 of AWS FCAJ Agent Forge has successfully bridged the gap between architectural theory and real-world deployment. Through practical labs exploring Bedrock AgentCore and inspiring real-world case studies, I now feel fully equipped to design, build, and monitor robust AI Agents that deliver real, secure, and manageable value to enterprise environments.