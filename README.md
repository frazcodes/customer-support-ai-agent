# Customer Support AI Agent

> A production-oriented AI customer support agent built with **Amazon Bedrock AgentCore, Strands Agents, Amazon Bedrock Knowledge Bases, AgentCore Memory, AgentCore Browser, Code Interpreter, and MCP-based Gateway tools**.

This project demonstrates how modern AI agent infrastructure can be combined to build an intelligent customer support system capable of answering product and policy questions, retrieving customer and order information, calculating loyalty discounts, interacting with web pages, processing refund workflows, and maintaining long-term customer context.

The agent is deployed as an **Amazon Bedrock AgentCore Runtime** and uses a modular tool architecture so that knowledge retrieval, customer operations, browser automation, code execution, and memory can be handled by specialized components.

---

## Overview

Traditional customer support systems typically depend on predefined workflows and static FAQ pages.

This project takes a different approach by combining a **reasoning-capable AI agent** with specialized tools and managed AWS services.

The agent can:

* Answer product and support questions using a Bedrock Knowledge Base
* Retrieve customer and order information
* Initiate and check refund workflows
* Generate return-label information
* Calculate loyalty discounts dynamically
* Execute calculations through AgentCore Code Interpreter
* Browse webpages through AgentCore Browser
* Use MCP tools exposed through AgentCore Gateway
* Remember relevant customer information across conversations
* Maintain session and customer-specific context
* Respond through an HTTP-based AgentCore Runtime

---

## Architecture

```text
                         Customer
                            │
                            ▼
                  ┌─────────────────────┐
                  │  AgentCore Runtime  │
                  │    CustomerSupport  │
                  └──────────┬──────────┘
                             │
                             ▼
                    ┌────────────────┐
                    │ Strands Agent  │
                    │ Amazon Nova 2  │
                    │     Lite       │
                    └───────┬────────┘
                            │
          ┌─────────────────┼──────────────────┐
          │                 │                  │
          ▼                 ▼                  ▼
 ┌────────────────┐ ┌────────────────┐ ┌─────────────────┐
 │ Bedrock        │ │ AgentCore     │ │ AgentCore       │
 │ Knowledge Base │ │ Memory        │ │ Browser         │
 │                │ │               │ │ + Playwright    │
 └────────────────┘ └────────────────┘ └─────────────────┘
          │                 │                  │
          │                 │                  │
          ▼                 ▼                  ▼
   Product / Policy    Customer Context    Live Web Pages


                            │
                            ▼
                   ┌──────────────────┐
                   │ AgentCore        │
                   │ Gateway          │
                   │ MCP Tools        │
                   └────────┬─────────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 ▼                     ▼
        ┌─────────────────┐   ┌──────────────────┐
        │ Order Tracker   │   │ Refund Processor │
        │ AWS Lambda      │   │ AWS Lambda       │
        └─────────────────┘   └──────────────────┘


                            │
                            ▼
                  ┌──────────────────┐
                  │ AgentCore Code   │
                  │ Interpreter      │
                  └──────────────────┘
```

---

## Core Capabilities

### 1. Knowledge Base Search

The agent uses an Amazon Bedrock Knowledge Base to retrieve relevant information before answering questions related to:

* Products
* Product policies
* Customer support information
* Loyalty programs
* Other indexed business knowledge

The knowledge retrieval tool uses the Bedrock Runtime `retrieve` API and returns relevant document chunks to the agent.

This helps prevent the agent from relying solely on model knowledge when answering business-specific questions.

---

### 2. Customer & Order Information

Customer-specific information is exposed through Gateway/MCP tools backed by AWS Lambda.

The current demonstration backend provides:

* Customer profiles
* Customer order history
* Individual order information
* Order status
* Tracking information
* Estimated delivery information

Example supported routes:

```text
GET /orders/{order_id}

GET /customers/{customer_id}/orders

GET /customers/{customer_id}
```

The current Lambda implementation uses mock in-memory data for demonstration purposes.

A production deployment could replace this layer with DynamoDB, Aurora, RDS, or another persistent transactional data source without changing the overall agent architecture.

---

### 3. Loyalty Discount Calculator

The agent includes a dedicated loyalty-discount tool powered by **Amazon Bedrock AgentCore Code Interpreter**.

The calculator considers:

* Customer loyalty points
* Loyalty tier
* Order total
* Product category
* Points redemption
* Tier-based discounts
* Discount limits
* Points earned from the purchase

Current product earning rates include different values for standard, device, and fresh categories.

Supported loyalty tiers include:

* Silver
* Gold
* Platinum

The calculation is intentionally executed through Code Interpreter rather than relying on the language model to perform the business calculation itself.

This provides a useful separation between:

```text
AI reasoning
      ↓
Tool selection
      ↓
Code execution
      ↓
Deterministic calculation
```

---

### 4. AgentCore Browser

The agent can use **AgentCore Browser** for requests that require live webpage interaction.

The project integrates browser functionality through Playwright and AgentCore Browser.

Browser-related requests are intentionally separated from knowledge-base questions.

For example:

```text
"Tell me about the return policy."
```

uses the Knowledge Base.

Whereas a request such as:

```text
"Open this webpage and check the current information."
```

can use the Browser tool.

This separation helps the agent select the appropriate information source based on the task.

---

### 5. AgentCore Gateway + MCP

The project uses an **AgentCore Gateway** as the integration layer for external business tools.

The Strands agent connects to the Gateway through an MCP client:

```text
Strands Agent
     │
     ▼
MCP Client
     │
     ▼
AgentCore Gateway
     │
     ├── Order tools
     │
     └── Refund tools
```

This architecture makes backend capabilities available to the agent without embedding all business logic directly into the agent application.

---

## Refund Processing

The refund backend exposes three MCP-compatible tools:

### `initiate_refund`

Starts a refund request for an order.

Inputs:

```json
{
  "order_id": "ORD-001",
  "reason": "Product damaged",
  "amount": 89.99
}
```

### `check_refund_status`

Checks the state of an existing refund.

```json
{
  "refund_id": "REF-XXXXXXXX"
}
```

### `get_return_label`

Generates simulated return-label information.

```json
{
  "order_id": "ORD-001"
}
```

The current implementation is a demonstration backend and does not connect to a real payment processor or fulfillment system.

---

## Long-Term Memory

The agent uses **Amazon Bedrock AgentCore Memory** to maintain customer context between interactions.

The implementation uses a `MemoryHook` abstraction that:

1. Identifies the customer
2. Identifies the conversation session
3. Retrieves relevant memories
4. Adds customer context to the agent prompt
5. Executes the agent
6. Saves the completed interaction

Conceptually:

```text
Previous conversations
        │
        ▼
AgentCore Memory
        │
        ▼
Memory retrieval
        │
        ▼
Customer Context
        │
        ▼
Strands Agent
        │
        ▼
New interaction
        │
        ▼
Memory persistence
```

This allows the support agent to become more context-aware across conversations.

---

## Agent Reasoning Flow

A typical request follows this process:

```text
Customer Request
       │
       ▼
AgentCore Runtime
       │
       ▼
Strands Agent
       │
       ├── Need business knowledge?
       │        └──► Knowledge Base
       │
       ├── Need customer/order data?
       │        └──► Gateway / MCP
       │
       ├── Need refund operation?
       │        └──► Refund Lambda
       │
       ├── Need deterministic calculation?
       │        └──► Code Interpreter
       │
       ├── Need live webpage interaction?
       │        └──► AgentCore Browser
       │
       └── Need previous customer context?
                └──► AgentCore Memory
```

The system prompt explicitly instructs the agent to use the appropriate tool instead of inventing information.

---

## Technology Stack

| Technology                    | Purpose                           |
| ----------------------------- | --------------------------------- |
| Python 3.14                   | Application runtime               |
| Strands Agents                | Agent orchestration               |
| Amazon Bedrock                | Foundation model infrastructure   |
| Amazon Nova 2 Lite            | Agent model                       |
| Bedrock AgentCore Runtime     | Agent deployment/runtime          |
| AgentCore Memory              | Long-term customer context        |
| AgentCore Browser             | Web interaction                   |
| AgentCore Code Interpreter    | Deterministic code execution      |
| AgentCore Gateway             | External tool integration         |
| MCP                           | Tool communication                |
| Amazon Bedrock Knowledge Base | Business knowledge retrieval      |
| AWS Lambda                    | Backend business tools            |
| Playwright                    | Browser automation                |
| uv                            | Python dependency management      |
| AWS CDK                       | Infrastructure/deployment support |

---

## Project Structure

```text
customer-support-ai-agent/
│
├── agentcore/
│   ├── agentcore.json
│   ├── aws-targets.json
│   └── cdk/
│
├── lambda/
│   ├── order_tracker.py
│   ├── refund_processor.py
│   └── lambda_schema
│
├── main.py
├── product_catalog.txt
├── order-tracker.zip
├── refund-processor.zip
│
├── pyproject.toml
├── uv.lock
├── README.md
└── .gitignore
```

---

## AgentCore Configuration

The AgentCore runtime is configured as:

```json
{
  "name": "CustomerSupport",
  "version": 1,
  "managedBy": "CDK"
}
```

Runtime configuration:

```text
Runtime:        CustomerSupport
Build:          CodeZip
Entrypoint:     main.py
Python:         3.14
Network Mode:   PUBLIC
Protocol:       HTTP
```

The project targets AWS:

```text
Region: us-east-1
```

AWS resource identifiers are intentionally kept out of this README where possible. Environment- or deployment-specific identifiers should be configured in the project rather than hard-coded into public documentation.

---

## Installation

### Prerequisites

You should have:

* Python 3.14+
* AWS CLI
* AWS credentials with the required permissions
* `uv`
* AgentCore Starter Toolkit
* Access to Amazon Bedrock
* An appropriately configured AWS environment

Verify Python:

```bash
python --version
```

Verify AWS:

```bash
aws sts get-caller-identity
```

Verify the region:

```bash
aws configure get region
```

---

## Install Dependencies

Clone the repository:

```bash
git clone https://github.com/frazcodes/customer-support-ai-agent.git
cd customer-support-ai-agent
```

Create/sync the environment:

```bash
uv sync
```

Activate the environment if required:

```bash
source .venv/bin/activate
```

---

## Configuration

The application currently reads its main AWS integration configuration from `main.py`.

The important components are:

```text
AWS Region
AgentCore Memory
Bedrock Knowledge Base
AgentCore Gateway
Amazon Bedrock Model
```

For a production deployment, these values should preferably be moved into environment variables or a secure configuration mechanism rather than committed directly to source code.

---

## Running Locally

The project uses:

```python
app = BedrockAgentCoreApp()
```

and exposes the agent through the AgentCore entrypoint:

```python
@app.entrypoint
async def invoke(payload, context=None):
```

The application can therefore be deployed and invoked through AgentCore.

The project also contains a local CLI implementation for development/testing.

---

## Deploying to AgentCore

The project uses the AgentCore configuration located in:

```text
agentcore/agentcore.json
agentcore/aws-targets.json
```

The configured runtime is:

```text
CustomerSupport
```

After AWS credentials and dependencies are configured, deployment can be performed through the AgentCore CLI.

Check deployment status:

```bash
agentcore status
```

Deploy:

```bash
agentcore deploy
```

Invoke the deployed agent:

```bash
agentcore invoke "Where is my order?"
```

For session-aware testing:

```bash
agentcore invoke --session-id <session-id>
```

> Exact deployment behavior depends on the installed AgentCore Starter Toolkit version and the AWS environment.

---

## Example Requests

### Product Question

```text
What is the return policy for this product?
```

Expected flow:

```text
Agent
  ↓
Knowledge Base
  ↓
Relevant policy information
  ↓
Customer response
```

---

### Order Tracking

```text
Where is my order ORD-001?
```

Expected flow:

```text
Agent
  ↓
Gateway
  ↓
Order Tracker Lambda
  ↓
Order information
  ↓
Customer response
```

---

### Customer Order History

```text
Show me my recent orders.
```

Expected flow:

```text
Agent
  ↓
Gateway / MCP
  ↓
Customer Orders API
  ↓
Order history
```

---

### Loyalty Discount

```text
I have 4250 loyalty points and a $200 device purchase. What discount can I get?
```

Expected flow:

```text
Agent
  ↓
Loyalty Discount Tool
  ↓
AgentCore Code Interpreter
  ↓
Deterministic calculation
  ↓
Discount result
```

---

### Refund

```text
I want a refund for ORD-001 because the product arrived damaged.
```

Expected flow:

```text
Agent
  ↓
Gateway / MCP
  ↓
Refund Processor Lambda
  ↓
Refund result
  ↓
Customer response
```

---

### Browser

```text
Open the requested webpage and check the current information.
```

Expected flow:

```text
Agent
  ↓
AgentCore Browser
  ↓
Playwright
  ↓
Live webpage
  ↓
Extracted information
```

---

## Design Principles

### Tool-Based Architecture

Business capabilities are implemented as tools instead of placing all functionality inside the model prompt.

This makes the system easier to extend and maintain.

### Grounded Responses

The agent is instructed to retrieve business information from the Knowledge Base rather than inventing product or policy information.

### Deterministic Computation

Financial and loyalty calculations are delegated to Code Interpreter rather than relying on free-form model arithmetic.

### Separation of Concerns

The project separates:

```text
Agent reasoning
Knowledge retrieval
Memory
Browser interaction
Business APIs
Refund operations
Code execution
```

This creates a cleaner architecture for future production integrations.

### Modular Integrations

Gateway/MCP allows additional backend tools to be introduced without rewriting the core agent.

---

## Security Considerations

The repository intentionally does not include:

* AWS access keys
* Secret keys
* API credentials
* Session credentials
* Private tokens
* `.env` files
* AgentCore CLI deployment state

Generated AgentCore CLI state is excluded through `.gitignore`.

For production deployments, sensitive configuration should be managed through appropriate AWS services such as:

* IAM roles
* AWS Secrets Manager
* AWS Systems Manager Parameter Store
* Environment-specific configuration

Least-privilege IAM permissions should also be applied to the runtime and backend services.

---

## Current Limitations

This repository is a **production-oriented reference implementation**, but several backend components are intentionally simplified for demonstration.

### Mock Customer Data

The order tracker currently uses hard-coded demonstration data.

A production implementation should replace this with a persistent database.

### Mock Refund Processing

The refund processor simulates refund operations.

A production system would integrate with an actual payment/refund provider.

### Demonstration Return Labels

Return-label generation is simulated rather than connected to a real shipping provider.

### Public Runtime

The current AgentCore configuration uses:

```text
networkMode: PUBLIC
```

A production environment may require a more restrictive networking and access model depending on the application's requirements.

---

## Future Improvements

Potential production enhancements include:

* DynamoDB-backed customer and order storage
* Real payment/refund provider integration
* Real shipping-provider integration
* Authentication and customer identity verification
* Role-based authorization
* Private networking where appropriate
* AWS Secrets Manager integration
* Automated CI/CD
* Automated integration tests
* Agent evaluation pipelines
* Guardrails and policy enforcement
* Structured observability dashboards
* Cost monitoring
* Conversation analytics
* Human-agent escalation
* Multi-language customer support
* Streaming responses
* Persistent business transaction auditing

---

## Observability

AgentCore and AWS logging can be used to inspect runtime behavior and troubleshoot:

* Agent invocation errors
* Tool execution
* Browser failures
* Knowledge Base retrieval
* Gateway/MCP calls
* Memory operations
* Lambda execution
* Runtime deployment issues

CloudWatch should be used as the primary operational logging layer for deployed workloads.

---

## Development Workflow

A typical development cycle is:

```text
1. Modify agent/tool code
        ↓
2. Run local validation
        ↓
3. Test individual tools
        ↓
4. Deploy AgentCore runtime
        ↓
5. Invoke with representative prompts
        ↓
6. Inspect CloudWatch logs
        ↓
7. Fix / improve
        ↓
8. Commit changes
        ↓
9. Push to GitHub
```

Git workflow:

```bash
git add .
git commit -m "describe the change"
git push
```

---

## What This Project Demonstrates

This project brings together several important concepts in modern AI engineering:

* AI agent orchestration
* Tool calling
* Retrieval-augmented generation
* Long-term agent memory
* MCP-based integrations
* Serverless backend services
* Browser automation
* Code execution
* AWS managed AI infrastructure
* Agent deployment
* Cloud observability
* Modular agent architecture

Rather than building a chatbot that only generates text, the project demonstrates an agent capable of **reasoning about a customer request and selecting the appropriate external capability to complete the task**.

---

## Repository

**GitHub:** `frazcodes/customer-support-ai-agent`

The repository contains the source code, AgentCore configuration, Lambda integrations, dependency configuration, and supporting project files required to reproduce the application in a suitably configured AWS environment.

---

## Author

**Ahmad Faraz**

Information Technology student and aspiring **AI Engineer / Full Stack Developer** focused on building AI-powered applications using Python, React, FastAPI, AWS, and modern agent technologies.

### Areas of Focus

* AI Engineering
* Agentic AI
* AWS Bedrock
* Amazon Bedrock AgentCore
* Strands Agents
* Python
* Full Stack Development
* React
* FastAPI
* Cloud & Serverless Architecture

---

## License

This project is intended primarily as a learning and portfolio project demonstrating modern AI agent architecture on AWS.

Add an appropriate open-source license before distributing the project for reuse.
