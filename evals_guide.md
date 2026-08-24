#### How is Error analysis helpful? What are the best practices while moving from generic evals to problem-specific diagnostics & domain specific observability?

1. Prioritize Manual error analysis:
    - Even with experience and despite the allure of automated benchmarks, the highest ROI activity is direct manual review of production logs.
    - By looking at the data over more and more real-world interactions can help define the product roadmap and identify gaps between the expected system behavior and user mental models. This way engineers can prioritize bug fixes and feature development based on real user pain points.

2. Custom data viewers over off-the-shelf dashboards:
   - Don't rely solely on off-the-shelf dashboards which often hide the nuance of specialized domains.
   - Developing a custom interface to visualize your specific data structure (e.g., voice/email/text threads), that allow you to replay the full context - human input, tool calls, model responses - removes the cognitive friction and allows for rapid, qualitative error analysis before moving to quantitative evals.

3. Shift from generic scores:
   - Move away from 'vanity' scores like generic hallucination, conciseness towards error taxonomy grounded in business logic (business logic-driven evals), e.g., "tour shcheduling errors", "handoff failures").
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
1. Manual foundation - Manually review your conversation data to identify recurring issues (e.g., missed handoffs, incorrect scheduling) and create descriptive notes for why these failures occurred.
2. Develop a Taxonomy: Once you understand the common failure patterns, define a clear set of categories or tags that represent these issues
3. Use LLMs for Categorization: Leverage LLMs to automatically label your raw conversation data against your defined taxonomy. By providing the model with your conversation logs and the set of categories, it can classify new interactions without manual intervention.
4. Aggregated Reporting: Use these automated labels to aggregate your data and count which errors occur most frequently. This turns qualitative logs into quantitative data that drives your product roadmap and engineering priorities


Reference - 
* [Error Analysis: The Highest ROI Technique In AI Engineering - Hamel Husain](https://youtu.be/e2i6JbU2R-s?si=fUaeOioS6IedrEY3)

# Reference reading sources
1. [Define success criteria and build evals (Anthropic Documentation)](https://docs.claude.com/en/docs/test-and-evaluate/develop-tests)
2. [Blog - Demystifying evals for AI agents - Anthropic - 2026](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
3. [LLM Evaluation: Methods, Best Practices, and a Practical Roadmap, Langfuse (Nov, 2025)](https://langfuse.com/blog/2025-11-12-evals)
4. [Blog - Your AI product needs evals, Hamel Husain](https://hamel.dev/blog/posts/evals/index.html)
5. [Blog - Everything you need to know about Evals (FAQ), Hamel Husain](https://hamel.dev/blog/posts/evals-faq/)
6. [Blog - Selecting the right AI evals tool, Hamel Husain](https://hamel.dev/blog/posts/eval-tools/)
7. [Blog - Using LLM-as-a-judge for evaluation: A complete guide, Hamel Husain](https://hamel.dev/blog/posts/llm-judge/)
8. [Blog - An LLM-as-a-judge won't save the product - fixing your process will (April, 2025)](https://eugeneyan.com/writing/eval-process/)
9. [Ground truth generation and review best practices for evaluating Gen-AI question-answering with FMEval - AWS (Mar, 2025)](https://aws.amazon.com/blogs/machine-learning/ground-truth-generation-and-review-best-practices-for-evaluating-generative-ai-question-answering-with-fmeval/)
10. [Blog - Evals Flash cards, Hamel Husain](https://hamel.dev/notes/llm/evals/flashcards/)
11. [Huggingface LLM evaluation guidebook](https://huggingface.co/spaces/OpenEvals/evaluation-guidebook)
12. [Huggingface LLM course - Evaluation](https://huggingface.co/learn/llm-course/en/chapter11/5)
13. [Golden dataset: Role in Custom LLM evals - Arize](https://arize.com/resource/golden-dataset/)
14. [Huggingface Using LLM-as-a-judge for an automated and versatile evaluation](https://huggingface.co/learn/cookbook/llm_judge)
15. [Video - Intro To Error Analysis: Creating Custom Data Annotation Apps, Hamel Husain](https://youtu.be/qH1dZ8JLLdU?si=NglOiQ2u3w26q6B2)
16. [Blog - Pass@k vs Pass^k: Understanding agent reliability (2025)](https://www.philschmid.de/agents-pass-at-k-pass-power-k)

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
