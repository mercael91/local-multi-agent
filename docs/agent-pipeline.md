# Agent Pipeline Deep Dive

> How 5 specialized agents collaborate to transform a business idea into a complete product specification

## Overview

The pipeline is sequential: each agent receives output from the previous one and adds its specialized analysis. JSON is the universal communication format.

## Agent 1: Router (llama3.1:8b)

### Purpose
Decomposes a free-form business idea into structured subtasks for 3 specialized agents.

### Input
Raw user text: "Сервис доставки здорового питания для офисов с подпиской на неделю"

### Output Schema
```json
{
  "project_name": "Short, memorable project name",
  "target_audience": "Who is the primary user",
  "subtasks": {
    "analyst_instruction": "Clear instruction for the analyst agent",
    "tool_instruction": "Which tool to call and with what parameters",
    "coder_instruction": "Technical specification for the programmer"
  },
  "estimated_complexity": "Low/Medium/High"
}
```

### Why llama3.1:8b?
Best at reasoning and decomposition among 8B models. Strong at understanding intent and creating structured plans.

### Token Limit
400 tokens max — forces concise decomposition without rambling.

## Agent 2: Analyst (qwen2.5:7b)

### Purpose
Creates a product specification from the router's instruction.

### Input
Router's `analyst_instruction` field.

### Output Schema
```json
{
  "core_features": ["Feature 1 description", "Feature 2", "Feature 3"],
  "database_entities": {
    "EntityName1": ["field1", "field2", "field3"],
    "EntityName2": ["field1", "field2"]
  },
  "potential_risks": ["Risk 1", "Risk 2"],
  "business_value": "Brief description of business value"
}
```

### Why qwen2.5:7b?
Excellent at structured analysis, business writing, and maintaining consistent JSON format.

## Agent 3: Tool Agent (llama3.1:8b)

### Purpose
Decides which external tool to call and with what arguments. Simulates API calls.

### Input
Router's `tool_instruction` field.

### Output Schema
```json
{
  "thought": "Why this tool was chosen",
  "tool_to_call": "function_name",
  "arguments": {"arg_name": "arg_value"}
}
```

### Available Tools

| Tool | Function | Simulation |
|------|----------|------------|
| `check_domain_availability` | Check if domain is free | Always returns "available" |
| `generate_api_key` | Generate test API key | Returns MD5-based fake key |

### Why llama3.1:8b?
Strong at tool-use reasoning — understanding when and how to call functions.

## Agent 4: Coder (qwen2.5-coder:7b)

### Purpose
Generates working code from the analyst's specification.

### Input
Router's `coder_instruction` + analyst's `database_entities` (passed by the pipeline).

### Output Schema
```json
{
  "language": "python",
  "description": "Brief description of what the code does",
  "code": "Full code as escaped string"
}
```

### Why qwen2.5-coder:7b?
Purpose-built for code generation. Better at producing syntactically correct, well-documented code than general-purpose models.

## Agent 5: Presenter (qwen2.5:7b)

### Purpose
Combines all agent outputs into a professional Markdown report.

### Input
All previous outputs + system metrics (time, tokens).

### Output
Raw Markdown document (not JSON) with:
- Project description
- Business value
- Technical architecture (DB entities, risks)
- Generated code
- Agent logs summary
- Performance metrics

### Why qwen2.5:7b?
Best at writing and formatting among the available models. Strong at synthesizing information from multiple sources.

## Error Handling

### JSON Parse Failures

```python
def safe_json_parse(text, agent_name, max_retries=2):
    for attempt in range(max_retries + 1):
        cleaned = extract_json(text)
        try:
            return json.loads(cleaned)
        except JSONDecodeError:
            if attempt < max_retries:
                # Retry with explicit JSON instruction
                retry_prompt = f"Your previous answer was not valid JSON..."
                text, _, _ = call_agent(retry_prompt, model)
            else:
                return {"error": "invalid_json", "raw": text}
```

### API Failures

```python
def call_agent(prompt, model, max_retries=2):
    for attempt in range(max_retries + 1):
        try:
            resp = requests.post(OLLAMA_URL, json={...}, timeout=180)
            return content, elapsed, tokens
        except RequestException:
            if attempt < max_retries:
                time.sleep(1.5)  # Backoff
            continue
    raise RuntimeError(f"Model {model} failed after {max_retries} retries")
```

## Metrics Collection

Each agent call tracks:
- **Elapsed time** (seconds) — wall clock time for the API call
- **Token count** — `eval_count + prompt_eval_count` from Ollama response

These are aggregated into total pipeline metrics and displayed in the final report.
