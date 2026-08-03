---
title: "Event 4: AgentForge - Deep Dive"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

### Event Information

* **Event Name:** AgentForge - Deep Dive
* **Date:** August 1, 2026
* **Location:** 26th Floor, Bitexco Tower, 02 Hai Trieu Street, Saigon Ward, Ho Chi Minh City
* **Role:** Attendee

### Event Objectives

A half-day, hands-on follow-up to the AgentForge L300-400 day (Event 2), focused on taking a single Amazon Bedrock AgentCore agent from zero to a working, authenticated web app: deploy a basic agent, wire it up to external tools and a Knowledge Base, then front it with a web UI secured by Amazon Cognito.

### Speakers

- **Nghia Tran** - Agentic SA
- **Anh Pham** - Cloud Consuitant C-Asiapacific Vietnam

### Key Highlights

- Morning split into two blocks: a 1-hour foundations talk covering the Amazon Bedrock AgentCore Overview (Runtime, Gateway, Identity) followed by a 1-hour hands-on lab
- Lab walked through deploying a base agent in AgentCore Runtime, then extending it by connecting external tools and a Knowledge Base through Gateway
- Final lab step wrapped the agent in a web UI backed by Amazon Cognito, turning a bare agent endpoint into something an actual user could log into and use
- Faster-paced and more compressed than the full AgentForge day, same Runtime/Gateway/Identity primitives, but the whole arc from "agent deployed" to "usable, authenticated app" fit into one hour

### Key Takeaways

- Runtime, Gateway, and Identity form a minimum viable stack for a production-facing agent: Runtime executes it, Gateway is how it reaches tools and Knowledge Bases, Identity is how both the agent and the end user get authenticated
- Cognito sitting in front of the web UI is the same "who is allowed to call this" problem as the Gateway/Identity discussion from Event 2, just moved to the human-facing side of the agent instead of the agent-to-tool side
- Connecting a Knowledge Base is treated as a first-class Gateway integration rather than a bolt-on RAG pipeline, retrieval is just another tool the agent reaches through the same access path as everything else

### Applying to Work

Cognito-in-front-of-a-web-UI is the missing piece on ServerlessFinance's human-facing side: the REST API (Phase 4) is gated by a single shared API key with no concept of individual users, and there's no UI at all today, just `curl` and `test-invoke-method` calls. The same pattern demoed here (Cognito at the front door, API/agent behind it) would give ServerlessFinance per-user auth instead of one shared key, plus an actual front end for the backtest/results endpoints instead of raw API calls.

### Event Experience

Going from "an agent is deployed" to "a logged-in user can open a web page and use it" in a single one-hour lab made the Runtime -> Gateway -> Identity stack feel a lot less abstract than the deep-dive talks at Event 2 did. Watching Cognito get bolted onto the front of an agent as just another auth boundary, the same shape as the Gateway/Identity discussion from the full AgentForge day, made it click that ServerlessFinance's missing "who's the user" layer isn't a separate problem from its missing "who's the agent" layer, but the same pattern applied twice.
