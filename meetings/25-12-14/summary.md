# Meeting 2025-12-14

## Participants
- Kirill (supervisor) — Speaker 0
- Misha — Speaker 1
- Sonya — Speaker 2

## Context
Follow-up to Nov 16 meeting. Students have been working on AI safety topics. Focus shifts from conceptual to implementation.

## Topics Discussed

### 1. System Architecture — Sensor/Actuator Model
- Kirill framed the system using a "sensor + actuator" metaphor:
  - **Sensor:** Runs daily (or nightly), analyzes user conversations, classifies state (green/orange/red zones)
  - **Actuator:** Takes action based on sensor output — modifies prompts, sends alerts, escalates
- Need personas in ALL zones (not just red) — same person ("Vasya") in different states
- Key questions: Can we detect "orange Vasya"? What actions do we take? Does intervention actually help?
- Escalation (Misha's thesis) vs. metrics collection (Sonya's thesis) are separate modules but interconnected

### 2. LiteLLM as Core Infrastructure
- LiteLLM chosen as the central proxy — becoming a de-facto standard for LLM management
- Already has built-in guardrails support (pre-call/post-call hooks), logging integrations
- Plan: extend LiteLLM with safety modules rather than building from scratch
- Kirill emphasized: integrate into existing ecosystem rather than building standalone tools nobody will use
- OpenRouter can work alongside LiteLLM (complementary, not competing)

### 3. Technical Implementation Discussion
- **Misha's direction:** LiteLLM → QwenGuard integration for real-time content filtering via hooks/endpoints
- **Logging pipeline:** LiteLLM has extensive logging integrations — can use these for nightly batch analysis
- **Notification:** Telegram bot for alerts (both Misha and Sonya have Telegram bot experience — Sonya manages two Telegram bot projects at work)
- Kirill pushed for concrete technical details: what tables, what Python procedures, what APIs — not just concepts
- Misha found LiteLLM docs already describe guardrails integration (pre-call, during-call hooks)

### 4. Questionnaire / User Onboarding Problem
- Challenge: can't show users 150 questions before they use the system
- Need lightweight profiling that doesn't drive users away
- Idea: use ChatGPT-generated baseline questionnaires, then validate with a psychologist if possible
- System should be parametrizable: different contexts (university, school, summer camp, office) need different questionnaires
- Questionnaire feeds into user profile → customized prompts for the LLM

### 5. Working Style — Independence Phase
- Kirill explicitly stepping back for December: "I'll be more hindrance than help at this point"
- Students should self-organize, communicate directly with each other
- Come back with problems only if stuck after 1-2 attempts

## Decisions
- **LiteLLM** is the chosen platform for the proxy/safety layer
- December goal: get something running — even basic, even ugly
- Students should study LiteLLM internals (guardrails hooks, logging integrations)
- Notification via **Telegram bot**
- Rapid prototyping approach: use ChatGPT for initial questionnaires, iterate later

## Action Items (December)
- **Misha:** Study LiteLLM source code, set up LiteLLM → QwenGuard proxy pipeline, get basic content filtering working
- **Sonya:** Define metrics (what to collect, how to store, how to use), understand LiteLLM logging capabilities, continue refining user personas
- **Both:** Draw out system architecture (Miro or similar) — where each component connects, data flow
- **Both:** Communicate directly, self-organize, bring problems to Kirill only when stuck
- **Kirill:** Step back, let students work independently through December

## Project Status
- **Phase:** Transitioning from conceptual design to first implementation iteration
- **Architecture:** Partially defined — LiteLLM-based proxy with guardrails hooks, nightly batch analysis, Telegram alerts
- **Code:** Nothing yet — December is "start coding" month
- **Personas:** Sonya has critical zone personas, needs gray/green zone ones
