# Meeting 2026-01-17

## Participants
- Misha (Speaker 0) — technical student, drove the discussion on architecture and implementation
- Sonya (Speaker 1) — PM student, raised UX and product concerns, asked clarifying questions on metrics

Kirill (supervisor) was absent; they plan to present their architectural decisions to him the next day.

## Context
A working session between Misha and Sonya to align on architecture and product direction for the AI safety proxy system. The system sits between users and LLM providers (via LiteLLM/OpenRouter), monitors conversations for harmful patterns, and escalates critical situations. Kirill is expected to review their proposals at the next meeting.

## Topics Discussed

### 1. Real-time (streaming) content filtering limitations
Misha explained that streaming classifiers are limited: modern LLMs already have built-in safety, and a token-level classifier cannot detect nuanced issues like sycophancy or hallucinations in real time. The only viable streaming protection is simple regex-based filtering (profanity, obvious stop-words). More sophisticated safety models he found cannot be prompted and cover only 6-10 narrow classes. He also floated the idea of zeroing out token probabilities for bad words, but acknowledged this is impossible through provider APIs since the system is a proxy.

### 2. Post-session batch analysis as the primary safety mechanism
Both agreed the main safety analysis should happen after a session ends. Conversation logs will be sent to a classifier (prompted LLM or dedicated safety model) that evaluates the full dialogue for risks: self-harm indicators, sycophancy, hallucinations, excessive anthropomorphization, etc. This avoids the technical limitations of streaming and allows richer analysis.

### 3. Metrics and classification approach
Misha outlined a two-tier classification strategy:
- **Binary classifier first** (safe / unsafe) to get a baseline — e.g., "our binary classifier works at 70%."
- **Multi-class breakdown second** (self-harm, hallucination, sycophancy, etc.) to identify which risk categories underperform.
- Standard classification metrics: precision, recall, F1-score per class.

### 4. User questionnaire — dropped in favor of behavioral analysis
Misha initially proposed a user intake questionnaire to customize the system prompt. Sonya pushed back strongly:
- Questionnaire results go stale quickly.
- Users skip questionnaires, hurting conversion.
- The system should be safe by default without requiring user self-reporting.

They agreed to drop the questionnaire and instead derive user risk profiles from accumulated session data. Sonya proposed using simple toggleable filters/settings (e.g., "hide 18+ content") instead.

### 5. Deployment model — fully local, open-source
The system will be distributed as an open-source project on GitHub. Users clone the repo and deploy locally via Docker. No centralized server, no user data stored by the team. This sidesteps all legal/privacy concerns for the thesis stage.

### 6. Alert delivery mechanism
- Telegram bot considered but rejected as too niche.
- Email alerts are the likely primary channel.
- Users can optionally create their own Telegram/VK bot and paste the token into config.
- Sonya suggested daily session summaries (session duration, message count, risk events).

### 7. LLM provider choice
OpenRouter selected as the unified provider for accessing multiple models. Need to check batch API support for cost reduction.

## Decisions
1. **Streaming safety: regex only.** Real-time filtering limited to regex-based stop-words and profanity.
2. **Primary safety: post-session analysis.** Full session logs analyzed in batch by a prompted LLM or safety classifier.
3. **No user questionnaire.** System must be safe by default; risk profiles built from behavioral data over time.
4. **Deployment: local Docker, open-source.** No centralized infrastructure.
5. **Provider: OpenRouter.** Unified access to all major LLM providers.
6. **Alert delivery: email + optional bot.**

## Action Items
- **Sonya:** Write up metrics definitions — name, meaning, formula for each classification metric.
- **Sonya:** Finish the three use-case diagrams.
- **Misha:** Research more safety/classifier models; investigate promptable options for post-session analysis.
- **Misha:** Check OpenRouter batch API support and pricing.
- **Both:** Prepare architecture summary to present to Kirill at the next meeting.
