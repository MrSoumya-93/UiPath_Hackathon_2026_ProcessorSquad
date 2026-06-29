# GuardianPulse AI

> **Telecom Customer Retention Platform — Agentic AI Solution built on UiPath**

GuardianPulse AI is a multi-agent, responsible-AI system that detects at-risk telecom customers, orchestrates a chain of specialised AI agents to evaluate churn risk, simulates intervention economics, and executes retention actions — all under two human-in-the-loop governance gates and a Responsible AI layer before any offer is sent or call is placed.

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Solution Components](#solution-components)
  - [Orchestration Layer](#orchestration-layer)
  - [AI Agents](#ai-agents)
  - [RPA Workflows (UiPath Studio)](#rpa-workflows-uipath-studio)
  - [Human-in-the-Loop Components](#human-in-the-loop-components)
  - [Data & Storage](#data--storage)
- [End-to-End Process Flow](#end-to-end-process-flow)
- [Agent Reference](#agent-reference)
- [LLM Model Strategy](#llm-model-strategy)
- [Responsible AI & Governance](#responsible-ai--governance)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Deployment Guide](#deployment-guide)
- [Configuration Reference](#configuration-reference)
- [Testing & Evaluation](#testing--evaluation)
- [Known Limitations & Roadmap](#known-limitations--roadmap)

---

## Architecture Overview

The solution follows a **C4 Level-2 Container Data-Flow** pattern (BPMN execution order). It is orchestrated end-to-end by **UiPath Maestro** using a BPMN process definition, and the reasoning backbone for all agents is provided by an **external LLM provider** (OpenAI GPT-5.4 / GPT-4o via UiPath AgentHub).

```
Customer Risk Signal Detected
        │
        ▼
[RPA] CRM DataLoader ──► Agent 1: CRM Intelligence ──► Agent 2: Sentiment Intelligence
                                                                  │
                        ┌─────────────────────────────────────────┘
                        ▼
              Agent 3: Behavioral Intelligence ──► Agent 4: Revenue Leakage
                        │                                   │
                        └──────────────┬────────────────────┘
                                       ▼
                             Agent 5: Intent Prediction
                                       │
                                       ▼
                             Agent 6: Unified Risk Scoring
                                       │
                                       ▼
                             Agent 7: Counterfactual Simulation
                                       │
                                       ▼
                             Agent 8: Economic Decision
                                       │
                              ┌────────┴────────┐
                              ▼                 ▼
                    Agent 9: Humanity    Agent 10: Governance & Responsible AI
                         Layer                       │
                              │               ┌──────┴──────────┐
                              ▼               ▼                 ▼
                       [Gate 10] ──► Manager Approval    Action Blocked
                    Human Review     [UiPath App]
                              │
                 ┌────────────┤
                 ▼            ▼
         [Step 10]     [Gate ①: Pre-Call]
     Offer Review     Human Validates
       (Human)          Offer
                              │
                        [RPA] Initiate AI Call ──► [RPA] Conduct Negotiation
                                                           │
                                                  Confirm & Update CRM
                                                           │
                                              [Optional] Social Advocacy
```

**Two human gates** are enforced on every execution path:
- **Gate ① — Step 10 (Pre-Call Offer Review):** An observer validates the AI-selected offer before any outbound call is placed.
- **Gate ② — Manager Approval (UiPath App):** A Retention Manager performs the final sign-off after the AI call; rejected cases escalate to a Senior Retention Manager.

---

## Solution Components

### Orchestration Layer

| Component | Type | Description |
|---|---|---|
| `Maestro BPMN` | Process Orchestration | BPMN definition that sequences all agents and RPA steps in execution order. Deployed as a **UiPath Maestro** process. Contains both `Process.bpmn` (stable) and `Process_new.bpmn` (in-development). |

### AI Agents

| Component | Folder | Agent Type | LLM |
|---|---|---|---|
| CRM Intelligence Agent | `GP_CRM_Intelligence` | Low-code Agent | GPT-5.4 |
| Sentiment Intelligence Agent | `GP_Sentiment_Analysis` | Low-code Agent | GPT-5.4 |
| Sentiment Analysis (Coded) | `Coded copy of GP_Sentiment_Analysis` | **Python / LangGraph Agent** | GPT-4o-2024-11-20 |
| Behavioral Intelligence Agent | `GP_Behavioral_Risk` | Low-code Agent | GPT-5.4 |
| Revenue Leakage Agent | `GP_Revenue_Leakage` | Low-code Agent | GPT-5.4 |
| Intent Prediction Agent | `GP_Intent_Prediction` | Low-code Agent | GPT-5.4 |
| Unified Risk Scoring Agent | `GP_Risk_Scoring` | Low-code Agent | GPT-4o-2024-05-13 |
| Counterfactual Simulation Agent | `GP_Counterfactual_Sim` | Low-code Agent | GPT-5.4 |
| Economic Decision Agent | `GP_Economic_Decision` | Low-code Agent | GPT-5.4 |
| Humanity Layer Agent | `GP_Humanity_Layer` | Low-code Agent | GPT-5.4 |
| Governance & Responsible AI Agent | `GP_Governance` | Low-code Agent | GPT-5.4 |
| Best Offer Selection Agent | `GP_BestOffer_Selection` | Low-code Agent | GPT-4o-2024-11-20 |
| Customer Retention Offer Agent | _(sub-component of BestOffer)_ | Low-code Agent | — |

### RPA Workflows (UiPath Studio)

| Component | Folder | Role |
|---|---|---|
| CRM DataLoader | `GP_CRM_DataLoader` | Fetches customer records from CRM via REST; populates the execution context. |
| Initiate Negotiation Call | `Initiate_Call` | Places the AI retention voice call via AI Voice API. |
| Calling API Workflow | `Calling_API_Workflow` | Handles call state machine: captures reply, logs outcome. |
| Calling Retainer API | `GP_Calling_Retainer_API` | Wrapper API workflow for the retention call service. |
| Action Executor | `GP_Action_Executor` | Executes approved retention actions (benefit application, CRM update). |
| Learning Logger | `GP_Learning_Logger` | Logs intervention outcomes to the feedback store for continuous improvement. |
| AddOffersToEntities | `AddOffersToEntities` | Standalone utility — syncs offer catalogue from CRM into UiPath Entities. Deployed as a separate `.uis` package. |

### Human-in-the-Loop Components

| Component | Type | Description |
|---|---|---|
| Offer Review (Human) | UiPath Action / Task | Gate ① — observer validates AI-selected offer in Orchestrator Action Centre before the call is placed. |
| Manager Approval | `uiapp_FinalReview` (UiPath App) | Gate ② — Retention Manager approves or rejects the intervention outcome after the AI call. |

### Data & Storage

| Resource | Type | Contents |
|---|---|---|
| `Customer_Data` | Orchestrator Bucket / Index | Inbound customer risk records. |
| `Best_Offers` / `BestOffer_CSV` | Orchestrator Bucket / Index | Offer catalogue with cost range and retention-lift values. |
| `CustomerOffers` | UiPath Entity | Structured entity store for approved offers per customer. |
| CRM System (external) | REST | Canonical customer record store and offer catalogue master. |
| Social Media | REST (planned) | Loyalty advocacy publisher — fires only on explicit customer consent. |

---

## End-to-End Process Flow

### Step 1 — Trigger
A **Customer Risk Signal** is detected (scheduled batch or real-time event). Maestro starts the BPMN process.

### Step 2 — CRM Data Load
`GP_CRM_DataLoader` (RPA) fetches the customer record from the CRM system and hydrates the execution context with: `CustomerID`, `CustomerName`, `CustomerType`, `CurrentPlan`, `MonthlyARPU`, `DaysSinceRecharge`, `LastComplaintDate`, `CRMComplaint`, `CallTranscript`, `CompetitorPricingVisited`, `NetworkIssue`, `AppUsage`.

### Steps 3–10 — Intelligence Pipeline (Agents 1–10)

| Step | Agent | Key Output |
|---|---|---|
| 3 | CRM Intelligence | Value tier, account health summary |
| 4 | Sentiment Intelligence | `CallSentiment`, `EmotionalRiskLevel`, `KeyRiskPhrases` |
| 5 | Behavioral Intelligence | `BehavioralRiskScore` (0–100) |
| 6 | Revenue Leakage | `ExpectedRevenueLoss`, `LeakageSeverity`, `RecoveryProbability` |
| 7 | Intent Prediction | `ChurnProbability`, `CancellationLikelihood` |
| 8 | Unified Risk Scoring | `UnifiedRiskScore` (0–100), `RiskTier` (Critical / Very High / Medium / Low) |
| 9 | Counterfactual Simulation | `SimulationResults` — ROI for each intervention option |
| 10 | Economic Decision | `RecommendedAction`, `EstimatedROI` |
| ∥ | Humanity Layer | `HardshipFlag`, `HardshipType`, `HardshipSeverity` |
| ∥ | Governance & Responsible AI | `GovernanceDecision` (Approved / Blocked / Escalation Required), `FairnessFlag` |

Humanity Layer and Governance run in parallel after Economic Decision to avoid blocking the pipeline.

### Gate ① — Pre-Call Offer Review (Step 10, Human)
The Best Offer Selection Agent proposes the ranked offer. A human observer validates it in UiPath Action Centre. If the call is not answered, a push notification is sent via the app instead.

### Steps 11–13 — AI Voice Intervention
- `Initiate_Call` places an outbound AI retention call via the AI Voice external service.
- `Calling_API_Workflow` conducts the negotiation: presents the offer and captures the customer reply.

### Step 14 — Hardship Check (Decision Gateway)
- **Hardship Detected (Yes):** escalation path → Manager Approval with a hardship flag; commercial offer is suppressed.
- **Hardship Detected (No):** Governance decision gateway.

### Step 14b — Governance Decision Gateway
- **Approved:** flows to Manager Approval.
- **Blocked:** terminates with "Action Blocked" end event.
- **Escalation Required:** flows to Retention Manager for human review before proceeding.

### Gate ② — Manager Approval (UiPath App: `uiapp_FinalReview`)
Final human sign-off.
- **Accepted:** `GP_Action_Executor` applies the benefit, `Calling_API_Workflow` confirms retention, CRM is updated via REST, and `GP_Learning_Logger` records the outcome.
- **Rejected:** case escalates to Senior Retention Manager.

### Social Advocacy (Optional, Post-Consent)
If the customer consents and the case is flagged for social promotion, `Advocacy / Social Publisher` (RPA, planned) posts loyalty content via the Social Media API.

---

## Agent Reference

### Agent 1 — CRM Intelligence Agent (`GP_CRM_Intelligence`)
**Purpose:** Assess customer value and account health from CRM data.  
**LLM:** GPT-5.4 | **Max Tokens:** 128,000 | **Temperature:** 0  
**Inputs:** CustomerID, CustomerName, CustomerType, CurrentPlan, MonthlyARPU, DaysSinceRecharge, LastComplaintDate, CRMComplaint  
**Outputs:** Value / account health summary passed downstream.

---

### Agent 2 — Sentiment Intelligence Agent (`GP_Sentiment_Analysis`)
**Purpose:** Analyse the customer's call transcript for emotional tone and churn risk language.  
**LLM:** GPT-5.4 (low-code) / GPT-4o-2024-11-20 (Python/LangGraph coded copy)  
**Inputs:** `CallTranscriptSummary`  
**Outputs:**

| Field | Type | Values |
|---|---|---|
| `CallSentiment` | string | `Positive`, `Negative` |
| `EmotionalRiskLevel` | string | `Critical`, `High`, `Medium`, `Low` |
| `KeyRiskPhrases` | string | Comma-separated phrases |

> A Python/LangGraph implementation (`Coded copy of GP_Sentiment_Analysis`) is included as an alternative deployment using `uipath_langchain` and `uipath-agent`. Both versions expose identical input/output contracts.

---

### Agent 3 — Behavioral Intelligence Agent (`GP_Behavioral_Risk`)
**Purpose:** Score churn risk from digital behaviour signals.  
**LLM:** GPT-5.4  
**Inputs:** `CompetitorPricingVisited`, `NetworkIssue`, `AppUsage`, `DaysSinceRecharge`  
**Outputs:** `BehavioralRiskScore` (0–100), behavioural risk summary.

---

### Agent 4 — Revenue Leakage Agent (`GP_Revenue_Leakage`)
**Purpose:** Quantify the annual revenue at risk if the customer churns.  
**LLM:** GPT-5.4  
**Inputs:** `MonthlyARPU`  
**Outputs:**

| Field | Formula / Values |
|---|---|
| `ExpectedRevenueLoss` | `MonthlyARPU × 12` |
| `LeakageSeverity` | `Critical` (>$150), `High` ($100–$150), `Medium` ($50–$99), `Low` (<$50) |
| `RecoveryProbability` | `High`, `Medium`, `Low` |

---

### Agent 5 — Intent Prediction Agent (`GP_Intent_Prediction`)
**Purpose:** Predict cancellation intent from complaint text and behavioural signals.  
**LLM:** GPT-5.4  
**Inputs:** DaysSinceRecharge, LastComplaintDate, CRMComplaint, CallSentiment, EmotionalRiskLevel, KeyRiskPhrases, CompetitorPricingVisited, NetworkIssue  
**Outputs:** `ChurnProbability` (%), `CancellationLikelihood` (Very High / High / Medium / Low).

---

### Agent 6 — Unified Risk Scoring Agent (`GP_Risk_Scoring`)
**Purpose:** Fuse all upstream signals into a single 0–100 composite risk score.  
**LLM:** GPT-4o-2024-05-13 | **Max Tokens:** 4,096  
**Scoring formula:**

```
Base = (ChurnProbability × 0.5) + (LeakageSeverityScore × 0.3) + (CallSentimentScore × 0.2)
Bonus = +10 if EmotionalRiskLevel = Critical; +5 if High
CLV Modifier applied from MonthlyARPU
UnifiedRiskScore = min(Base + Bonus + CLV, 100)
```

**Outputs:** `UnifiedRiskScore` (0–100), `RiskTier` (Critical / Very High / Medium / Low).

---

### Agent 7 — Counterfactual Simulation Agent (`GP_Counterfactual_Sim`)
**Purpose:** Simulate multiple intervention options and estimate ROI for each.  
**LLM:** GPT-5.4  
**Inputs:** UnifiedRiskScore, RiskTier, MonthlyARPU, CustomerType, CancellationLikelihood  
**Outputs:** `SimulationResults` (JSON array of options with retentionLift, cost, revenueRecovered), `BestIntervention`, `EstimatedROI`.

---

### Agent 8 — Economic Decision Agent (`GP_Economic_Decision`)
**Purpose:** Select the final action with the highest ROI, validate proportionality.  
**LLM:** GPT-5.4  
**Inputs:** SimulationResults, BestIntervention, EstimatedROI, MonthlyARPU, UnifiedRiskScore, RiskTier  
**Outputs:** `RecommendedAction`.

---

### Agent 9 — Humanity Layer (`GP_Humanity_Layer`)
**Purpose:** Detect genuine customer hardship so commercial interventions can be overridden with compassionate support.  
**LLM:** GPT-5.4  
**Inputs:** CallTranscriptSummary, DaysSinceRecharge, CRMComplaint, CallSentiment, EmotionalRiskLevel, KeyRiskPhrases  
**Outputs:**

| Field | Values |
|---|---|
| `HardshipFlag` | `Yes`, `No` |
| `HardshipType` | `Financial`, `Medical`, `Job Loss`, `Bereavement`, `Natural Disaster`, `None` |
| `HardshipSeverity` | `Critical`, `Moderate`, `Mild`, `None` |

---

### Agent 10 — Governance & Responsible AI (`GP_Governance`)
**Purpose:** Final policy and fairness gate before execution. Validates proportionality, non-discrimination, and Responsible AI policy compliance.  
**LLM:** GPT-5.4  
**Inputs:** RecommendedAction, HardshipFlag, HardshipType, HardshipSeverity, UnifiedRiskScore, RiskTier, CustomerType  
**Outputs:**

| Field | Values |
|---|---|
| `GovernanceDecision` | `Approved`, `Blocked`, `Escalation Required` |
| `AuditNote` | One-sentence audit trail entry |
| `FairnessFlag` | `Pass`, `Fail` |

---

### Best Offer Selection Agent (`GP_BestOffer_Selection`)
**Purpose:** Select and rank the top retention offer from the catalogue using a cost-weighted scoring formula.  
**LLM:** GPT-4o-2024-11-20  
**Scoring formula:**
```
Score = 0.60 × CostEfficiency + 0.40 × Satisfaction
CostEfficiency = 1 − (AvgCost ÷ MaxCost)
AvgCost = (CostLow + CostHigh) ÷ 2
```
Recommends the top-ranked offer with two fallbacks. Never invents offers or costs. Respects approval tiers.

---

## LLM Model Strategy

| Use case | Model | Rationale |
|---|---|---|
| High-reasoning agents (Governance, Economic, Behavioral, Simulation) | `gpt-5.4` | Frontier reasoning; large 128k context for complex multi-signal inputs |
| Structured scoring (Risk Scoring) | `gpt-4o-2024-05-13` | Deterministic arithmetic; 4k token budget sufficient |
| Sentiment / Offer selection | `gpt-4o-2024-11-20` | Proven at NLP classification and offer matching |

All agents run at **temperature 0** to ensure deterministic, reproducible outputs across identical inputs.

LLM calls are routed via **UiPath AgentHub** (`agenthub_config = "agentsruntime"`), providing centralised API key management, rate-limit handling, and call logging.

---

## Responsible AI & Governance

GuardianPulse AI embeds Responsible AI as a first-class architectural concern rather than a post-hoc check.

**Hardship Protection (Agent 9 — Humanity Layer)**
Any signal of genuine customer hardship (financial crisis, bereavement, job loss, medical emergency, natural disaster) causes the system to suppress all commercial retention offers and flag the case for compassionate human support. This gate fires before the Manager Approval step.

**Fairness & Proportionality (Agent 10 — Governance)**
The Governance agent checks:
- Whether the recommended action is proportional to the risk score.
- Whether the customer's segment type (`CustomerType`) is being targeted in a discriminatory pattern.
- Whether a Hardship override applies.
Outputs a `FairnessFlag` (Pass / Fail) and an `AuditNote` that is written to the audit trail regardless of the decision.

**Human-in-the-Loop Gates**
No offer is delivered and no call is placed without a human observer validating the AI decision (Gate ①). No benefit is applied without a Retention Manager sign-off (Gate ②).

**Social Publishing Consent Gate**
The Social Advocacy / Publisher step is gated on explicit customer consent. No loyalty posts are published if consent is absent.

**Audit Logging**
`GP_Learning_Logger` records every intervention outcome — accepted, rejected, escalated, or blocked — to enable continuous model and policy improvement.

---

## Repository Structure

```
GuardianPulse_AI.uis                  ← Main solution archive (UiPath Solution)
│
├── Maestro BPMN/                     ← BPMN orchestration process
│   ├── Process.bpmn                  ← Stable production BPMN
│   ├── Process_new.bpmn              ← In-development BPMN
│   └── bindings_v2.json              ← Agent/activity bindings
│
├── GP_CRM_DataLoader/                ← RPA: CRM data fetch (Studio XAML)
├── Initiate_Call/                    ← RPA: AI Voice call initiation
├── Calling_API_Workflow/             ← RPA: Call state machine
├── GP_Calling_Retainer_API/          ← API Workflow: retention call wrapper
├── GP_Action_Executor/               ← RPA: benefit execution
├── GP_Learning_Logger/               ← API Workflow: outcome logging
│
├── GP_CRM_Intelligence/              ← Agent 1
├── GP_Sentiment_Analysis/            ← Agent 2 (low-code)
├── Coded copy of GP_Sentiment_Analysis/  ← Agent 2 (Python/LangGraph)
│   ├── main.py
│   ├── utils.py
│   └── pyproject.toml
├── GP_Behavioral_Risk/               ← Agent 3
├── GP_Revenue_Leakage/               ← Agent 4
├── GP_Intent_Prediction/             ← Agent 5
├── GP_Risk_Scoring/                  ← Agent 6
├── GP_Counterfactual_Sim/            ← Agent 7
├── GP_Economic_Decision/             ← Agent 8
├── GP_Humanity_Layer/                ← Agent 9
├── GP_Governance/                    ← Agent 10
├── GP_BestOffer_Selection/           ← Best Offer Selection Agent
│
├── resources/
│   └── solution_folder/
│       ├── entity/CustomerOffers.json
│       ├── bucket/orchestratorBucket/
│       │   ├── Customer_Data.json
│       │   └── Best_Offers.json
│       ├── index/                    ← Context grounding indexes
│       ├── appVersion/uiapp_FinalReview.json
│       └── connection/               ← Connector credentials (Google Sheets)
│
└── SolutionStorage.json              ← Solution manifest (project IDs)

AddOffersToEntities.uis               ← Standalone utility solution
└── AddOffersToEntities/
    ├── Main.xaml
    └── .entities/EntitiesStore.json  ← CustomerOffers entity definitions

uiapp_FinalReview.uiapp               ← Manager Approval UiPath App
```

---

## Prerequisites

### UiPath Platform
- UiPath Orchestrator (Cloud or On-Premises) with Maestro enabled
- UiPath Studio 2024.10 or later (for XAML workflows)
- UiPath Agent Builder (for low-code agents)
- UiPath Apps (for `uiapp_FinalReview`)
- UiPath Action Centre (for Gate ① human task)

### LLM / AI
- OpenAI API access with models: `gpt-5.4`, `gpt-4o-2024-11-20`, `gpt-4o-2024-05-13`
- Configured as an LLM connection in UiPath AgentHub under the `agentsruntime` config name

### Python (Coded Sentiment Agent only)
- Python 3.11+
- Dependencies declared in `Coded copy of GP_Sentiment_Analysis/pyproject.toml`:
  - `uipath-langchain`
  - `langchain-core`
  - `pydantic`

### External Systems
- CRM system with REST API (customer fetch + record update)
- AI Voice provider (configured in `Initiate_Call` workflow)
- Google Sheets connection (configured under `somnathmaitra1@gmail.com` connector — used for offer catalogue or logging)
- Social Media API (planned — `Advocacy / Social Publisher`)

### Orchestrator Resources
- Two Storage Buckets: `Customer_Data`, `Best_Offers`
- One Entity definition: `CustomerOffers`
- Two Indexes: `Customer_Data`, `BestOffer_CSV`
- Action Centre queue for Gate ① offer review tasks

---

## Deployment Guide

### 1. Publish Agents and Workflows

Open `GuardianPulse_AI.uis` in UiPath Studio. Publish all projects to Orchestrator in this order to respect dependencies:

1. `GP_CRM_DataLoader`
2. All AI agents (`GP_CRM_Intelligence`, `GP_Sentiment_Analysis`, etc.)
3. `GP_Risk_Scoring`, `GP_Governance`, `GP_Humanity_Layer`
4. `GP_BestOffer_Selection`
5. `Initiate_Call`, `Calling_API_Workflow`, `GP_Calling_Retainer_API`
6. `GP_Action_Executor`, `GP_Learning_Logger`
7. `Maestro BPMN`

### 2. Deploy the Utility Solution

Publish `AddOffersToEntities.uis` separately. Run `AddOffersToEntities` once (or on a schedule) to populate the `CustomerOffers` entity store from the CRM offer catalogue.

### 3. Deploy the UiPath App

Import `uiapp_FinalReview.uiapp` into UiPath Apps and publish it. Assign it to the Retention Manager role in Orchestrator.

### 4. Configure Connections

In Orchestrator → Connections:
- Add the OpenAI LLM connection and map it to AgentHub as `agentsruntime`.
- Configure the CRM REST connection used by `GP_CRM_DataLoader` and `GP_Action_Executor`.
- Configure the AI Voice provider connection used by `Initiate_Call`.
- Set up the Google Sheets connector if using Sheets for offer catalogue or reporting.

### 5. Create Orchestrator Resources

- Create Storage Buckets named `orchestratorBucket` containing `Customer_Data` and `Best_Offers` files.
- Import the Entity schema from `resources/solution_folder/entity/CustomerOffers.json`.
- Create the Action Centre queue for human offer review tasks (Gate ①).

### 6. Configure the BPMN Process

In Maestro, import the BPMN file and bind each activity to its published package using `bindings_v2.json` as reference. Set the trigger (scheduled or event-based on risk signal).

---

## Configuration Reference

### Agent Settings (common across all low-code agents)

| Setting | Value | Notes |
|---|---|---|
| `engine` | `basic-v2` | UiPath agent execution engine |
| `maxIterations` | `25` | Maximum reasoning steps per agent |
| `mode` | `standard` | Non-conversational, single-turn |
| `temperature` | `0` | Deterministic output |
| `isConversational` | `false` | All agents are single-turn |

### Risk Score Tier Thresholds

| Tier | Score Range |
|---|---|
| Critical | 85–100 |
| Very High | 70–84 |
| Medium | 40–69 |
| Low | 0–39 |

### Offer Scoring Weights (Best Offer Agent)

| Component | Weight |
|---|---|
| Cost Efficiency | 60% |
| Customer Satisfaction / Retention Lift | 40% |

---

## Testing & Evaluation

Each AI agent ships with a default evaluation set under `<agent_folder>/evals/`:

```
evals/
├── eval-sets/
│   └── evaluation-set-default.json    ← Test cases with expected outputs
└── evaluators/
    ├── evaluator-default.json          ← Output correctness evaluator
    └── evaluator-default-trajectory.json  ← Reasoning trajectory evaluator
```

The coded Sentiment agent additionally has evaluations under `evaluations/` using the UiPath Python agent evaluation format.

To run evaluations in UiPath Agent Builder: open the agent project → select **Evaluate** → choose `evaluation-set-default`. Results are scored against both output correctness and reasoning trajectory.

For regression testing across the full pipeline, run the BPMN with a representative set of synthetic customer records covering all risk tiers and hardship scenarios.

---

## Known Limitations & Roadmap

### Current Limitations
- **Social Advocacy Publisher** is marked `[RPA - planned]`; the workflow scaffold exists but the Social Media API integration is not yet implemented.
- The `GP_BestOffer_Selection` agent reads the offer catalogue from a static CSV bucket file. Real-time CRM catalogue sync depends on the `AddOffersToEntities` utility running on schedule.
- The coded Python Sentiment agent (`Coded copy of GP_Sentiment_Analysis`) is a development/alternative version. The low-code `GP_Sentiment_Analysis` is the production default.
- Agent evals use default evaluation sets; domain-specific test cases should be added before production rollout.

### Roadmap
- Live CRM offer catalogue streaming via REST (remove dependency on bucket file).
- Social Advocacy Publisher implementation.
- Expand `GP_Learning_Logger` to feed outcome data back into agent evaluation sets automatically.
- Add A/B testing capability in Counterfactual Simulation agent for offer experimentation.
- Senior Retention Manager escalation App view in `uiapp_FinalReview`.

---

## Contributing

1. Create a feature branch from `main`.
2. Open or modify the relevant agent project in Agent Builder or Studio.
3. Update or add evaluation cases in `evals/eval-sets/`.
4. Run evaluations and confirm no regression before raising a pull request.
5. Update `bindings_v2.json` if BPMN activity bindings change.

---

## Licence

Internal — GuardianPulse AI is a proprietary solution. All agent prompts, BPMN definitions, and workflow code are confidential.

---

*Built with UiPath Maestro · Agent Builder · Studio · Apps · AgentHub*  
*LLM Provider: OpenAI via UiPath (gpt-5.4 / gpt-4o)*  
*Solution ID: `27e51f02-8e47-4056-cb9b-08decdd794b8`*
