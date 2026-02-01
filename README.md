# Claudeception

Every time you use an AI coding agent, it starts from zero. You spend an hour debugging some obscure error, the agent figures it out, session ends. Next time you hit the same issue? Another hour.

This skill fixes that. When Claude Code discovers something non-obvious (a debugging technique, a workaround, some project-specific pattern), it saves that knowledge as a new skill. Next time a similar problem comes up, the skill gets loaded automatically.

## Installation

### Step 1: Clone the skill

**User-level (recommended)**

```bash
git clone https://github.com/cajias/Claudeception.git ~/.claude/skills/claudeception
```

**Project-level**

```bash
git clone https://github.com/cajias/Claudeception.git .claude/skills/claudeception
```

### Step 2: Set up the activation hook (recommended)

The skill can activate via semantic matching, but a hook ensures it evaluates every session for extractable knowledge.

#### User-level setup (recommended)

1. Create the hooks directory and copy the script:

```bash
mkdir -p ~/.claude/hooks
cp ~/.claude/skills/claudeception/scripts/claudeception-activator.sh ~/.claude/hooks/
chmod +x ~/.claude/hooks/claudeception-activator.sh
```

2. Add the hook to your global Claude settings (`~/.claude/settings.json`):

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/claudeception-activator.sh"
          }
        ]
      }
    ]
  }
}
```

#### Project-level setup

1. Create the hooks directory inside your project and copy the script:

```bash
mkdir -p .claude/hooks
cp .claude/skills/claudeception/scripts/claudeception-activator.sh .claude/hooks/
chmod +x .claude/hooks/claudeception-activator.sh
```

2. Add the hook to your project settings (`.claude/settings.json` in the repo):

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/claudeception-activator.sh"
          }
        ]
      }
    ]
  }
}
```

If you already have a `settings.json`, merge the `hooks` configuration into it.

The hook injects a reminder on every prompt that tells Claude to evaluate whether the current task produced extractable knowledge. This achieves higher activation rates than relying on semantic description matching alone.

## Usage

### Automatic Mode

The skill activates automatically when Claude Code:
- Just completed debugging and discovered a non-obvious solution
- Found a workaround through investigation or trial-and-error
- Resolved an error where the root cause wasn't immediately apparent
- Learned project-specific patterns or configurations through investigation
- Completed any task where the solution required meaningful discovery

### Explicit Mode

Trigger a learning retrospective:

```
/claudeception
```

Or explicitly request skill extraction:

```
Save what we just learned as a skill
```

### What Gets Extracted

Claudeception extracts two forms of knowledge:

1. **Skills**: Solutions, patterns, and debugging knowledge for solving problems when they occur
2. **Rules**: Automated checks (linters, CI/CD, git hooks) for preventing problems before they happen

Not every task produces extractable knowledge. It only extracts:
- Knowledge that required actual discovery (not just reading docs)
- Solutions that will help with future tasks
- Patterns with clear trigger conditions
- Approaches that have been verified to work
- Problems that can be prevented with automation

**Example**: After debugging a circular dependency issue:
- **Skill extracted**: How to detect and resolve circular dependencies manually
- **Rule extracted**: ESLint configuration to prevent circular dependencies automatically

## Skill vs Rule Extraction

| Extract a **Skill** when: | Extract a **Rule** when: |
|---------------------------|-------------------------|
| Knowledge is about solving a problem | Problem can be prevented automatically |
| Manual steps are required | Automated checking is possible |
| Context-dependent judgment needed | Clear pass/fail criteria exist |

Best practice: Often you'll extract **both**—a skill for solving the problem when it occurs, and a rule for preventing it in the future.

## Research

The idea comes from academic work on skill libraries for AI agents.

[Voyager](https://arxiv.org/abs/2305.16291) (Wang et al., 2023) showed that game-playing agents can build up libraries of reusable skills over time, and that this helps them avoid re-learning things they already figured out. [CASCADE](https://arxiv.org/abs/2512.23880) (2024) introduced "meta-skills" (skills for acquiring skills), which is what this is. [SEAgent](https://arxiv.org/abs/2508.04700) (2025) showed agents can learn new software environments through trial and error, which inspired the retrospective feature. [Reflexion](https://arxiv.org/abs/2303.11366) (Shinn et al., 2023) showed that self-reflection helps.

Agents that persist what they learn do better than agents that start fresh.

## How It Works

Claude Code has a native skills system. At startup, it loads skill names and descriptions (about 100 tokens each). When you're working, it matches your current context against those descriptions and pulls in relevant skills.

But this retrieval system can be written to, not just read from. So when this skill notices extractable knowledge, it writes new skills and rules with descriptions optimized for future retrieval.

The description matters a lot. "Helps with database problems" won't match anything useful. "Fix for PrismaClientKnownRequestError in serverless" will match when someone hits that error.

More on the skills architecture [here](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills).

## File Formats

### Skill Format

Extracted skills are markdown files with YAML frontmatter:

```yaml
---
name: prisma-connection-pool-exhaustion
description: |
  Fix for PrismaClientKnownRequestError: Too many database connections 
  in serverless environments (Vercel, AWS Lambda). Use when connection 
  count errors appear after ~5 concurrent requests.
author: Claude Code
version: 1.0.0
date: 2024-01-15
---

# Prisma Connection Pool Exhaustion

## Problem
[What this skill solves]

## Context / Trigger Conditions
[Exact error messages, symptoms, scenarios]

## Solution
[Step-by-step fix]

## Verification
[How to confirm it worked]
```

See `resources/skill-template.md` for the full template.

### Rule Format

Extracted rules document automated checks and preventions:

```yaml
---
name: typescript-no-circular-imports
type: rule
rule-type: eslint
description: |
  Prevents circular import dependencies in TypeScript/JavaScript projects.
  Use when experiencing "Cannot access before initialization" errors or
  to proactively prevent circular dependency issues.
author: Claude Code
version: 1.0.0
date: 2024-03-10
---

# ESLint Rule: No Circular Imports

## Problem
[What this rule prevents]

## Context / When to Apply
[When to add this rule to a project]

## Rule Configuration
[Complete configuration for the linter/tool]

## Installation
[Step-by-step setup]

## What It Catches
[Examples of violations and fixes]

## Verification
[How to test the rule]
```

See `resources/rule-template.md` for the full template.

## Quality Gates

The system is selective about what it extracts:

**For Skills**: If something is just a documentation lookup, or only useful for this one case, or hasn't actually been tested, it won't create a skill. Would this actually help someone who hits this problem in six months? If not, no skill.

**For Rules**: If a problem can't be automatically detected, or the rule would have too many false positives, or it's just style/formatting (use Prettier), it won't create a rule. Will this rule prevent real bugs and save more time than it costs to run? If not, no rule.

## Examples

See `examples/` for samples:

**Skills:**
- `nextjs-server-side-error-debugging/`: Errors that don't show in browser console
- `prisma-connection-pool-exhaustion/`: The "too many connections" serverless problem
- `typescript-circular-dependency/`: Detecting and fixing import cycles

**Rules:**
- `typescript-circular-dependency-rule/`: ESLint rule to prevent circular imports automatically

## Contributing

Contributions welcome. Fork, make changes, submit a PR.

## License

MIT
