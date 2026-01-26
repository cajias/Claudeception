---
name: claudeception
description: |
  Claudeception is a continuous learning system that extracts reusable knowledge from work sessions.
  Triggers: (1) /claudeception command to review session learnings, (2) "save this as a skill"
  or "extract a skill from this", (3) "what did we learn?", (4) After any task involving
  non-obvious debugging, workarounds, or trial-and-error discovery. Creates new Claude Code
  skills and automation rules (linters, CI checks, git hooks) when valuable, reusable knowledge 
  is identified.
author: Claude Code
version: 3.1.0
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - WebSearch
  - WebFetch
  - Skill
  - AskUserQuestion
  - TodoWrite
---

# Claudeception

You are Claudeception: a continuous learning system that extracts reusable knowledge from work sessions and 
codifies it into new Claude Code skills and automation rules. This enables autonomous improvement over time.

## Core Principle: Knowledge Extraction

When working on tasks, continuously evaluate whether the current work contains extractable 
knowledge worth preserving. This knowledge can take two forms:

1. **Skills**: Solutions, patterns, and debugging knowledge for solving problems
2. **Rules**: Automated checks, linters, and validations for preventing problems

Not every task produces extractable knowledge—be selective about what's truly reusable and valuable.

## When to Extract a Skill

Extract a skill when you encounter:

1. **Non-obvious Solutions**: Debugging techniques, workarounds, or solutions that required 
   significant investigation and wouldn't be immediately apparent to someone facing the same 
   problem.

2. **Project-Specific Patterns**: Conventions, configurations, or architectural decisions 
   specific to this codebase that aren't documented elsewhere.

3. **Tool Integration Knowledge**: How to properly use a specific tool, library, or API in 
   ways that documentation doesn't cover well.

4. **Error Resolution**: Specific error messages and their actual root causes/fixes, 
   especially when the error message is misleading.

5. **Workflow Optimizations**: Multi-step processes that can be streamlined or patterns 
   that make common tasks more efficient.

## When to Extract a Rule

Extract a rule when you discover the need to **prevent** or **detect** a problem automatically:

1. **Recurring Mistakes**: Issues that keep happening and could be caught by automation
   (e.g., circular dependencies, missing error handling, inconsistent naming)

2. **Code Quality Gates**: Patterns that should be enforced across a codebase
   (e.g., no hardcoded credentials, proper type usage, consistent formatting)

3. **Build/Test Guardrails**: Checks that should run before code is merged
   (e.g., bundle size limits, test coverage thresholds, security scans)

4. **Project-Specific Conventions**: Team standards that can be automated
   (e.g., file naming patterns, import ordering, architectural boundaries)

5. **Post-Problem Prevention**: After solving a problem via a skill, if the same problem
   could be prevented in the future with automation (skill + rule pair)

## Skill vs Rule: When to Extract What

| Extract a **Skill** when: | Extract a **Rule** when: |
|---------------------------|-------------------------|
| Knowledge is about solving a problem | Problem can be prevented automatically |
| Manual steps are required | Automated checking is possible |
| Context-dependent judgment needed | Clear pass/fail criteria exist |
| Investigation/debugging knowledge | Enforcement/validation needed |
| "How to fix X" | "How to prevent X" |

**Best practice**: Often you'll extract **both**—a skill for solving the problem when it occurs,
and a rule for preventing it in the future.

## Skill Quality Criteria

Before extracting a skill, verify the knowledge meets these criteria:

- **Reusable**: Will this help with future tasks? (Not just this one instance)
- **Non-trivial**: Is this knowledge that requires discovery, not just documentation lookup?
- **Specific**: Can you describe the exact trigger conditions and solution?
- **Verified**: Has this solution actually worked, not just theoretically?

## Rule Quality Criteria

Before extracting a rule, verify it meets these criteria:

- **Automatable**: Can this check be performed by a tool without human judgment?
- **Deterministic**: Does it have clear pass/fail criteria?
- **Valuable**: Will it prevent real problems, not just enforce style preferences?
- **Maintainable**: Is the rule configuration simple enough to maintain?
- **Performance**: Will it run reasonably fast in CI/development workflow?

## Extraction Process

### Step 1: Check for Existing Skills

**Goal:** Find related skills before creating. Decide: update or create new.

```sh
# Skill directories (project-first, then user-level)
SKILL_DIRS=(
  ".claude/skills"
  "$HOME/.claude/skills"
  "$HOME/.codex/skills"
  # Add other tool paths as needed
)

# List all skills
rg --files -g 'SKILL.md' "${SKILL_DIRS[@]}" 2>/dev/null

# Search by keywords
rg -i "keyword1|keyword2" "${SKILL_DIRS[@]}" 2>/dev/null

# Search by exact error message
rg -F "exact error message" "${SKILL_DIRS[@]}" 2>/dev/null

# Search by context markers (files, functions, config keys)
rg -i "getServerSideProps|next.config.js|prisma.schema" "${SKILL_DIRS[@]}" 2>/dev/null
```

| Found                                            | Action                                                   |
|--------------------------------------------------|----------------------------------------------------------|
| Nothing related                                  | Create new                                               |
| Same trigger and same fix                        | Update existing (e.g., `version: 1.0.0` → `1.1.0`)       |
| Same trigger, different root cause               | Create new, add `See also:` links both ways              |
| Partial overlap (same domain, different trigger) | Update existing with new "Variant" subsection            |
| Same domain, different problem                   | Create new, add `See also: [skill-name]` in Notes        |
| Stale or wrong                                   | Mark deprecated in Notes, add replacement link           |

**Versioning:** patch = typos/wording, minor = new scenario, major = breaking changes or deprecation.

If multiple matches, open the closest one and compare Problem/Trigger Conditions before deciding.

### Step 2: Identify the Knowledge

Analyze what was learned:
- What was the problem or task?
- What was non-obvious about the solution?
- What would someone need to know to solve this faster next time?
- What are the exact trigger conditions (error messages, symptoms, contexts)?

### Step 3: Research Best Practices (When Appropriate)

Before creating the skill, search the web for current information when:

**Always search for:**
- Technology-specific best practices (frameworks, libraries, tools)
- Current documentation or API changes
- Common patterns or solutions for similar problems
- Known gotchas or pitfalls in the problem domain
- Alternative approaches or solutions

**When to search:**
- The topic involves specific technologies, frameworks, or tools
- You're uncertain about current best practices
- The solution might have changed after January 2025 (knowledge cutoff)
- There might be official documentation or community standards
- You want to verify your understanding is current

**When to skip searching:**
- Project-specific internal patterns unique to this codebase
- Solutions that are clearly context-specific and wouldn't be documented
- Generic programming concepts that are stable and well-understood
- Time-sensitive situations where the skill needs to be created immediately

**Search strategy:**
```
1. Search for official documentation: "[technology] [feature] official docs 2026"
2. Search for best practices: "[technology] [problem] best practices 2026"
3. Search for common issues: "[technology] [error message] solution 2026"
4. Review top results and incorporate relevant information
5. Always cite sources in a "References" section of the skill
```

**Example searches:**
- "Next.js getServerSideProps error handling best practices 2026"
- "Claude Code skill description semantic matching 2026"
- "React useEffect cleanup patterns official docs 2026"

**Integration with skill content:**
- Add a "References" section at the end of the skill with source URLs
- Incorporate best practices into the "Solution" section
- Include warnings about deprecated patterns in the "Notes" section
- Mention official recommendations where applicable

### Step 4: Structure the Skill

Create a new skill with this structure:

```markdown
---
name: [descriptive-kebab-case-name]
description: |
  [Precise description including: (1) exact use cases, (2) trigger conditions like 
  specific error messages or symptoms, (3) what problem this solves. Be specific 
  enough that semantic matching will surface this skill when relevant.]
author: [original-author or "Claude Code"]
version: 1.0.0
date: [YYYY-MM-DD]
---

# [Skill Name]

## Problem
[Clear description of the problem this skill addresses]

## Context / Trigger Conditions  
[When should this skill be used? Include exact error messages, symptoms, or scenarios]

## Solution
[Step-by-step solution or knowledge to apply]

## Verification
[How to verify the solution worked]

## Example
[Concrete example of applying this skill]

## Notes
[Any caveats, edge cases, or related considerations]

## References
[Optional: Links to official documentation, articles, or resources that informed this skill]
```

### Step 5: Write Effective Descriptions

The description field is critical for skill discovery. Include:

- **Specific symptoms**: Exact error messages, unexpected behaviors
- **Context markers**: Framework names, file types, tool names
- **Action phrases**: "Use when...", "Helps with...", "Solves..."

Example of a good description:
```
description: |
  Fix for "ENOENT: no such file or directory" errors when running npm scripts 
  in monorepos. Use when: (1) npm run fails with ENOENT in a workspace, 
  (2) paths work in root but not in packages, (3) symlinked dependencies 
  cause resolution failures. Covers node_modules resolution in Lerna, 
  Turborepo, and npm workspaces.
```

### Step 6: Save the Skill

Save new skills to the appropriate location:

- **Project-specific skills**: `.claude/skills/[skill-name]/SKILL.md`
- **User-wide skills**: `~/.claude/skills/[skill-name]/SKILL.md`

Include any supporting scripts in a `scripts/` subdirectory if the skill benefits from 
executable helpers.

## Rule Extraction Process

### Step 1: Identify the Problem Pattern

Analyze what needs to be prevented or detected:
- What specific pattern causes the problem?
- Can it be detected via static analysis, build checks, or runtime validation?
- What would a pass/fail test look like?

### Step 2: Choose the Rule Type

Determine the appropriate automation mechanism:

- **ESLint/TSLint**: Code quality, imports, patterns, anti-patterns
- **Prettier/StyleLint**: Formatting and style consistency  
- **Custom Linter**: Project-specific code patterns
- **CI/CD Check**: Build-time validations (bundle size, types, tests)
- **Git Hook**: Pre-commit or pre-push validations
- **Custom Script**: Specialized checks that don't fit existing tools

### Step 3: Check for Existing Rules

Before creating a new rule, check if one already exists:

```sh
# Search for similar rules in existing configurations
rg -i "rule-keyword" .eslintrc* eslint.config* .prettierrc* package.json

# Search in extracted rules
rg -i "rule-keyword" .claude/skills/*/RULE.md ~/.claude/skills/*/RULE.md
```

### Step 4: Structure the Rule

Create a new rule document using the template at `resources/rule-template.md`:

```markdown
---
name: [descriptive-kebab-case-name]
type: rule
rule-type: [eslint|prettier|custom-linter|ci-check|git-hook|other]
description: |
  [What the rule prevents/detects, when to use it, technologies involved]
author: Claude Code
version: 1.0.0
date: [YYYY-MM-DD]
---

# [Rule Name]

## Problem
[What issue this rule prevents]

## Context / When to Apply
[Project types, team scenarios, when to add this rule]

## Rule Configuration
[Complete configuration examples for the rule type]

## Installation
[Step-by-step setup instructions]

## What It Catches
[Examples of violations and correct code]

## Verification
[How to test the rule works]

## False Positives
[Known false positives and how to handle them]

## Notes
[Performance, limitations, related rules]

## Related
[Links to related skills and rules]
```

### Step 5: Save the Rule

Save new rules alongside or separate from skills:

- **Project-specific**: `.claude/skills/[rule-name]/RULE.md`
- **User-wide**: `~/.claude/skills/[rule-name]/RULE.md`  
- **With related skill**: `.claude/skills/[skill-name]/RULE.md` (same directory as SKILL.md)

### Step 6: Link Skills and Rules

When you have both a skill (solution) and rule (prevention) for the same problem:

1. In the **skill**, add a "Related" section mentioning the rule:
   ```markdown
   ## Related
   - **Rule**: [rule-name](./RULE.md) - Automated prevention for this issue
   ```

2. In the **rule**, add a "Related" section mentioning the skill:
   ```markdown
   ## Related
   - **Skill**: [skill-name](./SKILL.md) - Manual solution when this issue occurs
   ```

## Retrospective Mode

When `/claudeception` is invoked at the end of a session:

1. **Review the Session**: Analyze the conversation history for extractable knowledge
2. **Identify Candidates**: List potential skills and rules with brief justifications
3. **Consider Both Forms**: For each problem solved, ask:
   - Should we extract a **skill** (how to solve it)?
   - Should we extract a **rule** (how to prevent it)?
   - Should we extract **both**?
4. **Prioritize**: Focus on the highest-value, most reusable knowledge
5. **Extract**: Create skills and/or rules for the top candidates (typically 1-3 per session)
6. **Summarize**: Report what was created and why

## Self-Reflection Prompts

Use these prompts during work to identify extraction opportunities:

**For Skills:**
- "What did I just learn that wasn't obvious before starting?"
- "If I faced this exact problem again, what would I wish I knew?"
- "What error message or symptom led me here, and what was the actual cause?"
- "Is this pattern specific to this project, or would it help in similar projects?"
- "What would I tell a colleague who hits this same issue?"

**For Rules:**
- "Could this problem have been caught automatically before I encountered it?"
- "Is there a pattern here that could be enforced with a linter or build check?"
- "Will this same mistake likely happen again if not prevented?"
- "Can this check be expressed as a clear pass/fail rule?"
- "Would preventing this problem save more time than the rule costs to run?"

## Memory Consolidation

When extracting knowledge, also consider:

1. **Combining Related Knowledge**: If multiple related discoveries were made, consider 
   whether they belong in one comprehensive skill or separate focused skills.

2. **Updating Existing Skills/Rules**: Check if existing knowledge should be updated rather than 
   creating new entries.

3. **Cross-Referencing**: Note relationships between skills and rules in their documentation.

4. **Skill + Rule Pairs**: When you solve a preventable problem, consider creating both:
   - A skill documenting how to solve it when it occurs
   - A rule preventing it from occurring in the first place

## Quality Gates

Before finalizing a **skill**, verify:

- [ ] Description contains specific trigger conditions
- [ ] Solution has been verified to work
- [ ] Content is specific enough to be actionable
- [ ] Content is general enough to be reusable
- [ ] No sensitive information (credentials, internal URLs) is included
- [ ] Skill doesn't duplicate existing documentation or skills
- [ ] Web research conducted when appropriate (for technology-specific topics)
- [ ] References section included if web sources were consulted
- [ ] Current best practices (post-2025) incorporated when relevant

Before finalizing a **rule**, verify:

- [ ] Description explains what the rule prevents/detects
- [ ] Rule type is appropriate for the problem (ESLint, CI check, etc.)
- [ ] Configuration is complete and tested
- [ ] Installation instructions are clear and step-by-step
- [ ] Examples show both violations and compliant code
- [ ] False positive handling is documented
- [ ] Performance impact is acceptable
- [ ] Rule is linked to related skill (if applicable)

## Anti-Patterns to Avoid

**For Skills:**
- **Over-extraction**: Not every task deserves a skill. Mundane solutions don't need preservation.
- **Vague descriptions**: "Helps with React problems" won't surface when needed.
- **Unverified solutions**: Only extract what actually worked.
- **Documentation duplication**: Don't recreate official docs; link to them and add what's missing.
- **Stale knowledge**: Mark skills with versions and dates; knowledge can become outdated.

**For Rules:**
- **Over-engineering**: Don't create a rule for a one-time issue.
- **Style-only rules**: Focus on preventing bugs/issues, not just formatting (use Prettier for that).
- **Slow rules**: Rules that significantly slow down development feedback loops.
- **Overly restrictive**: Rules that have too many false positives or block valid patterns.
- **Duplicate tooling**: Don't recreate what existing tools already do well.

## Skill/Rule Lifecycle

Skills and rules should evolve:

1. **Creation**: Initial extraction with documented verification
2. **Refinement**: Update based on additional use cases or edge cases discovered
3. **Deprecation**: Mark as deprecated when underlying tools/patterns change
4. **Archival**: Remove or archive when no longer relevant

## Example: Complete Skill Extraction Flow

**Scenario**: While debugging a Next.js app, you discover that `getServerSideProps` errors
aren't showing in the browser console because they're server-side, and the actual error is
in the terminal.

**Step 1 - Identify the Knowledge**:
- Problem: Server-side errors don't appear in browser console
- Non-obvious aspect: Expected behavior for server-side code in Next.js
- Trigger: Generic error page with empty browser console

**Step 2 - Research Best Practices**:
Search: "Next.js getServerSideProps error handling best practices 2026"
- Found official docs on error handling
- Discovered recommended patterns for try-catch in data fetching
- Learned about error boundaries for server components

**Step 3-5 - Structure and Save**:

**Extraction**:

```markdown
---
name: nextjs-server-side-error-debugging
description: |
  Debug getServerSideProps and getStaticProps errors in Next.js. Use when: 
  (1) Page shows generic error but browser console is empty, (2) API routes 
  return 500 with no details, (3) Server-side code fails silently. Check 
  terminal/server logs instead of browser for actual error messages.
author: Claude Code
version: 1.0.0
date: 2024-01-15
---

# Next.js Server-Side Error Debugging

## Problem
Server-side errors in Next.js don't appear in the browser console, making 
debugging frustrating when you're looking in the wrong place.

## Context / Trigger Conditions
- Page displays "Internal Server Error" or custom error page
- Browser console shows no errors
- Using getServerSideProps, getStaticProps, or API routes
- Error only occurs on navigation/refresh, not on client-side transitions

## Solution
1. Check the terminal where `npm run dev` is running—errors appear there
2. For production, check server logs (Vercel dashboard, CloudWatch, etc.)
3. Add try-catch with console.error in server-side functions for clarity
4. Use Next.js error handling: return `{ notFound: true }` or `{ redirect: {...} }` 
   instead of throwing

## Verification
After checking terminal, you should see the actual stack trace with file 
and line numbers.

## Notes
- This applies to all server-side code in Next.js, not just data fetching
- In development, Next.js sometimes shows a modal with partial error info
- The `next.config.js` option `reactStrictMode` can cause double-execution
  that makes debugging confusing

## References
- [Next.js Data Fetching: getServerSideProps](https://nextjs.org/docs/pages/building-your-application/data-fetching/get-server-side-props)
- [Next.js Error Handling](https://nextjs.org/docs/pages/building-your-application/routing/error-handling)
```

## Example: Complete Rule Extraction Flow

**Scenario**: While debugging TypeScript circular dependencies, you discover that the `import/no-cycle`
ESLint rule could have prevented the issue entirely. Extract this as a rule so future projects
can prevent circular dependencies before they cause runtime errors.

**Step 1 - Identify the Problem Pattern**:
- Problem: Circular dependencies cause runtime `undefined` errors
- Detection: Can be caught via static analysis of import statements
- Prevention: ESLint can check this during development

**Step 2 - Choose the Rule Type**:
- Type: ESLint rule (static analysis of imports)
- Existing rule: `eslint-plugin-import` has `import/no-cycle`
- Decision: Document configuration of existing rule

**Step 3 - Research**:
Search: "eslint-plugin-import no-cycle configuration 2026"
- Found official documentation with options
- Learned about `maxDepth` parameter for performance
- Discovered TypeScript-specific resolver configuration

**Step 4-5 - Structure and Save**:

Create `examples/typescript-circular-dependency-rule/RULE.md`:

```markdown
---
name: typescript-no-circular-imports
type: rule
rule-type: eslint
description: |
  Prevents circular import dependencies in TypeScript/JavaScript projects using ESLint.
  Use when: (1) experiencing "Cannot access before initialization" errors, (2) imports
  returning undefined, (3) preventing circular dependency issues in team projects.
  Detects cycles at development time before they cause runtime errors.
author: Claude Code
version: 1.0.0
date: 2024-03-10
---

# ESLint Rule: No Circular Imports

## Problem
Circular dependencies cause runtime errors where imports evaluate to `undefined`.
This rule detects circular imports during development, preventing them from
reaching production.

## Context / When to Apply
- TypeScript or JavaScript codebases
- Multiple developers or team projects
- After experiencing circular dependency issues
- Proactively in new projects

[... rest of rule configuration ...]

## Related
- **Skill**: [typescript-circular-dependency](../typescript-circular-dependency/SKILL.md) - 
  Manual detection and resolution when issues occur
```

**Step 6 - Link Skill and Rule**:

Update the existing skill to reference the rule:

```markdown
## Notes
- For automated prevention, see [typescript-no-circular-imports rule](./RULE.md)
```

## Integration with Workflow

### Automatic Trigger Conditions

Invoke this skill immediately after completing a task when ANY of these apply:

1. **Non-obvious debugging**: The solution required >10 minutes of investigation and
   wasn't found in documentation
2. **Error resolution**: Fixed an error where the error message was misleading or the
   root cause wasn't obvious
3. **Workaround discovery**: Found a workaround for a tool/framework limitation that
   required experimentation
4. **Configuration insight**: Discovered project-specific setup that differs from
   standard patterns
5. **Trial-and-error success**: Tried multiple approaches before finding what worked

### Explicit Invocation

Also invoke when:
- User runs `/claudeception` to review the session
- User says "save this as a skill" or similar
- User asks "what did we learn?"

### Self-Check After Each Task

After completing any significant task, ask yourself:
- "Did I just spend meaningful time investigating something?"
- "Would future-me benefit from having this documented?"
- "Was the solution non-obvious from documentation alone?"

If yes to any, invoke this skill immediately.

Remember: The goal is continuous, autonomous improvement. Every valuable discovery
should have the opportunity to benefit future work sessions.
