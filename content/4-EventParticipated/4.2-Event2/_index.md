---
title: "Event 2: AgentForge - Ho Chi Minh City"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

### Event Information

* **Event Name:** AgentForge - Ho Chi Minh City
* **Date:** July 21, 2026
* **Location:** 26th Floor, Bitexco Tower, 02 Hai Trieu Street, Saigon Ward, Ho Chi Minh City
* **Role:** Attendee

### Event Objectives

An intensive full-day L300-400 training on **Amazon Bedrock AgentCore**, aimed at builders ready to go deep on agentic AI systems. Covering the full AgentCore stack (Runtime, Gateway, Identity, Memory, Evaluations, Observability, Registry, Policy) end-to-end, from architecture to production, through deep-dive sessions, a real-world DevOps use case, and hands-on labs.

### Speakers

- **Vasileios Vonikakis** - Sr. GTM Specialist Solution Architect
- **Isaac Ibrahim** - GTM Specialist SA – AI/ML
- **Tim Wu** - Sr. GTM Spec. SA AIML
- **Brian Bae** - Head of ASEAN AI GTM

### Key Highlights

- Deep-dive presentations across the entire Amazon Bedrock AgentCore platform - Runtime, Gateway, Identity, Memory, Evaluations, Observability, Registry, and Policy
- A real-world DevOps use case walkthrough showing AgentCore running in production
- Hands-on lab: deploy a base agent, then augment it with Evaluations and Policy enforcement, plus an open-ended enhancement of your own
- Best practices for designing, deploying, and operating production-ready agentic systems (L300-400 level - built for mastery, not just awareness)

### Key Takeaways

- AgentCore splits what used to feel like one monolithic "agent" into separate primitives - Runtime (execution), Gateway (tool/API access), Identity (auth for the agent itself), Memory (state across turns/sessions), Policy (guardrails), Evaluations (quality/regression testing), Observability (tracing) - each with its own concerns
- Policy enforcement and Evaluations were treated as part of getting a base agent to "production-ready," not optional extras bolted on afterward
- The DevOps use case surfaced real operational concerns beyond the happy path: authorization boundaries (Identity/Gateway), auditability, and how an agent's memory persists or expires across sessions

### Applying to Work

Seeing Gateway as the layer that exposes tools/APIs to an agent, paired with Identity giving each agent its own scoped auth, mapped directly onto a gap in ServerlessFinance's REST API (Phase 4) - right now a single API key grants full access to every endpoint. An AgentCore Gateway in front of it could scope exactly which endpoints an agent is allowed to call (e.g. read backtest results, but not place a paper-trading order), instead of an all-or-nothing key.

### Event Experience

A full day of hands-on labs, deploying a base agent and then layering on Evaluations, Policy enforcement, and my own enhancement, made AgentCore's pieces click in a way a single demo couldn't have. It also reframed how I think about ServerlessFinance's API: treating "who is allowed to call what" as its own explicit layer (Gateway + Identity) rather than a single shared API key is a small but real gap I hadn't thought about before this event.
