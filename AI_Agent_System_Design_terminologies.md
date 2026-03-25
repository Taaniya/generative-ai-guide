## Concepts -
1. [Backpressure and rate limiting](#what-is-backpressure-and-rate-limiting-why-is-it-imposed)
2. 

   
#### What is backpressure and Rate limiting? Why is it imposed?

* In agentic systems—AI architectures where autonomous agents use tools, call APIs, and reason over multiple steps—backpressure and rate limiting are critical flow-control mechanisms
used to prevent system failures, manage API costs, and handle uneven workloads.
* They ensure that agents do not overwhelm downstream services (like LLM providers or databases) with more requests than they can process.

**What is AI agent rate limiting?**
* AI agent rate limiting involves controlling how frequently agents make API calls, access resources, and consume credits to prevent service disruptions, manage costs, and stay within provider quotas.

**LLM API vs AI agent rate limiting?**
**LLM API rate limits -**
* LLM API rate limits (e.g., GPT-4, Claude) focus on protecting infrastructure, restricting RPM (requests per minute) and TPM (tokens per minute)
* These are enforced at provider side to protect GPUs, prevent abuse, ensure stability.
* Metrics: Rigid limits on RPM, TPM, and concurrent requests
* LLM API limits are about system capacity (how many requests to the model)

**AI Agent Rate Limits (Application/Agent Side) -**
* Agent rate limits are more dynamic, managing autonomous, multi-step workflows that generate unpredictable bursts of traffic, often requiring context-aware, adaptive rate limiting (ARL) rather than simple thresholds
* Goal: Manage budget, control complex, multi-step reasoning processes, and prevent runaway agents.
* Metrics: More granular; often tracks tokens per agent, total tokens per task, and cost per workflow.
* Agent rate limits are about application behavior (how much reasoning/token usage the agent is allowed)

References -
* https://fast.io/resources/ai-agent-rate-limiting/
* https://stackoverflow.com/questions/69697796/what-is-the-difference-between-rate-limiting-and-back-pressure
* https://nordicapis.com/how-ai-agents-are-changing-api-rate-limit-approaches/#:~:text=Traditional%20approaches%20to%20rate%20limiting,indicating%20a%20typical%20DDoS%20attack).
