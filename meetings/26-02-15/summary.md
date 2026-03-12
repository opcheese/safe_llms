# Meeting 2026-02-15

## Participants
- Misha (technical lead) — presented current architecture status, discussed datasets and classification approach
- Sonya (PM) — drove the data model / diagram work, coordinated action items

Kirill Aleksandrovich (supervisor) was **not present**; the meeting was a working session between Misha and Sonya preparing materials for the next supervisor review on Feb 21.

## Context
The project is an AI safety system built on top of LiteLLM. It acts as a proxy between users and target LLMs (via OpenRouter), monitoring conversations for harmful dependency, mental health risks, and destructive behavior. The system collects behavioral metrics, classifies messages, and escalates critical situations to an admin (e.g., a parent monitoring a child's LLM usage). Langfuse is used for logging all interactions, and a scheduler periodically analyzes conversation history.

## Topics Discussed

### 1. Architecture / Data Model Diagram
Sonya and Misha walked through the entire system architecture to build a data model diagram (in draw.io). They mapped out all components and their interactions:
- **Open Web UI** — user-facing chat interface
- **LiteLLM Proxy** — central application that proxies requests, runs custom logic, handles auth via tokens
- **Target LLM** — external model accessed via OpenRouter (not a separate microservice, just a config parameter in LiteLLM)
- **Langfuse** — log storage for all messages/sessions (used because native LiteLLM DB logging was broken)
- **Database (BD)** — stores user data, classification results, metrics
- **Input Classifier** — real-time classifier that checks each user message on input; runs as part of the proxy pipeline
- **Daily Classifier** — periodic classifier that analyzes full conversation history (not yet implemented)
- **Scheduler (cron)** — background task that pulls data from Langfuse, feeds it to the daily classifier, writes results to DB
- **Alert Bot** — Telegram bot for admin notifications (not yet implemented)
- **LiteLLM UI** — admin interface for creating users and API tokens

Data flow: Open Web UI -> LiteLLM Proxy -> (check user in DB) -> Input Classifier -> Target LLM -> response logged to Langfuse on return path. The scheduler separately reads from Langfuse and feeds data to the daily classifier.

### 2. User Roles and Authentication
- Two roles: admin and regular user, stored in a single users table with a role field
- Admin creates users in LiteLLM UI and distributes API tokens
- Each request is authenticated by token
- Use case: parental control scenario where admin = parent, users = children

### 3. Alert Bot Design
- Input classifier triggers immediate alerts for critical messages (e.g., suicidal ideation)
- Daily classifier produces periodic summary reports
- Alert frequency should be configurable by admin (not spam on every message)
- Bot should serve as a unified entry point with links to Langfuse (view messages) and LiteLLM UI (manage users)

### 4. Prompt Modification as Core Feature
Misha argued the system's unique value is **dynamic prompt personalization**: the system analyzes user history and automatically configures safety-oriented system prompts tailored to the individual user's psychological profile. This is something large LLM providers won't do due to privacy/legal risks, making it the project's competitive advantage. Sonya noted this may raise ethical questions that Kirill will likely push back on.

### 5. Datasets and Benchmarking
- Existing public datasets are too small (single-message exchanges); the project needs long multi-turn conversations grouped by user ID
- Goal: benchmark the system's safety improvements with concrete metrics — compare old vs. new models, with and without the proxy system
- Without a proper dataset, classifier quality cannot be validated

### 6. Langfuse Justification
Langfuse remains in the stack because native LiteLLM log storage didn't work (confirmed by community reports). While it adds an extra component, replacing it would require significant effort. It also serves as tamper-proof logging — users can delete chats in Open Web UI, but Langfuse retains originals.

## Decisions
- The data model diagram will be finalized in draw.io and presented to Kirill on Feb 21
- Alert bot will receive messages from both the input classifier (immediate alerts) and the scheduler (periodic summaries)
- Alert frequency should be configurable by the admin
- Alert bot will include links to Langfuse and LiteLLM UI as a unified admin interface
- Langfuse stays in the architecture as the primary log storage
- All classifiers run on LLM — whether to use one combined prompt or separate per-class prompts remains open (needs dataset to benchmark)

## Action Items
- **Sonya**: finalize the data model diagram in a clean format; prepare a second diagram variant showing the alert bot as a unified admin interface with links to Langfuse and LiteLLM UI; write the daily classifier prompt text (Misha to provide a template/structure)
- **Misha**: provide a prompt template for the daily classifier; search for large multi-turn conversation datasets suitable for benchmarking; continue refining the architecture
- **Both**: prepare materials for the Feb 21 meeting with Kirill — current system vision, data model, implementation plan, and task assignments
- **Open (for Kirill review)**: discuss the ethical implications of dynamic prompt modification; validate the overall approach and roadmap
