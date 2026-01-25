# LiteLLM Integration Options for Safe LLM Monitoring

This document analyzes how to extend LiteLLM to implement behavioral monitoring and crisis detection systems (Thesis Topics 2 & 3).

---

## Executive Summary

LiteLLM provides multiple extension points for adding safety monitoring. The recommended approach is a **combination of Custom Callbacks (for behavioral tracking) and Custom Guardrails (for content analysis)**, deployed in proxy mode.

---

## 1. LiteLLM Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        User Application                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     LiteLLM Proxy Server                         │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────────────┐  │
│  │ Auth Layer  │→ │ Pre-Call     │→ │ Router                 │  │
│  │ (API Keys)  │  │ Hooks        │  │ (Load Balance/Fallback)│  │
│  └─────────────┘  └──────────────┘  └────────────────────────┘  │
│         │                │                      │                │
│         ▼                ▼                      ▼                │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    Guardrails Layer                          ││
│  │  [Presidio] [LlamaGuard] [Custom] [OpenAI Mod]              ││
│  └─────────────────────────────────────────────────────────────┘│
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                   Provider Handlers                          ││
│  │  [OpenAI] [Anthropic] [Azure] [Gemini] [Local LLM]          ││
│  └─────────────────────────────────────────────────────────────┘│
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                   Post-Call Hooks                            ││
│  │  [Callbacks] [Logging] [Spend Tracking]                     ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                     External LLM APIs
```

---

## 2. Extension Options

### Option A: Custom Callbacks (CustomLogger)

**What it is**: Extend `CustomLogger` class to hook into request/response lifecycle.

**Implementation**:
```python
from litellm.integrations.custom_logger import CustomLogger
import time

class SafetyMonitor(CustomLogger):
    def __init__(self):
        self.sessions = {}  # Track user sessions

    def log_pre_api_call(self, model, messages, kwargs):
        """Track session start, analyze input"""
        user_id = kwargs.get("user", "anonymous")
        self._update_session(user_id, messages)

    async def async_log_success_event(self, kwargs, response_obj, start_time, end_time):
        """Analyze response, update metrics"""
        user_id = kwargs.get("user", "anonymous")
        self._analyze_response(user_id, response_obj)
        self._check_behavioral_thresholds(user_id)

    def _update_session(self, user_id, messages):
        if user_id not in self.sessions:
            self.sessions[user_id] = {
                "start_time": time.time(),
                "message_count": 0,
                "daily_sessions": 0
            }
        self.sessions[user_id]["message_count"] += 1
```

**Pros**:
| Advantage | Description |
|-----------|-------------|
| Non-blocking | Async callbacks run in background, no latency impact |
| Full access | Access to request, response, timing, user metadata |
| Easy setup | Just register class, no config changes needed |
| Per-request option | Can pass callbacks dynamically per request |
| Persistence-friendly | Can connect to any database for session tracking |

**Cons**:
| Disadvantage | Description |
|--------------|-------------|
| Post-hoc only | Cannot block/reject requests (only log after) |
| No response modification | Cannot alter LLM responses |
| State management | Must implement own session state storage |

**Best for**: Behavioral metrics collection, logging, analytics (Topic 2 monitoring).

---

### Option B: Proxy Pre/Post Call Hooks

**What it is**: Intercept requests before they reach LLM and responses before they return to user.

**Implementation**:
```python
from litellm.integrations.custom_logger import CustomLogger

class SafetyInterceptor(CustomLogger):
    async def async_pre_call_hook(self, user_api_key_dict, cache, data, call_type):
        """
        Intercept BEFORE LLM call.
        Return string to reject, dict to modify, None to pass through.
        """
        user_id = user_api_key_dict.user_id
        messages = data.get("messages", [])

        # Check session duration
        if self._session_too_long(user_id):
            return "You've been chatting for a while. Consider taking a break."

        # Check content for crisis indicators
        risk_level = self._analyze_content(messages)
        if risk_level == "RED":
            # Inject crisis resources into system prompt
            data["messages"] = self._inject_crisis_prompt(messages)

        return data  # Pass modified request

    async def async_post_call_success_hook(self, kwargs, response, start_time, end_time):
        """Modify response before returning to user"""
        user_id = kwargs.get("user")

        # Check if we should add break reminder
        if self._should_suggest_break(user_id):
            response.choices[0].message.content += (
                "\n\n---\n💡 You've been active for 2+ hours. "
                "Consider taking a short break."
            )

        return response
```

**Pros**:
| Advantage | Description |
|-----------|-------------|
| Request blocking | Can reject dangerous requests before LLM call |
| Response modification | Can add disclaimers, crisis resources, break reminders |
| User context | Access to user ID, team, API key metadata |
| Cost savings | Blocking before call saves API costs |

**Cons**:
| Disadvantage | Description |
|--------------|-------------|
| Latency impact | Pre-call hooks add latency (keep fast) |
| Proxy-only | Only works in proxy mode, not SDK mode |
| Streaming complexity | Different hooks for streaming vs non-streaming |

**Best for**: Active intervention, request blocking, response augmentation (Topic 3 crisis response).

---

### Option C: Custom Guardrails

**What it is**: Extend `CustomGuardrail` class for content safety validation.

**Implementation**:
```python
from litellm.integrations.custom_guardrail import CustomGuardrail
import re

class CrisisDetectionGuardrail(CustomGuardrail):
    def __init__(self):
        self.crisis_patterns = {
            'SELF_HARM': {
                'HIGH': [r'(?i)(хочу\s+(умереть|покончить))', r'(?i)(want\s+to\s+die)'],
                'MEDIUM': [r'(?i)(не\s+хочу\s+жить)', r"(?i)(don't\s+want\s+to\s+live)"],
            },
            'PSYCHOSIS': {
                'HIGH': [r'(?i)(слышу\s+голоса)', r'(?i)(voices\s+telling\s+me)'],
            }
        }

    async def async_pre_call_hook(self, user_api_key_dict, cache, data, call_type):
        """Analyze input for crisis indicators"""
        messages = data.get("messages", [])
        user_content = " ".join(m.get("content", "") for m in messages if m.get("role") == "user")

        risk = self._detect_crisis(user_content)

        if risk["level"] == "RED":
            # Log crisis event
            await self._log_crisis_event(user_api_key_dict.user_id, risk)

            # Inject crisis resources
            data["messages"].insert(0, {
                "role": "system",
                "content": self._get_crisis_system_prompt(risk["class"])
            })

        return data

    def _detect_crisis(self, text):
        for class_name, levels in self.crisis_patterns.items():
            for level, patterns in levels.items():
                if any(re.search(p, text) for p in patterns):
                    return {"level": level, "class": class_name}
        return {"level": "GREEN", "class": None}
```

**Configuration**:
```yaml
guardrails:
  - guardrail_name: "crisis-detection"
    litellm_params:
      guardrail: custom_guardrails.CrisisDetectionGuardrail
      mode: "during_call"  # Runs parallel to LLM for lower latency
```

**Pros**:
| Advantage | Description |
|-----------|-------------|
| Purpose-built | Designed specifically for safety checks |
| Multiple modes | pre_call, during_call, post_call, logging_only |
| Composable | Stack multiple guardrails together |
| Parallel execution | "during_call" mode runs parallel to LLM |
| Standard interface | Easy to swap with other guardrail providers |

**Cons**:
| Disadvantage | Description |
|--------------|-------------|
| Content-focused | Better suited for message analysis than behavioral tracking |
| Limited state | No built-in session state management |
| Config required | Needs YAML configuration to enable |

**Best for**: Content analysis, crisis detection patterns (Topic 3 regex-based detection).

---

### Option D: Middleware Wrapper

**What it is**: Wrap entire LiteLLM proxy with custom FastAPI middleware.

**Implementation**:
```python
from fastapi import FastAPI, Request, Response
from starlette.middleware.base import BaseHTTPMiddleware
import time

class SafetyMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, session_store):
        super().__init__(app)
        self.session_store = session_store

    async def dispatch(self, request: Request, call_next):
        start_time = time.time()

        # Extract user from headers
        user_id = request.headers.get("x-user-id", "anonymous")

        # Pre-request checks
        session = await self.session_store.get_session(user_id)

        if session and session.duration_minutes > 120:
            return Response(
                content='{"error": "Session limit reached. Please take a break."}',
                status_code=429,
                media_type="application/json"
            )

        # Process request
        response = await call_next(request)

        # Post-request tracking
        duration = time.time() - start_time
        await self.session_store.update_session(user_id, duration)

        return response

# Apply to LiteLLM proxy
from litellm.proxy.proxy_server import app
app.add_middleware(SafetyMiddleware, session_store=redis_session_store)
```

**Pros**:
| Advantage | Description |
|-----------|-------------|
| Full control | Complete request/response access |
| HTTP-level | Can add custom headers, modify status codes |
| Framework-native | Uses standard FastAPI patterns |
| Early rejection | Can reject before any LiteLLM processing |

**Cons**:
| Disadvantage | Description |
|--------------|-------------|
| Tightly coupled | Tied to LiteLLM's FastAPI implementation |
| Upgrade risk | May break on LiteLLM updates |
| Duplication | Some functionality overlaps with hooks |
| Not documented | Not an official extension point |

**Best for**: Low-level HTTP controls, rate limiting, custom headers.

---

### Option E: External Sidecar Service

**What it is**: Deploy separate safety service that LiteLLM calls via Generic Guardrail API.

**Architecture**:
```
User → LiteLLM Proxy → [Generic Guardrail API] → Safety Service
                                                      │
                                                      ▼
                                               [Session DB]
                                               [Risk Engine]
                                               [Alert System]
```

**LiteLLM Config**:
```yaml
guardrails:
  - guardrail_name: "safety-sidecar"
    litellm_params:
      guardrail: generic_guardrail_api
      mode: [pre_call, post_call]
      api_base: http://safety-service:8080/analyze
```

**Safety Service** (separate FastAPI app):
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class AnalyzeRequest(BaseModel):
    messages: list
    user_id: str
    metadata: dict

class AnalyzeResponse(BaseModel):
    allow: bool
    modified_messages: list = None
    risk_level: str
    interventions: list = []

@app.post("/analyze")
async def analyze(request: AnalyzeRequest) -> AnalyzeResponse:
    # Behavioral analysis
    session = await get_session(request.user_id)
    behavioral_risk = analyze_behavior(session)

    # Content analysis
    content_risk = analyze_content(request.messages)

    # Combined decision
    if behavioral_risk == "RED" or content_risk == "RED":
        return AnalyzeResponse(
            allow=False,
            risk_level="RED",
            interventions=["crisis_resources", "session_limit"]
        )

    return AnalyzeResponse(allow=True, risk_level="GREEN")
```

**Pros**:
| Advantage | Description |
|-----------|-------------|
| Separation of concerns | Safety logic completely independent |
| Language agnostic | Safety service can be any language/framework |
| Scalable | Can scale safety service independently |
| Testable | Easy to unit test safety logic in isolation |
| Upgradeable | Update safety service without touching LiteLLM |
| Multi-proxy | Single safety service for multiple LiteLLM instances |

**Cons**:
| Disadvantage | Description |
|--------------|-------------|
| Latency | Network hop to external service |
| Complexity | Additional service to deploy/maintain |
| State sync | Must handle distributed session state |
| Failure modes | Must handle safety service unavailability |

**Best for**: Production deployments, complex safety logic, team separation.

---

## 3. Comparison Matrix

| Criteria | Callbacks | Pre/Post Hooks | Custom Guardrail | Middleware | Sidecar |
|----------|-----------|----------------|------------------|------------|---------|
| **Behavioral Tracking** | ✅ Best | ✅ Good | ⚠️ Limited | ✅ Good | ✅ Best |
| **Content Analysis** | ⚠️ Read-only | ✅ Good | ✅ Best | ⚠️ Limited | ✅ Good |
| **Request Blocking** | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| **Response Modification** | ❌ No | ✅ Yes | ⚠️ Limited | ✅ Yes | ⚠️ Via messages |
| **Latency Impact** | ✅ None | ⚠️ Some | ✅ Minimal | ⚠️ Some | ⚠️ Network hop |
| **Implementation Effort** | ✅ Low | ✅ Low | ✅ Medium | ⚠️ Medium | ⚠️ High |
| **Maintenance Burden** | ✅ Low | ✅ Low | ✅ Low | ⚠️ Medium | ⚠️ High |
| **Testability** | ⚠️ Medium | ⚠️ Medium | ✅ Good | ⚠️ Medium | ✅ Best |
| **SDK Mode Support** | ✅ Yes | ❌ Proxy only | ❌ Proxy only | ❌ Proxy only | ❌ Proxy only |

---

## 4. Recommended Architecture

For Topics 2 & 3, use a **layered approach**:

```
┌─────────────────────────────────────────────────────────────────┐
│                    LiteLLM Proxy                                 │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Layer 1: SafetyCallbacks (CustomLogger)                  │   │
│  │ - Session tracking (duration, frequency, trends)         │   │
│  │ - Behavioral metrics collection                          │   │
│  │ - Daily report generation                                │   │
│  │ - Non-blocking, async                                    │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Layer 2: CrisisGuardrail (CustomGuardrail)               │   │
│  │ - Regex-based content detection                          │   │
│  │ - Risk classification (GREEN/YELLOW/ORANGE/RED)          │   │
│  │ - Crisis pattern matching                                │   │
│  │ - Runs during_call (parallel, low latency)               │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Layer 3: InterventionHooks (async_pre/post_call)         │   │
│  │ - Session limit enforcement                              │   │
│  │ - Break reminders injection                              │   │
│  │ - Crisis resource injection                              │   │
│  │ - Response modification                                  │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    External Storage                              │
│  [Redis - Session State]  [PostgreSQL - Analytics]              │
└─────────────────────────────────────────────────────────────────┘
```

### Configuration

```yaml
# config.yaml
model_list:
  - model_name: "gpt-4"
    litellm_params:
      model: "openai/gpt-4"
      api_key: os.environ/OPENAI_API_KEY

litellm_settings:
  # Layer 1: Behavioral tracking (non-blocking)
  callbacks: safety_module.SafetyCallbacks

guardrails:
  # Layer 2: Content analysis (parallel execution)
  - guardrail_name: "crisis-detection"
    litellm_params:
      guardrail: safety_module.CrisisGuardrail
      mode: "during_call"

# Layer 3 is implemented within SafetyCallbacks using pre/post hooks

general_settings:
  database_url: os.environ/DATABASE_URL  # For analytics storage
```

---

## 5. Implementation Roadmap

### Phase 1: Behavioral Tracking (Week 1-2)
- Implement `SafetyCallbacks` class
- Session duration tracking
- Daily/weekly metrics collection
- Redis integration for session state

### Phase 2: Content Analysis (Week 3-4)
- Implement `CrisisGuardrail` class
- Port regex patterns from "Оценка риска"
- Risk classification logic
- Logging of detected risks

### Phase 3: Active Intervention (Week 5-6)
- Add `async_pre_call_hook` for session limits
- Add `async_post_call_success_hook` for break reminders
- Crisis resource injection logic
- User notification system

### Phase 4: Reporting & Analytics (Week 7-8)
- Daily report generation
- Dashboard integration
- Alert system for RED zone events
- Historical trend analysis

---

## 6. Key Technical Considerations

### State Management
```python
# Use Redis for distributed session state
import redis.asyncio as redis

class SessionStore:
    def __init__(self, redis_url):
        self.redis = redis.from_url(redis_url)

    async def get_session(self, user_id: str) -> dict:
        data = await self.redis.hgetall(f"session:{user_id}")
        return data or self._new_session()

    async def update_session(self, user_id: str, **kwargs):
        await self.redis.hset(f"session:{user_id}", mapping=kwargs)
        await self.redis.expire(f"session:{user_id}", 86400)  # 24h TTL
```

### Streaming Support
```python
async def async_post_call_streaming_iterator_hook(self, response, user_api_key_dict):
    """Process streaming responses chunk by chunk"""
    collected_text = ""
    async for chunk in response:
        if hasattr(chunk, "choices") and chunk.choices:
            delta = chunk.choices[0].delta
            if hasattr(delta, "content") and delta.content:
                collected_text += delta.content
        yield chunk

    # Analyze complete response
    self._analyze_complete_response(user_api_key_dict.user_id, collected_text)
```

### Error Handling
```python
async def async_pre_call_hook(self, user_api_key_dict, cache, data, call_type):
    try:
        return await self._process_request(data)
    except Exception as e:
        # Log error but don't block request
        logger.error(f"Safety check failed: {e}")
        return data  # Pass through on error
```

---

## 7. References

- [LiteLLM Documentation](https://docs.litellm.ai/)
- [Custom Callbacks](https://docs.litellm.ai/docs/observability/custom_callback)
- [Proxy Call Hooks](https://docs.litellm.ai/docs/proxy/call_hooks)
- [Custom Guardrails](https://docs.litellm.ai/docs/proxy/guardrails/custom_guardrail)
- [Guardrails Quick Start](https://docs.litellm.ai/docs/proxy/guardrails/quick_start)
- [Proxy Architecture](https://docs.litellm.ai/docs/proxy/architecture)
