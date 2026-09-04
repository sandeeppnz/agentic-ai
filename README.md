# Payments Operations Agent

This project explores how to create and invoke an AI agent with Microsoft Foundry using a payments operations scenario.

The example is based on the core agent lifecycle:

1. Create an agent in Microsoft Foundry
2. Configure instructions and model behavior
3. Reference the agent through the Responses API
4. Send user requests and receive structured responses

## Project goal

The project demonstrates a simple Payments Operations Assistant that helps with common payment-processing topics such as:

- Failed payments
- Pending payments
- Settlement delays
- Reconciliation issues
- Operational troubleshooting guidance

The focus is on learning the fundamentals of agent creation and invocation rather than building a full production payment system.

## Repository contents

- `01_payments_operations_agent.md` — walkthrough and sample code for creating and invoking the agent

## Prerequisites

Before running the examples, make sure you have:

- An Azure subscription
- A Microsoft Foundry project
- A deployed model in that project
- Python installed locally
- Azure authentication configured (for example with `DefaultAzureCredential`)
- The Azure AI Projects SDK available in your environment

## Main concept

The agent is created with a clear operational role and guardrails. It helps with payments operations questions while avoiding unsupported assumptions, such as inventing transaction details or payment status.

## Example flow

The project follows this pattern:

```text
Microsoft Foundry
    │
    ├─ Create agent version
    ├─ Set instructions and model
    └─ Store the agent reference

Responses API
    │
    ├─ Send input request
    ├─ Resolve agent reference
    └─ Return the agent response
```

## Key idea

This project is intentionally simple and educational. It focuses on understanding:

- how agents are defined in Foundry
- how they are named and referenced
- how to invoke them through the Responses API
- how to apply instructions and safety boundaries to an agent

## Notes

This is a learning exercise and not a complete production payments tooling solution. It is meant to help understand the agent workflow before adding tools, knowledge bases, RAG, actions, or more advanced orchestration.

## Next steps

After this foundational example, the next evolution could include:

- connecting the agent to real payment system data
- adding retrieval-augmented generation (RAG)
- creating tool-backed operations
- implementing guardrails and response validation
- building a production workflow around the agent

## License

This project is intended for learning and experimentation.
