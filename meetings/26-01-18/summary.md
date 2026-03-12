# Meeting 2026-01-18

## Participants
- Kirill (supervisor) — guided discussion, pushed for concrete deliverables, shared research links
- Misha — presented technical findings on LiteLLM integration, streaming classifiers
- Sonya — presented architecture diagram (screen share), proposed narrowing to parental control use case

## Context
Second meeting after winter break. Kirill is frustrated that after a month of discussion there is still no working prototype. Pushes hard for concrete deliverables.

## Topics Discussed

### 1. Streaming Classifier Feasibility
Misha reported streaming safeguard classifiers are unavailable on OpenRouter and hard to find as hosted services. Kirill: not a blocker — can self-host a small model or run classification in a parallel thread post-response. If marginal value, drop it with benchmark justification.

### 2. System Architecture Walkthrough
Sonya presented: user routes traffic through the team's proxy URL (LiteLLM). Proxy intercepts prompts and responses, runs classifier, stores results in DB, sends Telegram alerts for flagged content. Daily summaries via bot. Kirill noted implicit assumptions not stated (protocol compatibility, agent-mode tool calls).

### 3. Target Audience — Parental Control Focus
Four segments identified: enterprises, app developers, non-technical home users, technical home users. Misha proposed focusing on **parental control** — parent installs system, child uses LLMs through it, parent receives alerts. Kirill agreed this is most compelling.

### 4. Safeguard Models and Provider Landscape
Relying on providers' built-in safety is insufficient — providers change, have varying safety levels. Chinese models (DeepSeek, MiniMax), French projects, Russian models all differ. The system should provide a provider-independent safety layer. Kirill shared links to OpenAI GPT-OS SafeGuard and RusTrust consortium.

### 5. Benchmarking and Datasets
Misha proposed building benchmark datasets (toxic content, sycophancy, hallucination). Kirill referenced: Toxic Chat, MB Guard, Anthropic's published dataset. Team must understand precision/recall, F-measure, Type I/II errors. Datasets likely on HuggingFace.

### 6. Adaptive Personalization
Misha proposed building user profiles from conversation history and generating custom system prompts to counteract detected patterns. Kirill: interesting but secondary to getting basic prototype running.

### 7. LiteLLM Integration Strategy
Two approaches: (a) fork and modify LiteLLM source, (b) use plugin/callback system. Kirill prefers (b). Misha noted LiteLLM has callbacks for nearly everything and built-in Postgres logging.

## Decisions
- Target audience narrowed to **home/family use case** (parental control) as primary focus
- Build a **working prototype first**, even if rudimentary
- Streaming classifier deprioritized — decision based on benchmark data
- Benchmark dataset creation is core workstream, not optional
- GitHub repository to be created

## Action Items
- **Misha:** Create GitHub repo; deep-dive into LiteLLM codebase (plugin/callback mechanisms); draw detailed architecture diagram; start building prototype
- **Sonya:** Research benchmark datasets (Toxic Chat, MB Guard, Anthropic); study evaluation metrics; review Kirill's shared links
- **Both:** Come to next meeting with concrete progress, not more abstract discussion
