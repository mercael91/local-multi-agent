<div align="center">

# 🏭 Local Multi-Agent System

### 5 AI Agents Running Entirely on Your Machine

**No cloud · No API keys · No data leaving your computer**

**Just Ollama + 3 open-source models (7B-8B params)**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python)]()
[![Streamlit](https://img.shields.io/badge/Streamlit-App-FF4B4B?logo=streamlit)]()
[![Ollama](https://img.shields.io/badge/Ollama-Local-000000)]()

</div>

---

## What Is This?

A demonstration of what **local LLMs can do**. Five specialized AI agents process a business idea through a complete pipeline — decomposition, analysis, tool calls, code generation, and presentation — all running on a single GPU with open-source models.

No OpenAI. No Claude. No API keys. No internet required.

> Just you, your GPU, and 3 open-source models working together.

---

## The Pipeline

```
Your Business Idea
        │
        ▼
┌───────────────────┐
│ 🧠 ROUTER         │  llama3.1:8b
│                   │
│ Decomposes idea   │
│ into 3 subtasks   │
│ for specialized   │
│ agents            │
└────────┬──────────┘
         │
    ┌────┴────┬────────────┐
    ▼         ▼            ▼
┌────────┐ ┌────────┐ ┌────────┐
│📊 ANALY│ │🔧 TOOL │ │💻 CODER│
│ST      │ │AGENT   │ │        │
│        │ │        │ │        │
│qwen2.5 │ │llama3.1│ │qwen2.5 │
│:7b     │ │:8b     │ │-coder  │
│        │ │        │ │:7b     │
│Spec:   │ │Calls:  │ │Writes: │
│features│ │domain  │ │working │
│DB schema│ │check  │ │code    │
│risks   │ │API keys│ │from spec│
└───┬────┘ └───┬────┘ └───┬────┘
    │          │           │
    └──────────┼───────────┘
               ▼
        ┌───────────────┐
        │📝 PRESENTER   │  qwen2.5:7b
        │               │
        │ Combines all  │
        │ outputs into  │
        │ professional  │
        │ Markdown      │
        │ report        │
        └───────────────┘
               │
               ▼
        Final Report
```

---

## The 5 Agents

| # | Agent | Model | Role | Output |
|---|-------|-------|------|--------|
| 1 | 🧠 **Router** | llama3.1:8b | Decomposes business idea into 3 subtasks | JSON with project name, target audience, complexity |
| 2 | 📊 **Analyst** | qwen2.5:7b | Creates product specification | Features, DB entities, risks, business value |
| 3 | 🔧 **Tool Agent** | llama3.1:8b | Calls external APIs (simulated) | Domain availability, API key generation |
| 4 | 💻 **Coder** | qwen2.5-coder:7b | Generates code from specification | Working Python code with documentation |
| 5 | 📝 **Presenter** | qwen2.5:7b | Creates final report | Professional Markdown document |

### Why Different Models?

Each agent uses a model specialized for its task:

- **llama3.1:8b** — Strong reasoning for routing and tool decisions
- **qwen2.5:7b** — Excellent at structured analysis and writing
- **qwen2.5-coder:7b** — Specialized for code generation

This demonstrates that **model diversity** beats using one model for everything.

---

## Inter-Agent Communication

Agents communicate via structured JSON:

### Router → Analyst/Tool/Coder

```json
{
  "project_name": "HealthBox",
  "target_audience": "Office workers",
  "subtasks": {
    "analyst_instruction": "Design product spec for healthy food delivery...",
    "tool_instruction": "Check domain availability for healthbox.ru...",
    "coder_instruction": "Write Python Flask API for subscription management..."
  },
  "estimated_complexity": "Средняя"
}
```

### Analyst → Presenter

```json
{
  "core_features": ["Subscription management", "Menu planning", "Delivery tracking"],
  "database_entities": {
    "User": ["id", "email", "subscription_plan"],
    "Order": ["id", "user_id", "items", "delivery_date"]
  },
  "potential_risks": ["Perishable goods logistics", "Seasonal pricing"],
  "business_value": "Recurring revenue from office subscriptions"
}
```

### Tool Agent → Presenter

```json
{
  "thought": "User needs a domain for their service",
  "tool_to_call": "check_domain_availability",
  "arguments": {"domain_name": "healthbox.ru"}
}
```

---

## Two Modes

### 🎭 Presentation Mode

For business audiences:
- Smooth animations between agents
- Only the final report shown
- Professional formatting
- No technical details

### 🔧 Technical Mode

For developers:
- Raw prompts visible
- JSON responses displayed
- Token counts and timing metrics
- Full agent logs

---

## System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **GPU** | GTX 1080 Ti (11GB) | RTX 3090 (24GB) |
| **VRAM** | 11 GB | 16+ GB |
| **RAM** | 16 GB | 32 GB |
| **Python** | 3.10+ | 3.11 |
| **Ollama** | Latest | Latest |

### Models to Install

```bash
ollama pull llama3.1:8b      # ~4.7 GB
ollama pull qwen2.5:7b       # ~4.4 GB
ollama pull qwen2.5-coder:7b # ~4.4 GB
# Total: ~13.5 GB disk space
```

### Fallback Config

For testing with fewer resources, use `config_fallback.py`:
- All 5 agents use the same model (qwen2.5:7b)
- Only ~4.7 GB VRAM needed
- Slightly lower quality but works on smaller GPUs

---

## Demo Ideas

Built-in examples for quick demos:

1. 🥗 **Healthy food delivery service** with weekly subscriptions
2. 💰 **Personal finance app** with AI advisor
3. 📚 **Online programming learning platform** with mentorship
4. 🌾 **Local farmer's marketplace** with same-day delivery
5. 👥 **Corporate HR portal** for onboarding automation

---

## Architecture

```
┌─────────────────────────────────────────────┐
│              STREAMLIT UI                    │
│  localhost:8501 · Dark theme · Animations    │
├─────────────────────────────────────────────┤
│              AGENT PIPELINE                  │
│                                              │
│  Router → [Analyst, Tool, Coder] → Presenter │
│                                              │
│  Each agent:                                 │
│  ├── Structured prompt with JSON schema      │
│  ├── Ollama API call (localhost:11434)        │
│  ├── JSON extraction + validation            │
│  ├── Retry on parse failure (max 2)          │
│  └── Metrics collection (time, tokens)       │
├─────────────────────────────────────────────┤
│              OLLAMA RUNTIME                  │
│  localhost:11434 · GPU inference              │
│  llama3.1:8b · qwen2.5:7b · qwen2.5-coder:7b│
└─────────────────────────────────────────────┘
```

### Tech Stack

| Layer | Technology |
|-------|-----------|
| UI | Streamlit (custom CSS, animations) |
| LLM Runtime | Ollama (local GPU inference) |
| Models | llama3.1:8b, qwen2.5:7b, qwen2.5-coder:7b |
| Language | Python 3.10+ |
| Dependencies | streamlit, ollama, requests |

---

## Results

| Metric | Value |
|--------|-------|
| Agents | 5 |
| Models | 3 (all open-source) |
| Lines of Code | 619 |
| VRAM Required | 11+ GB |
| Internet Required | No |
| Data Privacy | 100% local |
| Average Pipeline Time | 2-5 minutes |

---

## Project Status

- [x] 5-agent sequential pipeline
- [x] 3 specialized local models
- [x] JSON inter-agent communication
- [x] Presentation & Technical modes
- [x] Streamlit UI with animations
- [x] Tool simulation (domain check, API keys)
- [x] Fallback config for smaller GPUs
- [x] Demo ideas built-in
- [ ] Real API integration (domain check, etc.)
- [ ] Parallel agent execution
- [ ] Model hot-swapping
- [ ] Export to PDF

---

## Quick Start

```bash
# 1. Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# 2. Pull models
ollama pull llama3.1:8b
ollama pull qwen2.5:7b
ollama pull qwen2.5-coder:7b

# 3. Install dependencies
pip install streamlit requests

# 4. Run
streamlit run app.py
```

Open http://localhost:8501 and enter a business idea.

---

## Contact

- **Creator:** [mercael91](https://github.com/mercael91)
- **Telegram:** [@mercael](https://t.me/mercael)

## Related Projects

- **[AGI-Zarodysh](https://github.com/mercael91/embryo-agent)** — Autonomous AI agent for open-source contributions.
- **[LikAI](https://github.com/mercael91/likai)** — AI content platform with style mimicry.
- **[Nexus Analytica](https://github.com/mercael91/nexus-analitica)** — AI news intelligence with consensus analysis.
- **[Tinkoff Scalper](https://github.com/mercael91/tinkoff-scalper)** — Autonomous scalper bot for Russian stock market.

---

<div align="center">

**Your data. Your GPU. Your AI.**

*Твои данные. Твой GPU. Твой ИИ.*

</div>
