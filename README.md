# Chefs_Multiagent-Langgraph_Orchestration-
# LangGraph Orchestrator-Worker Menu Planner

A LangGraph orchestrator-worker pipeline that takes a free-text list of meals, breaks it into individual dishes using Gemini, then fans out to parallel "chef" workers that each generate a detailed cooking walkthrough for one dish.

## How it works

```
START → orchestrator → assign_workers (fan-out) → chef_worker (×N, parallel) → synthesizer → END
```

1. **`orchestrator`** — Takes a `meals` string (e.g. *"Italian pasta, Mexican tacos, Indian curry"*) and uses Gemini with a `PydanticOutputParser` to break it into structured `Dish` objects (name, ingredients, cuisine).
2. **`assign_workers`** — Fans out one `Send()` per dish to the `chef_worker` node, running them in parallel via a conditional edge.
3. **`chef_worker`** — For each dish, prompts Gemini to role-play as an expert chef from that cuisine and produce a step-by-step cooking guide.
4. **`synthesizer`** — Joins all the per-dish writeups into one final menu guide.

## Requirements

- Python 3.10+
- A Google AI Studio API key with access to Gemini models

```bash
pip install langgraph langchain-core langchain-google-genai pydantic
```

## Setup

Set your API key as an environment variable before running:

```bash
export GOOGLE_API_KEY="your-key-here"
```


```python
from google.colab import userdata
import os
os.environ["GOOGLE_API_KEY"] = userdata.get("GOOGLE_API_KEY")
```
## Colab Note
use google colab notebook or jupyter for testing the code by adding secret of gemini api key 

## Usage

```bash
python langgraph_orchestrator_fixed.py
```

Or import and call it directly:

```python
from langgraph_orchestrator_fixed import orchestrator_worker

state = orchestrator_worker.invoke({
    "meals": "Italian pasta, Mexican tacos, Indian curry, Thai stir-fry, American burgers"
})
print(state["final_menu_guide"])
```



## Known issues / debugging notes

- **`gemini-2.5-flash` occasionally returns empty `content`** under parallel `Send()` fan-out, which surfaces downstream as `ValueError: contents are required.` from the `google-genai` SDK. The current code guards against this in `chef_worker` by checking for empty content and logging `finish_reason` instead of crashing the whole graph.
- If you see this error, check the printed `DEBUG[chef_worker]` logs for the `finish_reason` on the failing call (commonly `MAX_TOKENS` if the model's reasoning budget is exhausted before producing visible output).

## License

MIT
