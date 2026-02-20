# LangChain Quickstart — Basic Agent

A hands-on notebook that walks from a minimal LangChain agent all the way to a production-ready one with external tools, structured output, runtime context, and conversational memory.

---

## What This Project Covers

The notebook (`notebooks/LLM-Chain.ipynb`) is split into two parts:

**Part 1 — Basic Agent**  
Spin up an agent in a few lines using `create_agent`, wire in a single tool, and get a response. The goal here is the simplest possible working example so you can see the moving parts clearly before adding complexity.

**Part 2 — Real-World Agent**  
Rebuild the agent step by step, introducing each production concept one cell at a time:

| Step | What it adds |
|------|--------------|
| 1. System prompt | Defines the agent's role and behavioral rules |
| 2. Tools | Two custom tools — one fetches weather, one reads user location from runtime context |
| 3. Model configuration | Temperature, timeout, retries, and token limits via `init_chat_model` |
| 4. Structured output | A `ResponseFormat` dataclass enforces a predictable response schema |
| 5. Memory | `InMemorySaver` checkpointer gives the agent cross-turn conversational memory |
| 6. Full run | Everything assembled and invoked end-to-end |

---

## Architecture

```
User message
     │
     ▼
  Agent  (create_agent)
     │
     ├──► get_user_location   (reads runtime context → returns city)
     │
     └──► get_weather_for_location  (city → weather string)
     │
     ▼
ResponseFormat(punny_response, weather_conditions)
```

The agent is powered by **Gemini 2.5 Flash** via `langchain-google-genai` and orchestrated by **LangGraph** under the hood.

---

## Repository Layout

```
.
├── notebooks/
│   └── LLM-Chain.ipynb      # Main tutorial notebook
├── requirements.txt          # Direct Python dependencies
├── .env.example              # Public template for environment variables
├── .gitignore                # Ignores secrets, caches, venvs, checkpoints
└── README.md
```

---

## Setup

**1. Create and activate a virtual environment**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

**2. Install dependencies**

```bash
pip install -r requirements.txt
```

**3. Configure environment variables**

```bash
cp .env.example .env
```

Open `.env` and set your Gemini API key:

```
GEMINI_API_KEY=your_key_here
```

The notebook loads this file automatically on startup via `python-dotenv`. If the variable is missing at runtime, it falls back to a `getpass` prompt so no key is ever hard-coded.

---

## Running the Notebook

```bash
jupyter notebook
```

Open `notebooks/LLM-Chain.ipynb` and run all cells from top to bottom. Each cell builds on the previous one, so order matters.

---

## Example Output

At the end of Part 2, the agent returns a structured object:

```python
ResponseFormat(
    punny_response="Looks like the weather in Florida is quite a *bright* spot! Hope you have a *sun-sational* day!",
    weather_conditions="It's always sunny in Florida!"
)
```

The `punny_response` field is always populated. `weather_conditions` carries any additional detail the model decides to surface.

---

## Key Concepts Explained

**`create_agent`**  
High-level LangChain factory that wires together a model, a list of tools, an optional system prompt, an optional response format, and an optional checkpointer into a runnable LangGraph agent.

**`ToolRuntime` and `context_schema`**  
Allows tools to receive request-scoped data (like a `user_id`) without it appearing in the model's tool signature. The agent receives a `Context` dataclass at invocation time and passes it transparently to any tool that declares a `runtime: ToolRuntime[Context]` parameter.

**`ToolStrategy(ResponseFormat)`**  
Instructs the agent to emit its final answer as a structured object matching the given schema rather than plain text. Backed by a hidden tool call internally.

**`InMemorySaver`**  
A LangGraph checkpointer that persists message history in memory, keyed by `thread_id`. Passing the same `thread_id` across multiple `.invoke()` calls gives the agent memory of previous turns.

---

## Security Notes

- Never commit `.env` or any file containing real API keys.
- Use `.env.example` as the public template — it should contain only placeholder values.
- Review notebook output cells before sharing; `print(result)` can expose full message history including tool responses.
- `GOOGLE_API_KEY` is explicitly removed from the environment at startup to prevent provider conflicts when `GEMINI_API_KEY` is present.
