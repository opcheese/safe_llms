# Meeting 2026-02-01

## Participants
- Kirill — supervisor. Guides discussion, asks probing questions, pushes for architecture diagrams and concrete deliverables.
- Misha — technical student. Reports on LiteLLM integration progress, proposes middleware-based architecture.
- Sonya — PM student. Reports on dataset research, proposes new metrics.

## Context
Building the AI safety proxy on LiteLLM. Langfuse adopted for tracing/logging. Focus shifting from conceptual design to concrete implementation.

## Topics Discussed

### 1. LiteLLM Integration Approach
Misha reported that LiteLLM Docker + native hook system is painful (outdated docs, debugging difficulties). Discovered LiteLLM can be imported directly as a FastAPI app, giving full middleware access. Kirill accepted this approach — simpler and easier to debug.

### 2. Escalation Mechanism
When classifier flags a critically harmful prompt (red alert), how to return a hardcoded safety message instead of forwarding to LLM? Middleware-based interception preferred — return custom response without passing request further. Kirill asked Misha to verify in LiteLLM source how hooks handle return values.

### 3. Two Categories of Metrics
- **Category 1 — Behavioral/temporal (no NLP):** Time of day, session duration, frequency, night-time usage. Computable with simple heuristics. Start here.
- **Category 2 — Semantic/content (NLP required):** Topic classification, sentiment, multi-stage analysis. Requires guard model or LLM. Harder to validate.

Kirill: prioritize by (ease of implementation) × (expected impact). Run candidates against user personas first.

### 4. Dataset Research (Sonya)
Existing datasets contain mostly obvious harmful prompts; the "grey zone" is underrepresented. Response-side dangers are rare. All available datasets are English-only. Options: find Russian datasets, translate English ones, reach out to Russian organizations (Sberbank, Yandex).

### 5. Testing and Validation
Kirill: system architecture must be designed first ("rectangles"), otherwise nothing concrete to test. End-to-end testing with real users needs 20-30 people minimum. Synthetic personas acceptable if justified with psychology literature.

## Decisions
1. Use LiteLLM as FastAPI app with custom Python middleware (not Docker hooks)
2. Middleware intercepts red-flag prompts and returns hardcoded safety message directly
3. Real-time classifiers on prompts only; response analysis via async cron job
4. Start with behavioral/temporal metrics before semantic ones
5. Working system first, testing second; architecture diagrams are prerequisite for both

## Action Items
- **Misha:** Draw complete system architecture diagram (all components, middleware, classifiers, DB, Langfuse, cron, escalation flow). Deadline: before Feb 14.
- **Sonya:** Prepare prioritized metrics list. Run candidates against user personas. Search for Russian-language AI safety datasets.
- **Both:** Sync on data model — what metrics, how computed, where stored, when recalculated.
- **Misha:** Verify in LiteLLM source how pre-call hooks handle return values.
