Consider your Role as a Developers Advocate and given a task to help them improve productivity.

prepare me Slides with gamma connector. This presentation will be mainly for developers, so slides can be technical

# Slide 1:
Topic: how to efficiently use Github Copilot and Boost Productivity
Set a relevant image, theme to be dark and a placeholder for name

# Slide 2:

## Prompting
show how RCTFC Framework helps to prepare a better prompt

where RCTFC stands for
1. R - Role
2. C - Context
3. T - Task
4. F - Format
5. C - Constraints

-> Describe in one sentence and share one example for each of the point.

# Slide 3:

## Task
Give an task to be interactive and ask team to prepare a prompt for a specific task. share the best prompt for that task with marking out in details.

# Slide 4:

## Why Skills instruction are required
Read https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf

# Slide 5:

## What is a skill?
A skill is a folder containing:
• SKILL.md (required): Instructions in Markdown with YAML frontmatter
• scripts/ (optional): Executable code (Python, Bash, etc.)
• references/ (optional): Documentation loaded as needed
• assets/ (optional): Templates, fonts, icons used in output

# Slide 5:

## The kitchen analogy
MCP provides the professional kitchen: access to tools, ingredients, and
equipment.
Skills provide the recipes: step-by-step instructions on how to create something
valuable.

# Slide 9:

Good use case definition on SKILL.md file

Common skill use case categories:
Category 1: Document & Asset Creation
Used for: Creating consistent, high-quality output including documents,
presentations, apps, designs, code, etc.
Real example: frontend-design skill (also see skills for docx, pptx, xlsx, and
ppt)

Category 2: Workflow Automation 
Used for: Multi-step processes that benefit from consistent methodology,
including coordination across multiple MCP servers.
Real example: skill-creator skill

Category 3: MCP Enhancement
Used for: Workflow guidance to enhance the tool access an MCP server provides.
Real example: sentry-code-review skill (from Sentry)

# Slide 10:

Skills file structure

File structure
your-skill-name/
├── SKILL.md # Required - main skill file
├── scripts/ # Optional - executable code
│ ├── process_data.py # Example
│ └── validate.sh # Example
├── references/ # Optional - documentation
│ ├── api-guide.md # Example
│ └── examples/ # Example
└── assets/ # Optional - templates, etc.
 └── report-template.md # Example

# Slide 11:

Keep SKILL.md under 5,000 words and also mention other good to have skills.

# Slide 9:
Distribution and sharing
How individual users get skills:
1. Download the skill folder
2. Zip the folder (if needed)
3. Upload to Claude.ai via Settings > Capabilities > Skills
4. Or place in Claude Code skills directory


# Slide 12:
Large context issues
Symptom: Skill seems slow or responses degraded
Causes:
• Skill content too large
• Too many skills enabled simultaneously
• All content loaded instead of progressive disclosure
Solutions:
1. Optimize SKILL.md size
– Move detailed docs to references/
– Link to references instead of inline
– Keep SKILL.md under 5,000 words
2. Reduce enabled skills
– Evaluate if you have more than 20 - 50 skills enabled simultaneously
– Recommend selective enablement
– Consider skill "packs" for related capabilities

# Slide 13:

Getting more from each token: How Copilot improves context handling and model routing

Read the link

- https://github.blog/ai-and-ml/github-copilot/getting-more-from-each-token-how-copilot-improves-context-handling-and-model-routing/ 

and prepare.

# Slide 14:
Calibrating effort and thinking depth
The effort parameter allows you to tune Claude's intelligence versus token spend, trading off capability for faster speed and lower costs. Start with the xhigh effort level for coding and agentic use cases, and use a minimum of high effort for most intelligence-sensitive use cases. Experiment with other effort levels to further tune token usage and intelligence:

max: Max effort can deliver performance gains in some use cases, but may show diminishing returns from increased token usage. This setting can also sometimes be prone to overthinking. Test max effort for intelligence-demanding tasks.
xhigh: Extra high effort is the best setting for most coding and agentic use cases.
high: This setting balances token usage and intelligence. For most intelligence-sensitive use cases, use a minimum of high effort.
medium: Good for cost-sensitive use cases that need to reduce token usage while trading off intelligence.
low: Reserve for short, scoped tasks and latency-sensitive workloads that are not intelligence-sensitive.

If you observe shallow reasoning on complex problems, raise effort to high or xhigh rather than prompting around it. If you need to keep effort at low for latency, add targeted guidance

# Slide 15:
CodeGraph

https://medium.com/kd-agentic/codegraph-the-open-source-knowledge-graph-that-makes-ai-coding-tools-dramatically-cheaper-190f8b89f8a7

Read and make a simple slide on the use cases

# Slide 14:
Most developers think AI costs are driven by the prompts they type.

That is actually only part of it.

The rest of it comes from the context being sent behind the scenes.

Every Copilot Chat request can include:
 ✅ Custom instructions
 ✅ Open files
 ✅ Prompt files
 ✅ MCP tool definitions
 ✅ Conversation history

That means a simple question like "What does this function do?" might actually send thousands of tokens to the model.

In my latest video, I break down my 5 Rules of Token Optimization for GitHub Copilot:

1️⃣ Treat Context Like a Budget
2️⃣ Load On-Demand, Not Always-On
3️⃣ Right-Size Your Model
4️⃣ Be Precise, Not Polite
5️⃣ Clean Your Context

A few takeaways:
🔹 Smaller models are often more than capable for everyday development tasks.
🔹 Prompt files are great for detailed guidance because they're only loaded when needed.
🔹 Long instruction files cost tokens on every request. Make every line earn its place.
🔹 Starting a fresh chat between tasks often improves response quality more than people realize.
🔹 Better context management doesn't just reduce costs. It frequently produces better answers.

The goal isn't to use less AI.

The goal is to use AI more intentionally.

What strategies have you found effective for managing context and getting better results from Copilot?
