# Meeting 2026-02-08

## Participants
- Kirill — supervisor. Demonstrates Langfuse evaluation capabilities, pushes for architecture diagrams and extensible design.
- Misha — technical student. Reports on LiteLLM proxy integration, middleware development, Langfuse logging.
- Sonya — PM student. Presents updated behavioral metrics table, shares research on Russian depression detection datasets.

## Context
The system now has a working proxy (LiteLLM as FastAPI + middleware). OpenWebUI is the frontend. Langfuse handles tracing. Focus on solidifying architecture, defining metrics formally, and designing extensible escalation.

## Topics Discussed

### 1. LiteLLM Architecture Refactoring
Misha abandoned Docker image approach. Now imports LiteLLM FastAPI app directly and wraps with custom middleware for Guardrails classifier. OpenWebUI connected as frontend chat interface.

### 2. Background Processing
Two options for periodic analysis: separate Docker container vs. embedded cron scheduler. Kirill: just use APScheduler within existing process. No separate infrastructure needed yet.

### 3. Escalation Strategy
Instead of raising errors (which might push vulnerable users to bypass the system), middleware should silently substitute the prompt or hardcode a helpful response. Kirill: design with extensibility — support error responses, prompt modification, and response substitution, switchable with minimal code changes.

### 4. Software Design Patterns
Kirill lectured on OOP principles for the middleware. Escalation behavior should follow strategy pattern — switching between behaviors requires changing a single line. One universal middleware with pluggable strategy, not chained middlewares.

### 5. Langfuse Integration and Sessions
All messages and classifier tags logged to Langfuse. Problem: no proper "session" concept linking related traces. Langfuse has sessions feature but neither has used it. Critical for Sonya's metrics.

### 6. Database Design
Need separate PostgreSQL tables for user profiles, computed metrics, historical risk data. Options: reuse Langfuse DB, LiteLLM user tables, or create separate tables. Team assigned to design data model.

### 7. Behavioral Metrics (Sonya)
Updated table with descriptions and risk associations. Kirill validated each against concrete data model attributes. Challenged some metrics (e.g., "time between question and response" reflects model speed, not user behavior). Linguistic pattern metrics may each require separate algorithms — use heuristics and LLM-based classification where possible.

### 8. Datasets
Sonya found 2021 Russian-language dataset of depressive social media posts. Would need conversion to synthetic chat-style conversations. Jailbreak datasets exist but lack Russian examples.

### 9. Langfuse as Experiment Platform (Kirill Demo)
Kirill demonstrated Langfuse beyond tracing: dataset management, prompt versioning, experiment runs with scored metrics. Showed fraud-detection experiment results using local models. No need for ClearML/MLflow.

## Decisions
1. No separate Docker container for background processing — use in-process scheduler
2. No error-raising for harmful content — substitute prompts/responses silently
3. Middleware must use strategy pattern for extensible escalation
4. Langfuse for experiment management (replaces ClearML discussion)
5. Separate database for user profiles and computed metrics
6. Session concept needs formal definition

## Action Items
- **Both:** Design complete system architecture + sequence diagram (1-3 weeks)
- **Misha:** Implement extensible middleware with pluggable escalation strategies; investigate Langfuse sessions
- **Sonya:** Define formal algorithms/formulas for each behavioral metric; evaluate found datasets; read Kirill's experiment results document
- **Both:** Design data model (tables, attributes, relationships) for user profiles, sessions, metrics
