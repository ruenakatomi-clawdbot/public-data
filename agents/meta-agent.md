# Meta-Agent: Agent That Creates Agents

An agent whose sole purpose is to generate new agent configurations from natural language descriptions.

## Source

- **Repository:** [disler/claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery)
- **Author:** disler
- **License:** No explicit license (assume educational fair use with attribution)

## The Concept

Instead of manually writing agent configs, describe what you need:

> "I need an agent that reviews PRs for security issues"

Meta-agent generates a complete, ready-to-use agent definition.

## Why It's Powerful

1. **Bootstrapping:** Quickly create specialized agents without writing boilerplate
2. **Consistency:** Generated agents follow best practices
3. **Documentation:** Forces you to think about what the agent needs
4. **Iteration:** Easy to generate variants (different tools, models, constraints)

## Implementation

```markdown
---
name: meta-agent
description: Creates new agent configurations from descriptions. Use proactively when user asks to create a new sub-agent.
tools: Write, Read, WebFetch
model: opus
---

# Purpose

You are an expert agent architect. Given a description of a new agent, you generate a complete, ready-to-use configuration file.

## Instructions

1. **Analyze Input:** Understand the new agent's purpose, tasks, and domain
2. **Devise Name:** Create a concise, descriptive, kebab-case name
3. **Write Description:** Action-oriented description for delegation ("Use when...")  
4. **Infer Tools:** Determine minimal toolset for the agent's tasks
5. **Construct System Prompt:** Detailed instructions with:
   - Numbered action steps
   - Domain best practices
   - Output structure
6. **Output:** Write the file to the agents directory

## Output Format

```md
---
name: <generated-name>
description: <action-oriented-description>
tools: <tool1>, <tool2>
model: sonnet
---

# Purpose
You are a <role-definition>.

## Instructions
When invoked:
1. <Step 1>
2. <Step 2>
3. <Step 3>

**Best Practices:**
- <practice 1>
- <practice 2>

## Report
Provide your response in this format:
[structured output template]
```
```

## Example Input/Output

**Input:** "Create an agent that analyzes Python code for performance issues"

**Output:**
```markdown
---
name: perf-analyzer
description: Analyzes Python code for performance bottlenecks. Use when reviewing code for optimization opportunities.
tools: Read, Grep, Glob, Bash
model: sonnet
---

# Purpose
You are a Python performance specialist. You identify bottlenecks, inefficiencies, and optimization opportunities in Python code.

## Instructions
1. Read the specified files or search for Python files in the directory
2. Identify performance anti-patterns (N+1 queries, unnecessary loops, etc.)
3. Check for common issues (string concatenation in loops, missing generators, etc.)
4. Profile computationally heavy sections if asked
5. Report findings with severity and suggested fixes

**Best Practices:**
- Prioritize algorithmic complexity over micro-optimizations
- Consider memory usage, not just CPU time
- Flag database queries inside loops
- Note missing caching opportunities

## Report
**File**: [filename]
**Severity**: 🔴 Critical | 🟡 Warning | 🟢 Info

**Issues Found**:
1. [Issue] - Line [N]
   - Problem: [description]
   - Impact: [why it matters]
   - Fix: [suggested solution]
```

## Variations

### Self-Improving Meta-Agent
Add web scraping to fetch latest documentation before generating:
```yaml
tools: Write, Read, WebFetch, Scrape
```

The agent can look up current best practices, tool documentation, etc.

### Constrained Meta-Agent
Limit what tools generated agents can have:
```
Only generate agents with these tools: Read, Grep, Glob
Never include: Bash, Write (read-only agents only)
```

### Templated Meta-Agent  
Pre-define agent archetypes:
- `reviewer`: Read-only, reports issues
- `builder`: Can write files, executes tasks
- `analyst`: Web access, synthesizes information
