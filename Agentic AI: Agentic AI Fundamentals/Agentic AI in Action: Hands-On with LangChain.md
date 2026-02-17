# LangChain Complete Course Notes

## Course Overview

LangChain is a powerful yet approachable framework that makes it easy to work with large language models (LLMs) by providing ready-made components for chaining, memory, and tool use. Its modular design means even beginners can quickly build practical applications without needing to reinvent the wheel.

### Course Learning Objectives

1. Establish a connection in LangChain to the Gemini model and execute a basic LLM call
2. Design a prompt template and construct a sequential chain
3. Implement a basic chatbot without memory
4. Integrate conversation buffer memory so the chatbot can retain context
5. Differentiate between full conversational memory and windowed memory
6. Understand how context is managed in LangChain applications

---

## Demo 1: Building a Simple LangChain App with Google Gemini

### Overview
Building a simple but complete LangChain app using Google Gemini as the underlying LLM, including installing dependencies, setting up the model, and composing a motivational quote generator using a parameterized prompt.

### Step 1: Install Required Packages

```bash
pip install langchain langchain-google-genai google-generativeai
```

**Package Breakdown:**
- **LangChain**: The core framework
- **langchain-google-genai**: Connects LangChain to Gemini
- **google-generativeai**: Gives access to Gemini models

### Step 2: Import Essential Libraries

```python
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_google_genai import ChatGoogleGenerativeAI
```

**Library Details:**
- **PromptTemplate**: Formats prompts with parameters
- **StrOutputParser**: Extracts plain text from responses
- **RunnablePassthrough**: Enables chaining steps
- **ChatGoogleGenerativeAI**: Works with Gemini models

### Step 3: Set Up Gemini API Key

**To create an API key:**

1. Go to Google Colab → Click on "Collab Secrets"
2. Click "Get Gemini API Key"
3. Select "Manage Gemini API keys" in Google AI Studio
4. Click "Create API key"
5. Select "default project" and confirm
6. Copy the generated key
7. Paste in Collab Secrets → Enable notebook access

**Secure Setup in Code:**

```python
import os
from google.colab import userdata

api_key = userdata.get('GEMINI_API_KEY')
os.environ['GOOGLE_API_KEY'] = api_key
```

### Step 4: Create LLM Instance

```python
llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0.7,
    convert_system_message_to_human=True
)
```

**Configuration Details:**
- **Model**: `gemini-2.0-flash` - Fast and lightweight
- **Temperature**: 0.7 - Encourages creativity (0.0 = deterministic, 1.0 = more random)
- **convert_system_message_to_human=True**: Makes responses more predictable with LangChain

### Step 5: Test Basic LLM Call

```python
response = llm.invoke("Explain AI in simple terms")
print(response.content)
```

**Expected Output:**
> "Imagine teaching a computer to think and learn like a human. That's basically what AI is..."

### Step 6: Create Prompt Template

```python
prompt_template = PromptTemplate(
    input_variables=["profession"],
    template="Generate a short motivational quote for a {profession}"
)
```

**Key Concept:**
- Makes prompts reusable for different inputs
- Can generate different outputs based on input parameters

### Step 7: Build the Chain

```python
chain = prompt_template | llm | StrOutputParser()
```

**Chain Flow:**
1. Format prompt with input
2. Send to LLM
3. Extract plain text from response

**What LangChain Does:**
- Connects components in a pipeline
- Manages data flow between steps
- Handles output parsing automatically

### Step 8: Test with Different Inputs

```python
# For a teacher
result = chain.invoke({"profession": "teacher"})
print(result)
```

**Output:**
> "You plant seeds of knowledge and kindness. Your impact ripples through generations. Keep growing!"

```python
# For a coder
result = chain.invoke({"profession": "coder"})
print(result)
```

**Output:**
> "Every line of code brings your vision closer to reality. Keep building, keep learning, keep creating!"

### Key Takeaways from Demo 1

- LangChain simplifies LLM integration with ready-made components
- Prompt templates make applications reusable
- Chaining components creates functional pipelines
- Temperature controls the creativity level of responses

---

## Demo 2: Building a Sequential Chain

### Overview
Chains two tasks together: First generate creative ideas on a topic, then summarize them in simple language. This demonstrates how LangChain flows outputs from one step into the next.

### Step 1: Install and Import

```python
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_google_genai import ChatGoogleGenerativeAI
```

### Step 2: Create LLM Instance

```python
llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0.7,
    convert_system_message_to_human=True
)
```

### Step 3: Define First Prompt (Idea Generation)

```python
idea_prompt = PromptTemplate(
    input_variables=["topic"],
    template="Come up with three creative ideas about {topic}"
)
```

**Purpose:** Generate creative ideas as the first step

### Step 4: Define Second Prompt (Summarization)

```python
summary_prompt = PromptTemplate(
    input_variables=["topic", "ideas"],
    template="""Here are some ideas about {topic}:
{ideas}

Please summarize these ideas in two easy to understand sentences."""
)
```

**Purpose:** Take ideas from step 1 and summarize them

### Step 5: Create First Chain

```python
idea_chain = idea_prompt | llm | StrOutputParser()
```

**Output:** Three creative ideas as plain text

### Step 6: Build Sequential Chain

```python
from langchain_core.runnables import RunnablePassthrough

# Create a dictionary mapping inputs to chains
chain_input = {
    "ideas": idea_chain,  # Will generate ideas
    "topic": RunnablePassthrough()  # Pass topic through unchanged
}

sequential_chain = chain_input | summary_prompt | llm | StrOutputParser()
```

**How It Works:**
- `idea_chain`: Generates ideas based on topic
- `RunnablePassthrough()`: Keeps topic available for next step
- Output from step 1 becomes input to step 2

### Step 7: Execute Sequential Chain

```python
topic = "AI to improve school homework feedback"

# Get ideas
ideas = idea_chain.invoke({"topic": topic})
print("Ideas:", ideas)

# Get summary
summary = sequential_chain.invoke({"topic": topic})
print("Summary:", summary)
```

### Example Output

**Ideas:**
- Personalized Feedback Narrative Generation with "Character" Tutoring
- AI-Powered Argumentation Reconstructor for Essay Feedback
- Adaptive Difficulty Feedback with "Gamified" Revision Challenges

**Summary:**
> "The ideas focus on making homework feedback more personalized and engaging through AI-powered systems that adapt to student needs and gamify the learning experience, ultimately helping students learn more effectively."

### Key Concepts from Demo 2

- **Sequential Processing**: Each step's output becomes the next step's input
- **Smart Chaining**: LangChain automatically manages data flow
- **RunnablePassthrough**: Keeps data available throughout the chain
- **Real Workflow**: Shows multi-step problem solving

---

## Demo 3: Chatbot Without Memory

### Overview
Demonstrates how a LangChain application behaves without any memory, establishing a baseline to understand the difference when memory is added.

### Problem Without Memory

When there's no memory, each interaction is independent. The chatbot forgets everything from previous conversations.

### Implementation

```python
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain import LLMChain
from langchain_google_genai import ChatGoogleGenerativeAI
```

### Step 1: Create LLM

```python
llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0.7,
    convert_system_message_to_human=True
)
```

### Step 2: Define Prompts

```python
# First prompt - ask for name
name_prompt = PromptTemplate(
    input_variables=["name"],
    template="My name is {name}. Please greet me and remember my name."
)

# Second prompt - check if bot remembers
followup_prompt = PromptTemplate(
    input_variables=["question"],
    template="{question}"
)
```

### Step 3: Create Chains (No Memory)

```python
name_chain = name_prompt | llm | StrOutputParser()
followup_chain = followup_prompt | llm | StrOutputParser()
```

### Step 4: Run the Chatbot

```python
# Get user's name
name = "John"

# First interaction
greeting = name_chain.invoke({"name": name})
print("Bot:", greeting)
# Output: "Hello John! It's nice to meet you. I'll remember your name."

# Second interaction - Ask if it remembers
question = "Do you remember my name?"
followup = followup_chain.invoke({"question": question})
print("Bot:", followup)
# Output: "As a large language model, I don't have memory of past conversations. 
#          Therefore, I don't remember your name. You would have to tell me again."
```

### Critical Observation

Despite greeting the user by name just moments ago, the chatbot **cannot remember** the name when asked again. This is because:
- No memory mechanism is implemented
- Each chain works in isolation
- Each request is treated as completely new

### Key Problems Without Memory

1. **No Context Persistence**: Information is lost between turns
2. **Poor User Experience**: Chatbot must be re-introduced constantly
3. **Inability to Maintain Relationships**: Can't build conversation history
4. **Inefficient for Dialogue**: Real conversations require context

---

## Demo 4: Chatbot With Conversation Buffer Memory

### Overview
Adds memory using LangChain's ConversationBufferMemory, allowing the chatbot to remember details from earlier parts of the conversation.

### What is Conversation Buffer Memory?

A memory system that stores all past user and AI messages, making them available for future responses.

### Step 1: Import Memory Module

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_google_genai import ChatGoogleGenerativeAI
```

### Step 2: Create LLM Instance

```python
llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0.7,
    convert_system_message_to_human=True
)
```

### Step 3: Initialize Memory

```python
memory = ConversationBufferMemory(
    memory_key="chat_history",
    input_key="input",
    output_key="text"
)
```

**Configuration:**
- **memory_key**: The variable name for storing chat history
- **input_key**: The field name for user input
- **output_key**: The field name for bot's response

### Step 4: Create Unified Prompt with Memory

```python
unified_prompt = PromptTemplate(
    input_variables=["chat_history", "input"],
    template="""Current conversation:
{chat_history}

User: {input}
Assistant:"""
)
```

**Key Feature:**
- Includes `chat_history` - This is visible to the model every time it generates a response
- This allows the model to understand context

### Step 5: Create Conversation Chain

```python
conversation_chain = LLMChain(
    llm=llm,
    prompt=unified_prompt,
    memory=memory,
    output_key="text"
)
```

**Chain Configuration:**
- Links LLM with prompt and memory
- Automatically manages conversation history
- Updates memory after each interaction

### Step 6: Run the Conversation

```python
# First interaction
name = "Alex"
response1 = conversation_chain.run(input=f"My name is {name}. Please remember it and greet me.")
print("Bot:", response1)
# Output: "Hello Alex! It's great to meet you. I'll remember your name!"

# Second interaction
response2 = conversation_chain.run(input="Do you remember my name?")
print("Bot:", response2)
# Output: "Yes, your name is Alex! We just met. How can I help you?"
```

### Memory State After Interaction

The memory buffer now contains:
```
User: My name is Alex. Please remember it and greet me.
Assistant: Hello Alex! It's great to meet you. I'll remember your name!

User: Do you remember my name?
Assistant: Yes, your name is Alex! We just met. How can I help you?
```

### Advantages of Buffer Memory

1. **Full Context Awareness**: Model sees entire conversation history
2. **Natural Dialogue**: Conversations feel human-like
3. **Information Retention**: No important details are lost
4. **Simple Implementation**: Automatic memory management

### Disadvantages of Buffer Memory

1. **Memory Growth**: Chat history grows indefinitely
2. **Increased Costs**: More tokens used as history grows
3. **Slower Responses**: Larger context takes more time to process
4. **Context Limit**: May exceed LLM's maximum context window

---

## Demo 5: Chatbot With Window Memory

### Overview
Explores how to limit chatbot memory using ConversationBufferWindowMemory, keeping only the most recent messages (short-term memory).

### What is Window Memory?

A memory system that remembers only the last `k` messages, maintaining a sliding window of conversation context.

### Use Cases for Window Memory

- Longer conversations where old context becomes irrelevant
- Cost optimization (fewer tokens)
- Focusing on recent topics
- Preventing context overload

### Step 1: Import Window Memory

```python
from langchain.memory import ConversationBufferWindowMemory
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_google_genai import ChatGoogleGenerativeAI
```

### Step 2: Create LLM Instance

```python
llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0.7,
    convert_system_message_to_human=True
)
```

### Step 3: Initialize Window Memory

```python
memory = ConversationBufferWindowMemory(
    memory_key="chat_history",
    input_key="input",
    output_key="text",
    k=2  # Remember only the last 2 messages
)
```

**Key Parameter:**
- **k=2**: Only the last 2 message exchanges are retained
- Older messages are automatically discarded

### Step 4: Create Prompt Template

```python
prompt = PromptTemplate(
    input_variables=["chat_history", "input"],
    template="""Current conversation:
{chat_history}

User: {input}
Assistant:"""
)
```

### Step 5: Create LLMChain with Window Memory

```python
conversation_chain = LLMChain(
    llm=llm,
    prompt=prompt,
    memory=memory,
    output_key="text"
)
```

### Step 6: Run Conversations

```python
# Interaction 1
name = "Alex"
r1 = conversation_chain.run(input=f"My name is {name}. Greet me.")
print("1. Bot:", r1)  # Greets with name

# Interaction 2 - Change topic
r2 = conversation_chain.run(input="Tell me something interesting about Mars.")
print("2. Bot:", r2)  # Responds to Mars topic (remembers name - it's in window)

# Interaction 3 - Another topic
r3 = conversation_chain.run(input="How do plants make food?")
print("3. Bot:", r3)  # Responds about plants

# Interaction 4 - Follow-up question
r4 = conversation_chain.run(input="Do you remember my name?")
print("4. Bot:", r4)  # May NOT remember (name dropped from k=2 window)
```

### Window Memory Visualization

With k=2 (remembers last 2 message pairs):

**After Interaction 1:**
```
Buffer:
- User: "My name is Alex. Greet me."
- Bot: "Hello Alex!"
```

**After Interaction 2:**
```
Buffer:
- User: "My name is Alex. Greet me."
- Bot: "Hello Alex!"
- User: "Tell me about Mars."
- Bot: "Mars is..."
```

**After Interaction 3:**
```
Buffer (k=2, only last 2):
- User: "Tell me about Mars."
- Bot: "Mars is..."
- User: "How do plants make food?"
- Bot: "Plants make food through..."
```

**After Interaction 4:**
The name "Alex" from Interaction 1 is **no longer in the window**, so the bot might say it doesn't remember.

### Key Parameters for Window Memory

| Parameter | Purpose |
|-----------|---------|
| `k` | Number of message exchanges to remember |
| `k=1` | Only current exchange (minimal memory) |
| `k=2` | Last 2 exchanges (short-term) |
| `k=5` | Last 5 exchanges (medium-term) |
| `k=10+` | Recent conversation focus |

### Window Memory Trade-offs

**Advantages:**
- ✅ Cost-effective (fewer tokens)
- ✅ Faster processing
- ✅ Prevents context bloat
- ✅ Focuses on recent conversation

**Disadvantages:**
- ❌ Forgets older but important information
- ❌ Can't reference early conversation
- ❌ May seem forgetful to users

---

## Comparison: No Memory vs Buffer vs Window

### Scenario: User tells name, discusses 3 topics, then asks if bot remembers name

| Question | No Memory | Buffer Memory | Window Memory (k=2) |
|----------|-----------|---------------|---------------------|
| Remembers name? | ❌ No | ✅ Yes | ❌ No (dropped from window) |
| Conversation feel | Robot-like | Natural | Somewhat natural |
| Memory usage | Minimal | Grows with time | Fixed/limited |
| Cost | Low | High (grows) | Low/medium |
| Best for | Stateless tasks | Long conversations | Cost-conscious apps |

---

## LangChain Architecture Summary

### Core Components

1. **LLM Layer**: The language model (Gemini, GPT, etc.)
2. **Prompt Layer**: Structured input templates
3. **Chain Layer**: Component orchestration
4. **Memory Layer**: Conversation history management
5. **Output Layer**: Response parsing and formatting

### Data Flow in a Conversation

```
User Input
    ↓
Memory (retrieves history)
    ↓
Prompt Template (formats with context)
    ↓
LLM (generates response)
    ↓
Output Parser (extracts text)
    ↓
Memory (stores new interaction)
    ↓
Response to User
```

---

## Best Practices for LangChain Development

### 1. Choose the Right Memory Type

```python
# For simple, quick tasks - no memory
chain = prompt | llm | parser

# For long conversations - full memory
memory = ConversationBufferMemory()

# For cost-conscious apps - window memory
memory = ConversationBufferWindowMemory(k=3)
```

### 2. Set Appropriate Temperature

```python
# For factual answers (e.g., fact lookup)
temperature=0.0

# For creative tasks (e.g., idea generation)
temperature=0.7-0.9

# For balanced tasks (e.g., general questions)
temperature=0.5
```

### 3. Design Clear Prompt Templates

```python
# Good - Clear variables and instructions
template = """You are a helpful teacher.
Student question: {student_question}
Grade level: {grade_level}
Please explain this clearly."""

# Avoid - Ambiguous variables
template = "Answer this: {input}"
```

### 4. Handle API Keys Securely

```python
# ✅ Good - Use environment variables
api_key = os.environ.get('GEMINI_API_KEY')

# ✅ Good - Use secret managers (Colab, etc.)
api_key = userdata.get('GEMINI_API_KEY')

# ❌ Bad - Hardcode keys
api_key = "sk-xyz123..."
```

### 5. Test Incrementally

```python
# Test LLM first
response = llm.invoke("Test prompt")

# Test prompt template
formatted = prompt_template.format(variable="value")

# Test chain
result = chain.invoke({"variable": "value"})

# Test with memory
result = chain.run(input="User message")
```

---

## Common Issues and Solutions

### Issue 1: Chatbot Forgets Information

**Solution:** Add memory to the chain

```python
# Before (no memory)
chain = prompt | llm | parser

# After (with memory)
chain = LLMChain(
    llm=llm,
    prompt=prompt,
    memory=ConversationBufferMemory()
)
```

### Issue 2: High Memory and Token Usage

**Solution:** Use window memory

```python
memory = ConversationBufferWindowMemory(k=3)  # Keep last 3 exchanges
```

### Issue 3: Response is Too Creative/Not Creative Enough

**Solution:** Adjust temperature

```python
llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0.3  # Lower = more factual
)
```

### Issue 4: API Key Errors

**Solution:** Verify key setup

```python
# Ensure key is set
import os
api_key = os.environ.get('GOOGLE_API_KEY')
assert api_key is not None, "API key not found!"
```

---

## Real-World Applications

### 1. Educational Assistant
- **Memory Type:** Buffer (remember student background)
- **Temperature:** 0.3 (accurate explanations)
- **Prompts:** Tailored to grade level

### 2. Customer Support Chatbot
- **Memory Type:** Window (k=5-10, recent context)
- **Temperature:** 0.4 (professional responses)
- **Prompts:** Company policies + conversation

### 3. Creative Writing Assistant
- **Memory Type:** Buffer (remember story continuity)
- **Temperature:** 0.8 (creative responses)
- **Prompts:** Story context and style guides

### 4. Homework Helper
- **Memory Type:** Buffer (remember problem context)
- **Temperature:** 0.5 (balanced approach)
- **Prompts:** Subject-specific guidance

---

## Key Takeaways

1. **LangChain Simplifies AI Development**: Pre-built components reduce development time
2. **Memory is Essential for Dialogue**: Without it, chatbots seem like robots
3. **Choose Memory Wisely**: 
   - No memory = stateless tasks
   - Buffer memory = deep conversations
   - Window memory = cost-effective conversations
4. **Prompt Engineering Matters**: Clear templates = better outputs
5. **Temperature Controls Creativity**: Adjust based on your use case
6. **Secure API Key Management**: Always use environment variables or secret managers

---

## Additional Resources

- **LangChain Documentation**: https://python.langchain.com/
- **Google Gemini API**: https://ai.google.dev/
- **Prompt Engineering Guide**: https://platform.openai.com/docs/guides/prompt-engineering
- **LangChain GitHub**: https://github.com/langchain-ai/langchain

---

*Course Notes Created: February 2026*
*Framework: LangChain with Google Gemini*
*Target Audience: Beginners to Intermediate Python Developers*

