# Agentic AI Architecture - Complete Course Notes

## Table of Contents
- [Introduction](#introduction)
- [Core Architecture Components](#core-architecture-components)
  - [1. Perception](#1-perception)
  - [2. Memory](#2-memory)
  - [3. Planning](#3-planning)
  - [4. Tool Use](#4-tool-use)
  - [5. Environment Integration](#5-environment-integration)
  - [6. Feedback & Learning](#6-feedback--learning)
- [Agent Architectures](#agent-architectures)
- [Real-World Example: Restaurant Booking Flow](#real-world-example-restaurant-booking-flow)
- [Key Terminology](#key-terminology)

---

## Introduction

**Agentic AI Architecture** is the blueprint behind intelligent, goal-driven AI systems. It explains how these systems:
- Understand their surroundings
- Set goals
- Make choices
- Take action
- Learn over time

This transforms AI from fixed models into **flexible, interactive systems** capable of autonomous decision-making and continuous improvement.

### What Makes an Agent Different?

Traditional AI responds to direct commands. Agentic AI:
- Interprets messy, real-world inputs
- Maintains context through memory
- Plans multi-step sequences
- Uses tools to interact with environments
- Learns from feedback and outcomes

---

## Core Architecture Components

### 1. Perception

**Definition:** The process of transforming raw inputs (user messages, triggers, data) into meaningful, structured information.

#### The Perception Pipeline

```
Raw Input → Interpreter → Context Enrichment → Validation → Structured Schema
```

**Steps:**
1. **Capture Input**: User message, system event, or data stream
2. **Interpret Intent**: Using LLM, classifier, or keyword extraction
3. **Layer Context**: Add memory and environmental signals
4. **Clean & Validate**: Ensure format compliance and logical consistency
5. **Reformat**: Convert to structured schema (task object, planning request)

#### Key Subtasks
- **Extract Intent & Entities**: What does the user want? What are the key elements?
- **Enrich Input**: Add context from memory and environment
- **Normalize**: Structure the data for downstream processing

#### Technologies Used
- **LLM Prompting**: For natural language understanding
- **Embedding-based Search**: From vector memory stores
- **Domain-specific Parsing**: Custom rules for specialized inputs

#### Design Considerations
- What types of inputs will the agent receive?
- Is deep language understanding required?
- Are inputs structured (API signals) or free-form (human chat)?
- How ambiguous or noisy are the inputs?

#### Best Practices
- Use prompting best practices to clarify expectations
- Apply feedback to refine input handling
- Design agents to ask clarifying questions when uncertain
- Handle ambiguity gracefully, don't just guess

#### Example
```
User Input: "order me something spicy"

Perception Output:
{
  "intent": "food_order",
  "preference": "spicy",
  "ambiguity": "high",
  "clarification_needed": true,
  "suggested_clarification": "Are you looking for spicy food delivery?"
}
```

---

### 2. Memory

**Definition:** The system that allows agents to remember, adapt, and stay context-aware over time.

#### Why Memory Matters
- Maintains continuity across multi-turn workflows
- Stores facts about users and domains
- Logs what worked and what didn't
- Personalizes interactions
- Enables learning from feedback

#### Three Types of Memory

##### 1. Short-term Memory (Working Memory)
- **Purpose**: Maintains current conversation/task context
- **Scope**: Session-level continuity
- **Use Cases**:
  - Follow multi-step instructions
  - Avoid repeating questions
  - Track recent interactions

**Analogy**: Like a mental whiteboard

##### 2. Long-term Memory (Knowledge Vault)
- **Purpose**: Stores persistent information
- **Scope**: Cross-session knowledge
- **Contents**:
  - User names and preferences
  - Past choices
  - Domain facts
  - Historical data

**Analogy**: Like a personal database

##### 3. Episodic Memory (Experience Log)
- **Purpose**: Records specific experiences
- **Scope**: Task-level history
- **Contents**:
  - What the agent did
  - When it happened
  - What the outcome was
- **Enables**:
  - Post-analysis
  - Learning from success/failure
  - Debugging and storytelling

**Analogy**: Like a journal

#### Memory Workflow

```
New Input → Check Memory Need → Query Memory System → Retrieve Relevant Data
→ Filter & Clean → Inject into Reasoning → Complete Task → Store New Insights
```

#### Memory Operations

**Retrieval Methods:**
- Vector similarity search
- Metadata tags
- Knowledge graphs

**Storage Strategy:**
- Store what's relevant to task, user, or goal
- Distinguish between short-term and long-term needs
- Enable intelligent retrieval at the right moment

#### Best Practices
- **Selective Storage**: Don't record everything, remember strategically
- **Smart Retrieval**: Surface context at the right moment
- **Adaptive Learning**: Memory should grow more relevant over time
- **Balance Efficiency**: Distinguish temporary vs. persistent information

---

### 3. Planning

**Definition:** The process of breaking down goals into sequenced, executable steps.

#### Why Planning Matters
Without planning, even the best perception and intent recognition can fail:
- Tasks with dependencies require proper sequencing
- Tool selection needs strategic thinking
- Complex goals need decomposition

#### Planning Sits Between Perception and Action

```
Perception → Memory Retrieval → Planning Engine → Tool Execution
```

#### The Planning Process

1. **Receive Structured Input**: From perception layer
2. **Query Planning Module**: Reasoning chain, rule-based planner, or policy model
3. **Decompose Goal**: Break into smaller tasks
4. **Arrange Logically**: Order based on dependencies
5. **Prioritize**: Determine execution sequence
6. **Map to Tools**: Assign appropriate tools or agents
7. **Incorporate Constraints**: Add policies and rules
8. **Define Control Logic**: Set fallback strategies
9. **Generate Executable Plan**: Pass to action executor

#### Three Planning Strategies

##### 1. Rule-Based Flowcharts
- **Type**: Fixed logic trees
- **Logic**: `if X then Y`
- **Pros**: Predictable, easy to debug
- **Cons**: Rigid, brittle with complexity
- **Best For**: Simple, deterministic tasks

##### 2. Chain-of-Thought Prompting
- **Type**: LLM-based reasoning
- **Logic**: Step-by-step natural language reasoning
- **Pros**: Flexible, context-aware
- **Cons**: Less structured, harder to guarantee outcomes
- **Best For**: Medium complexity with variability

##### 3. Dynamic Planning Graphs (e.g., LangGraph)
- **Type**: Graph-based state machines
- **Logic**: Nodes (tasks/decisions) + edges (transitions)
- **Pros**: Modular, adaptive, supports branching/looping
- **Cons**: More complex to design and maintain
- **Best For**: Complex, evolving workflows

#### Common Planning Pitfalls
- **Skipping Planning**: Diving straight into execution
- **Using Wrong Tools**: Poor goal decomposition
- **Overlooking Prior Steps**: Misalignment or failure
- **No Fallback Strategy**: No recovery from errors

#### Planning with Memory
Planning actively pulls from memory:
- Consults past interactions
- Reviews user preferences
- Checks task history
- Understands what worked/failed previously

#### Adaptive Planning Loop

```
Plan → Act → Observe → Adjust → Re-plan (if needed)
```

This loop makes agentic AI adaptive and resilient.

---

### 4. Tool Use

**Definition:** The agent's ability to invoke external code, services, and systems to take real-world actions.

#### What is a Tool?
In agent systems, tools are:
- **APIs**: REST, GraphQL endpoints
- **Cloud Functions**: Serverless compute
- **Search Engines**: Google, Bing APIs
- **Databases**: SQL, NoSQL stores
- **Web Browsers**: Headless automation
- **Other Agents**: Agent-to-agent communication

#### Why Tool Use Matters
Tools let agents:
- Go beyond internal reasoning
- Interact with the external world
- Retrieve data dynamically
- Trigger workflows
- Modify systems

**Without tools, agents can only think—they can't act.**

#### Tool Use Workflow

```
Action Plan → Map to Tool → Transform Input → Send Request
→ Manage Auth → Invoke Tool → Process Response → Update Memory → Log Result
```

**Detailed Steps:**
1. **Receive Action Plan**: From planner
2. **Map Action to Tool**: Match task to available function
3. **Transform Input**: Format for tool's expected schema
4. **Send Request**: With proper authentication
5. **Invoke Tool/API**: Execute the action
6. **Process Response**: Parse result
7. **Update Context**: Store in memory
8. **Log Metadata**: Track success/failure

#### Tool Interface Types

##### 1. Function Calls
- **Examples**: OpenAI functions, LangChain tools
- **Usage**: Direct function invocation with parameters
- **Best For**: Structured, predefined operations

##### 2. JSON Payloads
- **Format**: Structured data objects
- **Usage**: Fill parameters and format data
- **Best For**: API integrations

##### 3. Natural Language
- **Format**: Prompts or instructions
- **Usage**: Send human-like requests
- **Best For**: LLM-based tools

##### 4. Graph-Based Selectors
- **Format**: Structured relationships
- **Usage**: Choose based on metadata/dependencies
- **Best For**: Complex, dynamic tool selection

#### Tool Selection Strategies

**Basic: Hard-coded Logic**
```python
if task == "search":
    use_bing_api()
elif task == "email":
    use_gmail_api()
```

**Advanced: Memory & Feedback-Based**
- Select based on past performance
- Consider relevance to current goal
- Adapt based on success rates

#### Tool Chaining & Composition

Agents often chain multiple tools:

**Example Flow:**
1. Query database → Get results
2. Summarize with LLM → Generate insight
3. Send email → Deliver to user

**Orchestration Tools:**
- LangGraph
- LangChain
- Custom state machines

#### Tool Registration

Tools can be:
- **Pre-registered**: With metadata and schemas
- **Dynamically Discovered**: At runtime

---

### 5. Environment Integration

**Definition:** How agents connect to external systems and exchange information.

#### Why Integration Matters
Intelligence without integration is isolation. Agents must:
- Access the world reliably
- Communicate securely
- Operate in production environments

#### Three Aspects of Integration

##### 1. Access External Data & Services
- CRM systems
- Weather APIs
- Payment gateways
- Cloud services

##### 2. Connect with Software Systems & UIs
- Read from applications
- Respond to user actions
- Trigger alerts and notifications

##### 3. Set Up Input/Output Pipelines
- Structured channels for receiving information
- Mechanisms for sending results
- Cross-organizational data flow

#### Integration Methods

**Input Channels:**
- Webhooks
- REST/GraphQL APIs
- Message queues (Kafka, RabbitMQ)
- WebSocket connections
- File systems
- Database connections

**Output Channels:**
- API responses
- Database writes
- File generation
- Email/notifications
- Dashboard updates
- Event triggers

#### Integration Workflow

```
Input Channels → Agent Processing → External Interaction
→ Middleware/Adapters → Outbound Integration → Logging & Feedback
```

**Flow Details:**
1. **Input Channels**: Webhooks, APIs, message queues
2. **Agent Processing**: Perception, planning, decision-making
3. **External Interaction**: API calls, database queries
4. **Middleware**: Normalize formats and protocols
5. **Outbound Integration**: Send results, trigger systems
6. **Logging**: Track outcomes, send feedback

#### Technologies Used

- **REST APIs**: Standard web service calls
- **GraphQL**: Flexible query-based APIs
- **SQL/NoSQL Databases**: Data persistence
- **Event-Driven Architecture**: Message-based triggers
- **WebSockets**: Real-time bidirectional communication

#### Common Integration Issues

##### 1. Silent Tool Errors
- Failures not logged or caught
- Solution: Comprehensive error handling

##### 2. Data Mismatches
- Format incompatibilities
- Solution: Strong validation and adapters

##### 3. Authentication Issues
- Expired tokens, missing credentials
- Solution: Robust auth management and refresh logic

##### 4. Fragile Scripts
- One-off solutions that break easily
- Solution: Modular, maintainable connectors

#### Agents as Services

**Modern Pattern**: Deploy agents with APIs

**Capabilities:**
- Handle real-time requests
- Process batch jobs
- Integrate into pipelines
- Act as microservices

**Requirements:**
- Secure inputs
- Validate payloads
- Monitor outputs
- Ensure consistency

---

### 6. Feedback & Learning

**Definition:** How agents capture outcomes, learn from experience, and continuously improve.

#### Why Feedback Matters
- Enables adaptation
- Catches mistakes early
- Foundation for long-term improvement
- Turns experience into intelligence

#### Types of Feedback

##### 1. Explicit Feedback
- User ratings (thumbs up/down)
- Corrections
- Direct comments
- Star ratings

##### 2. Implicit Feedback
- Task success/failure
- Tool errors
- User behavior (clicks, skips)
- Completion rates

##### 3. System-Generated Feedback
- Performance logs
- Error traces
- Execution time
- Resource usage

#### The Learning Loop

```
Execute Task → Evaluate Output → Collect Feedback → Log Results
→ Recognize Patterns → Fine-tune Logic → Store Knowledge → Improve
```

**Detailed Steps:**

1. **Step-Level Logging**
   - Record every decision
   - Track every tool call
   - Log every output

2. **Evaluate Output**
   - Compare to intended goal
   - Check constraint compliance
   - Assess quality

3. **Collect Feedback**
   - User signals
   - Automated metrics
   - Performance indicators

4. **Log Records**
   - Store successes and failures
   - Tag contexts and conditions

5. **Recognize Patterns**
   - What consistently works
   - What tends to fail
   - In what contexts

6. **Fine-tune Logic**
   - Revise action strategies
   - Adjust decision weights
   - Update learning models

7. **Store Knowledge**
   - Save insights to memory
   - Prepare for future runs

8. **Deliver Improvements**
   - Better accuracy
   - Smarter decisions
   - More relevant responses

#### Feedback-Driven Improvements

**What Gets Better:**
- Action selection
- Tool choice
- Response relevance
- Personalization
- Error recovery

**How It Improves:**
- Reinforcement learning
- Model fine-tuning
- Strategy adjustment
- Weight updating
- Pattern recognition

#### Risks of Feedback Learning

##### 1. Reinforcing Bias
- May amplify existing biases in data
- Solution: Diverse feedback, bias monitoring

##### 2. Overfitting to Recent Feedback
- Forgets general rules
- Solution: Balance recent vs. historical data

##### 3. Gaming the System
- Optimizing for metrics, not quality
- Solution: Multi-dimensional evaluation

**Human oversight remains critical, especially for sensitive tasks.**

#### Best Practices for Feedback Systems

##### 1. Build Feedback In from the Start
- Not an afterthought
- Core architectural component

##### 2. Developer Insights
- Expose internal decisions
- Enable debugging and analysis

##### 3. User-Friendly Feedback
- Make it easy to provide
- Structure the feedback format

##### 4. Safe Iteration
- Rollback options
- A/B testing
- Gradual deployment

##### 5. Continuous Monitoring
- Track performance metrics
- Alert on anomalies
- Regular audits

---

## Agent Architectures

### Comparison of Three Architectural Styles

| Feature | Reactive | Deliberative | Learning-Based |
|---------|----------|--------------|----------------|
| **Response Time** | Instant | Slower | Variable |
| **Memory** | ❌ None | ✅ Yes | ✅ Yes |
| **Planning** | ❌ No | ✅ Yes | ✅ Yes |
| **Learning** | ❌ No | ❌ No | ✅ Yes |
| **Adaptability** | Rigid | Moderate | High |
| **Complexity** | Low | Medium | High |
| **Best For** | Simple triggers | Complex reasoning | Continuous improvement |

### 1. Reactive Agents

**Characteristics:**
- Respond instantly to inputs
- No memory retention
- No planning capability
- Rule-based or pattern-matching

**Architecture:**
```
Input → Pattern Match → Immediate Action
```

**Examples:**
- Motion sensor lights
- Basic chatbot responses
- Threshold-based alerts
- Simple automation scripts

**Pros:**
- Fast response
- Predictable behavior
- Easy to implement
- Low computational cost

**Cons:**
- No learning
- No context awareness
- Brittle to changes
- Limited to simple tasks

### 2. Deliberative Agents

**Characteristics:**
- Build internal world models
- Plan before acting
- Reason through complex goals
- Consider future outcomes

**Architecture:**
```
Input → Build Model → Reason & Plan → Execute → Monitor
```

**Examples:**
- Warehouse robots (path planning)
- Customer support agents (policy-based)
- Strategic game players
- Task scheduling systems

**Pros:**
- Thoughtful decisions
- Handles complexity
- Plans multi-step sequences
- Considers consequences

**Cons:**
- Slower response time
- No adaptation over time
- Fixed reasoning models
- Requires comprehensive initial design

### 3. Learning-Based Agents

**Characteristics:**
- Use memory and feedback
- Adapt through experience
- Refine internal models
- Discover effective strategies
- Continuously improve

**Architecture:**
```
Input → Reason & Plan → Execute → Collect Feedback
→ Learn & Adapt → Update Models → Improved Future Performance
```

**Examples:**
- Recommendation engines (Netflix, Spotify)
- Personalized assistants
- Adaptive game AI
- Dynamic pricing systems

**Pros:**
- Continuous improvement
- Personalization
- Adapts to change
- Discovers better strategies

**Cons:**
- Complex design
- Requires oversight
- Risk of bias
- Needs quality feedback

### Hybrid Agents (Most Common in Practice)

Real-world agents often **mix and match** architectural styles:

**Example: Customer Support Chatbot**
- **Reactive** for greetings and small talk
- **Deliberative** for complex problem-solving
- **Learning-Based** for improving support quality

**Benefits of Hybrid Approach:**
- Flexibility for different scenarios
- Optimize speed vs. intelligence trade-offs
- Leverage strengths of each style
- More robust overall system

---

## Real-World Example: Restaurant Booking Flow

### User Request
> "Can you book me a table for dinner tonight?"

### Agentic Flow Breakdown

#### Phase 1: Perception
**Input:** "book me a table for dinner tonight"

**Processing:**
- Extract intent: `restaurant_reservation`
- Extract entities: `time = tonight`, `party_size = unspecified`
- Detect ambiguity: `cuisine preference?`, `location?`

**Output:**
```json
{
  "intent": "restaurant_reservation",
  "time": "tonight",
  "ambiguities": ["cuisine", "location", "party_size"]
}
```

#### Phase 2: Memory Retrieval
**Query Memory:**
- User preferences
- Past behavior
- Historical bookings

**Retrieved Context:**
```json
{
  "preferred_cuisine": "Thai",
  "preferred_time": "7:00 PM",
  "last_visited": "Siam Kitchen",
  "rating_given": 5
}
```

#### Phase 3: Planning
**Goal:** Book a suitable restaurant

**Plan:**
1. Search Thai restaurants near user location
2. Filter by availability at 7:00 PM tonight
3. Prioritize based on past preferences
4. Check for "Siam Kitchen" availability first
5. If unavailable, find similar alternatives
6. Make reservation
7. Confirm with user

#### Phase 4: Tool Use
**Selected Tools:**
- Location API (get user location)
- Restaurant Search API (find options)
- OpenTable/Zomato API (check availability & book)
- Notification Service (send confirmation)

**Execution:**
```python
# Pseudo-code
location = get_user_location()
restaurants = search_restaurants(
    cuisine="Thai",
    location=location,
    time="19:00",
    date="today"
)
best_option = rank_by_preference(restaurants, user_memory)
booking = make_reservation(best_option, time="19:00", party_size=2)
send_confirmation(user, booking)
```

#### Phase 5: Environment Integration
**Systems Connected:**
- OpenTable API (reservation)
- Google Maps API (location)
- SMS/Email service (notification)
- Database (store booking record)

#### Phase 6: Feedback & Learning
**Outcome:** Booking successful at Siam Kitchen, 7:00 PM

**User Feedback:** ⭐⭐⭐⭐⭐ (5 stars)

**Learning Update:**
```json
{
  "reinforced_preferences": {
    "restaurant": "Siam Kitchen",
    "time": "7:00 PM",
    "cuisine": "Thai"
  },
  "success_pattern": "similar_to_past_choice",
  "future_strategy": "prioritize_siam_kitchen"
}
```

**Next Time:**
- Agent will check Siam Kitchen first
- Suggest 7:00 PM automatically
- Confidence in Thai cuisine selection increases

---

## Key Terminology

### Core Concepts

**Agent**
- An AI system that can act independently and achieve goals autonomously

**Agentic AI**
- AI systems that perceive, reason, plan, act, and learn in a continuous loop

**Autonomy**
- Ability to operate without constant human intervention

**Goal-Driven**
- Oriented toward achieving specific objectives

### Perception

**Intent**
- The underlying goal or purpose behind user input

**Entity Extraction**
- Identifying key pieces of information (names, dates, locations, etc.)

**Natural Language Processing (NLP)**
- Technology for computers to understand human language

**LLM (Large Language Model)**
- AI models trained on vast text data (e.g., GPT, Claude)

**Disambiguation**
- Resolving ambiguity to determine the correct meaning

### Memory

**Short-term Memory**
- Temporary storage for current session/conversation

**Long-term Memory**
- Persistent storage for lasting information

**Episodic Memory**
- Record of specific experiences and events

**Vector Memory**
- Embedding-based storage for semantic similarity search

**Knowledge Graph**
- Structured representation of relationships between concepts

**Context Window**
- Amount of information an AI can consider at once

### Planning

**Task Decomposition**
- Breaking large goals into smaller, manageable tasks

**Sequencing**
- Determining the order of operations

**Dependencies**
- Tasks that must be completed before others can start

**Chain-of-Thought**
- Step-by-step reasoning process

**State Machine**
- System that transitions between defined states based on inputs

**LangGraph**
- Framework for building stateful, multi-actor applications

### Tool Use

**API (Application Programming Interface)**
- Interface for software systems to communicate

**REST API**
- Standard web service communication protocol

**GraphQL**
- Query language for APIs with flexible data retrieval

**Webhook**
- Automated message sent from apps when events happen

**Function Calling**
- Ability for LLMs to invoke external functions

**Tool Chaining**
- Connecting multiple tools in sequence

### Environment Integration

**Middleware**
- Software that bridges different systems

**Adapter**
- Component that translates between different formats

**Event-Driven Architecture**
- System design based on producing/consuming events

**Microservice**
- Small, independent service in a larger system

**Authentication**
- Verifying identity to access systems

**Payload**
- Data transmitted in a request or response

### Learning & Feedback

**Reinforcement Learning**
- Learning through trial and error with rewards/penalties

**Fine-tuning**
- Adapting a model to specific tasks or data

**Overfitting**
- Model becomes too specific to training data, loses generalization

**Bias**
- Systematic errors or unfair preferences in decisions

**Model Weights**
- Parameters that determine how a model makes decisions

**Feedback Loop**
- Cycle of action, observation, and adjustment

### Architecture Types

**Reactive System**
- Responds immediately without planning

**Deliberative System**
- Plans and reasons before acting

**Hybrid System**
- Combines multiple architectural approaches

**Stateless**
- No memory between interactions

**Stateful**
- Maintains state/memory across interactions

---

## Summary

Agentic AI systems are built on **six core components** working in harmony:

1. **Perception** - Understanding inputs
2. **Memory** - Maintaining context
3. **Planning** - Sequencing actions
4. **Tool Use** - Taking real actions
5. **Environment Integration** - Connecting to the world
6. **Feedback & Learning** - Continuous improvement

These systems can be architected in **three main styles**:
- **Reactive** (fast, simple)
- **Deliberative** (thoughtful, complex)
- **Learning-Based** (adaptive, evolving)

The result is AI that doesn't just respond—it **thinks, acts, remembers, and improves**, transforming from static models into dynamic, intelligent assistants capable of handling real-world complexity.

---

*Course Summary: Understanding how modern agentic AI systems are built and how they think, act, and evolve.*

