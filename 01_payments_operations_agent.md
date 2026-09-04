# Agent Fundamentals - Building and Invoking an AI Agent with Microsoft Foundry — A Payments Operations Example

As part of my **AI-103 learning journey**, I've been exploring how to create and invoke AI agents using Microsoft Foundry.

Rather than reproducing the course's IT Help Desk example, I wanted to apply the same concept to a domain I'm familiar with: **payments**.

In this post, I'll walk through creating a simple **Payments Operations Agent** and then invoking it through the Responses API.

The goal is deliberately simple: understand the fundamental lifecycle of creating an agent and invoking it before introducing tools, knowledge, RAG, actions, and more advanced agentic patterns.

---

## What We're Building

The example will create an agent that acts as a Payments Operations Assistant.

The agent will be instructed to help operations teams understand common payment-processing issues such as:

- Failed payments
- Pending payments
- Settlement delays
- Reconciliation discrepancies

At this stage, the agent won't have access to databases, payment systems, APIs, or external tools.

We're focusing on the basic agent lifecycle:

```text
                 Microsoft Foundry
                       │
                       │ Create Agent Version
                       ▼
             ┌─────────────────────┐
             │ Payments Operations │
             │ Agent               │
             │                     │
             │ GPT-5.4             │
             │ Instructions        │
             └──────────┬──────────┘
                        │
                        │ Agent Reference
                        ▼
                  Responses API
                        │
                        ▼
                   Application
```

---

## Prerequisites

For this example, you'll need:

- An Azure subscription
- A Microsoft Foundry project
- A deployed model
- Python
- Azure authentication configured locally
- The relevant Azure AI Projects SDK

I'm using `DefaultAzureCredential`, which allows the application to authenticate using the available Azure identity mechanisms rather than embedding credentials in the source code.

---

# 1. Creating the Agent

The first step is to create a version of the agent in Microsoft Foundry.

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.projects.models import PromptAgentDefinition

PROJECT_ENDPOINT = "<foundry-project-endpoint>"
AGENT_NAME = "Payments-Operations-Agent"
DEPLOYMENT_NAME = "gpt-5.4"

client = AIProjectClient(
    endpoint=PROJECT_ENDPOINT,
    credential=DefaultAzureCredential()
)

agent = client.agents.create_version(
    agent_name=AGENT_NAME,
    definition=PromptAgentDefinition(
        model=DEPLOYMENT_NAME,
        instructions=(
            "You are a Payments Operations Assistant for a banking platform. "
            "Help operations teams understand payment processing issues, "
            "including failed payments, pending payments, settlement delays, "
            "and reconciliation discrepancies. "
            "Provide clear explanations and structured troubleshooting steps. "
            "Do not invent payment status or transaction information. "
            "If the question is outside payments operations, politely say so."
        )
    )
)

print("Agent created:")
print(f"  ID      : {agent.id}")
print(f"  Name    : {agent.name}")
print(f"  Version : {agent.version}")
```

### What is happening here?

There are a few important pieces in this example.

### `AIProjectClient`

`AIProjectClient` provides access to the Microsoft Foundry project.

```python
client = AIProjectClient(
    endpoint=PROJECT_ENDPOINT,
    credential=DefaultAzureCredential()
)
```

The project endpoint identifies the Foundry project, while `DefaultAzureCredential` handles authentication.

---

### Agent name

```python
AGENT_NAME = "Payments-Operations-Agent"
```

This identifies the agent.

Instead of creating a completely new agent for every request, the application can reference this agent when it needs to use it.

---

### Model deployment

```python
DEPLOYMENT_NAME = "gpt-5.4"
```

The agent is configured to use the specified model deployment.

The exact deployment name will depend on how the model is deployed in your Foundry project.

---

### Agent instructions

The most interesting part is probably the instruction itself:

```python
instructions=(
    "You are a Payments Operations Assistant for a banking platform. "
    "Help operations teams understand payment processing issues, "
    "including failed payments, pending payments, settlement delays, "
    "and reconciliation discrepancies. "
    "Provide clear explanations and structured troubleshooting steps. "
    "Do not invent payment status or transaction information. "
    "If the question is outside payments operations, politely say so."
)
```

This establishes the agent's role and behavioral boundaries.

One instruction is particularly important:

> **Do not invent payment status or transaction information.**

A real payments assistant should not make up transaction details simply because the user asks for them.

This is a very simple example of thinking about **AI reliability and guardrails**.

---

## Creating a Version

The API call is:

```python
client.agents.create_version(...)
```

This is worth paying attention to.

We're not just sending a prompt to an LLM and receiving a response.

We're creating a managed **agent version** with:

- A model
- A name
- Instructions

That gives us an identifiable agent configuration that we can subsequently invoke.

---

# 2. Invoking the Agent

Once the agent exists, the next step is to invoke it.

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

PROJECT_ENDPOINT = "<foundry-project-endpoint>"
AGENT_NAME = "Payments-Operations-Agent"

client = AIProjectClient(
    endpoint=PROJECT_ENDPOINT,
    credential=DefaultAzureCredential()
)

openai = client.get_openai_client()

response = openai.responses.create(
    extra_body={
        "agent_reference": {
            "name": AGENT_NAME,
            "type": "agent_reference"
        }
    },
    input="What are some common reasons a payment might remain pending?"
)

print(f"Response: {response.output_text}")
```

The important part is the agent reference:

```python
extra_body={
    "agent_reference": {
        "name": AGENT_NAME,
        "type": "agent_reference"
    }
}
```

Instead of putting the entire agent instruction set into the request, the request identifies the agent that should handle the interaction.

The input is then simply:

```python
input="What are some common reasons a payment might remain pending?"
```

---

# 3. Testing the Agent

Now that the agent can be invoked, we can test how it behaves.

## Test 1 — Payments Question

```text
What are some common reasons a payment might remain pending?
```

We would expect a response focused on legitimate payment-processing reasons such as:

- Awaiting processing
- Compliance checks
- Insufficient information
- Downstream system delays
- Settlement dependencies
- Retry or exception handling

The important point isn't the exact response.

We're testing whether the agent follows its intended role.

---

## Test 2 — Another Payments Question

Try:

```text
What is the difference between payment processing and settlement?
```

Again, the agent should provide an explanation relevant to payments operations.

---

## Test 3 — Out-of-Scope Question

Now try something completely unrelated:

```text
What's the best recipe for lasagna?
```

Our instructions explicitly tell the agent:

```text
If the question is outside payments operations, politely say so.
```

This provides a simple demonstration that the agent isn't just being used as a generic chatbot.

Its behavior is constrained by the instructions we defined.

---

# 4. Agent vs. A Simple LLM Request

At first glance, this example might look very similar to simply sending a prompt to an LLM.

And that's a reasonable observation.

This example is intentionally simple.

We're not yet giving the agent:

- Tools
- APIs
- Databases
- RAG
- External knowledge
- Actions
- Workflow orchestration

Instead, we're establishing the basic foundation.

The agent currently consists primarily of:

```text
Model
  +
Instructions
  +
Managed Agent Definition
```

The interesting part comes when we start adding capabilities.

---

# 5. Where This Gets More Interesting

Imagine extending this Payments Operations Agent.

Instead of only answering general questions, it could eventually have access to tools such as:

```text
Payments Operations Agent
          │
          ├── Payment Status API
          │
          ├── Transaction Search
          │
          ├── Reconciliation Service
          │
          ├── Payment Documentation
          │
          └── Incident Management System
```

Then an operations user could ask:

```text
Why is payment 123456 still pending?
```

The agent could potentially:

1. Identify the payment reference.
2. Call a payment-status tool.
3. Inspect the current state.
4. Retrieve relevant payment-processing information.
5. Explain the likely reason.
6. Recommend the next operational action.

At that point we're moving beyond a simple prompt-driven assistant and toward a more capable **agentic system**.

---

# 6. Why I'm Using Payments as the Example

The original learning material uses an IT Help Desk scenario.

That's useful for learning the mechanics, but I wanted to use a domain that is closer to the systems I work with.

Payments are also an interesting domain for exploring AI because they combine:

- Distributed systems
- Event-driven architecture
- APIs
- Messaging
- Data consistency
- Reconciliation
- Security
- Auditability
- Operational workflows

These characteristics make payments a particularly interesting environment for exploring where agents can help — and where strong controls are required.

An agent that can explain a problem is one thing.

An agent that can **take action against a financial system** is a very different proposition.

That distinction becomes increasingly important as we move toward more autonomous AI systems.

---

# 7. What I Learned

This small example helped clarify a few concepts for me.

### 1. An agent is more than just a chat prompt

The agent has a managed definition containing its model and instructions.

### 2. Agent instructions establish behavior

The instructions influence what the agent considers in-scope and how it should respond.

### 3. Agents can be referenced by applications

The application can invoke an existing agent rather than rebuilding its configuration for every request.

### 4. This is only the beginning

A prompt-based agent is a useful starting point, but real agentic applications become much more interesting when we introduce:

- Tools
- Knowledge
- Retrieval
- Function calling
- State
- Evaluation
- Security
- Human approval
- Observability

---

# 8. What's Next?

This is the first step in my AI-103 learning journey.

The next logical step is to give the Payments Operations Agent an actual capability.

For example:

> **"Give the agent a tool that can retrieve payment information."**

That introduces a much more interesting question:

> How do we allow an AI agent to interact with real systems while keeping the behavior controlled, observable, and safe?

That's where agentic AI starts becoming much more than simply asking an LLM questions.

---

## Final Thoughts

This example is intentionally small.

The objective wasn't to build a production-ready payments agent.

It was to understand the basic lifecycle:

```text
Create Agent
     ↓
Define Model + Instructions
     ↓
Create Agent Version
     ↓
Reference Agent
     ↓
Invoke Agent
     ↓
Receive Response
```

From here, the same foundation can be extended with tools, knowledge, RAG, evaluation, security controls, and eventually more sophisticated agentic workflows.

For me, the interesting part is not just learning how to create an agent.

It's understanding **how agents can be safely integrated with the enterprise systems that already exist**.

And payments provide a particularly good environment to explore that problem.

---

*This post is part of my AI-103 learning journey, where I'm exploring Microsoft Foundry and agentic AI concepts through practical examples.*
