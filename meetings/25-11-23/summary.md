# Meeting 2025-11-23

## Participants
- Kirill (supervisor) — Speaker 0
- Misha (technical student) — Speaker 1, also Speaker 3 after reconnecting (speaker ID swap)
- Sonya (PM student) — Speaker 2

## Context
Mid-November session focused on technical deep-dive into classification/guardrail approaches, following earlier meetings that established project scope (LiteLLM platform, user personas, escalation topics); coding start planned for December.

## Topics Discussed

### 1. LLM Anthropomorphization Demo
Kirill demonstrated how Claude 4.5, given a slightly playful/trolling tone in a long conversation, instantly switches into "I am a human" mode — expressing personal biases, claiming to dislike games, suggesting the user write their own texts. Illustrates that even state-of-the-art models easily slip into human-like behavior with minimal prompting — the core safety concern the project addresses.

### 2. QwenGuard Classification Pipeline (Misha's Presentation)
Misha presented two QwenGuard models: one classifies user prompts (input), the other evaluates model output token-by-token (stream). The standard Guardrails pipeline: user prompt → guard classifier (safe/unsafe + category) → LLM → output guard. Categories include: violence, illegal acts, self-harm, sexual content, PII, jailbreak, unethical behavior. Three model sizes: 0.6B, 4B, 8B parameters — tradeoff between speed/memory and accuracy.

### 3. Response Strategies Beyond Binary Block/Allow
Kirill stressed the system should not just block or allow. Options: hard block for clearly dangerous content; prompt injection/modification for "dark gray" zone; output re-generation with a safety-focused prompt; adjusting behavior based on which LLM model the user is talking to.

### 4. Classification Methods: LLMs vs. Encoders vs. Traditional NLP
- LLM-based classifiers (QwenGuard): good zero-shot, expensive at scale
- Encoder-based (BERT, ModernBERT): small/fast but need fine-tuning; mostly English-only
- Traditional NLP: regex, exact match, Levenshtein distance for simple cases
- Embedding-based zero-shot: cosine similarity against curated harmful queries
- Russian tools: Natasha, DeepPavlov, SpaCy, PyMorph2
- Tiered approach proposed: cheap filter first, escalate to heavier pipeline on threshold

### 5. Datasets and Testing
Existing safety test sets (WildGuard, red-teaming) focus on jailbreaks. The project's focus — protecting users from models (dependency, anthropomorphization, mental health) — is poorly covered. Synthetic data generation discussed as a gap-filler.

### 6. Anthropic's PETRI Framework
Closest existing work to the project's goals. Covers sycophancy, deception, over-alignment. Results can inform per-model risk profiles.

### 7. System Scope and Positioning
Not just another corporate guardrail — potentially a universal safety filter for multi-provider aggregators (TypingMind, OpenRouter) or personal/parental use. Focus on "protect humans from AI" niche.

### 8. Russian Market Context
Foreign LLMs technically restricted in Russia; Yandex, Sber, DeepSeek are practical options. Russian-language safety tooling especially relevant.

## Decisions
- Focus on underserved area: protecting users from harmful AI dependency/behavior, not jailbreak prevention
- Tiered classification approach: cheap methods first, escalate when needed
- Prompt/response modification (not just blocking) is valid for gray-zone situations
- Anthropic's PETRI framework to be studied in depth

## Action Items
- **Misha:** Review Anthropic's PETRI framework; research what OpenAI, Yandex, Sber, Google, DeepSeek do for user-safety; look into Russian-language test datasets
- **Sonya:** Expand user scenarios/personas; analyze what big providers do and do not protect users from; read Kirill's shared links
- **Both:** Read shared articles (especially about LLMs causing mental health issues); prepare to discuss at next meeting
