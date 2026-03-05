# 🤖 AI Agents — Complete Study Notes
> Unit 1 | From scratch to smolagents | All concepts, analogies & memory tricks

---

## 📌 TABLE OF CONTENTS
1. [What is an AI Agent?](#1-what-is-an-ai-agent)
2. [The ReAct Cycle — Thought → Action → Observation](#2-the-react-cycle)
3. [Thought — Internal Reasoning & CoT](#3-thought--cot--react)
4. [Actions — How Agents Do Things](#4-actions)
5. [Observations — Feedback & Memory](#5-observations)
6. [Serverless API](#6-serverless-api)
7. [Dummy Agent Library](#7-dummy-agent-library)
8. [smolagents — The Real Framework](#8-smolagents)
9. [Master Cheat Sheet](#9-master-cheat-sheet)
10. [All Analogies Quick Reference](#10-all-analogies)

---

## 1. What is an AI Agent?

### 🧠 One Line
> An AI Agent is an LLM given **tools** and allowed to **loop** (think → act → observe) until a task is done.

### 🍳 Analogy — The Chef
> A chef given a request: *"Make me a dish I'll love."*
> - **Thinks** → "They like spicy food, let me check the fridge"
> - **Acts** → Opens fridge
> - **Observes** → "We have eggs, chili, garlic"
> - Loops until dish is served ✅

### 🔑 Key Difference
| Chatbot | AI Agent |
|---|---|
| One question → one answer | Loops until task is done |
| No tools | Has tools (APIs, code, search) |
| Static knowledge | Can fetch real-time data |
| Single LLM call | Multiple LLM calls in a loop |

### 💡 Memory Trick
> **Agent = LLM + Tools + Loop**
> Remove any one → it's no longer an agent

---

## 2. The ReAct Cycle

### 🧠 One Line
> ReAct = **Re**ason + **Act** — the agent loops through Think → Act → Observe until done.

### 🕵️ Analogy — The Detective
> A detective doesn't just arrest someone — they:
> - **Think** → "The fingerprints match but motive is unclear"
> - **Act** → Calls the lab for phone records
> - **Observe** → "Suspect was in another city"
> - Loops until case is solved ✅

### 📊 The Loop
```
User Question
      ↓
   THOUGHT  ←─────────────────────┐
   (What should I do next?)       │
      ↓                           │
   ACTION                         │
   (Call a tool with JSON/code)   │
      ↓                           │
   OBSERVATION                    │
   (Tool returns result)          │
      ↓                           │
   Is task done? ──── NO ─────────┘
      │
     YES
      ↓
   FINAL ANSWER → User ✅
```

### 🔑 Key Points
- The **LLM never runs tools** — it only writes text describing what it wants to call
- The **framework** reads that text and actually runs the tool
- The loop continues until `Final Answer` is produced or max iterations hit
- **Always cap iterations** → prevents infinite loops

### 💡 Memory Trick
> **"While not done: Think → Act → Observe"**
> It's literally a `while` loop in your code

---

## 3. Thought / CoT / ReAct

### 🧠 Three Levels of Thinking

| Method | What it is | Uses Tools? | How to trigger |
|---|---|---|---|
| **Plain LLM** | Direct answer | ❌ | Just ask |
| **CoT** | Step-by-step reasoning | ❌ | "Let's think step by step" |
| **ReAct** | Reasoning + tool calls | ✅ | System prompt with tools |
| **`<think>` tokens** | Built-in reasoning (o1, R1) | ✅ | Automatic (model-level) |

### 🎓 CoT — Chain of Thought
> Just add **"Let's think step by step"** to your prompt.
> The LLM breaks the problem into small parts before answering.

```
# Without CoT:  "What is 15% of 200?" → LLM guesses → "30" (sometimes wrong)
# With CoT:
"Let's think step by step. What is 15% of 200?"
→ 10% of 200 = 20
→ 5%  of 200 = 10
→ 15% = 20 + 10 = 30 ✅  (reliable)
```

### 8 Types of Thought
| Type | Example |
|---|---|
| Planning | "Break this into 3 steps" |
| Analysis | "The error is in line 42" |
| Decision Making | "Given the budget, use mid-tier" |
| Problem Solving | "Profile first, then optimise" |
| Memory Integration | "User prefers Python examples" |
| Self-Reflection | "Last approach failed, try differently" |
| Goal Setting | "First establish acceptance criteria" |
| Prioritization | "Fix security before new features" |

### ⚠️ CoT Gotcha
> CoT can **confidently hallucinate** — it reasons well on false premises.
> If the starting fact is wrong, the conclusion will be wrong too (but sound logical).
> **Fix:** Use ReAct to fact-check via tools first.

### 💡 Memory Trick
> **CoT = Thinking out loud (no phone)**
> **ReAct = Thinking out loud + making phone calls**

---

## 4. Actions

### 🧠 One Line
> An Action is what the agent actually **does** after thinking — calling a tool with specific inputs.

### 🗒️ Analogy — The Sticky Note
> The agent writes its instruction on a sticky note in a specific format, then hands it over.
> Someone else reads the note, executes it, brings back the result.
> The agent **stops writing** once the note is complete = **Stop and Parse**.

### 3 Types of Agents by Action Format

#### 🟡 JSON Agent
```json
{
  "action": "get_weather",
  "action_input": {"location": "New York"}
}
```
- Simple, structured, easy to parse
- Best for single tool calls

#### 🔵 Code Agent
```python
result = get_weather("New York")
print(f"Final Answer: {result}")
```
- More powerful — can do loops, conditions
- ⚠️ Security risk — never `exec()` untrusted code

#### 🟢 Function-calling Agent
```python
{"name": "get_weather", "arguments": '{"location": "New York"}'}
```
- Fine-tuned model feature (OpenAI, Anthropic)
- Easiest to parse — built into the API

### 🛑 Stop and Parse — The Most Important Concept
```
1. LLM generates action (JSON/code)
2. LLM STOPS ← critical — extra text breaks the parser
3. Framework PARSES the output
4. Framework EXECUTES the tool
5. Result returned as Observation
```
> ⚠️ Without STOP: LLM hallucinates the Observation and makes up fake tool results!

### 4 Types of Actions
| Type | Example |
|---|---|
| Information Gathering | Web search, DB query |
| Tool Usage | API calls, calculations |
| Environment Interaction | Click button, control device |
| Communication | Chat with user, message other agents |

### 💡 Memory Trick
> **"LLM writes the order. Framework delivers it."**
> The LLM is the manager writing instructions. Your code is the worker executing them.

---

## 5. Observations

### 🧠 One Line
> An Observation is the **tool's result appended to the agent's memory** so it can decide what to do next.

### 🍳 Analogy — Opening the Oven Door
> You put cake in oven (Action) → open door to check (Observation) → "still raw, 10 more minutes"
> Without opening the door → you're blind → you'd just guess

### The 3-Step Framework Process
```
After every Action:

STEP 1 — PARSE    → Which tool? What arguments?
STEP 2 — EXECUTE  → Actually call the Python function
STEP 3 — APPEND   → Add result to context as "Observation: ..."
```

### Why Append to Context?
```python
# LLMs are STATELESS — zero memory between calls
# The ONLY memory = text in the context window

# The agent's "notepad" (context list):
context = [
    "User: What's the weather in Tokyo?",
    "Thought: I'll call the weather tool.",
    "Action: get_weather('Tokyo')",
    "Observation: Sunny, 22°C",     # ← THIS LINE changes everything
    "Thought: I have the data. Final answer time!"
]
# "\n".join(context) → sent to LLM on every cycle
```

### 5 Types of Observations
| Type | Example |
|---|---|
| System Feedback | `"HTTP 404 Not Found"` |
| Data Changes | `"3 rows updated in DB"` |
| Environmental Data | `"CPU 87%, Temp 72°C"` |
| Response Analysis | `"{'price': 182.34}"` |
| Time-based Events | `"Backup completed at 3am"` |

### ⚠️ The Golden Rules
```python
# ✅ ALWAYS produce an observation — even on failure
try:
    result = call_tool()
    context.append(f"Observation: {result}")
except Exception as e:
    context.append(f"Observation: Tool failed → {e}. Try differently.")
    # Without this → agent loops forever repeating the same failed action

# ✅ ALWAYS convert tool output to string before appending
context.append(f"Observation: {str(result)}")  # not context.append(result_dict)

# ✅ ALWAYS append — never just store in a variable
observation = tool()        # ← computed
context.append(observation) # ← this line IS the memory update
```

### 💡 Memory Trick
> **"The Observation is writing in the agent's diary."**
> Skip it → agent has amnesia → repeats same action forever.

---

## 6. Serverless API

### 🧠 One Line
> Run code/models in the cloud — no server to set up, pay only per use.

### 🏨 Analogy
> **Traditional Server = Owning a Hotel** — pay 24/7, manage everything
> **Serverless = Airbnb** — room appears when guest arrives, disappears when they leave, pay per stay

### Key Facts
```
✅ No server to manage
✅ Pay per request (idle = free)
✅ Auto-scales
✅ Just write a function, get a URL
⚠️ Cold start — first request slightly slow
⚠️ Stateless — remembers nothing between calls (like LLMs!)
⚠️ Max runtime limit (15 min on AWS Lambda)
```

### In Python (HuggingFace)
```python
from huggingface_hub import InferenceClient  # [LIBRARY]

client = InferenceClient(model="moonshotai/Kimi-K2.5")

output = client.chat.completions.create(
    messages=[{"role": "user", "content": "Hello"}],
    stream=False,
    max_tokens=1024,
)
print(output.choices[0].message.content)
```

### 💡 Memory Trick
> **"Serverless = the cloud runs your function when called, bills you per second, then shuts it down."**
> You write the function. The cloud owns the server.

---

## 7. Dummy Agent Library

### 🧠 One Line
> Building an agent completely by hand — so you understand what frameworks do automatically.

### Why "Dummy"?
> Not stupid — simplified on purpose.
> Like learning to drive with no power steering → harder, but you understand every mechanism.

### The Hallucination Problem
```
# ❌ WITHOUT stop token:
LLM output:
  Thought: I need weather in London.
  Action: {"action": "get_weather", ...}
  Observation: It's 12°C ← LLM MADE THIS UP! No API was called!
  Final Answer: It's 12°C ← based on a lie!

# ✅ WITH stop=["Observation:"]:
LLM output:
  Thought: I need weather in London.
  Action: {"action": "get_weather", ...}
  ← STOPS HERE. You call get_weather() for real, append real result.
```

### The Full Manual Loop
```python
import json, re
from huggingface_hub import InferenceClient

client = InferenceClient(model="moonshotai/Kimi-K2.5")

# Step 1 — Send system prompt + user question
messages = [
    {"role": "system", "content": SYSTEM_PROMPT},
    {"role": "user",   "content": "What's the weather in London?"},
]

# Step 2 — LLM thinks and writes action, STOPS before observation
response = client.chat.completions.create(
    messages=messages,
    max_tokens=150,
    stop=["Observation:"],   # ← THE magic line
)
llm_text = response.choices[0].message.content

# Step 3 — Parse → Execute → Append
parsed      = json.loads(re.search(r"\{.*\}", llm_text, re.DOTALL).group(0))
tool_result = get_weather(parsed["action_input"]["location"])

# Step 4 — Append real observation, send back
messages.append({
    "role":    "assistant",
    "content": llm_text + "Observation:\n" + tool_result
})

# Step 5 — LLM reads real observation → Final Answer
final = client.chat.completions.create(messages=messages, max_tokens=200)
print(final.choices[0].message.content)
```

### Manual vs Framework
| Task | Manual (Dummy) | smolagents |
|---|---|---|
| System prompt | You write ~50 lines | Auto-generated |
| Stop token | `stop=["Observation:"]` | Built in |
| JSON parsing | `json.loads()` manually | Built in |
| Observation append | `.append()` manually | Built in |
| The loop | `while` loop you write | `agent.run()` |
| Lines of code | ~50+ | ~10 |

### 💡 Memory Trick
> **"Dummy agent = driving with manual transmission."**
> Hard, but once you understand it — automatics (frameworks) make total sense.

---

## 8. smolagents

### 🧠 One Line
> A lightweight Hugging Face library that runs the full ReAct loop automatically — you just define tools and call `agent.run()`.

### Install
```bash
pip install smolagents
```

### Minimum Working Example
```python
from smolagents import CodeAgent, tool, HfApiModel

# Define a tool — @tool decorator registers it
@tool
def get_weather(location: str) -> str:
    """Get the current weather for a given city.   ← LLM reads this docstring
    Args:
        location: The city name to get weather for
    """
    return f"Sunny and 22°C in {location}"

# Set up the LLM
model = HfApiModel(model_id="Qwen/Qwen2.5-Coder-32B-Instruct")

# Create the agent
agent = CodeAgent(tools=[get_weather], model=model)

# Run — this IS the entire ReAct loop
result = agent.run("What's the weather in London?")
print(result)
```

### Key Classes
| Class | What it does |
|---|---|
| `CodeAgent` | Agent that writes Python code to call tools |
| `ToolCallingAgent` | Agent that writes JSON to call tools |
| `HfApiModel` | Connects to HuggingFace Serverless API |
| `InferenceClientModel` | Same as above (alternative name) |
| `DuckDuckGoSearchTool()` | Ready-made web search tool |
| `load_tool("...")` | Loads a tool from HuggingFace Hub |
| `FinalAnswerTool()` | Required tool — lets agent finish |

### @tool Decorator Rules
```python
@tool
def my_tool(arg1: str, arg2: int) -> str:  # ✅ type hints required
    """Short description of what this does   # ✅ docstring required — LLM reads it
    Args:
        arg1: description of arg1            # ✅ every arg must be described
        arg2: description of arg2
    """
    return "result"                          # ✅ always return a string
```

### Adding Tools to Agent
```python
agent = CodeAgent(
    model=model,
    tools=[
        final_answer,                  # ← NEVER remove this
        DuckDuckGoSearchTool(),        # ← web search
        image_generation_tool,         # ← generate images
        get_current_time_in_timezone,  # ← timezone tool
        my_custom_tool,                # ← your own tools
    ],
    max_steps=6,                       # ← safety cap on loop iterations
)
```

### System Prompt
```python
# Stored in prompts.yaml — loaded like this:
import yaml
with open("prompts.yaml", 'r') as stream:
    prompt_templates = yaml.safe_load(stream)

agent = CodeAgent(..., prompt_templates=prompt_templates)
```

### 💡 Memory Trick
> **"smol = small. smolagents = small but complete agent framework."**
> `@tool` = register. `agent.run()` = entire ReAct loop. That's it.

---

## 9. Master Cheat Sheet

### The Full Picture
```
┌─────────────────────────────────────────────────────────────────┐
│                      AI AGENT ANATOMY                           │
├─────────────────────────────────────────────────────────────────┤
│  LLM (brain)     → Does the thinking (Thought step)            │
│  Tools           → Python functions the agent can call          │
│  System Prompt   → Rules + available tools baked in at start    │
│  Context Window  → The agent's only memory (a list of strings)  │
│  ReAct Loop      → while not done: Think → Act → Observe        │
│  Stop Token      → Prevents LLM from hallucinating observations │
│  Framework       → Automates the loop (smolagents, LangChain)   │
└─────────────────────────────────────────────────────────────────┘
```

### Critical Rules — Never Break These
```
1. ALWAYS append observations to context      → memory update
2. ALWAYS handle tool errors as observations  → prevents infinite loops
3. ALWAYS use stop=["Observation:"]           → prevents hallucination
4. ALWAYS cap max_iterations                  → prevents infinite loops
5. NEVER remove final_answer from tools list  → agent can never finish
6. NEVER exec() untrusted code                → security risk
7. NEVER store function with () in TOOLS dict → calls it immediately
```

### Python Concepts Used
| Concept | Tag | Example |
|---|---|---|
| `import` | [KEYWORD] | `import json` |
| `def` | [KEYWORD] | `def get_weather(city):` |
| `return` | [KEYWORD] | `return "result"` |
| `for`, `in` | [KEYWORD] | `for item in list:` |
| `if`, `elif`, `else` | [KEYWORD] | `if status == 200:` |
| `try`, `except` | [KEYWORD] | `try: ... except Exception as e:` |
| `class` | [KEYWORD] | `class MyAgent:` |
| `json` | [MODULE] | `json.loads()`, `json.dumps()` |
| `re` | [MODULE] | `re.search(r"\{.*\}", text)` |
| `os` | [MODULE] | `os.environ.get("HF_TOKEN")` |
| `smolagents` | [LIBRARY] | `pip install smolagents` |
| `huggingface_hub` | [LIBRARY] | `pip install huggingface_hub` |
| `requests` | [LIBRARY] | `pip install requests` |
| `pytz` | [LIBRARY] | `pip install pytz` |
| `print()` | [BUILT-IN] | `print(result)` |
| `len()` | [BUILT-IN] | `len(context)` |
| `enumerate()` | [BUILT-IN] | `for i, v in enumerate(list)` |
| `range()` | [BUILT-IN] | `for i in range(5)` |
| `str()` | [BUILT-IN] | `str(error)` |
| `**kwargs` | Python feature | `tool_fn(**{"city": "Tokyo"})` |
| `f-string` | Python feature | `f"Hello {name}"` |
| `[:n]` slice | Python feature | `context[-1]`, `text[:50]` |
| `@decorator` | Python feature | `@tool` |

---

## 10. All Analogies

| Concept | Analogy |
|---|---|
| AI Agent | Chef cooking in a loop — checks, adjusts, serves |
| ReAct Cycle | Detective investigating — thinks, checks evidence, adapts |
| Thought step | Chess player's silent planning before moving |
| CoT | Showing your working in a maths exam |
| Action | Sticky note handed to someone else to execute |
| Stop and Parse | Finishing a sentence and putting the pen down |
| Observation | Opening the oven door to check the cake |
| Appending observation | Writing in a diary — the only memory |
| Serverless API | Airbnb vs owning a hotel |
| Dummy Agent | Manual transmission car — hard but teaches the mechanics |
| smolagents | Automatic transmission — same mechanics, much easier |
| Context window | Agent's notepad — the ONLY memory it has |
| LLM statelessness | Talking to someone with amnesia — must restate everything each time |

---

## 🏆 Top 10 Most Important Things to Remember

```
1.  Agent = LLM + Tools + Loop (remove any one = not an agent)
2.  LLM never RUNS tools — it only WRITES text. Framework runs tools.
3.  stop=["Observation:"] prevents hallucination — most critical line
4.  Appending observation to context IS the memory update
5.  Errors must also be observations — silence causes infinite loops
6.  CoT = "Let's think step by step" — no tools, pure reasoning
7.  ReAct = CoT + tool calls interleaved
8.  <think> tokens (o1, R1) = training-level, not prompting-level
9.  @tool decorator + docstring = how smolagents registers tools
10. agent.run() = entire manual ReAct loop in one line
```

---

*Notes compiled from Unit 1 of the Hugging Face Agents Course*
*Topics: ReAct Cycle, CoT, Actions, Observations, Serverless API, Dummy Agent, smolagents*
