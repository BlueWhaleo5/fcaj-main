---
title: "Event 3: FCAJ x Agentic AI Build Week"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

### Event Information

* **Event Name:** FCAJ x Agentic AI Build Week
* **Date:** July 25, 2026
* **Location:** 36th Floor, Bitexco Tower, 02 Hai Trieu Street, Saigon Ward, Ho Chi Minh City
* **Role:** Attendee

### Event Objectives

A hackathon/demo day where teams built and presented AI-powered solutions to five different real-world problem statements.

### Topics

- **AI-powered Conversation Ordering** - Inspired by McDonald's AI Drive-Thru problem, an agent that takes a customer's order conversationally, handling substitutions and upsells
- **Signal Scout** - Aggregates scattered public information about a business into a single, structured, in-depth profile
- **Solution Architect Professional (Native App)** - A native app to help candidates study and practice for the AWS Solutions Architect Professional certification
- **Smart Human Flow** - Evaluation, prediction, and hazard detection for human/crowd flow, with automated respond-and-dispatch
- **Adaptive AML Workflow Engine** - Automates investigative data enrichment for anti-money-laundering cases, turning hours of manual searching into a legally compliant report

### Speakers
*Teams:*  
- **One Team**  
- **Dream AI Team**  
- **Plan V**  
- **Team 3KA**  
- **Six Pillars Team**  

### Key Highlights

- Five teams each pitched and demoed a working prototype built for one of the five problem statements above
- A strong theme across teams: using an LLM/agent to compress hours of manual, fragmented work - data gathering, report writing, order-taking - into a single structured output
- Q&A largely focused on production-readiness concerns: data accuracy, compliance, and latency under real-world load

### Key Takeaways

- The most convincing demos were the ones with a clear "before vs. after" for a genuinely tedious manual process (e.g. the AML report writing, the business research in Signal Scout) rather than just a novel chatbot interface
- Producing structured, verifiable output (like a compliance report) is a much harder bar than a general Q&A chatbot - accuracy and traceability of sources matter more than fluency
- Real-time/voice interfaces (the conversational ordering agent) surface a different set of engineering problems - latency, interruption handling - than research/reporting-style agents

### Applying to Work

The Signal Scout / AML pattern - turning scattered raw data into one structured report - is the same shape as ServerlessFinance's own pipeline: pulling raw OHLCV data through the data lake and producing a single structured backtest report. Seeing the same "aggregate -> structure -> report" pattern applied to compliance and business intelligence reinforced that it generalizes well beyond finance.

### Event Experience

Watching five very different teams tackle their problem statements in one afternoon was a good reminder of how much the "shape" of a good AI product repeats across domains, even when the subject matter (drive-thru orders vs. AML investigations) has nothing in common. The AML and Signal Scout demos stuck with me most, since the underlying pattern, pull scattered data, structure it, produce one trustworthy report, is close to what ServerlessFinance already does for backtest results.
