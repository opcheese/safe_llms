# Meeting 2026-02-22

## Participants
- **Kirill** (supervisor/senior lecturer) — guides discussion, asks probing questions, sets priorities
- **Misha** (technical student) — presents working prototype, shares research on benchmarks
- **Sonya** (PM student) — raises UX concerns about multiple interfaces, works on metrics/diagrams

## Context
Thesis project on AI safety: a system built on LiteLLM that monitors user interactions with LLMs, collects behavioral metrics, detects harmful dependency patterns, and escalates critical situations to administrators. The target deployment scenario is educational institutions (universities, schools) providing students with supervised LLM access.

## Topics Discussed

### 1. Working Prototype Demo (Misha)
Misha demonstrated the current state of the system:
- **LiteLLM** migrated to WSL with database support, enabling user registration and per-user tracking (sessions, messages, limits)
- **Open WebUI** integrated, passing chat IDs downstream to Langfuse for session tracking
- **Classifiers** built using LLM prompts — currently classifying: self-harm, psychosis, delusion, and anthropomorphization. Each returns a boolean label + confidence score
- **Scheduler** (cron-based) periodically scrapes all user sessions from Langfuse, reconstructs full dialogues per chat, and runs them through the classifier. Results are stored in the database with append-only logging
- Live demo showed self-harm classification with 0.95 confidence on a test conversation about a fictional "quadro bulper"

### 2. Confidence Score Reliability
Kirill questioned how confidence values are elicited from the LLM — noting that without careful prompt engineering, models produce arbitrary numbers. Misha acknowledged this is imperfect but argued it correlates with reality in practice. Both agreed it needs refinement but works as an MVP.

### 3. Escalation Pipeline Design
Discussion on what happens after classification:
- When a red flag is detected, the system should alert an administrator (e.g., school director) via a messaging channel (Telegram bot planned)
- Kirill pushed on the decision logic: should escalation trigger on any single flagged session, an average across sessions, or a 95th percentile threshold?
- Misha proposed escalating on any red flag occurrence
- Kirill noted legal considerations around message privacy that organizations would need to handle

### 4. Deployment Scenario and Architecture
Kirill walked through the end-to-end user story: an institution deploys the system via Docker, creates API keys per student in LiteLLM, distributes keys. Students use Open WebUI with their keys; the system monitors silently and escalates to administrators. LiteLLM internals are never exposed to end users.

Discussion of inference costs: Misha suggested serverless GPU providers (RunPod-style), Kirill noted that university budgeting favors one-time hardware purchases over recurring cloud costs.

### 5. UX Concerns — Multiple Interfaces (Sonya)
Sonya flagged that the current architecture exposes three separate interfaces to users: Open WebUI, LiteLLM admin panel, and the Telegram alert bot. Kirill reframed this as reasonable role separation (admin interface vs. user interface vs. alert channel) and deprioritized the concern for now, noting it matters more for a "home use" scenario than institutional deployment.

### 6. Metrics and User Profile
Kirill pressed hard on metrics — asking Sonya to specify for each metric: which database table, which field, what query extracts it. Key points:
- Temporal/quantitative metrics (session duration, frequency) are straightforward but need precise data source mapping
- Behavioral/linguistic pattern metrics require more work
- **User profile** concept introduced as central: a structured record per user aggregating all metrics, session history, and behavioral patterns
- Profiles serve three purposes: (a) enriching LLM prompts with personalization context (citing the RACE framework — up to 43% improvement in safety benchmarks), (b) enabling rule-based limits (e.g., "max 1 hour LLM use per day"), (c) giving administrators data to define healthy vs. unhealthy usage patterns

### 7. Benchmarks and Research (Misha)
Misha presented two relevant benchmarks:
- **HyperSearch** — agent framework showing personalization improves safety metrics by 42-43%; includes the **Penguin** benchmark (14K scenarios) and the **RACE** approach (model proactively asks users for context)
- A second benchmark closely aligned with the project's classification categories (psychosis, delusion, escalation, harmful advice)
- All benchmarks use synthetic users (LLM-as-user talking to LLM-as-assistant) evaluated by LLM judges
- Kirill noted these benchmarks can't be used one-to-one since the project is a multi-component system, not a single model — they need adaptation

### 8. Thesis Chapter 1 — Literature Review
Kirill gave detailed guidance on writing the first chapter (due March 13):
- The chapter must argue *why* the work is needed, not just describe the domain
- **Misha's angle**: existing LLM safety mechanisms (OpenAI, Anthropic built-in guardrails) are insufficient because they don't escalate externally — the chapter should prove that a full escalation system is necessary
- **Sonya's angle**: LLMs lack user profiling and behavioral metrics — the chapter should prove that a user profile system is necessary for safe institutional LLM deployment
- Chapters will overlap but diverge on the second chapter's focus
- Kirill urged them to submit drafts well before the deadline to allow iteration

## Decisions
- **Escalation trigger**: any single red-flag classification triggers an alert (simplest approach for now)
- **The system is a general framework** — legal/privacy concerns are left for deploying organizations to handle
- **User profile** is the next key architectural concept to define — a concrete list of fields that describe a user
- **Starting next week**: switching from joint meetings to individual 1-on-1 sessions (two separate hour-long slots)
- **Benchmarks**: will evaluate using Penguin and potentially adapt existing benchmarks for the multi-component system

## Action Items
- **Misha**: Complete the escalation pipeline (classifier -> threshold decision -> alert via Telegram). Continue improving classifiers. Write Chapter 1 draft focusing on why external escalation systems are needed. Submit draft before March 13, ideally by next meeting
- **Sonya**: Define the user profile schema — concrete fields, tables, and queries for each metric. Write formulas for how each metric is computed from the database. Write Chapter 1 draft focusing on why user profiling/metrics are needed for safe LLM use. Submit draft before March 13, ideally by next meeting
- **Both**: Coordinate on Chapter 1 since there is shared content. Decide between themselves on preferred meeting time slots (2pm or 4pm)
