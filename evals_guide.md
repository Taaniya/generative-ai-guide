## Contents:
* [How is Error analysis helpful?](#how-is-error-analysis-helpful-what-are-the-best-practices-while-moving-from-generic-evals-to-problem-specific-diagnostics--domain-specific-observability)
* [What is the paradigm shift in observability and evals from traditional deterministic software engineering to non-deterministic agentic systems?](#what-is-the-paradigm-shift-in-observability-and-evals-from-traditional-deterministic-software-engineering-to-non-deterministic-agentic-systems)
* [How are observability and evaluations tightly coupled?](#how-are-observability-and-evaluations-tightly-coupled)
* [What is offline evaluation? When should we run offline evaluations?](#what-is-offline-evaluation-when-should-we-run-offline-evaluations)
* [How do you automate offline test cases?](#how-do-you-automate-offline-test-cases)
* [What is online evaluation? How is it different from offline evals?](#what-is-online-evaluation-how-is-it-different-from-offline-evals)
* [What metrics are best for online evaluation?](#what-metrics-are-best-for-online-evaluation)
* [What triggers an online evaluation? Can it detect every error?](#what-triggers-an-online-evaluation-can-it-detect-every-error)
* [How are offline evals better for accuracy?](#how-are-offline-evals-better-for-accuracy)
* [What is an ad hoc evaluation? When should it be used?](#what-is-an-ad-hoc-evaluation-when-should-it-be-used)

### How is Error analysis helpful? What are the best practices while moving from generic evals to problem-specific diagnostics & domain-specific observability?

1. Prioritize Manual error analysis:
    - Even with experience and despite the allure of automated benchmarks, the highest ROI activity is direct manual review of production logs.
    - By looking at the data over more and more real-world interactions can help define the product roadmap and identify gaps between the expected system behavior and user mental models. This way engineers can prioritize bug fixes and feature development based on real user pain points.

2. Custom data viewers over off-the-shelf dashboards:
   - Don't rely solely on off-the-shelf dashboards which often hide the nuance of specialized domains.
   - Developing a custom interface to visualize your specific data structure (e.g., voice/email/text threads), that allow you to replay the full context - human input, tool calls, model responses - removes the cognitive friction and allows for rapid, qualitative error analysis before moving to quantitative evals.

3. Shift from generic scores:
   - Move away from 'vanity' scores like generic hallucination and conciseness towards error taxonomy grounded in business logic (business logic-driven evals), e.g., "tour scheduling errors", "handoff failures").
   - By using LLMs to categorize logs based on these specific failure modes, you can prioritize your engineering roadmap with objective data

4. Modeling the 'Model's Mind':
   - Systematic review of thousands of interactions helps you predict how your system will behave. This intuition is critical when designing tools and RAG pipelines. You begin to anticipate failure scenarios—such as date parsing or tool call triggers—before they hit production.

6. The 'Evergreen' Nature of Conversations:
   - A significant takeaway for system design is that conversational AI in enterprise settings often isn't a transactional "start-to-finish" event but rather about managing state over an extended, potentially indefinite timeline. Recognizing this changes how you structure your evaluation datasets and long-term memory systems.
   - Use Targeted Evaluations: Once you identify specific failure modes—such as a model struggling with date references relative to a conversation's start—create isolated, scenario-based tests. These "playgrounds" allow you to simulate various user inputs and refine the model's ability to handle state across the thread 

High-Impact recommended workflow:
* Read the raw data manually to identify gaps .
* Take detailed, context-aware notes on why failures occur.
* Categorize and count the most frequent failure modes to prioritize engineering bandwidth 
* Automate the discovery of those specific categories using LLM-as-a-judge patterns 

Automate conversation labeling:
1. **Manual foundation:** Manually review your conversation data to identify recurring issues (e.g., missed handoffs, incorrect scheduling) and create descriptive notes for why these failures occurred.
2. **Develop a Taxonomy:** Once you understand the common failure patterns, define a clear set of categories or tags that represent these issues
3. **Use LLMs for Categorization:** Leverage LLMs to automatically label your raw conversation data against your defined taxonomy. By providing the model with your conversation logs and the set of categories, it can classify new interactions without manual intervention.
4. **Aggregated Reporting:** Use these automated labels to aggregate your data and count which errors occur most frequently. This turns qualitative logs into quantitative data that drives your product roadmap and engineering priorities

Reference - 
* [Error Analysis: The Highest ROI Technique In AI Engineering - Hamel Husain](https://youtu.be/e2i6JbU2R-s?si=fUaeOioS6IedrEY3)

### What is the paradigm shift in observability and evals from traditional deterministic software engineering to non-deterministic agentic systems? 
1. The Shift to Emergent Behavior:
   * Unlike traditional software where source code is the sole source of truth, agentic systems are non-deterministic.
   * Behavior is emergent, driven by natural language inputs and dynamic tool-use.
   * This renders traditional stack traces obsolete for debugging; instead, the execution trace becomes the new source of truth.

2. Debugging agents is harder:
   * Debugging AI agents is significantly more complex than traditional software due to a fundamental shift in how they function :
       * **Non-Determinism:** Unlike traditional software, which is deterministic (the same code produces the same result every time), agents use Large Language Models (LLMs) that are inherently non-deterministic. Their behavior is emergent, meaning it unfolds as the agent runs.
       * **Lack of Traditional Stack Traces:** In traditional programming, code is the source of truth, and you can identify bugs via stack traces. With agents, the logic does not reside in a single line of code, but in the evolving context of the interaction. Since there is no failing code path to inspect, you must rely on execution traces to understand the agent's reasoning process.
       * **Unconstrained Inputs:** Agents accept natural language as input. Because this input is completely unconstrained, developers cannot predict or write comprehensive tests for every possible user action ahead of time. You often don't know exactly what your agent will do until it is actively used in a production environment.

3. Primitives of Agent Observability:
To effectively debug agent reasoning, you must implement observability across three granularities:
   * **Run (Single LLM call):** Focus on capturing input context (system prompts, tool definitions) and raw output (including reasoning blocks and tool calls).
   * **Trace (Execution flow):** A collection of runs in sequence. Critical for understanding how previous decisions influence subsequent steps.
   * **Thread (Multi-turn):** A sequence of traces. Necessary for debugging state maintenance and context retention over extended interactions.

4. Redefining Evaluation:
Agent evaluation shifts from testing code paths to testing reasoning and context.
    * **Single-step Evals:** Unit tests for decisions; fast, clear pass/fail criteria.
    * **Trace Evals:** Validate entire tool-use trajectories and state changes.
    * **Multi-turn Evals:** Most realistic but complex; tests the agent's ability to maintain context over long-horizon tasks.

5. Evaluation Lifecycle:
    * **Offline:** Essential for benchmarking and catching regressions before production deployment using curated datasets.
    * **Online:** Runs in production post-execution. Cannot rely on ground truth but is vital for flagging trajectory, efficiency, or quality failures in real-time.
    * **Ad-hoc:** Exploratory analysis (e.g., clustering failure modes, identifying user frustration) used when specific hypothesis-driven debugging is required.

5. Coupling Observability and Evals:
    * Observability powers the entire evaluation pipeline. Production traces serve as the data source for test cases.
    * When an agent fails, developers extract the state from the trace, anonymize if necessary, and promote it to an offline test case to prevent future regressions.
    * This feedback loop is the foundation of production-grade agent reliability.


### How are observability and evaluations tightly coupled?
* In the context of AI agents, observability and evaluation are tightly coupled because production traces serve as the source of truth.
* Unlike traditional software where the code is the primary point of debugging, agentic behavior is non-deterministic and emerges only when users interact with the system

This coupling manifests in several key ways:
* Production-to-Offline Pipeline:
    * Because you often cannot predict every failure mode before deployment, production traces become your primary dataset for offline evaluation.
    * When an agent fails in production, builders extract the specific state (messages, tool outputs) from the trace to create a reproducible offline test case.
    * This allows developers to fix the logic and verify the correction before redeploying.
* Continuous Feedback Loops:
    * While online evaluations monitor live traffic to identify process issues like efficiency or unexpected trajectories, they feed insights back into the development lifecycle.
    * These insights often highlight new edge cases that were previously unidentified, prompting the creation of new  benchmarks to prevent future regressions.
* Shared Observability Primitives:
    * Both evaluation methods rely on the same observability infrastructure—runs, traces, and threads.
    * By maintaining a consistent mental model and data structure for these execution steps, developers can seamlessly switch between manual debugging, ad-hoc analysis, and automated testing

### What is offline evaluation? When should we run offline evaluations?
* Offline evaluation happens in a controlled environment and tests an AI agent against a fixed dataset.
* You should run offline evaluations before deploying your agent to production. This process is critical for ensuring reliability and catching potential issues early.

Key scenarios for running offline evaluations include:
* **Before each deployment:** It is common to run a battery of tests before shipping new code to production to ensure everything is working as expected
* **Regression testing:** You should maintain a set of core tasks that your agent must always pass. Running these periodically or with every commit helps catch regressions, ensuring that new changes don't break existing, stable functionality.
* **Benchmarking and improvement:** Beyond just checking for failures, you can use a benchmark dataset to "hill climb" and systematically improve your agent's reasoning capabilities over time
* **Fixing production issues:** When a user reports a failure in production, you should capture that trace, create a test case from it, and use offline evaluation to verify your fix before updating the agent


### How do you automate offline test cases?
* Automating offline test generation for AI agents centers on utilizing production traces as the foundation for your evaluation datasets. Since agent behavior is emergent and unpredictable until deployed, production is the primary site for discovering where your agent fails.
* Here is the workflow for converting these production failures into automated offline tests:
    * **Identify the Failure:** When a user reports incorrect behavior or an evaluation flag highlights an issue, you locate the specific production trace in your observability tool.
    * **Extract State:** You capture the exact state at the point of failure, including the full conversation history, the specific inputs, and the files or tools available to the agent at that moment.
    * **Sanitize Data:** If the trace contains sensitive user information, you scrub or anonymize the data to ensure it is safe to use in a testing environment.
    * **Create the Test Case:** The sanitized state is saved as a new test case in your evaluation suite. This allows you to recreate the exact condition that caused the error .
    * **Run Offline Evals:** You can then run this new test case—alongside your existing benchmark dataset—using offline evaluation to verify that your code or prompt adjustments fix the issue without causing regressions.

### What is online evaluation? How is it different from offline evals?
Online evaluation measures the performance of Agents on live production traffic after launch.
Offline and online evals based on when they occur and their reliance on ground truth:
* **Offline Evaluation:** Conducted before production deployment. It involves building a dataset of inputs and known ground truth outputs to catch regressions and perform benchmarking. It serves as a way to test logic changes against a standard.
* **Online Evaluation:** Runs in production immediately after an agent executes. Because it happens in real-time on live user data, it cannot rely on ground truth. Instead, it focuses on monitoring metrics like trajectory, efficiency, and quality to flag issues as they happen.

### What metrics are best for online evaluation?
* For online evaluation—which occurs in real-time within production environments—you cannot rely on "ground truth" or expected outputs because production interactions are unpredictable. Instead, metrics focus on **monitoring process**, **quality**, and **efficiency** rather than simple correctness.
* Key areas for online metrics include:
    * **Trajectory tracking:** Monitoring the steps an agent takes to ensure it is not looping unnecessarily or getting stuck in invalid states
    * **Efficiency metrics:** Measuring the cost, latency, or number of tool calls required to complete a task
    * **Quality assessment:** Using secondary model-based evaluators or sentiment analysis to flag potential issues like user frustration or non-compliant responses 

**Note:** To catch errors related to specific, ground-truth-dependent outcomes, you must rely on **offline evaluation**, where you can test against predefined expected results using datasets derived from previous production failures


### What triggers an online evaluation? Can it detect every error?
* In a production environment, an online evaluation is triggered immediately after an agent completes its execution run
* No, online evaluations cannot detect every error. Because online evaluations run on live production traces, they lack access to the ground truth (the known correct answer) required to verify factual accuracy or semantic correctness
* The process functions as follows:
    * **Trace Capture:** Once the agent finishes, the entire execution trace—which contains the input, context, tool calls, and final output—is sent to an observability platform, such as LangSmith
    * **Evaluator Execution:** Pre-defined evaluators run automatically over this captured trace to analyze specific behaviors
    * **Metric Focused Analysis:** Because online evaluations occur in production without known "ground truth" or expected outputs, they cannot check for factual accuracy or semantic correctness. Instead, they are best suited for process-oriented and quality metrics such as trajectory, efficiency, and quality / sentiments.

### How are offline evals better for accuracy?
* Offline evaluation is superior for measuring accuracy because it allows you to test against ground truth, which is impossible to consistently determine in real-time production settings.
* Here is how offline evaluation ensures accuracy:
    * **Curated Test Datasets:** You can build a dataset of specific inputs paired with verified, expected outputs. This creates a controlled benchmark to verify if the agent is producing correct information.
    * **Regressions and Benchmarking:** By running these tests before deploying code, you can catch regressions and systematically "hill climb" to improve the agent’s reasoning capabilities over time.
    * **Controllable Environment:** Unlike online monitoring, which is subject to the randomness of user inputs, offline evaluation provides a consistent environment to repeatedly test logic, tool usage, and reasoning

### What is an ad hoc evaluation? When should it be used?
* An ad hoc evaluation is an exploratory approach to analysis that allows you to investigate specific hunches or user feedback without needing to pre-configure automated tests
* Key characteristics of ad hoc evaluations include:
    * **Flexibility:** Unlike online evaluations that run automatically on all production traces, ad hoc evaluations are performed on demand
    * **Exploratory Analysis:** It functions much like exploratory data analysis, where you can dive into specific segments of data—such as filtering for instances where a user expressed frustration—to cluster behaviors or understand patterns
    * **Purpose:** It is used to identify failure modes or compare successful versus failed executions based on specific interests, rather than for catching general regressions

You should use ad hoc evaluation when you want to perform exploratory data analysis to investigate specific hunches or address user feedback. Unlike automated online evaluations that run continuously, ad hoc evaluation is a manual, on-demand approach ideal for:
* **Surface-level exploration:** When you need to filter traces to understand specific patterns, such as identifying when a user expressed frustration
* **Failure mode investigation:** When you have a suspicion about a specific issue and need to dive into the data to identify why the agent is failing
* **Comparative analysis:** When you want to manually compare successful agent executions against failed ones to spot differences in behavior or reasoning

It is a highly flexible, exploratory tool, used at any time to gain insights that pre-configured tests might miss.

Reference:
* [Observability and Evals for AI Agents: A Simple Breakdown - Harrison Chase, LangChain (Feb, 2026)](https://youtu.be/FDVdLrloFOw?si=fGFGYkTqYEh1hQ7t)


# Reference reading sources
1. [Define success criteria and build evals (Anthropic Documentation)](https://docs.claude.com/en/docs/test-and-evaluate/develop-tests)
2. [Blog - Demystifying evals for AI agents - Anthropic - 2026](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
3. [Blog - What is AI Agent Evaluation - Databricks - 2026](https://www.databricks.com/blog/what-is-agent-evaluation)
4. [Confident AI - AI Agent evaluation: Metrics, Traces, Human review & Workflows (Apr 2026)](https://www.confident-ai.com/blog/definitive-ai-agent-evaluation-guide)
5. [LLM Evaluation: Methods, Best Practices, and a Practical Roadmap, Langfuse (Nov, 2025)](https://langfuse.com/blog/2025-11-12-evals)
6. [Blog - Your AI product needs evals, Hamel Husain](https://hamel.dev/blog/posts/evals/index.html)
7. [Blog - Everything you need to know about Evals (FAQ), Hamel Husain](https://hamel.dev/blog/posts/evals-faq/)
8. [Blog - Selecting the right AI evals tool, Hamel Husain](https://hamel.dev/blog/posts/eval-tools/)
9. [Blog - Using LLM-as-a-judge for evaluation: A complete guide, Hamel Husain](https://hamel.dev/blog/posts/llm-judge/)
10. [Blog - An LLM-as-a-judge won't save the product - fixing your process will (April, 2025)](https://eugeneyan.com/writing/eval-process/)
11. [Ground truth generation and review best practices for evaluating Gen-AI question-answering with FMEval - AWS (Mar, 2025)](https://aws.amazon.com/blogs/machine-learning/ground-truth-generation-and-review-best-practices-for-evaluating-generative-ai-question-answering-with-fmeval/)
12. [Blog - Evals Flash cards, Hamel Husain](https://hamel.dev/notes/llm/evals/flashcards/)
13. [Huggingface LLM evaluation guidebook](https://huggingface.co/spaces/OpenEvals/evaluation-guidebook)
14. [Huggingface LLM course - Evaluation](https://huggingface.co/learn/llm-course/en/chapter11/5)
15. [Golden dataset: Role in Custom LLM evals - Arize](https://arize.com/resource/golden-dataset/)
16. [Huggingface Using LLM-as-a-judge for an automated and versatile evaluation](https://huggingface.co/learn/cookbook/llm_judge)
17. [Video - Intro To Error Analysis: Creating Custom Data Annotation Apps, Hamel Husain](https://youtu.be/qH1dZ8JLLdU?si=NglOiQ2u3w26q6B2)
18. [Blog - Pass@k vs Pass^k: Understanding agent reliability (2025)](https://www.philschmid.de/agents-pass-at-k-pass-power-k)

# Eval Frameworks 
## Phoenix (Arize AI)
* Evaluation - https://arize.com/docs/phoenix/evaluation/llm-evals
* Using human annotations for eval-driven development - https://arize.com/docs/phoenix/cookbook/human-in-the-loop-workflows-annotations/using-human-annotations-for-eval-driven-development
* Annotate traces on the UI for analysis and dataset creation - https://arize.com/docs/phoenix/tracing/how-to-tracing/feedback-and-annotations/annotating-in-the-ui
* Building your own evals - https://arize.com/docs/phoenix/evaluation/concepts-evals/building-your-own-evals
* Creating datasets - https://arize.com/docs/phoenix/datasets-and-experiments/how-to-datasets/creating-datasets
* Running evals on traces - https://arize.com/docs/phoenix/tracing/how-to-tracing/feedback-and-annotations/evaluating-phoenix-traces
* Customize eval template - https://arize.com/docs/phoenix/evaluation/tutorials/customize-eval-template
* Pre-built eval metrics - https://arize.com/docs/phoenix/evaluation/pre-built-metrics

## LangFuse
* [LLM-as-a-judge](https://langfuse.com/docs/evaluation/evaluation-methods/llm-as-a-judge)


## ADK
* [Evaluating agents with ADK](https://codelabs.developers.google.com/adk-eval/instructions#0)
