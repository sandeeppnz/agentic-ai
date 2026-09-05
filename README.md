# 💳 Payments Operations Agent with Microsoft Foundry

A practical learning project exploring how to build and invoke AI agents in Microsoft Foundry using a payments operations scenario.

The focus is intentionally simple: understand the core agent lifecycle first, then extend it with tool use and function calling in a way that feels grounded in real operational work.

## ✨ Why this project matters

This repo walks through the progression from a basic prompt-based assistant to a more capable, tool-enabled agent.

The examples are designed to answer a few important questions:

- How do you create an agent in Microsoft Foundry?
- How do you define its behavior and guardrails?
- How do you invoke it through the Responses API?
- How does an agent decide when it needs a tool?
- How do we keep the AI grounded in real operational workflows?

The scenario centers on a Payments Operations Assistant helping teams understand issues such as:

- failed payments
- pending payments
- settlement delays
- reconciliation discrepancies
- payment-processing troubleshooting

## 📚 Repository contents

- [01_payments_operations_agent.md](01_payments_operations_agent.md) — introduces the fundamentals of creating a model-backed agent and invoking it through the Responses API.
- [02_payment_ops_function_calling.md](02_payment_ops_function_calling.md) — expands the example with function calling and tool use so the agent can request operational capabilities from the application layer.

## 🚀 Learning journey

This project follows a clear progression:

1. Create a simple agent in Microsoft Foundry
2. Define its model, instructions, and operational boundaries
3. Reference the agent through the Responses API
4. Send a real user request and receive a structured response
5. Add tools so the agent can call specific payment-related capabilities when needed

## 🧠 Core concepts covered

### Agent creation

The first phase focuses on what distinguishes an agent from a plain prompt:

- a model deployment
- a named agent definition
- operational instructions and guardrails
- a reusable agent reference for future invocation

### Tool use and function calling

The second phase introduces a more agentic pattern:

- the model decides whether a tool is required
- the application executes the actual function
- the tool result is returned to the model
- the agent synthesizes a final, grounded response

This is the bridge between "chatting with an LLM" and "using an AI system in an operational workflow."

## 🏗️ Example architecture

```text
User
  │
  ▼
Payments Operations Agent
  │
  ├─ Model + instructions
  ├─ Tool definitions
  └─ Agent reference
  │
  ▼
Responses API / Foundry Project
  │
  ├─ Agent decides whether a tool is needed
  ├─ Function call is executed by the app
  └─ Final response is returned
```

## ✅ Prerequisites

Before exploring the examples, make sure you have:

- an Azure subscription
- a Microsoft Foundry project
- a deployed model in that project
- Python configured locally
- Azure authentication set up using `DefaultAzureCredential` or another supported identity flow
- the Azure AI Projects SDK installed

## 🔄 Example flow

The pattern in this repo looks like this:

```text
Microsoft Foundry
    │
    ├─ Create agent version
    ├─ Set model and instructions
    ├─ Attach tools if needed
    └─ Store the agent reference

Application / Responses API
    │
    ├─ Send user input
    ├─ Resolve the agent reference
    ├─ Run tool logic when required
    └─ Return the final answer
```

## 🛡️ Guardrails and design principles

These examples also reinforce a few important AI practices:

- never invent payment or transaction details
- keep the assistant scoped to the payments domain
- make tool descriptions clear and operationally grounded
- separate the model's decision-making from the application's execution logic
- treat agent integration as a controlled enterprise workflow, not just a chatbot

## 🚫 What this project is not

This repo is not a production payment platform and not a complete financial operations system. It is a learning-focused project designed to teach the patterns behind agent development and tool-enabled AI workflows.

## 🔜 Next steps

A natural next evolution would be to extend this foundation with:

- real payment-status APIs or data access
- stronger validation and safety checks
- retrieval-augmented generation (RAG)
- human approval for sensitive operations
- broader observability and evaluation

## 📌 License

This project is intended for learning, experimentation, and technical exploration.
