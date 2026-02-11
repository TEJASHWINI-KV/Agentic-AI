# Agentic AI Frameworks - Quick Notes

## What is Agentic AI?
AI that can perceive, reason, act, and adapt autonomously (not just chatbots)

---

## The 7 Frameworks

### 1. **LangChain** - The Foundation
- **What**: Building blocks for AI agents
- **Best for**: Quick prototyping, custom agents
- **Key parts**: Chains (sequences), Tools (APIs), Agents (decision-makers)
- **Example**: Research assistant that searches → summarizes → answers

### 2. **LangGraph** - The Flow Controller  
- **What**: Models workflows as graphs with loops/retries
- **Best for**: Complex agents with branching logic
- **Key parts**: Nodes (steps), Edges (connections), State (memory)
- **Example**: Customer support that retries different solutions

### 3. **AutoGen** - The Team Coordinator
- **What**: Multiple agents talking to each other (Microsoft)
- **Best for**: Agent collaboration, verification workflows
- **Key parts**: Agent roles, Conversation loops, Human-in-loop
- **Example**: Code debugger where agents review each other

### 4. **CrewAI** - The Role-Based Team
- **What**: Agents with specific jobs working sequentially
- **Best for**: Organized teams with clear handoffs
- **Key parts**: Agent (role), Task (job), Crew (orchestrator)
- **Example**: Researcher → Writer → Reviewer pipeline

### 5. **LlamaIndex** - The Librarian
- **What**: Connects LLMs to your documents (RAG)
- **Best for**: Answering from PDFs/databases
- **Key parts**: Indexes, Retrievers, Query engines
- **Example**: HR bot answering from 10,000 company policies

### 6. **Haystack** - The Production Pipeline
- **What**: Enterprise-grade RAG system (by deepset)
- **Best for**: Production systems, large scale
- **Key parts**: Retriever → Ranker → Reader → Generator
- **Example**: Legal research across millions of documents

### 7. **n8n** - The Action Taker
- **What**: No-code workflow automation
- **Best for**: Real-world actions (emails, APIs, databases)
- **Key parts**: Webhooks, 350+ integrations, visual workflows
- **Example**: Auto-send emails, update CRM, post to Slack

---

## Quick Comparison

| Framework | Purpose | When to Use |
|-----------|---------|-------------|
| LangChain | Foundation | Quick prototyping |
| LangGraph | Flow control | Loops & retries |
| AutoGen | Multi-agent chat | Collaboration |
| CrewAI | Role-based teams | Sequential workflows |
| LlamaIndex | Document search | Answer from docs |
| Haystack | Enterprise RAG | Production scale |
| n8n | Real actions | Connect apps/APIs |

---

## When to Use What?

**Need to build fast?** → LangChain or n8n  
**Need loops/retries?** → LangGraph  
**Multiple agents talking?** → AutoGen  
**Team with roles?** → CrewAI  
**Search documents?** → LlamaIndex (simple) or Haystack (enterprise)  
**Take real actions?** → n8n  

---

## Key Terms

- **Agent** = AI that makes decisions & takes actions
- **RAG** = Retrieval-Augmented Generation (find data before answering)
- **Tool** = Specific ability (calculator, search, API)
- **Memory** = Remember past interactions
- **Pipeline** = Series of connected steps
- **Multi-agent** = Multiple AIs working together

---

## Real Example: Bank Customer Service

**Problem**: 50M customers, 100K daily requests, $200M/year cost

**Solution Stack**:
1. **LangChain** - Basic conversation
2. **LangGraph** - Decision flow with loops
3. **AutoGen** - Specialist agents collaborate (Fraud + Risk + Policy experts)
4. **CrewAI** - Mortgage processing team (5 specialized agents)
5. **LlamaIndex** - Search 100K fraud cases
6. **Haystack** - Search policies at scale
7. **n8n** - Execute actions (refund, email, freeze card, update CRM)

**Result**: 18 min → 2 min, $15 → $0.75 per interaction, $155M saved/year

---

## Memory Trick: "LLACHHN"

- **L**angChain - **L**EGO blocks
- **L**angGraph - **L**oops
- **A**utoGen - **A**gents chatting
- **C**rewAI - **C**oordinated team
- **H**aystack - **H**uge pipelines
- **H** (LlamaIndex) - **H**ow to find data?
- **N**8n - **N**ow take action!

---

## How They Work Together

```
LangChain (Foundation)
    ↓
LangGraph (adds loops/branches)
    ↓
AutoGen/CrewAI (multi-agent)
    ↓
LlamaIndex/Haystack (data retrieval)
    ↓
n8n (real-world actions)
```

---

## Getting Started

```bash
pip install langchain langgraph llama-index
pip install pyautogen crewai farm-haystack
# n8n via Docker
docker run -p 5678:5678 n8nio/n8n
```

**First Project**: LangChain + LlamaIndex → Q&A bot over your documents

---

**Last Updated**: Feb 2026

