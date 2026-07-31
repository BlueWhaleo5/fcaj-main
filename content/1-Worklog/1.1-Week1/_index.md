---
title: "Week 1 Worklog"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 1 Objectives
  - Tasks to be carried out this week
  - Week 1 Achievements
reportType: worklog
---

### Week 1 Objectives:

* Get familiar with the FCAJ program, mentors, and the AWS Console/CLI.
* Understand core AWS service categories (Compute, Storage, Networking, Database) before picking a project.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | Onboarding: program rules, meet mentors and other interns | 01/05/2026 | 01/05/2026 | |
| 2 | Learn AWS service categories (Compute, Storage, Networking, Database, Serverless) | 02/05/2026 | 04/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | Create AWS account, install & configure AWS CLI, learn IAM basics | 05/05/2026 | 05/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | Explore serverless services relevant to the project idea: Lambda, S3, DynamoDB, API Gateway | 06/05/2026 | 07/05/2026 | AWS documentation |
| 5 | Sketch the initial idea for **ServerlessFinance** (a serverless backtesting engine) and check it against AWS Free Tier / Pricing Calculator | 08/05/2026 | 08/05/2026 | |

### Week 1 Achievements:

* Working AWS account with AWS CLI configured (`aws sts get-caller-identity` verified).
* Clear picture of which AWS services a fully serverless data + compute pipeline would need: S3, Lambda, Step Functions, DynamoDB, API Gateway.
* Settled on the project direction: a distributed strategy-backtesting engine plus a real-time paper-trading module, both serverless (see the [Proposal](../../2-Proposal/)).
