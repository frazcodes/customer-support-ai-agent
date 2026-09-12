# Customer Support AI Agent

> A production-oriented AI customer support agent built with **Amazon Bedrock AgentCore, Strands Agents, Amazon Bedrock Knowledge Bases, AgentCore Memory, AgentCore Browser, AgentCore Code Interpreter, and MCP-based AgentCore Gateway tools**.

This project demonstrates how modern AI agent infrastructure can be combined to build an intelligent customer support system capable of answering product and policy questions, retrieving customer and order information, calculating loyalty discounts, interacting with live webpages, processing refund workflows, and maintaining long-term customer context across conversations.

The agent is deployed as an **Amazon Bedrock AgentCore Runtime** and uses a modular tool architecture in which knowledge retrieval, customer operations, browser automation, code execution, backend integrations, and memory are handled by specialized components.

---

## Overview

Traditional customer support applications often depend on static FAQ pages and predefined workflows.

This project takes a more agentic approach by combining a reasoning-capable **Strands Agent** with specialized AWS services and external tools.

The agent can:

* Answer product and support questions using an Amazon Bedrock Knowledge Base
* Retrieve customer and order information
* Check order status and tracking information
* Initiate and check refund workflows
* Generate return-label information
* Calculate loyalty discounts using AgentCore Code Interpreter
* Browse live webpages using AgentCore Browser
* Use MCP tools exposed through AgentCore Gateway
* Retrieve relevant long-term customer memories
* Persist customer interactions for future conversations
* Maintain customer-specific context across sessions
* Run as an HTTP-based Amazon Bedrock AgentCore Runtime

The architecture is intentionally modular so individual backend capabilities can be replaced or extended without rewriting the core agent.

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
 │ Amazon Bedrock │ │ AgentCore     │ │ AgentCore       │
 │ Knowledge Base │ │ Memory        │ │ Browser         │
 │                │ │               │ │ + Playwright    │
 └────────────────┘ └────────────────┘ └─────────────────┘
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

The agent uses an **Amazon Bedrock Knowledge Base** to retrieve relevant information before answering business-specific questions.

Knowledge Base retrieval is used for topics such as:

* Products
* Product specifications
* Return policies
* Warranty information
* Loyalty programs
* Support policies
* Other indexed business knowledge

The application uses the Bedrock Runtime retrieval functionality to obtain relevant document content and provide grounded information to the agent.

The system prompt explicitly instructs the agent to use the Knowledge Base for business-specific knowledge instead of relying on general model knowledge.

This helps reduce hallucination when answering questions about company-specific policies and products.

---

## 2. Customer and Order Information

Customer-specific capabilities are exposed through **AgentCore Gateway / MCP tools** backed by AWS Lambda.

The demonstration backend provides capabilities for:

* Customer profiles
* Customer order history
* Individual order information
* Order status
* Tracking information
* Estimated delivery information

The architecture separates agent reasoning from backend business logic:

```text
Customer Request
       │
       ▼
Strands Agent
       │
       ▼
MCP Client
       │
       ▼
AgentCore Gateway
       │
       ▼
AWS Lambda
       │
       ▼
Customer / Order Data
```

The current backend uses demonstration data.

A production implementation could replace the Lambda data layer with services such as:

* Amazon DynamoDB
* Amazon Aurora
* Amazon RDS
* Existing enterprise APIs
* Other transactional data systems

The agent architecture would remain largely unchanged.

---

## 3. Loyalty Discount Calculator

The project includes a dedicated loyalty-discount capability that uses **Amazon Bedrock AgentCore Code Interpreter** for deterministic calculations.

The calculation can consider:

* Customer loyalty points
* Loyalty tier
* Order total
* Product category
* Points redemption
* Tier-based discounts
* Discount limits
* Points earned from purchases

The demonstration supports loyalty tiers such as:

* Silver
* Gold
* Platinum

Instead of asking the language model to perform business calculations directly, the agent delegates calculation logic to executable code.

The conceptual flow is:

```text
Customer Request
       │
       ▼
Strands Agent
       │
       ▼
Loyalty Discount Tool
       │
       ▼
AgentCore Code Interpreter
       │
       ▼
Deterministic Calculation
       │
       ▼
Result
```

This provides a useful separation between:

```text
AI reasoning
      ↓
Tool selection
      ↓
Code execution
      ↓
Deterministic result
```

---

## 4. AgentCore Browser

The agent integrates **Amazon Bedrock AgentCore Browser** for tasks requiring live webpage interaction.

Browser capabilities are exposed through the Strands tool system and use Playwright-based browser automation.

The system prompt distinguishes between static business knowledge and live web information.

For example:

```text
"What is the return policy?"
```

should use:

```text
Amazon Bedrock Knowledge Base
```

while a request such as:

```text
"Open this webpage and check the current information."
```

can use:

```text
AgentCore Browser
       ↓
Playwright
       ↓
Live Webpage
```

The agent is instructed not to claim that it visited a webpage unless the Browser tool was actually used.

---

## 5. AgentCore Gateway + MCP

The project uses **Amazon Bedrock AgentCore Gateway** as the integration layer for external business capabilities.

The Strands agent connects to the Gateway through an MCP client.

```text
Strands Agent
      │
      ▼
MCP Client
      │
      ▼
AgentCore Gateway
      │
      ├── Order Tools
      │
      └── Refund Tools
```

This architecture allows backend functionality to be exposed as tools without embedding all business logic directly inside the AI agent.

It also provides a modular foundation for adding additional business capabilities later.

---

# Refund Processing

The demonstration refund backend exposes MCP-compatible operations for refund workflows.

## `initiate_refund`

Starts a refund request for an order.

Example input:

```json
{
  "order_id": "ORD-001",
  "reason": "Product damaged",
  "amount": 89.99
}
```

---

## `check_refund_status`

Checks the status of an existing refund.

Example input:

```json
{
  "refund_id": "REF-XXXXXXXX"
}
```

---

## `get_return_label`

Generates simulated return-label information for an order.

Example input:

```json
{
  "order_id": "ORD-001"
}
```

The current implementation is a demonstration backend and does not connect to a real payment processor or shipping provider.

A production implementation could integrate these tools with actual payment, fulfillment, and shipping systems.

---

# Long-Term Customer Memory

One of the key features of the project is **Amazon Bedrock AgentCore Memory**.

The agent uses a custom `MemoryHook` to retrieve and persist customer context.

The memory architecture supports customer-specific information such as:

* Preferences
* Previously stated facts
* Relevant conversation context

The current memory configuration includes semantic facts and user preferences.

Example namespaces:

```text
cs_agent/{actorId}/facts
cs_agent/{actorId}/preferences
```

For a customer such as:

```text
CUST-FINAL
```

the runtime resolves these to:

```text
cs_agent/CUST-FINAL/facts
cs_agent/CUST-FINAL/preferences
```

This keeps memory isolated by customer actor ID.

---

## Memory Retrieval Flow

When a user message arrives, the `MemoryHook`:

1. Identifies the customer
2. Extracts the incoming user query
3. Resolves the configured memory namespaces
4. Retrieves relevant memories
5. Adds the retrieved context to the user message
6. Allows the Strands agent to use that context when responding

Conceptually:

```text
Customer Message
       │
       ▼
MemoryHook
       │
       ▼
AgentCore Memory
       │
       ▼
Relevant Customer Context
       │
       ▼
Strands Agent
       │
       ▼
Customer Response
```

---

## Memory Persistence

After an agent invocation completes, the memory hook uses the Strands `AfterInvocationEvent`.

The callback:

```python
registry.add_callback(
    AfterInvocationEvent,
    self.save_support_interaction,
)
```

extracts the relevant user and assistant messages from:

```python
event.agent.messages
```

and persists the interaction using AgentCore Memory.

This keeps memory persistence inside the agent lifecycle rather than requiring a manual memory-save call from the runtime entrypoint.

The resulting lifecycle is:

```text
User Request
      │
      ▼
Memory Retrieval
      │
      ▼
Agent Invocation
      │
      ▼
Agent Response
      │
      ▼
AfterInvocationEvent
      │
      ▼
Memory Persistence
```

---

## Cross-Session Memory

The implementation was tested using the same customer ID across different sessions.

For example, a customer can state:

```text
Remember that I prefer Japan for travel and I like wireless headphones.
```

Later, in a new session, the customer can ask:

```text
What travel destination do I prefer, and what type of product do I like?
```

The deployed runtime can retrieve the previously stored customer context and respond with the remembered preferences.

This demonstrates that the memory is associated with the customer actor rather than being limited to a single runtime session.

---

# Agent Reasoning Flow

A typical customer request follows this process:

```text
Customer Request
       │
       ▼
AgentCore Runtime
       │
       ▼
Strands Agent
       │
       ├── Business knowledge required?
       │        └──► Amazon Bedrock Knowledge Base
       │
       ├── Customer/order information required?
       │        └──► AgentCore Gateway / MCP
       │
       ├── Refund operation required?
       │        └──► Refund Lambda
       │
       ├── Deterministic calculation required?
       │        └──► AgentCore Code Interpreter
       │
       ├── Live webpage required?
       │        └──► AgentCore Browser
       │
       └── Previous customer context required?
                └──► AgentCore Memory
```

The system prompt instructs the agent to select the appropriate tool and avoid inventing customer, order, refund, product, or policy information.

---

# System Prompt Design

The agent is instructed to follow several important rules.

### Ground business-specific answers

For product information, policies, warranties, loyalty benefits, and other Knowledge Base content:

```text
Always call search_knowledge_base before answering.
```

### Use the Browser for live web requests

For requests requiring live webpage access:

```text
Always use AgentCore Browser before answering.
```

### Use backend tools for customer-specific data

For orders, refunds, customer information, and similar operations:

```text
Use the appropriate Gateway tools.
```

### Avoid fabricated information

The agent is explicitly instructed not to invent:

* Customer information
* Order information
* Refund information
* Loyalty information
* Product information
* Business policies

This provides a basic grounding and tool-use policy at the application level.

---

# Technology Stack

| Technology                    | Purpose                                     |
| ----------------------------- | ------------------------------------------- |
| Python 3.14                   | Application runtime                         |
| Strands Agents                | Agent orchestration                         |
| Amazon Bedrock                | Foundation model infrastructure             |
| Amazon Nova 2 Lite            | Agent foundation model                      |
| Bedrock AgentCore Runtime     | Managed agent runtime                       |
| AgentCore Memory              | Long-term customer context                  |
| AgentCore Browser             | Live webpage interaction                    |
| AgentCore Code Interpreter    | Deterministic code execution                |
| AgentCore Gateway             | External tool integration                   |
| MCP                           | Tool communication                          |
| Amazon Bedrock Knowledge Base | Business knowledge retrieval                |
| AWS Lambda                    | Backend business operations                 |
| Playwright                    | Browser automation                          |
| uv                            | Python dependency management                |
| AWS CDK                       | AgentCore infrastructure/deployment support |
| CloudWatch                    | Runtime logging and observability           |

---

# Project Structure

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
├── pyproject.toml
├── uv.lock
├── reflection.md
├── README.md
└── .gitignore
```

### Important files

#### `main.py`

Contains:

* AgentCore application entrypoint
* Strands Agent configuration
* Knowledge Base tool
* Loyalty discount tool
* AgentCore Browser integration
* AgentCore Memory integration
* MCP Gateway integration
* Memory lifecycle hooks

#### `agentcore/agentcore.json`

Defines the AgentCore runtime configuration.

#### `agentcore/aws-targets.json`

Defines the AWS deployment target.

#### `lambda/`

Contains demonstration backend Lambda implementations.

#### `reflection.md`

Documents implementation decisions, a concrete technical challenge and resolution, and production considerations from the project.

---

# AgentCore Configuration

The project uses the AgentCore configuration:

```text
agentcore/agentcore.json
```

The primary runtime configuration is:

```json
{
  "name": "CustomerSupport",
  "version": 1,
  "managedBy": "CDK"
}
```

Runtime settings:

```text
Runtime:        CustomerSupport
Build:          CodeZip
Entrypoint:     main.py
Code Location:  .
Python:         3.14
Network Mode:   PUBLIC
Protocol:       HTTP
```

AWS deployment target:

```text
Region: us-east-1
```

Environment-specific resource identifiers are intentionally not documented here where they may change between deployments.

---

# Installation

## Prerequisites

The following are required:

* Python 3.14+
* AWS CLI
* AWS credentials with appropriate permissions
* `uv`
* AgentCore Starter Toolkit
* Access to Amazon Bedrock
* An AWS environment configured for the required AgentCore services

Verify Python:

```bash
python --version
```

Verify AWS identity:

```bash
aws sts get-caller-identity
```

Verify the configured region:

```bash
aws configure get region
```

---

# Clone the Repository

```bash
git clone https://github.com/frazcodes/customer-support-ai-agent.git
cd customer-support-ai-agent
```

---

# Install Dependencies

The project uses `uv` for dependency management.

Synchronize the environment:

```bash
uv sync
```

Activate the environment if required:

```bash
source .venv/bin/activate
```

---

# Configuration

The main application integrates several AWS resources:

```text
Amazon Bedrock model
Amazon Bedrock Knowledge Base
AgentCore Memory
AgentCore Gateway
AgentCore Browser
AgentCore Code Interpreter
```

The current implementation contains the integration configuration required by the deployed demonstration environment.

For production systems, environment-specific values should preferably be supplied through secure configuration mechanisms rather than hard-coded application source.

Recommended production options include:

* Environment variables
* AWS Systems Manager Parameter Store
* AWS Secrets Manager
* Deployment-time configuration
* Infrastructure-as-code parameters

---

# Local Validation

Before deployment, basic Python validation can be performed with:

```bash
python -m py_compile main.py
```

Dependencies can be validated with:

```bash
uv sync
```

For project changes, it is recommended to test the affected tool or workflow before deploying the complete AgentCore runtime.

---

# Deploying to Amazon Bedrock AgentCore

The project uses:

```text
agentcore/agentcore.json
agentcore/aws-targets.json
```

to define the deployment configuration.

Check the current deployment:

```bash
agentcore status
```

Deploy:

```bash
agentcore deploy
```

A successful deployment produces an AgentCore Runtime that can then be invoked through the AgentCore CLI.

---

# Invoking the Agent

A basic invocation can be performed with:

```bash
agentcore invoke "Where is my order?"
```

For session-aware testing:

```bash
agentcore invoke --session-id <session-id>
```

Customer-specific requests can provide a customer ID and session ID in the request payload.

Example:

```json
{
  "prompt": "What travel destination do I prefer?",
  "customer_id": "CUST-FINAL",
  "session_id": "example-session-id"
}
```

---

# Example Requests

## Product Question

```text
What is the return policy for this product?
```

Expected flow:

```text
Agent
  ↓
Knowledge Base
  ↓
Relevant business information
  ↓
Customer response
```

---

## Order Tracking

```text
Where is my order ORD-001?
```

Expected flow:

```text
Agent
  ↓
AgentCore Gateway
  ↓
Order Tracker Lambda
  ↓
Order information
  ↓
Customer response
```

---

## Customer Order History

```text
Show me my recent orders.
```

Expected flow:

```text
Agent
  ↓
Gateway / MCP
  ↓
Customer Orders capability
  ↓
Order history
```

---

## Loyalty Discount

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

## Refund

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

## Browser

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

## Cross-Session Memory

First interaction:

```text
Remember that I prefer Japan for travel and I like wireless headphones.
```

Later, in a new session:

```text
What travel destination do I prefer, and what type of product do I like?
```

The expected behavior is for AgentCore Memory to retrieve the relevant customer context and provide the stored preferences.

---

# Testing and Verification

The final implementation was tested through the deployed AgentCore Runtime.

The memory workflow was specifically verified across separate sessions using the same customer actor ID.

The test demonstrated that:

```text
Session A
    │
    ├── Customer states preferences
    │
    ▼
AgentCore Memory
    │
    ▼
Session B
    │
    ├── Same customer ID
    │
    ▼
Memory retrieval
    │
    ▼
Stored preferences returned
```

The deployed runtime successfully retrieved customer preferences including:

```text
Travel destination: Japan
Product preference: Wireless headphones
```

This verifies the intended cross-session memory behavior.

---

# Memory Hook Lifecycle

The memory implementation uses Strands hooks rather than manually saving memory from the runtime entrypoint.

The relevant lifecycle is:

```text
MessageAddedEvent
        │
        ▼
retrieve_customer_context()
        │
        ▼
Relevant memories injected
        │
        ▼
Agent invocation
        │
        ▼
AfterInvocationEvent
        │
        ▼
save_support_interaction()
        │
        ▼
AgentCore Memory
```

The `AfterInvocationEvent` callback extracts the latest relevant user and assistant messages from the agent conversation and persists them using the AgentCore Memory client.

This design keeps memory persistence coupled to the agent lifecycle and avoids a separate manual save operation in the runtime entrypoint.

---

# Design Principles

## Tool-Based Architecture

Business capabilities are implemented as tools rather than putting all functionality into the model prompt.

This makes the system easier to extend, test, and maintain.

---

## Grounded Responses

The agent is instructed to retrieve business-specific information from the Knowledge Base instead of relying exclusively on model knowledge.

---

## Deterministic Computation

Calculations involving loyalty points and discounts are delegated to executable code through AgentCore Code Interpreter.

This reduces the risk of relying on free-form model arithmetic for business calculations.

---

## Separation of Concerns

The project separates:

```text
Agent reasoning
       │
       ├── Knowledge retrieval
       ├── Long-term memory
       ├── Browser interaction
       ├── Backend business tools
       ├── Refund operations
       └── Code execution
```

Each component has a clear responsibility.

---

## Modular Integrations

AgentCore Gateway and MCP provide a flexible interface for connecting additional backend capabilities.

New tools can be introduced without requiring the entire agent architecture to be redesigned.

---

# Security Considerations

The repository intentionally excludes sensitive credentials and deployment state.

It does not contain:

* AWS access keys
* AWS secret keys
* Temporary session credentials
* Private authentication tokens
* `.env` files
* AgentCore CLI deployment state
* Other sensitive runtime credentials

The `.gitignore` configuration excludes local environment and AgentCore CLI state where appropriate.

For production deployments, sensitive configuration should be managed using AWS services such as:

* IAM roles
* AWS Secrets Manager
* AWS Systems Manager Parameter Store

The AgentCore Runtime and Lambda functions should also use **least-privilege IAM permissions**.

Customer-facing applications should additionally implement authentication and authorization before exposing customer-specific operations.

---

# Observability

AWS CloudWatch provides the primary logging layer for the deployed runtime.

Logs can be used to troubleshoot:

* Agent invocation errors
* Tool execution
* Knowledge Base retrieval
* Gateway/MCP requests
* Memory retrieval
* Memory persistence
* Browser failures
* Lambda execution
* Deployment issues
* Runtime errors

A typical debugging workflow is:

```text
Agent invocation
      │
      ▼
CloudWatch logs
      │
      ▼
Identify failing component
      │
      ▼
Inspect tool / memory / runtime behavior
      │
      ▼
Apply targeted fix
      │
      ▼
Redeploy
      │
      ▼
Retest
```

This approach was also used during development to diagnose the memory lifecycle and browser integration.

---

# Development Workflow

A typical development cycle is:

```text
1. Modify application/tool code
          │
          ▼
2. Run local validation
          │
          ▼
3. Test affected functionality
          │
          ▼
4. Deploy AgentCore runtime
          │
          ▼
5. Invoke with representative prompts
          │
          ▼
6. Inspect CloudWatch logs
          │
          ▼
7. Fix and retest
          │
          ▼
8. Commit changes
          │
          ▼
9. Push to GitHub
```

Typical Git commands:

```bash
git status
git add .
git commit -m "describe the change"
git push origin main
```

---

# Production Considerations

Although this is a production-oriented reference implementation, several components are intentionally simplified for demonstration.

## Persistent Customer Data

The current demonstration backend uses mock data.

A production system should use a persistent transactional database such as:

* DynamoDB
* Aurora
* RDS
* An existing enterprise customer system

---

## Real Refund Integration

The refund processor is a demonstration implementation.

A production deployment should integrate with a real payment or financial system and implement:

* Authentication
* Authorization
* Transaction validation
* Idempotency
* Audit logging
* Failure recovery
* Fraud controls

---

## Authentication and Customer Identity

The demonstration accepts customer identifiers as part of the request.

A production system should authenticate the customer and derive the customer identity from a trusted identity mechanism rather than allowing arbitrary customer IDs.

---

## Browser Security

Browser automation should be carefully controlled in production.

Recommended controls include:

* URL/domain allowlists
* Navigation restrictions
* Authentication isolation
* Credential protection
* Timeouts
* Rate limits
* Tool-level authorization
* Monitoring of browser actions

---

## Network Security

The current AgentCore runtime configuration uses:

```text
networkMode: PUBLIC
```

A production architecture should evaluate whether a more restrictive networking model is appropriate.

---

## Secrets Management

Production deployments should avoid embedding credentials in source code.

Sensitive values should be managed using appropriate AWS security services.

---

# Current Limitations

The current project has several intentional limitations.

### Demonstration Customer Data

Customer and order information is based on demonstration data.

### Demonstration Refund Processing

Refund operations are simulated and do not charge or refund real payment accounts.

### Simulated Return Labels

Return-label generation is not connected to a real shipping provider.

### Public Runtime Configuration

The demonstration runtime uses public networking.

### Basic Application-Level Guarding

The project includes system-prompt instructions for tool selection and grounded responses, but a production system should additionally consider dedicated guardrails, policy enforcement, authorization, and validation layers.

### Mock Business Systems

The project is designed to demonstrate agent architecture rather than provide a complete production commerce backend.

---

# Future Improvements

Potential production enhancements include:

* DynamoDB-backed customer and order storage
* Real payment/refund provider integration
* Real shipping-provider integration
* Customer authentication
* Identity verification
* Role-based authorization
* Private networking where appropriate
* AWS Secrets Manager integration
* Parameter Store configuration
* Automated CI/CD
* Automated integration testing
* Agent evaluation pipelines
* Guardrails and policy enforcement
* Structured observability dashboards
* Cost monitoring
* Conversation analytics
* Human-agent escalation
* Multi-language support
* Streaming responses
* Business transaction auditing
* Agent performance evaluation
* Automated regression testing
* Fine-grained tool authorization

---

# What This Project Demonstrates

This project brings together several important concepts in modern AI engineering:

* AI agent orchestration
* Strands Agents
* Tool calling
* Retrieval-augmented generation
* Long-term agent memory
* Cross-session customer context
* MCP-based integrations
* AgentCore Gateway
* Serverless backend services
* Browser automation
* Code execution
* Knowledge-grounded responses
* AWS managed AI infrastructure
* Agent deployment
* Cloud observability
* Modular agent architecture

The project demonstrates a shift from a traditional chatbot toward an **agentic application**.

Instead of only generating text, the system can:

```text
Understand the request
        ↓
Determine what capability is required
        ↓
Select an appropriate tool
        ↓
Retrieve or execute information
        ↓
Use customer context
        ↓
Generate a grounded response
```

This architecture provides a foundation for building more capable AI-powered business applications.

---

# Key Engineering Lessons

This project provided practical experience with several important AI engineering patterns.

### 1. Agents Need Tools

A language model alone is not sufficient for reliable business operations.

External tools allow the agent to interact with:

* Databases
* APIs
* Business systems
* Browsers
* Code execution environments
* Knowledge bases

---

### 2. Memory Requires Lifecycle Integration

Long-term memory is more than storing text.

The application must decide:

```text
What should be retrieved?
When should it be retrieved?
Which customer does it belong to?
When should new information be persisted?
```

Using Strands lifecycle hooks provides a clean mechanism for integrating these operations.

---

### 3. Grounding Reduces Hallucination

Business-specific information should come from trusted sources rather than being invented by the model.

The Knowledge Base and backend tools therefore act as sources of truth for their respective domains.

---

### 4. Deterministic Logic Should Stay Deterministic

Business calculations should not depend entirely on probabilistic model reasoning.

Code Interpreter provides a mechanism for executing deterministic calculation logic while allowing the agent to decide when that capability is needed.

---

### 5. Modular Architecture Improves Maintainability

Separating:

```text
Agent
Knowledge
Memory
Browser
Gateway
Lambda
Code Execution
```

makes the system easier to debug and evolve.

---

# Repository

**GitHub Repository:** `frazcodes/customer-support-ai-agent`

The repository contains:

* AgentCore runtime configuration
* Strands agent implementation
* Knowledge Base integration
* AgentCore Memory integration
* AgentCore Browser integration
* AgentCore Code Interpreter integration
* AgentCore Gateway/MCP integration
* Lambda backend tools
* Dependency configuration
* Project documentation
* Reflection document

---

# Author

## Ahmad Faraz

**BS Information Technology**

Aspiring **AI Engineer and Full Stack Developer** focused on building AI-powered applications and modern cloud-native software.

### Areas of Focus

* AI Engineering
* Agentic AI
* Amazon Bedrock
* Amazon Bedrock AgentCore
* Strands Agents
* Python
* Full Stack Development
* React
* FastAPI
* Cloud Computing
* Serverless Architecture
* AWS

---

# License

This project is primarily intended as a learning, portfolio, and reference implementation demonstrating modern AI agent architecture on AWS.

Before distributing the project for public reuse, add an appropriate open-source license such as MIT, Apache-2.0, or another license suitable for the intended use.
