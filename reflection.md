# Project Reflection

## Implementation Choice

A key implementation choice was using Amazon Bedrock AgentCore Memory with a custom Strands `MemoryHook`. I designed the hook to retrieve relevant customer context when a user message is added and to persist the completed interaction after the agent finishes. In particular, `AfterInvocationEvent` is used to save the final user query and assistant response automatically. This keeps memory persistence separate from the main agent workflow and avoids relying on manual memory-saving logic inside the entrypoint. The approach also supports cross-session customer context by using the customer ID as the memory actor.

## Challenge and Resolution

One concrete challenge was running the AgentCore Browser in the headless AgentCore runtime. The browser tool initially encountered Playwright/Node.js permission and execution issues that did not appear in the normal development workflow. I investigated the runtime logs to identify the actual failure instead of changing the agent logic blindly. I resolved the issue by configuring the runtime for headless operation with `BYPASS_TOOL_CONSENT=true` and explicitly setting `PLAYWRIGHT_NODEJS_PATH=/usr/local/bin/node`. After redeployment, I verified the browser integration through runtime invocation and logs.

## Production Consideration

Security and operational monitoring would be important before moving this project to production. The current project uses a public AgentCore runtime and configured AWS resource identifiers, so a production version should apply least-privilege IAM policies, protect sensitive configuration with AWS Secrets Manager or environment configuration, and restrict network access where appropriate. The Gateway currently connects the agent to order and refund functionality, so those backend tools should also validate authorization for each customer rather than relying only on the agent's instructions. CloudWatch logging and metrics should be monitored for failed tool calls, latency, memory operations, and unexpected costs. The Code Interpreter and Browser integrations should also be monitored because they can increase runtime usage and therefore operational cost.
