---
title: "Week 2 Worklog"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 2 Objectives
  - Tasks to be carried out this week
  - Week 2 Achievements
reportType: worklog
---

### Week 2 Objectives:

* Learn AWS CDK (Python) as the Infrastructure-as-Code tool for the whole project.
* Scaffold the ServerlessFinance repository (Phase 0) and get a first `cdk synth` working.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | Study AWS CDK concepts: App, Stack, Construct (L1/L2), synth vs. deploy | 11/05/2026 | 12/05/2026 | AWS CDK docs |
| 2 | `cdk bootstrap` the target account/region; write a trivial CDK app and deploy it | 12/05/2026 | 13/05/2026 | AWS CDK docs |
| 3 | Design the repo layout: `infra/` (CDK), `services/` (Lambda handlers), `strategies/`, `tests/` | 13/05/2026 | 14/05/2026 | |
| 4 | Set up Python virtualenv, `requirements.txt`, separate infra vs. runtime dependencies | 14/05/2026 | 14/05/2026 | |
| 5 | Get `cdk synth` producing a valid (empty) CloudFormation template - Phase 0 done | 15/05/2026 | 16/05/2026 | |

### Week 2 Achievements:

* AWS CDK (Python) chosen as the IaC tool for the whole project - see [Proposal §4](../../2-Proposal/#4-technical-implementation).
* Repo scaffolding in place: `infra/app.py`, `infra/stacks/`, `services/`, `strategies/`, `tests/`.
* `cdk synth` runs clean - Phase 0 verified. See [Workshop - Prerequisites](../../5-Workshop/5.2-Prerequisites/) for the exact setup steps.
