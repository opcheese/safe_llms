# Meeting 2026-01-25

## Participants
- Kirill (supervisor) — directed discussion, shared metric prioritization document
- Misha — demonstrated LiteLLM Docker Compose deployment with custom guardrail class
- Sonya — presented dataset research findings

## Context
Follow-up to Jan 18 meeting. Kirill shared a document shifting metric priorities toward simpler behavioral metrics. First live demo of LiteLLM integration.

## Topics Discussed

### 1. Metric Prioritization Shift
Kirill's document advocates prioritizing simpler behavioral metrics (session duration, time of use, frequency) over complex content-based ones. These are easier to implement and more informative for detecting concerning patterns.

### 2. Sonya's Dataset Research
Found 4 datasets: NVIDIA safety set, Toxic Chat, two Reddit-based datasets. Kirill pointed to additional resources: Awesome Safety Tools, Anthropic Peril, and Safety Agent resources on GitHub.

### 3. Misha's LiteLLM Demo
Docker Compose deployment with custom guardrail class calling OpenAI SafeGuard as binary classifier in pre-call mode. Message logging was broken (known LiteLLM bug). First working proof-of-concept of the proxy pipeline.

### 4. Experiment Pipeline Design
Baseline comparison approach: raw LLM responses vs. system-filtered responses. ClearML discussed for experiment tracking. Cost optimization via using existing dataset responses instead of making new API calls.

### 5. Architecture Beyond Guardrails
Need for cron-based background analysis (nightly batch processing of accumulated conversations). Custom database needed for behavioral metrics beyond what LiteLLM provides.

### 6. Meeting Transcription
Sonya records meetings with OBS. Needs to learn transcription tooling for meeting documentation.

### 7. Repository Setup
GitHub repository access needs to be shared with all team members.

## Decisions
- Prioritize simple behavioral metrics first
- Use open-source models as baseline for comparison
- Evaluate ClearML for experiment tracking
- Use LangFuse/hooks for logging

## Action Items
- **Misha:** Fix logging issues, continue LiteLLM integration, share repo access
- **Sonya:** Deep-dive into found datasets, study evaluation metrics
- **Both:** End of February deadline — runnable system that processes a dataset through LiteLLM and produces measurable results
