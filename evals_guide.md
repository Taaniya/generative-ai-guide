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


Reference - 
* [Error Analysis: The Highest ROI Technique In AI Engineering - Hamel Husain](https://youtu.be/e2i6JbU2R-s?si=fUaeOioS6IedrEY3)

# Reference reading sources
1. [Define success criteria and build evals (Anthropic Documentation)](https://docs.claude.com/en/docs/test-and-evaluate/develop-tests)
2. [Blog - Demystifying evals for AI agents - Anthropic - 2026](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
3. [Blog - Your AI product needs evals, Hamel Husain](https://hamel.dev/blog/posts/evals/index.html)
4. [Blog - Everything you need to know about Evals (FAQ), Hamel Husain](https://hamel.dev/blog/posts/evals-faq/)
5. [Blog - Selecting the right AI evals tool, Hamel Husain](https://hamel.dev/blog/posts/eval-tools/)
6. [Blog - Using LLM-as-a-judge for evaluation: A complete guide, Hamel Husain](https://hamel.dev/blog/posts/llm-judge/)
7. [Blog - An LLM-as-a-judge won't save the product - fixing your process will](https://eugeneyan.com/writing/eval-process/)
8. [Blog - Evals Flash cards, Hamel Husain](https://hamel.dev/notes/llm/evals/flashcards/)
9. [Huggingface LLM evaluation guidebook](https://huggingface.co/spaces/OpenEvals/evaluation-guidebook)
10. [Huggingface LLM course - Evaluation](https://huggingface.co/learn/llm-course/en/chapter11/5)
11. [Golden dataset: Role in Custom LLM evals - Arize](https://arize.com/resource/golden-dataset/)
12. [Huggingface Using LLM-as-a-judge for an automated and versatile evaluation](https://huggingface.co/learn/cookbook/llm_judge)
13. [Video - Intro To Error Analysis: Creating Custom Data Annotation Apps, Hamel Husain](https://youtu.be/qH1dZ8JLLdU?si=NglOiQ2u3w26q6B2)
14. [Blog - Pass@k vs Pass^k: Understanding agent reliability (2025)](https://www.philschmid.de/agents-pass-at-k-pass-power-k)

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
