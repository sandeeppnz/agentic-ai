# Building a Tool-Using AI Agent with Microsoft Foundry — Giving a Payments Agent Its First Capability

In my previous post, I created and invoked a simple AI agent using Microsoft Foundry.

The agent could answer questions about payments operations based on its instructions.

But there was an obvious limitation.

It could **talk about payments**, but it couldn't actually **do anything**.

That leads to the next concept I'm exploring as part of my **AI-103 learning journey**:

> **How can we give an AI agent access to capabilities that exist outside the model itself?**

In this exercise, I'm exploring **function calling / tool use**.

Rather than using the IT Help Desk example from the learning material, I've adapted the same concept to a payments operations scenario.

The result is a simple **Payments Operations Agent with tools**.

---

# From Agent to Tool-Using Agent

The first version of my agent looked roughly like this:

```text
User
  │
  ▼
Payments Agent
  │
  ▼
LLM
  │
  ▼
Response
```

The agent could explain concepts such as:

- Why payments might remain pending
- What settlement means
- What reconciliation is
- Common payment-processing issues

But it had no way to retrieve information from an external system.

Now we're changing the architecture.

```text
User
  │
  ▼
Payments Operations Agent
  │
  ▼
LLM decides whether a tool is required
  │
  ▼
Function Call
  │
  ▼
Application executes function
  │
  ▼
Tool Result
  │
  ▼
Agent
  │
  ▼
Final Response
```

This is the key concept I'm exploring in this exercise.

---

# What Is a Tool?

A useful way to think about a tool is:

> **A capability that the AI agent can request from the surrounding application.**

The model doesn't directly execute our Python function.

Instead, we describe a function that the agent can use.

For example:

```text
get_payment_status
```

The agent can decide:

> "I need payment information to answer this question."

It then generates a function call.

Our application receives that function call, executes the actual code, and sends the result back to the agent.

This distinction is important.

The model decides **what capability it needs**.

The application controls **what actually gets executed**.

---

# The Payments Scenario

For this exercise, I'm going to give the agent three simple capabilities:

```text
Payments Operations Agent
        │
        ├── get_payment_status
        │
        ├── get_payment_processing_guidance
        │
        └── get_reconciliation_guidance
```

These are deliberately simple local functions.

In a real application, these could eventually represent:

- REST APIs
- Database queries
- Microservices
- Internal platforms
- Search systems
- Other enterprise capabilities

The goal here isn't to build those integrations yet.

It's to understand the **tool-calling lifecycle**.

---

# Creating the Tools

The first step is describing the tools to the agent.

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.projects.models import PromptAgentDefinition, FunctionTool

PROJECT_ENDPOINT = "<foundry-project-endpoint>"
AGENT_NAME = "Payments-Operations-Agent"
DEPLOYMENT_NAME = "gpt-4.1-mini"

client = AIProjectClient(
    endpoint=PROJECT_ENDPOINT,
    credential=DefaultAzureCredential()
)

tools = [
    FunctionTool(
        name="get_payment_status",
        description="Returns the current processing status and operational information for a payment.",
        parameters={
            "type": "object",
            "properties": {
                "payment_id": {
                    "type": "string",
                    "description": "The unique payment identifier."
                }
            },
            "required": ["payment_id"],
            "additionalProperties": False
        }
    ),

    FunctionTool(
        name="get_payment_processing_guidance",
        description="Returns operational guidance for investigating payment processing issues.",
        parameters={
            "type": "object",
            "properties": {},
            "required": [],
            "additionalProperties": False
        }
    ),

    FunctionTool(
        name="get_reconciliation_guidance",
        description="Returns guidance for investigating payment reconciliation discrepancies.",
        parameters={
            "type": "object",
            "properties": {},
            "required": [],
            "additionalProperties": False
        }
    )
]
```

There are two important things happening here.

First, we're giving each function a name and description.

Second, we're describing the parameters the function expects.

For example:

```python
FunctionTool(
    name="get_payment_status",
    ...
)
```

requires:

```text
payment_id
```

This gives the agent enough information to understand when the function might be useful and what information it needs to provide.

---

# Why the Description Matters

Consider:

```python
description="Returns the current processing status and operational information for a payment."
```

The model uses the tool definition to understand what the capability is for.

It's therefore important to make tool descriptions clear.

Compare:

```text
get_payment_status
```

with:

```text
Returns the current processing status and operational information for a payment.
```

The second description gives the model considerably more context about when it should use the function.

Tool definitions effectively become part of the interface between the AI model and our application.

---

# Creating the Agent

Now we can create the agent and attach the tools.

```python
agent = client.agents.create_version(
    agent_name=AGENT_NAME,
    definition=PromptAgentDefinition(
        model=DEPLOYMENT_NAME,
        instructions=(
            "You are a Payments Operations Assistant for a banking platform. "
            "Help operations teams understand payment processing issues, "
            "including failed payments, pending payments, settlement delays, "
            "and reconciliation discrepancies. "
            "Use the available tools when specific operational information "
            "is required. "
            "Provide clear, structured explanations. "
            "Do not invent payment status or transaction information. "
            "If the question is outside payments operations, politely say so."
        ),
        tools=tools
    )
)

print("Agent created:")
print(f"  ID      : {agent.id}")
print(f"  Name    : {agent.name}")
print(f"  Version : {agent.version}")
```

Notice the difference from the previous exercise.

Previously we had:

```python
definition=PromptAgentDefinition(
    model=DEPLOYMENT_NAME,
    instructions=...
)
```

Now we also have:

```python
tools=tools
```

The agent has gained access to capabilities that it can request.

---

# Implementing the Functions

Now we need to implement the actual functions.

For this learning exercise, I'm keeping the implementations local and deliberately simple.

```python
def get_payment_status(payment_id: str) -> str:
    payments = {
        "PAY-1001": {
            "status": "PENDING",
            "reason": "Awaiting downstream settlement confirmation."
        },
        "PAY-1002": {
            "status": "COMPLETED",
            "reason": "Payment successfully processed and settled."
        },
        "PAY-1003": {
            "status": "FAILED",
            "reason": "Payment rejected by the downstream processing system."
        }
    }

    payment = payments.get(payment_id)

    if not payment:
        return f"No payment information found for '{payment_id}'."

    return (
        f"Payment {payment_id}: "
        f"Status = {payment['status']}. "
        f"Reason = {payment['reason']}"
    )
```

We can also define other capabilities:

```python
def get_payment_processing_guidance() -> str:
    return (
        "Payment investigation guidance: "
        "1. Confirm the payment identifier. "
        "2. Check the current processing state. "
        "3. Review downstream processing responses. "
        "4. Check whether the payment is awaiting settlement. "
        "5. Check retry or exception queues. "
        "6. Escalate unresolved issues to the payments operations team."
    )
```

And:

```python
def get_reconciliation_guidance() -> str:
    return (
        "Reconciliation investigation guidance: "
        "1. Identify the payment or settlement reference. "
        "2. Compare the source transaction with the settlement record. "
        "3. Check for timing differences. "
        "4. Check for missing or duplicate transactions. "
        "5. Review suspense or exception records. "
        "6. Escalate unresolved discrepancies for investigation."
    )
```

---

# Connecting Tool Names to Functions

We now need a small dispatcher.

```python
def run_local_function(function_name: str, arguments: dict) -> str:

    if function_name == "get_payment_status":
        return get_payment_status(**arguments)

    if function_name == "get_payment_processing_guidance":
        return get_payment_processing_guidance()

    if function_name == "get_reconciliation_guidance":
        return get_reconciliation_guidance()

    return f"Unknown function requested: {function_name}"
```

This is intentionally straightforward.

In a real application, this layer could instead route to services such as:

```text
get_payment_status
        ↓
Payment Service
        ↓
Payment Database / API
```

or:

```text
get_reconciliation_guidance
        ↓
Knowledge / Documentation Service
```

The architecture remains conceptually similar.

---

# Invoking the Agent

Now comes the interesting part.

We create a conversation and invoke the agent.

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
import json

PROJECT_ENDPOINT = "<foundry-project-endpoint>"

project_client = AIProjectClient(
    endpoint=PROJECT_ENDPOINT,
    credential=DefaultAzureCredential()
)

client = project_client.get_openai_client()

AGENT_NAME = "Payments-Operations-Agent"
AGENT_VERSION = "1"

conversation = client.conversations.create()

response = client.responses.create(
    conversation=conversation.id,
    input="Why is payment PAY-1001 still pending?",
    extra_body={
        "agent_reference": {
            "name": AGENT_NAME,
            "type": "agent_reference",
            "version": AGENT_VERSION
        }
    }
)
```

At this point, something different can happen.

The agent may determine that it needs to call:

```text
get_payment_status
```

because the question asks about a specific payment.

---

# Detecting the Function Call

Our application can inspect the response:

```python
tool_outputs = []

for item in response.output:

    if item.type == "function_call":

        function_name = item.name
        arguments = json.loads(item.arguments)

        print(f"Function requested: {function_name}")
        print(f"Arguments received: {arguments}")

        function_result = run_local_function(
            function_name,
            arguments
        )

        tool_outputs.append(
            {
                "type": "function_call_output",
                "call_id": item.call_id,
                "output": function_result,
            }
        )
```

This is probably the most important part of the entire exercise.

The model has effectively said:

```text
I need:

get_payment_status(
    payment_id="PAY-1001"
)
```

But the model isn't executing our Python function.

Our application receives the request.

---

# The Application Executes the Tool

We then execute:

```python
function_result = run_local_function(
    function_name,
    arguments
)
```

For our example, that ultimately calls:

```python
get_payment_status("PAY-1001")
```

which might return:

```text
Payment PAY-1001:
Status = PENDING.
Reason = Awaiting downstream settlement confirmation.
```

We then package that result as a function output:

```python
{
    "type": "function_call_output",
    "call_id": item.call_id,
    "output": function_result,
}
```

The `call_id` is important because it associates the result with the function call requested by the model.

---

# Sending the Tool Result Back

Now we send the result back to the agent.

```python
if tool_outputs:

    final_response = client.responses.create(
        conversation=conversation.id,
        input=tool_outputs,
        extra_body={
            "agent_reference": {
                "type": "agent_reference",
                "name": AGENT_NAME,
                "version": AGENT_VERSION,
            }
        },
    )

    print(final_response.output_text)

else:
    print(response.output_text)
```

The agent can now use the tool result to produce its final response.

Conceptually:

```text
User
 │
 │ "Why is PAY-1001 still pending?"
 ▼
Agent
 │
 │ "I need payment status"
 ▼
Function Call
 │
 │ get_payment_status("PAY-1001")
 ▼
Application
 │
 │ Executes Python function
 ▼
Tool Result
 │
 │ PENDING
 │ Awaiting settlement confirmation
 ▼
Agent
 │
 ▼
Final Answer
```

This is the fundamental **tool-calling loop**.

---

# What Actually Changed?

Compared with my first agent, we now have another layer.

### Before

```text
User
  ↓
Agent
  ↓
Model
  ↓
Answer
```

### Now

```text
User
  ↓
Agent
  ↓
Model
  ↓
Tool decision
  ↓
Function
  ↓
Tool result
  ↓
Model
  ↓
Answer
```

The model can now use information supplied by the application.

That is a significant step forward.

---

# The Agent Isn't the Tool

One of the most useful things I took away from this exercise is that it's important to separate the **agent** from the **capabilities it can use**.

The agent decides:

> "I need payment status."

The application decides:

> "Here is the implementation of `get_payment_status`."

This gives the application an important control boundary.

For example, the application can:

- Validate arguments
- Authorize access
- Apply business rules
- Log the request
- Handle errors
- Rate-limit calls
- Reject unsafe operations

The model doesn't automatically receive unrestricted access to the underlying system.

---

# Why This Matters in Payments

This distinction becomes particularly important in financial systems.

Imagine a future agent with a tool:

```text
initiate_payment()
```

Giving an AI model access to that capability is very different from giving it:

```text
get_payment_status()
```

A read-only tool might allow an agent to retrieve information.

A write operation could potentially change the state of a financial system.

That immediately raises questions around:

- Authorization
- Human approval
- Transaction limits
- Audit logging
- Fraud controls
- Idempotency
- Validation
- Rollback
- Monitoring

So while this exercise is technically about **function calling**, it also starts to demonstrate why agentic AI needs to be treated as an engineering problem rather than simply an LLM integration.

---

# Is This Agent + Knowledge?

Not quite.

This exercise is primarily:

> **Agent + Tools**

The distinction is useful.

| Capability | What it provides |
|---|---|
| Agent instructions | Behaviour and role |
| Tools / functions | Ability to interact with external capabilities |
| Knowledge / RAG | Ability to retrieve relevant information from a knowledge source |
| Tool + Knowledge | Ability to retrieve information and take actions using external capabilities |

Our current functions return predefined information.

We haven't introduced document retrieval or a knowledge base yet.

That's coming next.

---

# What's Next?

The next step I'm interested in is adding **knowledge**.

For example, imagine giving the Payments Operations Agent access to:

```text
Payment Operations Documentation
        │
        ├── Payment Processing Guide
        ├── Settlement Procedures
        ├── Reconciliation Runbook
        ├── Exception Handling
        └── Incident Procedures
```

Then we could have:

```text
                Payments Operations Agent
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
            Tools                Knowledge
              │                     │
       Payment APIs          Payment Documentation
              │                     │
              └──────────┬──────────┘
                         ▼
                   Final Response
```

That would move us from:

**Agent → Agent + Tools → Agent + Knowledge**

and eventually towards:

**Agent + Tools + Knowledge.**

---

# Final Thoughts

This exercise was a small but important step in my AI-103 learning journey.

My first agent could answer questions.

This agent can now **request capabilities from the surrounding application**.

The key pattern is:

```text
Agent
  ↓
Decides which capability it needs
  ↓
Function Call
  ↓
Application executes capability
  ↓
Tool Result
  ↓
Agent
  ↓
Final Response
```

The most important thing I took away is that the agent doesn't need direct unrestricted access to everything.

Instead, we can expose **specific capabilities** through well-defined tools.

That creates an interesting engineering boundary between:

**AI reasoning**

and

**system execution.**

As agents become more capable, I think this boundary becomes increasingly important.

The next question is:

> **What happens when we give the agent both tools and a knowledge base?**

That's the next step in my exploration.

---

*This post is part of my AI-103 learning journey, where I'm exploring Microsoft Foundry and agentic AI through practical, domain-focused examples.*