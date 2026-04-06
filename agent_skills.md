**What are skills?**
 * In the context of modern AI development, Agent Skills refer to an open standard for extending the capabilities of AI agents through modular, portable packages of instructions and resources.
 * Unlike general knowledge built into a model, a "Skill" is a specific folder that provides an agent with specialized domain expertise on demand. This allows agents to remain efficient by only loading the context they need for a particular task—a pattern known as progressive disclosure


**Core Components of a Skill -**

Each skill typically consists of a directory containing: 
* SKILL.md: A markdown file with YAML frontmatter (name and description) and detailed instructions.
* Specialized Instructions: Procedural knowledge that guides the agent through complex workflows.
* Scripts & Tools: Executable code (Python, Bash, etc.) that the agent can run to perform actions like editing files or fetching data.
* Resources: Templates, examples, or reference materials the agent can load as needed.


**How Skills Work (3-Stage Loading)-**
* Discovery: At startup, the agent only sees the name and description of available skills (~100 tokens).
* Activation: If your request matches a skill's description, the agent dynamically loads the full instructions.
* Execution: The agent follows the instructions and accesses scripts or resources only if the task requires them.


**Popular Examples of Skills-**
* Data & Productivity: Skills for managing Excel (xlsx), creating PowerPoint decks, or analyzing CSV files.
* Development: Automated code review, testing (Playwright), and GitHub operations.
* Design: Frontend-design tools, SVG/GIF creation, and brand guideline enforcement.
* Integrations: Specialized skills for AWS, Spotify, or Slack


**Benefits for Users & Teams-**
* Consistency: Ensures agents follow specific company or project standards every time.
* Portability: You can "build once and deploy everywhere"—the same skill can work in Claude.ai, Cursor, or via the Claude API.
* Efficiency: Prevents "context bloat" by keeping irrelevant instructions out of the agent's immediate memory.


**References -**
* https://agentskills.io/home
* https://agentskills.so/
* https://agentskills.io/what-are-skills
* Specification (Complete format specification for skills.md files) - https://agentskills.io/specification
* https://learn.microsoft.com/en-us/agent-framework/agents/skills?pivots=programming-language-csharp
  
