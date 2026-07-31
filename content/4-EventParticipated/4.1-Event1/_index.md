---
title: "Event 1: FCAJ Community Day"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

### Event Information

* **Event Name:** FCAJ Community Day
* **Date:** June 27, 2026
* **Location:** 26th Floor, Bitexco Tower, 02 Hai Trieu Street, Saigon Ward, Ho Chi Minh City
* **Role:** Attendee

### Event Objectives

A multi-topic sharing day where speakers from several different companies each presented on a real-world AWS/AI topic, 5 talks total, covering secure agent infrastructure, enterprise AI productivity tools, DevOps agents, voice agents, and operating agentic systems in the cloud.

### Topics

- Building secure private MCP for Quick
- AI-powered productivity worker planning for Enterprise
- AWS DevOps Agent
- Building Voice Agent At Scale
- AgenticOps for your Cloud

### Speakers

- **Mr.Steve Tran** - CEO of Cloud Thinker
- **Mr.Nghi Danh** - Renova Cloud
- **Mr.Trung** - AWS Builder
- **Mr.Trung** - CEO Revve AI
- **Ms.Bao** - Cloud Engineer
- **Mr.Nguyen** - Could Engineer
- **Mr.Truong** - Noventiq Viet Nam
- **Ms.Minh Anh** - Noventiq Viet Nam
- **Mr.Toan Nguyen** - AWS Security Builder

### Key Highlights

- **Building secure private MCP for Quick**: architecture for running a private Model Context Protocol (MCP) server so internal tools and data can be exposed to AI agents without leaving the company's network boundary
- **AI-powered productivity worker planning for Enterprise**: how enterprises are piloting AI agents to help plan and prioritize work across teams
- **AWS DevOps Agent**: an agent that assists with CI/CD pipelines, incident triage, and infrastructure changes on AWS
- **Building Voice Agent At Scale**: architecture and scaling challenges of running voice-based AI agents for a large number of concurrent users
- **AgenticOps for your Cloud**: the operational practices (monitoring, guardrails, rollback) needed once agents start making changes to cloud infrastructure themselves

### Key Takeaways

- Multiple, unrelated companies independently converging on the same problem - giving AI agents safe, scoped access to internal tools and data - shows this is a real production concern, not a hypothetical one
- Voice agents and DevOps agents both need the same core reliability discipline highlighted at the other AgentForge events: guardrails, tracing, and a human-in-the-loop fallback
- "AgenticOps" is emerging as its own operational discipline, distinct from traditional DevOps, once agents can act on infrastructure directly instead of just answering questions

### Applying to Work

The private MCP talk pointed directly at a gap in ServerlessFinance - the REST API (Phase 4) currently has no scoped, agent-safe access layer. Exposing it as an MCP server instead of a plain REST API would let an AI agent query backtest results or place a paper-trading order within the kind of access boundaries discussed in the talk, instead of needing raw API-key access to everything.

### Event Experience

Having five different companies each present in one afternoon gave a much broader, more practical view than a single-vendor talk would have. The MCP security and AgenticOps talks stood out most to me since they pointed at gaps I could immediately connect to my own project, while the voice agent and DevOps agent talks were useful context even though less directly applicable to what I was building.
