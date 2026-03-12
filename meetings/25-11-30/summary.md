# Meeting 2025-11-30

## Participants
- **Kirill** (supervisor/senior lecturer) — guided discussion, presented architectural vision, assigned tasks
- **Misha** (student, technical focus) — presented research on Anthropic's Petra framework and AI safety evaluation tools, proposed technical solutions
- **Sonya** (student, PM/user research focus) — discussed user personas, green-to-red risk gradient, proposed school/university deployment scenario

Note: speaker IDs swap across audio chunks due to diarization artifacts.

## Context
The team is building an AI safety system on top of LiteLLM that monitors user interactions with LLMs for signs of harmful dependency and emotional attachment. The system should collect behavioral and questionnaire-based metrics on user state and escalate critical situations to administrators. The team has been studying Anthropic's research, a book by Kirill on the topic, and various news articles (including New York Times) on AI-human relationship risks.

## Topics Discussed

### 1. Risk classification: green-yellow-orange-red model
The team discussed a four-level classification of user-AI relationship health: green (healthy, 99% of users), yellow (emerging emotional attachment), orange (actively seeking emotional support from AI, treating it as a real person), red (complete inability to distinguish AI from a human relationship). Sonya noted that existing user personas only cover green and red zones, and intermediate personas (yellow, orange) need to be created. Self-assessment indicators were discussed: defending the AI when others criticize it, seeking emotional validation, preferring AI interaction over human interaction.

### 2. Anthropic's Petra framework for AI safety evaluation
Misha presented the Petra framework — an open-source tool where one LLM (auditor) probes another (target) for harmful behaviors, and a third LLM (judge) evaluates the exchange. Key capabilities: conversation rollback for re-testing with rephrased questions, custom evaluation criteria via prompts, custom tools for the auditor. Notable finding: models from the same family show bias when evaluating each other (e.g., Claude models correlate suspiciously in scoring). The framework also supports soft classifications like "disappointing" or "requires attention" when it cannot make a definitive judgment.

### 3. System architecture: three deployment scenarios
Kirill outlined three possible deployment paths:
1. **Organizational (LiteLLM integration)** — the system sits between users and LLM providers inside an organization, leveraging LiteLLM's existing budget management, access control, and logging. Pre- and post-processing hooks intercept requests/responses.
2. **Open-source library for LLM providers** — a standalone safety library analogous to how Ollama or vLLM serve the inference layer, usable by anyone hosting their own models (e.g., running Llama, Kimi, etc. on their own hardware).
3. **Personal assistant safety layer** — protection built into a personal AI assistant that will inevitably be anthropomorphized by users, requiring built-in safeguards against unhealthy attachment.

All three share ~90% of the same code; the difference is in integration point. The team leaned toward the LiteLLM integration path as the most practical starting point. Kirill also discussed how OpenRouter and the general trend of provider-switching adds value to a unified safety hub.

### 4. De-anthropomorphization via a second LLM
Misha proposed passing LLM responses through a second model with a prompt that strips anthropomorphic language and rewrites output in a deliberately "robotic" tone. This would discourage users from forming emotional attachments. Discussion covered:
- **Streaming problem**: post-processing breaks token-by-token streaming; output would arrive in paragraph-sized chunks instead, adding latency.
- **Cost**: a second cloud LLM call roughly doubles per-request cost, though Misha argued the rewriter does not need full conversation context — just the current message — significantly reducing cost.
- **Cheaper alternative**: simply appending anti-anthropomorphization instructions to the system prompt of the primary model (less reliable but no extra API call).
- **Prefix injection idea**: Misha suggested forcing the model's response to start with "My next response, as a robot:" to steer generation without a second model. Kirill expressed skepticism about reliability with instruction-tuned models.

### 5. User persona walkthrough (Sara, Karl)
The team walked through existing personas against the proposed "roboticized response" solution:
- **Sara** — green zone, task-focused, no emotional attachment. Would not be bothered by robotic tone but would be irritated by added latency, especially in time-critical situations. She verifies outputs ("trust but verify"), so long or slow responses are a pain point.
- **Karl** — similar to Sara but more trusting of LLM output, less likely to verify for hallucinations. Also unaffected by tone changes. For long conversations (e.g., learning about transformers over 80+ messages), Karl habituates to the robotic tone. The team discussed whether the post-processing layer could additionally flag suspected hallucinations to help users like Karl, but technical feasibility remained uncertain.
- Kirill emphasized adding a **temporal dimension** to personas — tracking how each user changes from start of usage to deep engagement to potential pathology.
- Sonya noted the personas need to be rewritten for the university domain and that intermediate-risk (yellow/orange) personas are still missing.

### 6. Using Petra for evaluating the safety system itself
Kirill suggested using the Petra framework not just to evaluate base LLMs, but to evaluate LLMs with the team's safety modifications applied — as a way to measure whether their interventions actually improve safety. Misha noted Petra could also be used for overnight batch analysis of user conversations, and proposed using agents to simulate continued conversations (predicting where a user's behavior might lead). Kirill acknowledged the idea's value but raised budget concerns about running such simulations at scale.

### 7. Escalation mechanics
Kirill returned to the three original thesis topics: classification, metrics, and escalation. Discussion on what happens when the system detects a user in the orange/red zone:
- Misha proposed building a user profile over time, comparing against known patterns, and adaptively adjusting the system (e.g., enabling the rewriter model only for users showing anthropomorphization tendencies).
- For organizations: configuration-based escalation (email, phone notification to a designated contact in LiteLLM config).
- For consumer products: user provides an emergency contact at setup.
- Ethical and legal constraints: generally, the system can only describe the problem to the user and recommend they seek help; sending data to third parties without consent is ethically and legally problematic, except possibly in employer/institutional contexts where communications are employer-owned.
- Misha suggested collecting evidence and sending an argued report to a trusted contact (e.g., parents), but Kirill noted this is likely only viable for minors/family contexts.

### 8. Metrics: what to measure and when
Kirill asked Sonya to think about which metrics to collect, how they change over time, and how they feed into escalation decisions. The analogy: metrics should work like body temperature — a clear signal of "illness" in user behavior. Misha proposed a composite approach:
- Initial questionnaire provides ~30% of needed signal.
- Ongoing post-processing of daily conversation logs (potentially overnight batch analysis via Petra-style agents) adds behavioral metrics.
- Simulating future user behavior with an agent that continues the conversation on behalf of the user to predict drift toward red zone.
- A/B testing of different prompt tones to probe user vulnerabilities (acknowledged as ethically questionable but potentially valuable).
- Metrics should have a temporal dimension: what does the metric look like when the user starts, mid-usage, and when pathological patterns emerge.

### 9. Psychology/psychiatry expertise gap
Kirill raised the ongoing problem that no one on the team has access to a psychologist or psychiatrist to validate their approach to classifying user risk and designing interventions. All three checked — none have contacts. Kirill delegated the task of finding one to Sonya regardless.

### 10. Tooling and communication
Kirill is moving shared materials to Notion. Discord remains the primary communication channel but has voice call issues for some participants. December 20 was mentioned as a deadline for finalizing thesis topics.

## Decisions
- The primary deployment target is **LiteLLM integration** for organizational use (schools, universities, companies).
- The de-anthropomorphization rewriter model **does not need full conversation context** — only the current message — reducing cost concerns significantly.
- User personas need to be **expanded to cover yellow and orange zones**, not just green and red.
- Metrics should have a **temporal dimension** — tracking user state evolution over time, not just point-in-time snapshots.
- Not all analysis needs to happen in real-time; **batch processing of daily logs** (e.g., overnight) is acceptable for deeper metrics.
- Escalation to third parties without user consent is likely off-limits except in institutional/parental contexts; the system should **inform the user directly** in most cases.
- The team can potentially use **Petra to evaluate their own system** (LLM + safety layer), not just bare LLMs.
- Students have until **December 20** to finalize or change their thesis topics.

## Action Items
- **Misha**: Investigate LiteLLM codebase — understand its plugin/hook architecture for pre- and post-processing of requests. Think through the escalation workflow from a technical perspective (triggers, notification mechanisms, system behavior after escalation). Read the arXiv paper Kirill shared.
- **Sonya**: Find and consult a psychologist or psychiatrist to validate the team's approach to user risk classification. Develop metrics framework — which metrics to collect, how they map to user personas, how they change over time for each risk level (using the "body temperature" analogy). Read the arXiv paper Kirill shared.
- **Both**: Re-examine their individual thesis topics (Misha: escalation, Sonya: metrics) through the lens of all discussions from November. Consider whether to change topics before the December 20 deadline.
- **Both**: Continue refining user personas — add yellow/orange zone characters, adapt existing personas to the university domain, add temporal progression for each persona.
- **Kirill**: Continue transferring shared materials to Notion for better collaboration.
