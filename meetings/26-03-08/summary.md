# Meeting 2026-03-08

## Participants
- **Kirill (Speaker 0 / Speaker 2)** -- supervisor/senior lecturer (speaker ID swaps across chunks)
- **Misha (Speaker 1 / Speaker 0)** -- technical student, focuses on AI safety engineering, escalation, open-source LLMs
- **Sonya (Speaker 2 / Speaker 1)** -- PM student, focuses on behavioral metrics, user profiling, metric aggregation

## Context
This is a thesis supervision meeting for two undergraduate (4th year) students working on a shared project: an AI safety system built on LiteLLM that monitors user interactions with LLMs for harmful dependency, collects behavioral metrics, and escalates critical situations. The meeting covers both bureaucratic/academic writing guidance and substantive project discussion.

## Topics Discussed

### 1. What a qualifying thesis (VKR) is and how defense works
Kirill gave an extended lecture on the nature of a bachelor's thesis as a *qualification* work (proving competence, not making discoveries). He explained how the defense commission works: they have ~7 minutes per student, typically only read the introduction and conclusion, and form their impression from those plus one section they find interesting. Key takeaway: introduction and conclusion must be written perfectly.

### 2. Academic writing requirements for the introduction
Detailed guidance on writing the introduction as a coherent narrative text (not a bulleted checklist from the guidelines):
- Start with a "water paragraph" to orient the reader on the subject domain
- Every claim must be backed by evidence: logical proof, citations, or (sparingly) common sense
- Avoid anglicisms and jargon -- use accepted Russian terminology (e.g., no "LLM" abbreviation without spelling out the Russian equivalent; no English terms like "control" when a Russian equivalent exists)
- The introduction must flow: context -> relevance/problem -> goal -> tasks
- **Goal must directly correspond to the thesis title** -- the single most scrutinized alignment by any commission
- Tasks describe *what* needs to be done, not *how* -- avoid premature technical specifics (no "LiteLLM" or "LlamaGuard" at this stage)
- Formal sections like "object/subject," "methods," "statements for defense" are largely redundant for a 4th-year thesis but can be included if required
- Never leave room for commission interpretation -- ambiguity always works against the student

### 3. Citations and references
- Citations are mandatory for virtually any claim in scientific text
- First chapter should contain the majority of all references; minimum 10-15 for a 4th-year thesis
- Preference hierarchy: scientific journals > arXiv > books/monographs > GitHub/Habr/docs
- Blocked websites in Russia: the commission reads paper copies, nobody actually clicks links; the "last accessed" date is what matters
- Meta (banned organization): avoid explicit references in the introduction; deeper in the text a "Facebook Research" reference is acceptable; don't draw unnecessary attention
- Wikipedia references are frowned upon but not as controversial as they used to be
- Always actually read the sources you cite

### 4. Anti-plagiarism and AI-generated text detection
Brief discussion about upcoming anti-plagiarism checks (May). Misha shared experience from a workplace case about detecting AI-generated text via word frequency anomalies (e.g., GPT overusing "demand" post-2022). Kirill noted that AI-written text is detectable by heuristics like em-dash usage patterns. One of Kirill's other students has conspicuously AI-generated text. Consensus: minimal effort to humanize text is usually sufficient; real risk is unknown anti-plagiarism tool behavior.

### 5. Deadlines and submission process
- **March 13** -- deadline to submit the current version of the thesis (introduction + chapter 1) for internal university evaluation
- Kirill wants at least one review iteration before the 13th: students send updated text by Tuesday-Wednesday, he reviews by Wednesday-Thursday, final iteration by Friday
- This is an internal checkpoint, not a state exam; Kirill is the one grading, so stakes are moderate
- Both students' introductions need minor fixes: Misha needs a couple more paragraphs (pre-relevance context and relevance-to-problem transition); Sonya's text also needs review with the document open
- The introduction will be rewritten 3-5 more times before final defense

### 6. Scheduling future sessions
Kirill plans to start splitting sessions -- working with Misha and Sonya separately, as the project moves into per-student text editing. Agreed to meet again at 16:00 the same day to discuss substance. Future sessions will be scheduled individually.

### 7. Project substance: Misha's approach to the escalation module
After Kirill left temporarily, Misha and Sonya discussed the project direction. Misha's key ideas:
- **Focus on open-source LLMs** rather than proprietary ones, since proprietary models (GPT, Claude) already have good safety wrappers; open-source models (Qwen, LLaMA) significantly lag on safety benchmarks and lack surrounding safety infrastructure
- **Use open-source safety tools** (LlamaGuard, QwenGuard, GPT-OSS SafeGuard) rather than proprietary classifiers, to keep data in a closed loop and make the solution practically deployable
- **Longitudinal / multi-session analysis** as the key differentiator: existing guardrails operate per-message or per-session, but real user crises develop across weeks/months (job loss -> breakup -> substance abuse -> crisis). Aggregating cross-session context should measurably improve early detection metrics
- Found an open longitudinal early-risk-detection dataset; plans to benchmark single-session vs. multi-session classification to prove the hypothesis
- Additional ideas: user persona construction from conversation history, clustering personal vs. work messages, early depression detection (Reddit-based datasets exist)
- On escalation behavior: models currently just refuse/deflect dangerous requests; better approach is to provide immediate first-response help (grounding exercises, hotline info) while escalating to humans

### 8. Project substance: Sonya's approach to behavioral metrics
Sonya described her part of the work:
- Focuses on **defining and aggregating behavioral metrics**: temporal (time-of-day, frequency), behavioral (session patterns), and contextual (content analysis)
- Her unique contribution is **linking these metric types together** -- existing tools measure them independently but nobody combines temporal + behavioral + contextual signals (e.g., daytime study queries + nighttime suicide queries = elevated risk when correlated)
- Will not implement escalation/session-interruption herself; instead, she produces metrics and statistics that Misha's escalation module (and administrators) can consume
- Acknowledged that their code bases will overlap significantly (shared LiteLLM infrastructure); they plan to fork at the module level

## Decisions
- Introduction should be concise (1-1.5 pages ideally, max 2 pages) and follow the narrative structure Kirill described
- Formal sections (object/subject, statements for defense, chapter-by-chapter overview) can be omitted or kept minimal -- Kirill will not penalize their absence
- The project emphasizes open-source LLMs and open-source safety tools as the target deployment context
- Longitudinal multi-session analysis is adopted as Misha's key differentiator for the escalation module
- Sonya's differentiator is cross-type metric correlation (temporal + behavioral + contextual)
- Shared codebase is acceptable; they will branch/fork for individual thesis components

## Action Items
- [ ] **Misha**: revise introduction (add context paragraphs, fix relevance-to-problem transition, align tasks with escalation focus); send to Kirill by Tuesday-Wednesday (March 10-11)
- [ ] **Sonya**: revise introduction similarly; send to Kirill by Tuesday-Wednesday (March 10-11)
- [ ] **Kirill**: review both drafts by Wednesday-Thursday; provide feedback for a final iteration before March 13
- [ ] **Both**: submit thesis (introduction + chapter 1) by **March 13** for internal evaluation
- [ ] **Both**: check GOST formatting for numbered lists in the tasks section
- [ ] **Misha**: run benchmark comparison -- single-session vs. multi-session classification on the longitudinal early-risk-detection dataset he found
- [ ] **Misha**: prepare to present the open-source + longitudinal approach to Kirill at the 16:00 session
- [ ] **Sonya**: prepare to discuss her metric aggregation approach with Kirill (separately or at 16:00)
- [ ] **Kirill**: check Telegram chat for exact submission format/process for March 13 deadline
