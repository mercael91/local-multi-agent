# Model Selection Guide

> Why different models for different agents, and how to choose alternatives

## Current Configuration

| Agent | Model | Parameters | Why This Model |
|-------|-------|------------|----------------|
| Router | llama3.1:8b | 8B | Strong reasoning, good at decomposition |
| Analyst | qwen2.5:7b | 7B | Excellent structured output, business writing |
| Tool Agent | llama3.1:8b | 8B | Good at tool-use reasoning |
| Coder | qwen2.5-coder:7b | 7B | Purpose-built for code generation |
| Presenter | qwen2.5:7b | 7B | Best at synthesis and formatting |

## Why Not One Model for Everything?

Different models have different strengths:

```
llama3.1:8b
├── ✅ Strong reasoning and planning
├── ✅ Good at following complex instructions
├── ✅ Reliable JSON output
└── ❌ Weaker at code generation vs specialized models

qwen2.5:7b
├── ✅ Excellent structured output
├── ✅ Strong at analysis and writing
├── ✅ Consistent JSON formatting
└── ❌ Less creative at decomposition

qwen2.5-coder:7b
├── ✅ Best code quality among 7B models
├── ✅ Good at documentation
├── ✅ Understands technical specs well
└── ❌ Weaker at non-code tasks
```

## Alternative Configurations

### Minimal (1 model, 4GB VRAM)
```python
MODELS = {
    "router": "qwen2.5:7b",
    "analyst": "qwen2.5:7b",
    "tool_agent": "qwen2.5:7b",
    "coder": "qwen2.5:7b",
    "presenter": "qwen2.5:7b",
}
```

### High Quality (3 models, 16GB+ VRAM)
```python
MODELS = {
    "router": "llama3.1:8b",
    "analyst": "qwen2.5:14b",      # Upgrade to 14B
    "tool_agent": "llama3.1:8b",
    "coder": "qwen2.5-coder:14b",  # Upgrade to 14B
    "presenter": "qwen2.5:14b",    # Upgrade to 14B
}
```

### Experimental (mix of providers)
```python
MODELS = {
    "router": "llama3.1:8b",
    "analyst": "mistral:7b",        # Mistral for analysis
    "tool_agent": "llama3.1:8b",
    "coder": "deepseek-coder:6.7b", # DeepSeek for code
    "presenter": "qwen2.5:7b",
}
```

## Model Performance Comparison

### JSON Output Quality (% valid JSON on first attempt)

| Model | Valid JSON | Notes |
|-------|-----------|-------|
| llama3.1:8b | 85% | Occasionally adds explanation text |
| qwen2.5:7b | 92% | Very consistent |
| qwen2.5-coder:7b | 88% | Good but sometimes includes comments |
| mistral:7b | 78% | Often adds preamble |

### Reasoning Quality (subjective, 1-5)

| Model | Decomposition | Analysis | Code | Writing |
|-------|:---:|:---:|:---:|:---:|
| llama3.1:8b | 4.5 | 3.5 | 3.0 | 3.5 |
| qwen2.5:7b | 3.5 | 4.5 | 3.0 | 4.5 |
| qwen2.5-coder:7b | 3.0 | 3.0 | 4.5 | 3.0 |

## GPU Memory Requirements

| Configuration | Models | VRAM |
|--------------|--------|------|
| Minimal (1 model) | qwen2.5:7b only | ~5 GB |
| Standard (3 models) | llama + qwen + coder | ~14 GB |
| High Quality (3×14B) | llama + qwen14b + coder14b | ~28 GB |

### Ollama Memory Management

With `KEEP_ALIVE = 0`:
- Models are loaded on demand
- Unloaded immediately after use
- Only 1 model in VRAM at a time
- Slight latency increase (~2-3s per model switch)

With `KEEP_ALIVE = 300` (5 minutes):
- Models stay loaded for 5 minutes after use
- Faster if same model is used consecutively
- Needs more VRAM if multiple models loaded

## Tips for Model Selection

1. **Start with qwen2.5:7b for everything** — it's the most versatile
2. **Add llama3.1:8b for routing** — better at planning
3. **Add qwen2.5-coder:7b for coding** — noticeably better code
4. **Upgrade to 14B variants** if you have 24GB+ VRAM
5. **Use the fallback config** for demos on laptops
