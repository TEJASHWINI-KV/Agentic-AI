# 🧠 Agentic AI — Module 1 Complete Notes
> Personal reference notes with analogies, memory tricks & examples  
> Topics: LLMs → Tokens → Attention → Prompting → Tools → Python Basics

---

## 📌 TABLE OF CONTENTS
1. [What is an LLM?](#1-what-is-an-llm)
2. [Special Tokens](#2-special-tokens)
3. [Next Token Prediction & Autoregressive Decoding](#3-next-token-prediction--autoregressive-decoding)
4. [Beam Search](#4-beam-search)
5. [Attention Mechanism](#5-attention-mechanism)
6. [Prompting & Training](#6-prompting--training)
7. [Messages & Chat Templates](#7-messages--chat-templates)
8. [AI Tools](#8-ai-tools)
9. [Python — Functions](#9-python--functions)
10. [Python — Decorators](#10-python--decorators)
11. [Python — f-strings](#11-python--f-strings)
12. [Python — import vs from...import](#12-python--import-vs-fromimport)

---

## 1. What is an LLM?

### 🎯 One Line
> An LLM predicts the next token given all previous tokens. Everything else emerges from doing this at massive scale.

### 🌍 Analogy
> Like your phone's autocomplete — but trained on the entire internet, billions of times smarter.

### 🔑 Key Facts
| Fact | Detail |
|---|---|
| Full form | Large Language Model |
| Core job | Predict next token |
| Built on | Transformer architecture |
| Works with | Tokens (not whole words) |
| Vocabulary size | ~32,000 tokens (vs 600,000 English words) |

### 📦 3 Types of Transformers
| Type | Job | Analogy | Example | Size |
|---|---|---|---|---|
| **Encoder** | Reads → understands | Person who reads & summarises | BERT | Millions |
| **Decoder** | Writes → generates | Person who writes word by word | GPT-4, Llama | Billions |
| **Seq2Seq** | Reads then writes | Translator | T5, BART | Millions |

> ⚡ Most modern LLMs = **Decoder-only** models

### 🧠 Memory Trick
```
E = Encoder  → Reads (E for Examine)
D = Decoder  → Writes (D for Draft)
S = Seq2Seq  → Both   (S for Switch)
```

### 🪙 What is a Token?
- Sub-word unit — not a whole word
- `"interesting"` → `"interest"` + `"ing"` (2 tokens)
- `"interested"` → `"interest"` + `"ed"` (2 tokens)
- Think of tokens like **LEGO bricks** — small pieces that build any word

---

## 2. Special Tokens

### 🎯 One Line
> Special tokens are control signals that tell the LLM where things start, end, or change — like punctuation for AI.

### 🌍 Analogy
> A book with no punctuation, chapters, or paragraphs would be unreadable. Special tokens are the LLM's punctuation system.

### 🔑 Most Important: EOS Token
**EOS = End of Sequence** — tells the model to STOP generating.

### 📊 EOS Tokens by Model
| Model | Provider | EOS Token |
|---|---|---|
| GPT-4 | OpenAI | `<\|endoftext\|>` |
| Llama 3 | Meta | `<\|eot_id\|>` |
| DeepSeek-R1 | DeepSeek | `<\|end_of_sentence\|>` |
| SmolLM2 | HuggingFace | `<\|im_end\|>` |
| Gemma | Google | `<end_of_turn>` |

> ⚠️ Every model has its OWN special tokens — not universal

### 🧠 Memory Trick
> `im_end` = **i**nstruction **m**essage **end**  
> Special tokens are always wrapped in `< >` or `<| |>` so the model knows they're instructions, not real words.

---

## 3. Next Token Prediction & Autoregressive Decoding

### 🎯 One Line
> LLM generates one token at a time, feeding each output back as input, until it produces the EOS token.

### 🌍 Analogy
> Playing "finish the sentence" — you say one word, pass it back, say next word, repeat until done.

### 🔑 The Loop
```
Input → Tokenize → Model scores all vocab → Pick token → Append → Loop back
                                                                        ↓
                                                               Stop if EOS
```

### 🎛️ 3 Decoding Strategies
| Strategy | How | When to Use | Speed |
|---|---|---|---|
| **Greedy** | Always pick highest score | Factual Q&A, chatbots | ⚡ Fastest |
| **Sampling + Temperature** | Pick randomly (weighted) | Creative writing | Medium |
| **Beam Search** | Track top-N paths simultaneously | Translation, summarisation | 🐢 Slowest |

### 🌡️ Temperature Guide
```
0.1  → Almost no randomness  → Factual, consistent
0.7  → Balanced              → General chat
1.2+ → Very random           → Creative, surprising
```

### 🧠 Memory Trick
> **G**reedy = **G**o with #1 always  
> **S**ampling = **S**urprise me  
> **B**eam = **B**est overall path

---

## 4. Beam Search

### 🎯 One Line
> Keep multiple candidate sequences alive at every step, pick the best one at the end.

### 🌍 Analogy
> Like keeping 4 GPS routes open simultaneously instead of blindly following the first one. Pick the best route AFTER seeing how they all end.

### 🌍 Chess Analogy
| Type | Behaviour |
|---|---|
| Beginner (Greedy) | Best move RIGHT NOW → walks into trap 3 moves later |
| Expert (Beam Search) | Thinks 5 moves ahead → finds truly best move |

### 📊 Score System
- Scores are always **negative** (log probabilities)
- **Closer to 0 = more confident**
- `-0.1` = 90% sure ✅ | `-3.5` = barely fits ❌

### 🔑 Final Score Formula
```
Final Score = Cumulative Score / (output_length ^ length_penalty)
```
- Positive length penalty → prefers shorter sequences  
- Negative length penalty → prefers longer sequences

### 🧠 Memory Trick
> **B**eam = **B**ranching paths. Width = how many branches you keep alive.

---

## 5. Attention Mechanism

### 🎯 One Line
> Each word looks at ALL other words and decides which ones are most relevant — this gives transformers their deep language understanding.

### 🌍 Analogy
> "The trophy didn't fit in the suitcase because **it** was too big."  
> Your brain instantly focuses on "trophy" and ignores "the", "fit", "because".  
> **That selective focus = Attention.**

### 🔑 How it Works
```
"The capital of France is ___"

Attention scores:
  "The"     → 0.02  (ignore)
  "capital" → 0.45  ✅ important
  "of"      → 0.03  (ignore)
  "France"  → 0.48  ✅ important
  "is"      → 0.02  (ignore)

Result → "Paris"
```

### 🔑 Q / K / V (Query, Key, Value)
| Part | Role | Search Engine Analogy |
|---|---|---|
| **Q** (Query) | "What am I looking for?" | Your search term |
| **K** (Key) | "What do I contain?" | Webpage titles |
| **V** (Value) | "What info do I give?" | Webpage content |

### 📏 Context Length
| Model | Context Length |
|---|---|
| GPT-2 (2019) | 1,024 tokens |
| GPT-3 (2020) | 4,096 tokens |
| GPT-4 (2023) | 128,000 tokens |
| Gemini (2024) | 1,000,000 tokens |

> ⚠️ Attention cost = O(n²) — doubling context = **4x** the computation

### 🧠 Memory Trick
> **A**ttention = **A**sk every word "who matters to me?"  
> Multi-head = Multiple experts reading the same sentence, each noticing something different.

---

## 6. Prompting & Training

### 🎯 One Line
> Your prompt IS your program — the exact wording determines what the LLM gives back.

### 🌍 Analogy — Prompting
> Hiring a contractor: "Fix my house" → confused. "Repaint living room walls white, two coats, by Friday" → perfect result.

### 🌍 Analogy — Training
```
Phase 1 (Pre-training)  = University — reads everything, learns language
Phase 2 (Fine-tuning)   = Job training — specialised for a specific role
```

### 🔑 Pre-training (Self-Supervised)
```
"The Eiffel Tower is located in ___"
Model guesses "Paris" → ✅ reward
Model guesses "London" → ❌ penalty
Repeats billions of times → learns language
```
> No human labels needed — the next word IS the label.

### 🔑 Fine-tuning Types
| Type | What it Teaches | Example |
|---|---|---|
| Instruction tuning | Follow instructions | GPT-4 |
| Conversation tuning | Multi-turn dialogue | ChatGPT |
| Code tuning | Write/debug code | GitHub Copilot |
| Tool-use tuning | Call APIs/functions | Llama with tools |

### 🔑 Local vs Cloud API
| | Local | Cloud API |
|---|---|---|
| Cost | Free after setup | Pay per token |
| Privacy | 🔒 Data stays local | Data sent to server |
| Setup | Complex | Simple |
| Best for | Sensitive data | Prototyping |

### 📝 Prompt Best Practices
```
❌  "Explain AI"
✅  "Explain AI to a 10-year-old in 5 bullet points"

❌  No role given
✅  "You are an expert Python developer..."

❌  No format specified
✅  "Respond in a JSON format" / "Use bullet points"
```

### 🧠 Memory Trick
> **P**rompt = **P**rogram. Write it precisely or get garbage back.

---

## 7. Messages & Chat Templates

### 🎯 One Line
> The chat UI is an illusion — all messages are stitched into ONE big formatted string and fed to the LLM every single time.

### 🌍 Analogy
> A customer support hotline:  
> - Receptionist script = System Message  
> - Your conversation = User + Assistant messages  
> - Full call transcript handed to agent each time = what LLM actually receives

### 🔑 3 Message Types
```python
{"role": "system",    "content": "You are a polite assistant."}  # Rules/personality
{"role": "user",      "content": "I need help with my order"}    # Human input
{"role": "assistant", "content": "Sure! What's your order?"}     # AI response
```

### 🔑 What LLM Actually Sees (SmolLM2)
```
<|im_start|>system
You are helpful<|im_end|>
<|im_start|>user
I need help<|im_end|>
<|im_start|>assistant
Sure!<|im_end|>
<|im_start|>assistant    ← model continues here
```

### 🔑 Key Function
```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("HuggingFaceTB/SmolLM2-1.7B-Instruct")

rendered_prompt = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True   # adds opening tag for next assistant turn
)
```

### 🔑 Base vs Instruct Model
| | Base Model | Instruct Model |
|---|---|---|
| Training | Raw internet text | + Human conversations |
| Knows | Language patterns | How to follow instructions |
| Example | SmolLM2-135M | SmolLM2-135M-Instruct |
| Analogy | Read every book | Trained to be a helpful assistant |

### 🧠 Memory Trick
> **No memory in LLMs** — what looks like memory is the ENTIRE conversation history copy-pasted into every new prompt.  
> `apply_chat_template()` does the copy-pasting in the right format.

---

## 8. AI Tools

### 🎯 One Line
> A Tool is a Python function given to the LLM. The LLM writes text saying which tool to call — the Agent actually runs it.

### 🌍 Analogy
> Brilliant consultant (LLM) can't leave their office.  
> You give them tools: 📞 phone (web search), 🔢 calculator, 🌐 browser.  
> They tell YOU "call this number" — you do it, report back. They give the final answer.

### 🔑 What Every Tool Needs
| Part | Example |
|---|---|
| Name | `calculator` |
| Description | `"Multiply two integers"` |
| Arguments + types | `a: int, b: int` |
| Output type | `int` |

### 🔑 Tool-Calling Flow
```
1. Tool description → injected into System Prompt
2. User asks question
3. LLM outputs TEXT: "TOOL_CALL: get_weather('Paris')"
4. AGENT reads this, actually RUNS get_weather('Paris')
5. Result fed back into conversation
6. LLM generates final answer using real data
7. User sees only the final answer ← all tool steps hidden
```

### 🔑 `@tool` Decorator Pattern
```python
@tool
def calculator(a: int, b: int) -> int:
    """Multiply two integers."""
    return a * b

print(calculator.to_string())
# Tool Name: calculator, Description: Multiply two integers., Arguments: a: int, b: int, Outputs: int
```

### 🔑 Why Tools Exist
| Problem | Tool Solution |
|---|---|
| LLM bad at maths | Calculator tool |
| LLM has training cutoff | Web search tool |
| LLM can't send emails | Email API tool |
| LLM hallucinates weather | get_weather() tool |

### 🔑 MCP (Model Context Protocol)
> MCP = USB standard for AI tools.  
> Write a tool once → works in ANY MCP-compatible framework (LangChain, AutoGen, SmolAgents...)

### 🧠 Memory Trick
> LLM = **Brain** (thinks, decides)  
> Tools = **Hands** (acts in the world)  
> Agent = **Person** (has both brain and hands)

---

## 9. Python — Functions

### 🎯 One Line
> A function is a reusable block of code — define once, use many times.

### 🌍 Analogy
> A microwave — you don't know how it works inside. You just put food in, press a button, get hot food out. Use it as many times as you want.

### 🔑 Basic Structure
```python
#    ② name   ③ parameters
#      ↓           ↓
def calculate_area(length, width):   # ← ① def keyword
    area = length * width
    return area                      # ← ④ return value
```

### 🔑 Key Concepts
| Concept | Meaning | Example |
|---|---|---|
| `def` | "I am defining a function" | `def greet():` |
| Parameter | Empty slot in definition | `def greet(name)` ← name |
| Argument | Actual value passed in | `greet("Alice")` ← "Alice" |
| `return` | Give back a value | `return result` |
| Default value | Optional parameter | `def greet(name, greeting="Hello")` |

### 🔑 print vs return
```
Without return:  Chef cooks pizza, eats it himself → you can't use it ❌
With return:     Chef cooks pizza and HANDS IT TO YOU 🍕 ✅
```

```python
# Without return — result is lost
def add(a, b):
    print(a + b)       # shows it but doesn't give it back

x = add(3, 5)
print(x)               # None ← nothing!

# With return — result is usable
def add(a, b):
    return a + b

x = add(3, 5)
print(x)               # 8 ✅
```

### 🧠 Memory Trick
> **D**ef = **D**efine (build the microwave)  
> **Calling** = pressing the button  
> **return** = the food coming out

---

## 10. Python — Decorators

### 🎯 One Line
> A decorator wraps a function with extra behaviour — without changing the original function.

### 🌍 Analogy
> Plain coffee ☕ + milk + sugar = latte (still coffee inside, just wrapped with extras).  
> A decorator adds "extras" around a function without touching the function itself.

### 🔑 The Blueprint (memorise this!)
```python
def my_decorator(func):           # 1. Takes a function as input
    def wrapper(*args, **kwargs):  # 2. Defines a new wrapper function
        # --- extra code BEFORE ---
        result = func(*args, **kwargs)  # 3. Calls the original function
        # --- extra code AFTER ---
        return result              # 4. Returns the result
    return wrapper                 # 5. Returns wrapper (NOT wrapper()!)

@my_decorator                      # 6. Apply with @
def my_function():
    ...
```

### 🔑 `@` Symbol
```python
@twice
def wink():
    print("😉")

# EXACTLY same as:
wink = twice(wink)
```

### 🔑 Stacked Decorators — Bottom to Top
```python
@A      ← applied 2nd (outer)
@B      ← applied 1st (inner)
def f():
    ...

# Same as: f = A(B(f))
```

### 🔑 What Makes Something a Decorator?
| Factor | Matters? |
|---|---|
| Name (func, f, pizza...) | ❌ No — name anything |
| `@` symbol | ✅ Yes — this is the signal |
| Takes a function as input | ✅ Yes — required |
| Returns a function | ✅ Yes — required |

### ⚠️ Most Common Mistake
```python
return wrapper()   # ❌ WRONG — this CALLS wrapper immediately
return wrapper     # ✅ RIGHT — this RETURNS wrapper to be called later
```

### 🔑 Real-World Decorators You'll See
| Decorator | Where | What it does |
|---|---|---|
| `@login_required` | Flask/Django | Block unauthenticated users |
| `@app.route("/")` | Flask | Map URL to function |
| `@staticmethod` | Classes | Mark as static method |
| `@tool` | AI Agents | Turn function into a Tool object |
| `@timer` | Custom | Measure execution time |

### 🧠 Memory Trick
> `@` = "Apply this wrapper to the function below"  
> Decorator = function that **takes** a function and **returns** a function

---

## 11. Python — f-strings

### 🎯 One Line
> The `f` before a string turns it into a template — `{variable}` gets replaced with the actual value.

### 🌍 Analogy
> A fill-in-the-blank form:  
> `"Hello, ______!"` → `{name}` is the blank → Python fills it with the value of `name`

### 🔑 Basic Usage
```python
name = "Alice"

# Normal string — prints literally
print("Hello, {name}!")    # Hello, {name}!  ❌

# f-string — replaces {name} with value
print(f"Hello, {name}!")   # Hello, Alice!   ✅
```

### 🔑 What Goes Inside `{ }`
```python
name  = "Alice"
score = 95

print(f"Name: {name}")              # variable
print(f"Score: {score}")            # number (auto-converted to string)
print(f"2+2 = {2 + 2}")            # calculation
print(f"Upper: {name.upper()}")     # method call
print(f"Pass: {score >= 50}")       # comparison → True/False
```

### 🔑 3 Rules
```
1. f directly before quote — no space:  f"..."  not  f "..."
2. Variable inside { }:  f"{name}"  not  f"name"
3. Variable must exist before the f-string runs
```

### 🔑 f-string vs Old Way
```python
# Old messy way ❌
print("Hello " + name + "! Score: " + str(score))

# Clean f-string ✅
print(f"Hello {name}! Score: {score}")
```

### 🧠 Memory Trick
> `f` = **F**ill in the blanks  
> `{ }` = the blanks to fill

---

## 12. Python — import vs from...import

### 🎯 One Line
> Both load code from a library. The difference is HOW MUCH you bring in and HOW you use it.

### 🌍 Analogy
> `import requests` = Bring the **whole toolbox** into your garage  
> `from requests import get` = Bring **just the hammer**

### 🔑 Side by Side
```python
# import — brings whole library, use with prefix
import requests
response = requests.get("https://...")   # must say requests.get

# from...import — brings one specific thing, use directly
from requests import get
response = get("https://...")            # just get, no prefix
```

### 🔑 Built-in vs Third-Party
| | Built-in | Third-Party |
|---|---|---|
| Example | `typing`, `os`, `math` | `requests`, `transformers` |
| Comes with Python? | ✅ Yes | ❌ No — pip install |
| Works at runtime? | Labels only (typing) | Does real work |
| Syntax | Both `import` and `from` work | Both `import` and `from` work |

> ⚠️ `import` vs `from` has NOTHING to do with built-in vs third-party!

### 🔑 Module System Hierarchy
```
📦 Package     (a folder of related code)
  └── 📄 Module   (a single .py file)
        └── 🔧 Function / Class / Variable
```

### 🔑 When to Use Which
| Use | When |
|---|---|
| `import requests` | Need many things from library |
| `from requests import get` | Need just one specific thing |
| `import requests as req` | Want a shorter alias |

### 🧠 Memory Trick
> Choose based on **how much you need** — not where it came from.  
> Whole toolbox = `import`. Just the hammer = `from x import y`.

---

## 🗺️ The Full Agentic AI Picture

```
LLM (Brain)
  │
  ├── Trained on text (pre-training → fine-tuning)
  ├── Reads tokens (not whole words)
  ├── Uses Attention to understand relationships
  ├── Generates one token at a time (autoregressive)
  ├── Stops at EOS token
  │
  ├── Receives input via Chat Template
  │     ├── System Message  (rules/personality)
  │     ├── User Messages   (human input)
  │     └── Assistant Messages (AI responses)
  │
  └── Extended with Tools
        ├── Tool descriptions injected in System Prompt
        ├── LLM writes TOOL_CALL: tool_name(args)
        ├── Agent runs the actual tool
        └── Result fed back → LLM gives final answer
```

---

## ⚡ Master Cheat Sheet

| Concept | One Line | Memory Trick |
|---|---|---|
| LLM | Predicts next token | Phone autocomplete × 1 billion |
| Token | Sub-word unit | LEGO brick |
| EOS | Stop signal | Period at end of sentence |
| Autoregressive | Output becomes next input | Finish-the-sentence game |
| Greedy | Always pick #1 token | Fast but boring |
| Beam Search | Track N best paths | GPS with 4 routes open |
| Temperature | Randomness knob | 0=robot, 1.5=poet |
| Attention | Words looking at words | Highlighting what matters |
| Context Length | Max tokens LLM can see | Spotlight (not memory) |
| Prompt | Your input to LLM | Your prompt IS your program |
| System Message | AI's backstage rules | Receptionist script |
| Chat Template | Formats messages into one string | Mail merge template |
| Tool | Function given to LLM | LLM's hands |
| Decorator | Wraps function with extras | Coffee with milk & sugar |
| f-string | Template string | Fill-in-the-blank form |
| `import` | Load whole library | Whole toolbox |
| `from x import y` | Load one thing | Just the hammer |

---

> 📅 Last updated: March 2026  
> 🗺️ Next: Unit 2 — Agent Actions, Planning & Memory
