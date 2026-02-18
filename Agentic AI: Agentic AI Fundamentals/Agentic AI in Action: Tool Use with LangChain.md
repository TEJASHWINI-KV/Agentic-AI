# LangChain Tool Use: Complete Guide for Future Analysis

**Date:** February 18, 2026  
**Source:** LangChain Tool Use Course Transcript  
**Purpose:** Comprehensive reference for implementing agent tools in production

---

## Table of Contents

1. [Overview & Key Concepts](#overview--key-concepts)
2. [Installation & Setup](#installation--setup)
3. [Demo 1: Calculator Tool](#demo-1-calculator-tool)
4. [Demo 2: Built-in Search Tools](#demo-2-built-in-search-tools)
5. [Demo 3: Custom Weather Tool](#demo-3-custom-weather-tool)
6. [Demo 4: Multi-Tool Reasoning](#demo-4-multi-tool-reasoning)
7. [Best Practices & Lessons Learned](#best-practices--lessons-learned)
8. [Comparison: Tools vs Plain LLMs](#comparison-tools-vs-plain-llms)
9. [Implementation Patterns](#implementation-patterns)
10. [Common Use Cases](#common-use-cases)

---

## Overview & Key Concepts

### What Are Tools?

Tools are **wrapped Python functions** that LangChain agents can call dynamically. They extend LLM capabilities by providing:

- **Deterministic operations** (math, logic)
- **Real-time data access** (web searches, APIs)
- **Business logic integration** (custom functions)
- **Safe execution** (sandboxed operations)

### Why Tools Matter

| Aspect | Plain LLM | With Tools |
|--------|-----------|-----------|
| **Math Accuracy** | ❌ Unreliable, patterns-based | ✅ Deterministic, exact |
| **Current Info** | ❌ Limited to training data | ✅ Real-time via APIs |
| **Reliability** | ❌ Hallucinations possible | ✅ Function guarantees |
| **Speed** | ✅ Fast | ✅ Depends on tool |
| **Customization** | ❌ Hard to align | ✅ Easy to control |

### Tool Lifecycle

```
1. User Query
   ↓
2. Agent Interprets Question
   ↓
3. Agent Selects Appropriate Tool(s)
   ↓
4. Tool Executes (Safe, Sandboxed)
   ↓
5. Agent Processes Results
   ↓
6. Agent Returns Final Answer
```

---

## Installation & Setup

### Required Libraries

```bash
# Core LangChain
pip install langchain -qU

# Gemini Integration
pip install langchain-google-genai -qU
pip install google-generativeai -qU

# Search Tools
pip install ddgs -qU  # DuckDuckGo Search (updated name)
pip install wikipedia -qU

# Community Tools
pip install langchain-community -qU
```

### API Key Configuration

```python
import os
from google.colab import userdata

# Securely fetch API key
GEMINI_API_KEY = userdata.get('GEMINI_API_KEY')
os.environ['GOOGLE_API_KEY'] = GEMINI_API_KEY

print("✓ Gemini API key configured successfully")
```

### Core Imports Pattern

```python
# Python Standard Library
import ast
import operator
from typing import Union, List
from datetime import datetime
from zoneinfo import ZoneInfo

# LangChain Core
from langchain.tools import tool
from langchain.chat_models import ChatGoogleGenerativeAI
from langchain.prompts import ChatPromptTemplate
from langchain.agents import AgentExecutor, create_tool_calling_agent

# Community Tools
from langchain.agents import initialize_agent, AgentType
from langchain_community.tools import DuckDuckGoSearchRun, WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
```

---

## Demo 1: Calculator Tool

### Problem Statement

Language models struggle with exact arithmetic due to their token-based nature. They predict patterns but don't compute reliably.

**Example of LLM failure:**
```
Query: Compute (12345 * 6789) + 98765
LLM Output: Approximately 83 million (incorrect/vague)
Expected: 83,812,690
```

### Solution: Safe Calculator Tool

#### Step 1: Define Safe Operations Mapping

```python
import ast
import operator

_OPS = {
    ast.Add: operator.add,
    ast.Sub: operator.sub,
    ast.Mult: operator.mul,
    ast.Div: operator.truediv,
    ast.Mod: operator.mod,
    ast.Pow: operator.pow,
    ast.USub: operator.neg,
    ast.UAdd: operator.pos,
}
```

**Why this is safe:**
- Only whitelisted operations allowed
- No file system access
- No imports or arbitrary code execution
- Prevents malicious input

#### Step 2: Implement Safe Node Evaluation

```python
def _eval_node(node):
    """Recursively evaluate an AST node safely."""
    
    # Base case: constant number
    if isinstance(node, ast.Constant):
        return node.value
    
    # Binary operations (a + b, a * b, etc.)
    if isinstance(node, ast.BinOp):
        left = _eval_node(node.left)
        right = _eval_node(node.right)
        op = _OPS.get(type(node.op))
        if op is None:
            raise ValueError(f"Unsupported operation: {type(node.op)}")
        return op(left, right)
    
    # Unary operations (-a, +a, etc.)
    if isinstance(node, ast.UnaryOp):
        operand = _eval_node(node.operand)
        op = _OPS.get(type(node.op))
        if op is None:
            raise ValueError(f"Unsupported unary operation: {type(node.op)}")
        return op(operand)
    
    # Wrapped expression
    if isinstance(node, ast.Expression):
        return _eval_node(node.body)
    
    # If we get here, something is unsupported
    raise ValueError(f"Unsupported AST node type: {type(node)}")
```

#### Step 3: User-Facing Calculator Function

```python
def safe_calc(expression: str) -> float:
    """Safe, sandboxed calculator.
    
    Args:
        expression: Math expression as string (e.g., "12 + 5 * 3")
    
    Returns:
        float: The computed result
    
    Raises:
        ValueError: If expression is invalid or too complex
    """
    
    # Clean input
    expression = expression.strip()
    
    # Prevent abuse - reject overly long expressions
    if len(expression) > 200:
        raise ValueError("Expression too long (max 200 chars)")
    
    try:
        # Parse into Abstract Syntax Tree
        tree = ast.parse(expression, mode='eval')
        # Evaluate safely
        result = _eval_node(tree.body)
        return float(result)
    
    except (SyntaxError, ValueError) as e:
        raise ValueError(f"Invalid expression: {e}")
```

#### Step 4: Wrap as LangChain Tool

```python
@tool
def calculate(expression: str) -> str:
    """Execute a math expression safely and return the result.
    
    This tool:
    - Supports: +, -, *, /, %, ** (power)
    - Rejects: file operations, imports, arbitrary code
    - Handles errors gracefully
    
    Examples:
        - "12 + 5" → 17.0
        - "(100 - 50) / 2" → 25.0
        - "2 ** 10" → 1024.0
    
    Args:
        expression: Math expression as string
    
    Returns:
        String with result or error message
    """
    try:
        result = safe_calc(expression)
        
        # Handle very large numbers
        if abs(result) > 10**15:
            return f"Error: Result too large: {result}"
        
        # Return clean integer if possible
        if result == int(result):
            return str(int(result))
        
        return str(result)
    
    except Exception as e:
        return f"Error: {str(e)}"
```

### Building the Calculator Agent

```python
from langchain.chat_models import ChatGoogleGenerativeAI
from langchain.prompts import ChatPromptTemplate
from langchain.agents import create_tool_calling_agent, AgentExecutor

# Initialize LLM
llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0  # Deterministic output
)

# System prompt with strict rules
prompt = ChatPromptTemplate.from_messages([
    ("system", """You are a precise math assistant. RULES:
    1. ALWAYS use the calculate tool - never do math in your head
    2. Convert symbols: ^ becomes **, / stays as /, etc.
    3. Return ONLY the clean numeric result when asked
    4. For compound expressions, show your work step by step
    """),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

# Create agent
agent = create_tool_calling_agent(
    llm=llm,
    tools=[calculate],
    prompt=prompt
)

# Wrap in executor
agent_executor = AgentExecutor(
    agent=agent,
    tools=[calculate],
    verbose=False
)

# Test
result = agent_executor.invoke({
    "input": "Compute (12345 * 6789) + 98765"
})
print(result["output"])  # 83812690
```

### Key Insights from Demo 1

| Aspect | Learning |
|--------|----------|
| **LLM vs Tool** | LLMs make mistakes; tools are deterministic |
| **Safety** | Whitelist operations, reject unknown AST nodes |
| **Length Limits** | Prevent abuse: cap input at 200 chars |
| **Type Hints** | Return float internally, string for tool output |
| **Error Handling** | Catch all exceptions, return user-friendly messages |

---

## Demo 2: Built-in Search Tools

### Problem Statement

Language models have knowledge cutoff dates. They can't answer:
- Current events or news
- Real-time data
- Recent developments
- Live market information

**Example of limitation:**
```
Q: "Who won the FIFA World Cup 2022?"
LLM might say: "I don't know, my training data ends in April 2023"
With DuckDuckGo: Gets current, sourced answer
```

### Solution: DuckDuckGo Search Tool

#### Step 1: Helper Function to Format Results

```python
def _format_hits(hits: List[dict], max_results: int = 5) -> str:
    """Format DuckDuckGo search results into readable list.
    
    Raw API response is messy; this cleans it up.
    
    Args:
        hits: Raw search results from DDGS
        max_results: Max results to return
    
    Returns:
        Formatted string with numbered results
    """
    
    if not hits:
        return "No results found."
    
    formatted = []
    for i, hit in enumerate(hits[:max_results], 1):
        title = hit.get('title', 'No title')
        url = hit.get('href', 'No URL')
        snippet = hit.get('body', 'No snippet')[:150]  # First 150 chars
        
        formatted.append(f"{i}. [{title}]({url})\n   {snippet}...")
    
    return "\n".join(formatted)
```

#### Step 2: Create Web Search Tool

```python
from duckduckgo_search import DDGS

@tool
def web_search(query: str, max_results: int = 5, timelimit: str = None) -> str:
    """Search the web using DuckDuckGo for current information.
    
    This tool:
    - Returns real-time results
    - Supports filtering by recency
    - Includes titles, URLs, and snippets
    
    Args:
        query: Search query string
        max_results: Number of results (default 5)
        timelimit: Filter by recency ('d' for day, 'w' for week, 
                   'm' for month, 'y' for year)
    
    Examples:
        - "latest AI developments 2025"
        - "Bitcoin price today"
        - "LangChain recent updates"
    
    Returns:
        Formatted list of search results with links
    """
    try:
        with DDGS() as ddgs:
            hits = ddgs.text(
                query,
                max_results=max_results,
                timelimit=timelimit
            )
            return _format_hits(hits, max_results)
    
    except Exception as e:
        return f"Search error: {str(e)}"
```

**Key parameters:**
- `timelimit='d'` - Past day
- `timelimit='w'` - Past week
- `timelimit='m'` - Past month
- `timelimit='y'` - Past year

### Solution: Wikipedia Tool

#### Implementation

```python
import wikipedia
from wikipedia import exceptions

@tool
def wiki_summary(topic: str) -> str:
    """Get structured Wikipedia summary about a topic.
    
    This tool:
    - Returns factual background information
    - Includes page URL
    - Handles disambiguation and errors gracefully
    
    Args:
        topic: Topic name (e.g., "Albert Einstein", "Machine Learning")
    
    Returns:
        Wikipedia summary (first 500 chars) with URL
    """
    
    # Set language to English
    wikipedia.set_lang("en")
    
    try:
        # Try to get the page
        page = wikipedia.page(topic, auto_suggest=True)
        
        # Extract summary (first few sentences)
        summary = page.summary[:500]
        
        return f"**{page.title}**\n\n{summary}\n\nURL: {page.url}"
    
    except exceptions.DisambiguationError as e:
        # Multiple pages with this name
        options = e.options[:5]  # Show first 5 options
        return f"Ambiguous topic. Did you mean:\n" + \
               "\n".join(f"- {opt}" for opt in options)
    
    except exceptions.PageError:
        # Page doesn't exist
        return f"Page not found for '{topic}'"
    
    except Exception as e:
        return f"Wikipedia error: {str(e)}"
```

### Building the Search Agent

```python
# Initialize LLM
llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0.2  # Slightly creative for natural language
)

# System prompt guiding tool selection
prompt = ChatPromptTemplate.from_messages([
    ("system", """You are a research assistant. RULES:
    1. Use web_search for: current events, news, recent data, live info
    2. Use wiki_summary for: background, definitions, history, biography
    3. ALWAYS cite sources with [Title](URL) format
    4. Keep answers concise (3-5 sentences max)
    5. Never hallucinate - admit if you don't find information
    """),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

# Create tools list
tools = [web_search, wiki_summary]

# Create agent
agent = create_tool_calling_agent(llm, tools, prompt)
executor = AgentExecutor(agent=agent, tools=tools, verbose=False)

# Test queries
queries = [
    "Who won the FIFA World Cup 2022?",
    "Explain Retrieval-Augmented Generation in 3 bullets",
    "Find 3 recent headlines about generative AI"
]

for q in queries:
    result = executor.invoke({"input": q})
    print(f"Q: {q}\nA: {result['output']}\n")
```

### Expected Agent Behavior

| Query | Tool Used | Why |
|-------|-----------|-----|
| "Who won World Cup 2022?" | web_search | Needs current/recent info |
| "What is RAG?" | wiki_summary | Needs definition/background |
| "Latest AI news" | web_search | Needs real-time data |
| "Albert Einstein biography" | wiki_summary | Needs historical info |

### Key Insights from Demo 2

| Aspect | Learning |
|--------|----------|
| **Tool Selection** | Agent learns from descriptions when to use each tool |
| **Formatting** | Raw API responses need cleanup for readability |
| **Error Handling** | Wikipedia disambiguation/404 errors handled gracefully |
| **Citations** | Always include [Title](URL) for credibility |
| **Temperature** | 0.2 for more natural (not robotic) language |

---

## Demo 3: Custom Weather Tool

### Problem Statement

Every business has unique APIs and logic. Need a pattern to integrate them as tools.

### Implementation

#### Step 1: Create Mock Data Source

```python
# In production, replace this with real API calls
WEATHER_DATA = {
    "Mumbai": {
        "condition": "Partly cloudy",
        "temperature": 28,
        "humidity": 75,
        "rain_chance": 20
    },
    "Bengaluru": {
        "condition": "Clear",
        "temperature": 24,
        "humidity": 60,
        "rain_chance": 5
    },
    "Delhi": {
        "condition": "Foggy",
        "temperature": 12,
        "humidity": 85,
        "rain_chance": 15
    },
    "Goa": {
        "condition": "Sunny",
        "temperature": 32,
        "humidity": 70,
        "rain_chance": 2
    }
}
```

#### Step 2: Helper Function for Timestamps

```python
from datetime import datetime
from zoneinfo import ZoneInfo

def get_ist_time() -> str:
    """Get current time in Indian Standard Time (IST)."""
    ist = ZoneInfo("Asia/Kolkata")
    return datetime.now(ist).strftime("%Y-%m-%d %H:%M:%S IST")
```

#### Step 3: Implement Weather Tool

```python
@tool
def get_weather(city: str) -> str:
    """Get weather information for a city.
    
    This is a business-specific tool that:
    - Calls your weather data source/API
    - Returns formatted, human-readable output
    - Handles errors gracefully
    
    Args:
        city: City name (e.g., "Mumbai", "Bengaluru")
    
    Returns:
        Formatted weather report
    
    Note:
        This demo uses mock data. In production:
        - Replace WEATHER_DATA with API call
        - Add error handling for timeouts
        - Cache results to avoid rate limits
    """
    
    if not city:
        return "Please specify a city name (e.g., 'Mumbai', 'Delhi')"
    
    # Try to find the city (case-insensitive)
    city_lower = city.lower().strip()
    
    # Check if city in our data
    for known_city, data in WEATHER_DATA.items():
        if known_city.lower() == city_lower:
            current_time = get_ist_time()
            
            report = f"""
🌤️  **Weather Report for {known_city}**

⏰ Time: {current_time}
🌡️  Temperature: {data['temperature']}°C
☁️  Condition: {data['condition']}
💧 Humidity: {data['humidity']}%
🌧️  Rain Chance: {data['rain_chance']}%
"""
            return report.strip()
    
    # City not found - return default or error
    return f"""City '{city}' not found in our system.
Available cities: {', '.join(WEATHER_DATA.keys())}

For unknown cities, assuming default:
⏰ Time: {get_ist_time()}
🌤️  Condition: Partly cloudy
🌡️  Temperature: ~30°C
💧 Humidity: Moderate (60-75%)
🌧️  Rain Chance: Low
"""
```

### Building the Weather Agent

```python
# Initialize LLM
llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0  # Deterministic for weather
)

# System prompt
prompt = ChatPromptTemplate.from_messages([
    ("system", """You are a weather assistant. RULES:
    1. ALWAYS use get_weather tool for weather queries
    2. Ask user to specify a city if not provided
    3. Return clear, friendly weather reports
    4. Format output nicely
    """),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

# Create agent
agent = create_tool_calling_agent(llm, [get_weather], prompt)
executor = AgentExecutor(agent=agent, tools=[get_weather], verbose=False)

# Test
test_queries = [
    "What's the weather in Mumbai?",
    "Is it raining in Bengaluru?",
    "What city?"  # Should ask for clarification
]

for query in test_queries:
    result = executor.invoke({"input": query})
    print(f"Q: {query}\nA: {result['output']}\n")
```

### Why Mock Data?

✅ **Advantages:**
- No API keys needed
- Predictable, repeatable results
- Easy to test and debug
- No rate limiting
- Works offline

➡️ **Migration Path:**
Replace this:
```python
WEATHER_DATA = {...}  # Mock data
```

With this:
```python
import requests

def get_weather_api(city: str, api_key: str) -> dict:
    url = f"https://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}"
    response = requests.get(url)
    return response.json()
```

The rest of your agent code stays identical.

### Key Insights from Demo 3

| Aspect | Learning |
|--------|----------|
| **Mock Data** | Use for safe learning; migrate to real APIs later |
| **Formatting** | Make output user-friendly with emojis, sections |
| **Error Handling** | Handle missing cities, invalid input gracefully |
| **Business Logic** | Any company logic can be wrapped this way |
| **Extensibility** | Easy to add more parameters (units, forecast, etc.) |

---

## Demo 4: Multi-Tool Reasoning

### Problem Statement

Real-world queries often need multiple tools. Example:

```
"Who is the current president of France, and what's the 
square root of 144?"
```

This needs:
1. **DuckDuckGo** for current political info
2. **Calculator** for math

Agent must plan and chain tools together.

### Implementation

#### Step 1: Define All Tools

```python
# Tool 1: Calculator (from Demo 1)
@tool
def calculate(expression: str) -> str:
    """Execute math expressions safely."""
    # ... implementation from Demo 1 ...
    pass

# Tool 2: Web Search (from Demo 2)
@tool
def web_search(query: str, max_results: int = 5) -> str:
    """Search the web using DuckDuckGo."""
    # ... implementation from Demo 2 ...
    pass

# Tool 3: Wikipedia (from Demo 2)
@tool
def wiki_summary(topic: str) -> str:
    """Get Wikipedia summary."""
    # ... implementation from Demo 2 ...
    pass
```

#### Step 2: Initialize Language Model

```python
from langchain.chat_models import ChatGoogleGenerativeAI

llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0  # Deterministic reasoning
)

# Important settings
llm.convert_system_message_to_human = True  # Ensures system prompt works
```

#### Step 3: Create Multi-Tool Agent

```python
from langchain.agents import initialize_agent, AgentType

# Gather all tools
tools = [calculate, web_search, wiki_summary]

# Create ZERO_SHOT_REACT agent
# This means: Agent decides tools based on descriptions ONLY
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,  # See reasoning steps
    handle_parsing_errors=True  # Recover from LLM formatting mistakes
)
```

**Agent Types:**
- `ZERO_SHOT_REACT_DESCRIPTION` - No prior training; uses tool descriptions
- `STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION` - Better for complex reasoning

#### Step 4: Helper Function for Demos

```python
def run_demo():
    """Run predefined examples showcasing multi-tool use."""
    
    test_queries = [
        # Simple math
        "Compute 15 * 23 + 45",
        
        # Mixed: fact + math
        "Who is the current president of France? Also, what's √144?",
        
        # Live news
        "Find 3 recent headlines about generative AI in 2025",
        
        # Background + calculation
        "Tell me about Albert Einstein. Also calculate 299792458 / 1000000",
        
        # Weather + math
        "If a city's temperature is 45°C, what is it in Fahrenheit? (F = C*9/5 + 32)"
    ]
    
    print("=" * 60)
    print("MULTI-TOOL AGENT DEMOS")
    print("=" * 60)
    
    for i, query in enumerate(test_queries, 1):
        print(f"\n[DEMO {i}]")
        print(f"Query: {query}")
        print("-" * 40)
        try:
            result = agent.run(query)
            print(f"Result: {result}")
        except Exception as e:
            print(f"Error: {str(e)}")
    
    print("\n" + "=" * 60)
    print("INTERACTIVE MODE")
    print("=" * 60)
    print("Type your question (or 'exit' to quit):\n")
    
    while True:
        user_input = input("You: ").strip()
        if user_input.lower() in ['exit', 'quit', 'bye']:
            print("Goodbye!")
            break
        
        try:
            response = agent.run(user_input)
            print(f"Agent: {response}\n")
        except Exception as e:
            print(f"Error: {str(e)}\n")

# Run when script executes
if __name__ == "__main__":
    print("Checking Gemini API key...")
    if not os.environ.get('GOOGLE_API_KEY'):
        print("ERROR: GOOGLE_API_KEY not set")
        exit(1)
    
    print("✓ API key found")
    print("\nOptions:")
    print("1. Run predefined demos")
    print("2. Interactive chat")
    
    choice = input("Enter choice (1/2): ").strip()
    
    if choice == "1":
        run_demo()
    else:
        # Interactive mode
        while True:
            query = input("Ask me anything: ").strip()
            if query.lower() in ['exit', 'quit']:
                break
            response = agent.run(query)
            print(f"Response: {response}\n")
```

### Expected Agent Reasoning

#### Example 1: Simple Math
```
Input: "Compute 15 * 23 + 45"

Agent Reasoning:
→ This is pure math
→ Use: calculate tool
→ Expression: "15 * 23 + 45"
→ Result: 390

Output: 390
```

#### Example 2: Fact + Math
```
Input: "Who is the current president of France? 
        Also, what's √144?"

Agent Reasoning:
→ Part 1: "current president of France" = current info
   → Use: web_search tool
   → Result: Emmanuel Macron
→ Part 2: "√144" = math
   → Use: calculate tool
   → Expression: "144 ** 0.5"
   → Result: 12.0

Output:
Emmanuel Macron is the current president of France.
The square root of 144 is 12.0.
```

#### Example 3: Live News
```
Input: "Find 3 recent headlines about generative AI in 2025"

Agent Reasoning:
→ Needs current information
→ Use: web_search tool
→ Query: "generative AI 2025"
→ Results: [Latest article 1, Latest article 2, Latest article 3]

Output:
Here are 3 recent headlines about generative AI in 2025:

1. [Headline A](url)
   Summary of latest development...

2. [Headline B](url)
   Another recent update...

3. [Headline C](url)
   Third development...
```

#### Example 4: Background + Calculation
```
Input: "Tell me about Albert Einstein. 
        Also calculate 299792458 / 1000000"

Agent Reasoning:
→ Part 1: "Albert Einstein" = background knowledge
   → Use: wiki_summary tool
   → Result: [Bio, achievements, Nobel Prize]
→ Part 2: "299792458 / 1000000" = math
   → Use: calculate tool
   → Result: 299.792458

Output:
Albert Einstein was a physicist who developed the theory 
of relativity... [full summary from Wikipedia]

The calculation 299792458 / 1000000 = 299.792458
(This is the speed of light in vacuum in meters/microsecond)
```

### Key Insights from Demo 4

| Aspect | Learning |
|--------|----------|
| **Tool Selection** | Agent reads tool descriptions to choose wisely |
| **Tool Chaining** | Can solve multi-part problems using multiple tools |
| **Reasoning Steps** | Verbose=True shows agent's thought process |
| **Error Recovery** | handle_parsing_errors=True prevents crashes |
| **Temperature** | 0 for logical reasoning, 0.2 for natural language |
| **Real-World Power** | Combines current info + facts + calculations |

---

## Best Practices & Lessons Learned

### 1. Safety Design

✅ **DO:**
- Whitelist allowed operations
- Validate input length/type
- Use AST parsing instead of `eval()`
- Catch exceptions and return user-friendly messages
- Log attempts for security monitoring

❌ **DON'T:**
- Use `eval()` directly on user input
- Allow arbitrary file system access
- Import untrusted modules
- Skip input validation
- Return raw system errors

### 2. Tool Descriptions Matter

Tool descriptions are critical. Agent reads them to decide which tool to use.

**Bad description:**
```python
@tool
def calculate(x: str) -> str:
    """Do math."""
```

**Good description:**
```python
@tool
def calculate(expression: str) -> str:
    """Execute safe mathematical expressions.
    
    Use this when:
    - User asks for arithmetic (add, subtract, multiply, divide)
    - Solving algebraic expressions
    - Computing powers, roots, percentages
    
    Supports: +, -, *, /, %, ** (power)
    
    Examples:
    - "12 + 5 * 3" → 27
    - "(100 - 50) / 2" → 25
    - "2 ** 10" → 1024
    
    Note: Returns exact numeric result, not approximation.
    """
```

### 3. System Prompts Guide Agent Behavior

```python
# Weak prompt:
"You are a helpful assistant."

# Strong prompt:
"""You are an expert research assistant. RULES:
1. ALWAYS use web_search for current events/news
2. Use wiki_summary for historical/background info
3. Use calculate for mathematical problems
4. NEVER hallucinate - admit if you don't know
5. ALWAYS cite sources with [Title](URL) format
6. Keep answers concise (max 3-5 sentences)
7. If multiple tools needed, use them sequentially
8. Show calculation steps for complex math
"""
```

### 4. Error Handling Patterns

```python
@tool
def your_tool(param: str) -> str:
    """Your tool description."""
    
    try:
        # Validate input
        if not param:
            return "Error: Parameter required"
        
        if len(param) > MAX_LENGTH:
            return f"Error: Parameter too long (max {MAX_LENGTH})"
        
        # Call your logic
        result = your_logic(param)
        
        # Return user-friendly result
        return str(result)
    
    except ValueError as e:
        return f"Invalid input: {str(e)}"
    
    except TimeoutError:
        return "Error: Operation timed out"
    
    except Exception as e:
        # Log error for debugging
        import logging
        logging.error(f"Tool error: {str(e)}")
        return "An unexpected error occurred. Please try again."
```

### 5. Temperature Settings

| Temperature | Use Case | Example |
|-------------|----------|---------|
| **0** | Deterministic, factual, math | Calculations, tool use decisions |
| **0.2** | Natural language, slightly varied | Summaries, explanations |
| **0.5** | Balanced | General-purpose agents |
| **0.7+** | Creative, varied | Brainstorming, story generation |

### 6. Testing Tools

```python
def test_tools():
    """Unit tests for individual tools."""
    
    # Test calculator
    assert calculate("2 + 2") == "4"
    assert calculate("10 / 2") == "5"
    assert "Error" in calculate("import os")
    
    # Test web search
    result = web_search("Python programming")
    assert len(result) > 0
    assert "http" in result or "Error" in result
    
    # Test Wikipedia
    result = wiki_summary("Albert Einstein")
    assert "Einstein" in result
    assert "URL:" in result
    
    print("✓ All tool tests passed")
```

---

## Comparison: Tools vs Plain LLMs

### Mathematical Accuracy

```python
expression = "12345 * 6789 + 98765"

# Plain LLM (without tools)
prompt = f"Calculate: {expression}"
llm_output = llm.invoke(prompt)
# Output: "Approximately 83 million" (vague, wrong)

# With Calculator Tool
agent_output = agent.run(f"Calculate: {expression}")
# Output: "83,812,690" (exact)

# Verification
expected = 83_812_690
assert int(agent_output) == expected
```

### Real-Time Information

```
Query: "What's trending on Twitter today?"

Plain LLM:
- Training data may be months/years old
- Cannot access real-time info
- Makes guesses ("probably AI or politics")
- Unreliable

With DuckDuckGo Tool:
- Fetches live search results
- Returns current trends
- Provides sources
- Reliable and verified
```

### Business Logic Integration

```
Query: "What's the weather in Mumbai and price discount?"

Plain LLM:
- Hallucinates weather data
- Makes up pricing
- No connection to real systems
- Unreliable

With Custom Tools:
- get_weather() calls real API
- calculate_discount() uses business rules
- Integrates with actual systems
- Reliable and consistent
```

### Speed Comparison

| Operation | Plain LLM | With Tool |
|-----------|-----------|-----------|
| 2 + 2 | ✅ Fast, wrong answer | ✅ Fast, correct |
| Current news | ❌ Slow (thinks), no data | ✅ Fast API call |
| Weather | ❌ Hallucinated | ✅ Real-time |
| Simple calculation | ✅ Fast, often wrong | ✅ Fast, always right |

---

## Implementation Patterns

### Pattern 1: Wrapping Python Functions

```python
# Generic pattern for any Python function

@tool
def my_tool(param1: str, param2: int = 10) -> str:
    """Tool description with clear use cases.
    
    Args:
        param1: Description of first parameter
        param2: Description of second parameter (default: 10)
    
    Returns:
        String with result or error message
    
    Examples:
        - my_tool("input1") → "output1"
        - my_tool("input2", param2=20) → "output2"
    """
    try:
        # Validate inputs
        if not param1:
            return "Error: param1 is required"
        
        # Execute logic
        result = some_function(param1, param2)
        
        # Return formatted result
        return str(result)
    
    except Exception as e:
        return f"Error: {str(e)}"
```

### Pattern 2: External API Integration

```python
@tool
def external_api_tool(query: str) -> str:
    """Call external API safely.
    
    Args:
        query: API query string
    
    Returns:
        Formatted API response
    """
    import requests
    from requests.exceptions import Timeout, RequestException
    
    try:
        # Validate input
        if len(query) > 500:
            return "Error: Query too long"
        
        # Call API with timeout
        response = requests.get(
            f"https://api.example.com/search",
            params={"q": query},
            timeout=10  # Prevent hanging
        )
        
        # Check status
        if response.status_code != 200:
            return f"Error: API returned {response.status_code}"
        
        # Parse and format response
        data = response.json()
        return format_response(data)
    
    except Timeout:
        return "Error: API request timed out"
    
    except RequestException as e:
        return f"Error: API error: {str(e)}"
    
    except Exception as e:
        return f"Unexpected error: {str(e)}"
```

### Pattern 3: Database Queries

```python
@tool
def query_database(sql: str) -> str:
    """Execute safe database queries.
    
    Args:
        sql: SELECT query (read-only)
    
    Returns:
        Query results as formatted string
    """
    import sqlite3
    
    try:
        # Security: Only allow SELECT queries
        if "SELECT" not in sql.upper():
            return "Error: Only SELECT queries allowed"
        
        # Block dangerous operations
        dangerous = ["DELETE", "UPDATE", "INSERT", "DROP", "ALTER"]
        if any(word in sql.upper() for word in dangerous):
            return "Error: Query not allowed"
        
        # Connect and query
        conn = sqlite3.connect(":memory:")
        cursor = conn.cursor()
        cursor.execute(sql)
        
        # Fetch and format results
        results = cursor.fetchall()
        conn.close()
        
        # Format for readability
        formatted = "\n".join(str(row) for row in results[:10])
        
        if len(results) > 10:
            formatted += f"\n... and {len(results) - 10} more rows"
        
        return formatted
    
    except sqlite3.Error as e:
        return f"Database error: {str(e)}"
```

### Pattern 4: File System Operations

```python
@tool
def read_file_content(file_path: str, max_lines: int = 50) -> str:
    """Read file content safely.
    
    Args:
        file_path: Path to file (relative to allowed directory)
        max_lines: Maximum lines to read
    
    Returns:
        File content or error message
    """
    import os
    from pathlib import Path
    
    try:
        # Security: Only allow reading in safe directory
        safe_dir = Path("/app/safe_files")
        requested_path = safe_dir / file_path
        
        # Prevent directory traversal
        if not requested_path.resolve().is_relative_to(safe_dir.resolve()):
            return "Error: Access denied"
        
        # Check if file exists
        if not requested_path.exists():
            return f"Error: File not found: {file_path}"
        
        # Read with line limit
        with open(requested_path, 'r') as f:
            lines = f.readlines()[:max_lines]
        
        return "".join(lines)
    
    except Exception as e:
        return f"Error reading file: {str(e)}"
```

---

## Common Use Cases

### Use Case 1: E-commerce Chatbot

```python
# Tools needed:
# 1. product_search(query) - Search product catalog
# 2. check_inventory(product_id) - Check stock levels
# 3. get_price(product_id, quantity) - Price with discounts
# 4. calculate(expression) - Apply coupons
# 5. place_order(items) - Confirm purchase

example_query = """
Customer: "Do you have blue shirts in size L? 
           What's the price with my 20% employee discount?"

Agent Flow:
1. Use product_search("blue shirt size L")
   → Found product_id: SKU_123
2. Use check_inventory("SKU_123")
   → 15 in stock
3. Use get_price("SKU_123", 1)
   → $50
4. Use calculate("50 * (1 - 0.20)")
   → $40
5. Return: "Yes, 15 available. Price: $50, 
   With your discount: $40"
"""
```

### Use Case 2: Research Assistant

```python
# Tools needed:
# 1. web_search(query) - Search recent papers/news
# 2. wiki_summary(topic) - Background info
# 3. arxiv_search(keywords) - Research papers
# 4. calculate(formula) - Data analysis
# 5. pdf_extract(url) - Extract from papers

example_query = """
Researcher: "What are the latest developments 
            in Transformer architectures? 
            Compare attention heads in GPT-4 vs Claude."

Agent Flow:
1. Use web_search("Transformer architecture 2025")
   → Get latest papers/news
2. Use arxiv_search("Transformer attention mechanisms")
   → Find relevant research
3. Use wiki_summary("Transformer neural network")
   → Background context
4. Summarize findings with comparisons
5. Return: Comprehensive research overview with sources
"""
```

### Use Case 3: Financial Analysis Bot

```python
# Tools needed:
# 1. stock_price(ticker) - Get current price
# 2. financial_data(ticker) - Get P/E, earnings, etc.
# 3. web_search(query) - Latest financial news
# 4. calculate(formula) - Financial ratios
# 5. currency_converter(amount, from, to) - Convert

example_query = """
Investor: "Is Apple overvalued? 
          Show me P/E ratio compared to industry average."

Agent Flow:
1. Use stock_price("AAPL")
   → Current: $185.50
2. Use financial_data("AAPL")
   → P/E ratio: 28.5, Earnings: $6.05
3. Use financial_data("MSFT")
   → Industry average P/E: 25.3
4. Use calculate("28.5 / 25.3")
   → AAPL premium: 1.13x
5. Use web_search("AAPL valuation analysis 2025")
   → Latest analyst opinions
6. Return: "Apple trades at 1.13x industry average.
   Recent analysis suggests..."
"""
```

### Use Case 4: Educational Tutoring Bot

```python
# Tools needed:
# 1. calculate(expression) - Solve math problems
# 2. wiki_summary(topic) - Explain concepts
# 3. generate_practice_problem(topic) - Create problems
# 4. check_answer(problem, answer) - Verify solutions
# 5. retrieve_lesson(topic) - Get lesson materials

example_query = """
Student: "I don't understand quadratic equations. 
         Can you solve x² + 5x + 6 = 0?"

Agent Flow:
1. Use wiki_summary("quadratic equation")
   → Explain what it is, real-world examples
2. Use calculate("(-5 + sqrt(25 - 24)) / 2")
   → Solution 1: -2
3. Use calculate("(-5 - sqrt(25 - 24)) / 2")
   → Solution 2: -3
4. Return: Full explanation with step-by-step solution
   "The solutions are x = -2 and x = -3 because..."
"""
```

### Use Case 5: Content Moderation

```python
# Tools needed:
# 1. check_policy(text) - Check against policies
# 2. toxicity_score(text) - Rate toxicity
# 3. classify_content(text, categories) - Categorize
# 4. similar_posts(content_id) - Find similar posts

example_query = """
Content: "I hate this product, waste of money!"

Agent Flow:
1. Use toxicity_score(text)
   → Score: 0.3 (mild)
2. Use classify_content(text, ["complaint", "feedback"])
   → Category: Legitimate complaint
3. Use check_policy(text)
   → Status: Allowed (critical feedback accepted)
4. Use similar_posts(content_id)
   → Found 45 similar complaints
5. Return: "Allow - this is constructive negative feedback"
"""
```

---

## Summary & Takeaways

### What We Learned

1. **Tools extend LLM capabilities** - Make agents practical for real work
2. **Safety first** - Whitelist operations, validate inputs, handle errors
3. **Tool descriptions guide agents** - Clear descriptions → better decisions
4. **Multiple tools enable reasoning** - Agents plan and chain tools
5. **Mock data for learning** - Easy transition to real APIs
6. **Error handling matters** - Graceful failures build trust

### Next Steps for Implementation

1. **Start simple** - One tool, one agent
2. **Test thoroughly** - Unit test each tool
3. **Add error handling** - Graceful failure messages
4. **Optimize descriptions** - Refine tool descriptions based on behavior
5. **Monitor usage** - Log tool calls for debugging
6. **Iterate** - Improve based on real-world usage

### Production Checklist

- [ ] All tools have comprehensive docstrings
- [ ] Input validation on all parameters
- [ ] Error handling with user-friendly messages
- [ ] Rate limiting for external API calls
- [ ] Logging for monitoring and debugging
- [ ] Security review (no path traversal, injection, etc.)
- [ ] Performance benchmarks (timeout settings)
- [ ] Unit tests for each tool
- [ ] Integration tests for agent workflows
- [ ] API key management (env variables, secrets)
- [ ] Documentation for team
- [ ] Monitoring and alerts in production

---

## References & Resources

### Official Documentation
- [LangChain Tools](https://python.langchain.com/docs/modules/tools)
- [LangChain Agents](https://python.langchain.com/docs/modules/agents)
- [Google Gemini API](https://ai.google.dev/)

### Related Tools & Libraries
- **DuckDuckGo**: `pip install ddgs`
- **Wikipedia**: `pip install wikipedia`
- **LangChain Community**: `pip install langchain-community`
- **Google GenAI**: `pip install google-generativeai`

### Best Practices
- Keep tools focused (single responsibility)
- Use type hints for clarity
- Write comprehensive docstrings
- Test tools independently before agent integration
- Monitor tool usage in production
- Version your tools for backward compatibility

---

**Last Updated:** February 18, 2026  
**Author:** AI Education Course Notes  
**Status:** Complete & Production-Ready

