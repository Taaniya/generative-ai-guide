## Concepts -
1. [AI agents](#ai-agents)
2. [What is Loop engineering?](#what-is-loop-engineering)
3. [What is Harness engineering?](#what-is-harness-engineering)
4. [Prompt engineering vs Context engineering vs Harness engineering](#prompt-engineering-vs-context-engineering-vs-harness-engineering)
5. [Backpressure and rate limiting](#what-is-backpressure-and-rate-limiting-why-is-it-imposed)
6. [Why is session state management required for agentic workflows?](why-is-session-state-management-critical-for-agentic-workflows)
7. [How to build stateful experience with stateless agentic app for production?](#how-to-build-stateful-experience-with-stateless-agentic-app-for-production)
8. [Stateful vs stateless AI agents](#stateless-vs-stateful-ai-agents)

#### AI agents
An agent is an autonomous software system that can perceive its environment, makes independent decisions through reasoning, takes actions using tools / APIs in multiple steps in loop to achieve a goal.

How AI Agents Work?

AI agents function through a continuous reasoning loop: 
* Plan: The agent breaks a main goal down into smaller, sequential steps.
* Act: It uses external tools, software functions, or database queries to execute a step.
* Observe: It looks at the resulting output or change in the environment.
* Adapt: It evaluates progress and decides the next action until the task is complete.

Core Components:

* The Brain: Usually powered by a large language model (LLM) or foundation model that handles logic and language processing.
* Memory: Short-term and long-term storage that allows the agent to recall past actions, user preferences, and prior context.
* Tools: External capabilities like code execution environments, web browsers, calculators, or specific APIs.
* Planning / Orchestration: Rules and structures that guide how the agent splits work and corrects errors.


References:
1.  [What are AI agents? (Google Cloud)](https://cloud.google.com/discover/what-are-ai-agents)
2.  [Architecting Agent Memory: Principles, Patterns, and Best Practices — Richmond Alake, MongoDB (Tech Talk)](https://youtu.be/W2HVdB4Jbjs?si=-H16PSXUBv8hzmZX)
3.  [AI agents explained (Google Cloud)](https://youtu.be/Zqno_vux6d8?si=j9kLAOUmwVZTi4af)



#### What is Loop engineering?
* Loop engineering is the practice of designing automated iterative workflows where AI agents act, observe, verify, and self-correct their outputs until a goal is met, shifting the developer's role from writing single prompts to building self-governing control loops. 

Core Components of Loop Engineering:
* **Intent / Purpose:** A clearly defined user goal or target outcome that establishes the boundary of what the agent needs to achieve. 
* **Action:** The operational step where the agent executes a task, such as writing code, calling an API, or searching a database. 
* **Observation / Verifier:** An external evaluation mechanism or tool (like a compiler, linter, or test suite) that checks if the output satisfies the objective.
* **Feedback & Adjustment:** The process where error logs or critique results are fed back into the agent so it can revise its subsequent steps. 
* **Stopping Condition:** Explicit termination logic that stops the recursive cycle once success is reached or a compute-budget limit is hit. 


Prompt Engineering vs. Loop Engineering -
* **Prompt Engineering:** Focuses on optimizing a single instruction for a one-off model response, requiring humans to manually re-prompt upon failure.
* **Loop Engineering:** Focuses on orchestrating the entire multi-step lifecycle, allowing the system to dynamically self-prompt and correct errors over a long-running execution

References -
* [What is Loop engineering - IBM](https://www.ibm.com/think/topics/loop-engineering)

#### What is Harness engineering?

* Harness engineering is the practice of building the structured environment, tools, and feedback loops around an AI model so it can run reliably as an autonomous agent.
* Defined as Agent = Model + Harness, it moves past simple prompt or context tuning to govern execution, verify outputs, and prevent repeat mistakes

Core Components:
* Guides (Feedforward): Rules, system instructions, and architecture boundaries that steer the agent before it acts.
* Sensors (Feedback): Custom linters, tests, and validation loops that let the agent self-correct its errors.
* State Management: Persistent logs and file-based records so an agent recovers cleanly across sessions.
* Tool Orchestration: Sandboxes, Model Context Protocol (MCP) servers, and sub-agent spawning that expand functional range safel

Evolution Beyond Prompting:
* Prompt Engineering: Optimizes a single response or turn.
* Context Engineering: Manages what information fits inside the active window.
* Harness Engineering: Designs the operational world and safety constraints for multi-hour, autonomous tasks


References -
* [Harness engineering for coding agent users](https://martinfowler.com/articles/harness-engineering.html)
* [Harness design for long-running application development - Anthropic](https://www.anthropic.com/engineering/harness-design-long-running-apps)
* [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
* [Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/)
* [How to build a better agent harness with traces and evals](https://arize.com/blog/improve-ai-agents-traces-evals-harness/)


#### Prompt engineering vs Context engineering vs Harness engineering
Prompt engineering, context engineering, and harness engineering represent three progressive layers of applied AI development. Prompt engineering shapes model behavior through instruction text, context engineering curates the knowledge and data fed into the window, and harness engineering builds the outer operational system and controls for safety and reliability

Prompt Engineering -
* **Focuses on** the precise wording, role assignments, tone constraints, and few-shot examples of a single interaction.
* **Asks the question:** "How should I phrase this instruction so the model understands what to do?"
* **Operates at** the text and message level.
* **Fails when** the wording is ambiguous or instructions conflict

Context Engineering -
* **Focuses on** managing the information ecosystem, memory, retrieval results (RAG), and tool outputs loaded into the model's window.
* **Asks the question:** "What exact data, files, or background facts does the model need to see right now?"
* **Operates at** the data architecture and token management level.
* **Fails when** data is stale, irrelevant, or causes context rot/overflow


Harness Engineering -
* **Focuses on** the deterministic software scaffolding, execution loops, permissions, sandboxes, and human-in-the-loop approval gates surrounding the AI agent.
* **Operates at** the system infrastructure and workflow governance level.
* **Fails when** the system lacks proper retries, evaluation loops, or safety guardrails.


Key Differences:
| Dimension |	Prompt Engineering	| Context Engineering |	Harness Engineering |
|:--------:|:--------:|:--------:|:--------:|
| Core Focus|	Instructions & Behavior	| Knowledge & Memory	| Control & System Reliability |
| Primary Domain	| Linguistics & Phrasing |	Data Architecture & Retrieval |	Software Architecture & Sandboxing |
| Scope	| Single prompt/turn	| Dynamic data streams/window	| Full operational | lifecycle/loops |


References -
* [Harness Engineering vs Prompt Engineering vs Context Engineering - Atlan](https://atlan.com/know/harness-engineering-vs-prompt-engineering/)
  


#### What is backpressure and Rate limiting? Why is it imposed?

* In agentic systems—AI architectures where autonomous agents use tools, call APIs, and reason over multiple steps—backpressure and rate limiting are critical flow-control mechanisms
used to prevent system failures, manage API costs, and handle uneven workloads.
* They ensure that agents do not overwhelm downstream services (like LLM providers or databases) with more requests than they can process.

**What is AI agent rate limiting?**
* AI agent rate limiting involves controlling how frequently agents make API calls, access resources, and consume credits to prevent service disruptions, manage costs, and stay within provider quotas.

LLM API vs AI agent rate limiting?

LLM API rate limits -

* LLM API rate limits (e.g., GPT-4, Claude) focus on protecting infrastructure, restricting RPM (requests per minute) and TPM (tokens per minute)
* These are enforced at provider side to protect GPUs, prevent abuse, ensure stability.
* Metrics: Rigid limits on RPM, TPM, and concurrent requests
* LLM API limits are about system capacity (how many requests to the model)

AI Agent Rate Limits (Application/Agent Side) -
* Agent rate limits are more dynamic, managing autonomous, multi-step workflows that generate unpredictable bursts of traffic, often requiring context-aware, adaptive rate limiting (ARL) rather than simple thresholds
* Goal: Manage budget, control complex, multi-step reasoning processes, and prevent runaway agents.
* Metrics: More granular; often tracks tokens per agent, total tokens per task, and cost per workflow.
* Agent rate limits are about application behavior (how much reasoning/token usage the agent is allowed)

References -
* https://fast.io/resources/ai-agent-rate-limiting/
* https://stackoverflow.com/questions/69697796/what-is-the-difference-between-rate-limiting-and-back-pressure
* [How AI Agents Are Changing API Rate Limit Approaches](https://nordicapis.com/how-ai-agents-are-changing-api-rate-limit-approaches/#:~:text=Traditional%20approaches%20to%20rate%20limiting,indicating%20a%20typical%20DDoS%20attack)


#### Why is session state management critical for agentic workflows?
* Allows agents to track goals across interactions
* Enables coherent conversations, interactions, retain context, provide personalized experiences
* Without state management, each prompt is handled in isolation, making it impossible for the agent to refer prior context, track ongoing tasks.


#### How to build stateful experience with stateless agentic app for production?
* Since cloud environments, need applications to be stateless and scalable, the solution is to externalize state to a persitent storage
* This lets an agent to reconstruct prior context e.g., build conversation history on demand, delivering a seamless stateful experience while keeping the agentic app itself stateless for scalability and resilience.

Reference -
* [Effectively building AI agents on AWS Serverless](https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-agentcore-memory-building-context-aware-agents/#:~:text=The%20memory%20problem%20in%20AI%20agents&text=When%20implementing%20memory%20for%20AI,patterns%20that%20matter%20to%20users.)
* [Amazon Bedrock AgentCore Memory: Building context-aware agents](https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-agentcore-memory-building-context-aware-agents/#:~:text=The%20memory%20problem%20in%20AI%20agents&text=When%20implementing%20memory%20for%20AI,patterns%20that%20matter%20to%20users.)

#### Stateless vs stateful AI agents
Stateful AI agents -
* Stateful AI agents retain memory of past interactions (context) to inform future decisions, allowing for context-aware, complex, multi-turn, and personalized tasks/ conversations over time

Stateless agents -
* Stateless agents treat each request independently with no memory, offering faster, simpler, and more scalable performance suitable for isolated tasks

Key differences -
1. Memory & context:
   - Stateful agents store user preferences, conversation history and intermediate steps. 
   - Stateless agents (like goldfish) forget everything between interactions, requiring full context in every request.
2. Use case:
   - Stateful: Personalized assistants, complex workflow orchestration, multi-step planning.
   - Stateless: Single-turn Q&A, simple chatbots, spam filters.
3. Scalability:
   - Stateless systems are easier to scale using load balancing, as any server can handle a new request.
   - Stateful systems are harder to scale because they require managing session data across requests.
4. Performance:
   - Stateful agents are more complex to implement and resource-intensive.
   - Stateless agents are typically faster and cheaper.

**References -**
* https://tacnode.io/post/stateful-vs-stateless-ai-agents-practical-architecture-guide-for-developers

