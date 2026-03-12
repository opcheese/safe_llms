# Thesis Project Timeline

**Project:** AI Safety System for LLM Interactions
**Supervisor:** Kirill (senior lecturer)
**Students:** Misha (technical/escalation), Sonya (PM/metrics)
**Period:** November 2025 — March 2026

---

## Phase 1: Conceptual Foundation (November 2025)

### Nov 2 — Topic Selection and Team Formation
First meeting. Three interconnected thesis topics defined within Kirill's larger agent system architecture:
1. Content safety classification (guardrails)
2. User behavior metrics (interaction patterns)
3. Escalation (alerting when danger detected)

Misha chose escalation. Sonya was undecided (considered an art-related topic). Team agreed to work together on a shared project rather than in isolation.

Core motivation: real cases of people forming unhealthy attachments to LLMs, bypassing safety measures by switching providers. Need for a client-side safety layer independent of any LLM provider.

### Nov 16 — User Profiling and Gray Zones
Deep dive into diagnostic approaches. Two competing methods:
- Questionnaire-based scoring (like psychological tests)
- Financial/material harm measurement (concrete, harder to game)

Kirill presented himself as a gray-zone case study — heavy AI user, no obvious harm, but some concerning patterns. Key insight: **most users fall in the gray zone**, not the clear red/green extremes. Sonya started writing user personas by risk zone.

Decision: will NOT pursue "provocation testing" (ethically too dangerous).

### Nov 23 — Classification Deep Dive
Misha presented QwenGuard classification pipeline (input guard + output guard, 3 model sizes). Team designed a **tiered classification approach**: cheap filters first (regex, embeddings), escalate to heavier models on threshold.

Classification methods ranked: LLM-based (best zero-shot, expensive) > encoder-based (fast, needs fine-tuning) > traditional NLP (regex, exact match). Russian NLP tools identified: Natasha, DeepPavlov, SpaCy, PyMorph2.

Key framing: project focuses on **protecting users from AI** (dependency, anthropomorphization), not jailbreak prevention — an underserved niche.

### Nov 30 — Architecture and Strategy
Three deployment scenarios defined:
1. **Organizational** (LiteLLM integration) — primary target
2. **Open-source library** for self-hosted LLM providers
3. **Personal assistant** safety layer

All share ~90% code; differ only in integration point.

Misha proposed de-anthropomorphization via a second LLM that rewrites responses in robotic tone. Streaming problem identified (breaks token-by-token delivery). Cheaper alternative: anti-anthropomorphization instructions in system prompt.

Petra framework (Anthropic) identified as closest existing work — can be used to evaluate the team's own safety system.

Decisions: LiteLLM integration as primary target. Batch processing of daily logs acceptable for deeper metrics. Escalation to third parties without consent likely off-limits except in institutional/parental contexts.

---

## Phase 2: Technical Foundation (December 2025 — January 2026)

### Dec 14 — Start Coding
Kirill framed the system as **sensor + actuator**: sensor analyzes conversations and classifies state; actuator takes action (modifies prompts, sends alerts, escalates).

LiteLLM formally chosen as the central proxy. Plan: extend LiteLLM with safety modules via its built-in guardrails hooks and logging integrations.

Kirill stepped back for December — students to self-organize and come back with a working prototype.

### Jan 17 — Students' Working Session (No Kirill)
Misha and Sonya aligned on architecture independently:
- **Streaming safety limited to regex** — sophisticated analysis impossible in real-time
- **Post-session batch analysis** is the primary safety mechanism
- **Questionnaire dropped** — Sonya pushed back hard (goes stale, users skip it, system should be safe by default)
- Risk profiles built from accumulated behavioral data instead
- Deployment: **local Docker, open-source** (sidesteps privacy/legal concerns)
- Provider: **OpenRouter** for unified multi-model access
- Alerts: email + optional Telegram/VK bot

### Jan 18 — Kirill Pushes for Prototype
Kirill frustrated by lack of working code after months of discussion. Parental control narrowed as the primary use case (parent installs, child uses LLMs through it, parent receives alerts).

Streaming classifier deprioritized. Benchmark dataset creation declared a core workstream. Students told to come back with concrete progress.

### Jan 25 — First Working Demo
Misha demonstrated LiteLLM Docker Compose deployment with custom guardrail class calling OpenAI SafeGuard as binary classifier. First working proof-of-concept.

Kirill shifted metric priorities toward **simpler behavioral metrics** (session duration, time of use, frequency) over complex content-based ones. Sonya found 4 datasets (NVIDIA safety set, Toxic Chat, Reddit-based).

Architecture beyond guardrails discussed: need for cron-based background analysis and custom database for behavioral metrics.

---

## Phase 3: Implementation and Refinement (February — March 2026)

### Feb 1 — Middleware Architecture
Misha abandoned Docker image approach. Now imports LiteLLM as a FastAPI app directly, wrapping with custom middleware for the guardrails classifier. OpenWebUI connected as frontend.

Two metric categories formalized:
1. **Behavioral/temporal** (no NLP): time of day, session duration, frequency — start here
2. **Semantic/content** (NLP required): topic classification, sentiment — harder, later

Prioritization formula: (ease of implementation) x (expected impact).

### Feb 8 — Extensible Design
Kirill lectured on OOP principles. Escalation behavior should follow **strategy pattern** — switching between behaviors (error response, prompt modification, response substitution) requires changing a single line.

Kirill demonstrated **Langfuse** beyond tracing: dataset management, prompt versioning, experiment runs with scored metrics. Langfuse replaces ClearML for experiment management.

Decisions: in-process scheduler (APScheduler, no separate Docker container), silent prompt/response substitution (no error-raising for harmful content), separate database for user profiles and computed metrics.

### Feb 15 — Students' Working Session (No Kirill)
Sonya and Misha built the data model diagram in draw.io. Full component mapping:
- Open WebUI → LiteLLM Proxy → Input Classifier → Target LLM (via OpenRouter)
- Langfuse for log storage (native LiteLLM DB logging was broken)
- Scheduler pulls from Langfuse, feeds daily classifier, writes to DB
- Alert bot (Telegram) for admin notifications

Misha argued the system's unique value is **dynamic prompt personalization** based on user psychological profile — something large providers won't do due to privacy/legal risks.

Dataset gap identified: existing datasets are single-message; project needs long multi-turn conversations grouped by user.

### Feb 22 — Working Prototype with Classifiers
Major milestone. Misha demonstrated:
- LiteLLM on WSL with database support, user registration, per-user tracking
- OpenWebUI integrated, passing chat IDs to Langfuse for session tracking
- LLM-based classifiers for: self-harm, psychosis, delusion, anthropomorphization (boolean + confidence)
- Scheduler scraping sessions from Langfuse, reconstructing dialogues, running classifiers
- Live demo: self-harm classification with 0.95 confidence on test conversation

**User profile** concept formalized as central: structured record per user aggregating all metrics. Profiles serve three purposes: (a) enriching LLM prompts with personalization (citing RACE framework — up to 43% improvement), (b) enabling rule-based limits, (c) giving administrators behavioral data.

Kirill gave Chapter 1 guidance: Misha must argue existing guardrails don't escalate; Sonya must argue user profiling is necessary for safe institutional deployment.

Decision: switching from joint meetings to individual 1-on-1 sessions.

### Mar 8 — Thesis Writing and Differentiation
Extended guidance on VKR (qualifying thesis) writing: introduction structure, citation requirements, anti-plagiarism considerations.

Key strategic decisions crystallized:
- **Misha's differentiator**: longitudinal multi-session analysis — existing guardrails operate per-message, but real crises develop across weeks/months. Found a longitudinal early-risk-detection dataset for benchmarking single-session vs. multi-session classification.
- **Sonya's differentiator**: cross-type metric correlation — combining temporal + behavioral + contextual signals (e.g., daytime study queries + nighttime crisis queries = elevated risk when correlated).
- **Focus on open-source LLMs** — proprietary models already have decent safety; open-source models (Qwen, LLaMA) significantly lag on safety benchmarks.

Deadline: March 13 — submit introduction + Chapter 1 for internal evaluation.

---

## Current State (as of March 2026)

### What Exists
- **Working prototype**: LiteLLM proxy + OpenWebUI frontend + Langfuse logging + LLM-based classifiers + cron scheduler
- **Classifiers**: self-harm, psychosis, delusion, anthropomorphization (LLM-prompted, boolean + confidence)
- **Infrastructure**: Docker-based deployment, user registration, per-user session tracking
- **Architecture**: fully mapped in draw.io (data model diagram)

### What's In Progress
- Chapter 1 drafts (both students, due March 13)
- Misha: longitudinal multi-session classification benchmark
- Sonya: formal metric definitions and aggregation formulas
- Escalation pipeline: classifier → threshold decision → Telegram alert

### What's Not Started
- Daily classifier (periodic full-conversation analysis)
- Telegram alert bot
- Dynamic prompt personalization based on user profiles
- Formal user profile schema implementation
- Dataset creation/adaptation for benchmarking
- De-anthropomorphization response rewriting

### Key Architecture

```
User → Open WebUI → LiteLLM Proxy → Target LLM (via OpenRouter)
                         ↓                    ↓
                   Input Classifier      Response logged
                         ↓                    ↓
                    [block/modify]         Langfuse
                                              ↓
                                    Scheduler (APScheduler)
                                              ↓
                                      Daily Classifier
                                              ↓
                                     PostgreSQL (metrics)
                                              ↓
                                    Alert Bot (Telegram)
                                              ↓
                                         Admin
```

### Key Decisions Log
| # | Decision | Meeting |
|---|----------|---------|
| 1 | Focus on protecting users from AI, not jailbreak prevention | Nov 23 |
| 2 | LiteLLM as central proxy platform | Dec 14 |
| 3 | Parental/institutional control as primary use case | Jan 18 |
| 4 | Questionnaire dropped — behavioral data instead | Jan 17 |
| 5 | Post-session batch analysis as primary safety mechanism | Jan 17 |
| 6 | FastAPI middleware approach (not Docker hooks) | Feb 1 |
| 7 | Strategy pattern for extensible escalation | Feb 8 |
| 8 | Langfuse for experiment management (replaces ClearML) | Feb 8 |
| 9 | Open-source LLMs as target (not proprietary) | Mar 8 |
| 10 | Longitudinal multi-session analysis as key differentiator | Mar 8 |

### Thesis Structure
- **Misha's thesis**: Escalation system for AI safety — argues existing guardrails don't escalate externally; differentiates via longitudinal multi-session analysis using open-source safety tools
- **Sonya's thesis**: Behavioral metrics and user profiling for safe LLM deployment — argues user profiling is necessary; differentiates via cross-type metric correlation (temporal + behavioral + contextual)
- **Shared**: LiteLLM infrastructure, OpenWebUI frontend, Langfuse logging, classifier framework
