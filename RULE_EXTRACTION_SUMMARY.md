# Rule Extraction Feature - Summary

## Overview

Claudeception now extracts **both skills and rules** to create a comprehensive learning system:

- **Skills**: How to solve problems when they occur
- **Rules**: How to prevent problems automatically

## What Changed

### 1. Core Skill Definition (SKILL.md)
- ✅ Updated to version 3.1.0
- ✅ Description mentions "automation rules"
- ✅ New section: "When to Extract a Rule"
- ✅ Comparison table: Skill vs Rule
- ✅ Complete rule extraction process (6 steps)
- ✅ Rule-specific quality criteria
- ✅ Rule extraction example

### 2. Documentation (README.md)
- ✅ Explains skill vs rule distinction
- ✅ Shows both file formats
- ✅ Updated quality gates for both types
- ✅ Lists example rules alongside skills

### 3. Templates
- ✅ `resources/rule-template.md` - Complete rule template
- ✅ Covers all rule types: ESLint, Prettier, CI/CD, Git hooks, custom

### 4. Examples
- ✅ `examples/typescript-circular-dependency-rule/` - Full example
  - RULE.md: ESLint `import/no-cycle` configuration
  - README.md: Explains skill+rule pattern
- ✅ Cross-reference from existing skill to new rule

### 5. Testing
- ✅ `TESTING.md` - Test scenarios and verification checklist

## How It Works

### The Skill + Rule Pattern

When you solve a preventable problem, Claudeception can now create both:

```
Problem: Circular dependencies causing runtime errors

↓

Skill: typescript-circular-dependency
├─ How to detect cycles (madge, visual graphs)
├─ Resolution strategies (interfaces, DI, dynamic imports)
└─ Manual verification steps

↓

Rule: typescript-no-circular-imports
├─ ESLint configuration (import/no-cycle)
├─ Automated checking in development
├─ CI/CD integration
└─ Prevention at build time
```

### Decision Tree

```
After solving a problem, ask:

1. Can this be prevented automatically?
   ├─ YES → Extract a RULE
   └─ NO → Extract only a SKILL

2. Does manual intervention ever need?
   ├─ YES → Extract both SKILL + RULE
   └─ NO → Extract only a RULE

3. Is prevention possible but expensive?
   ├─ Consider trade-offs
   └─ Document in rule's "Performance" section
```

## Example Usage

### Scenario 1: Console.log in Production

**Problem**: Accidentally shipped console.log with sensitive data

**Extraction**:
- **Skill**: How to find and remove console statements
- **Rule**: ESLint `no-console` configuration for production builds

### Scenario 2: Missing Error Handling

**Problem**: Unhandled promise rejections in async code

**Extraction**:
- **Skill**: How to add proper error handling to async functions
- **Rule**: ESLint `no-floating-promises` or custom AST rule

### Scenario 3: Bundle Size Bloat

**Problem**: Accidentally imported entire lodash library

**Extraction**:
- **Skill**: How to analyze bundle and tree-shake properly
- **Rule**: Webpack bundle analyzer + size limit CI check

## Rule Types Supported

| Type | Use Case | Example |
|------|----------|---------|
| **ESLint** | Code patterns, imports, anti-patterns | `import/no-cycle` |
| **Prettier** | Code formatting | `printWidth: 100` |
| **StyleLint** | CSS/SCSS patterns | `no-duplicate-selectors` |
| **CI/CD** | Build-time checks | Bundle size, test coverage |
| **Git Hooks** | Pre-commit validation | `husky` + `lint-staged` |
| **Custom** | Project-specific checks | Custom scripts |

## File Locations

Rules can be saved:
- **Project-specific**: `.claude/skills/[rule-name]/RULE.md`
- **User-wide**: `~/.claude/skills/[rule-name]/RULE.md`
- **With skill**: `.claude/skills/[skill-name]/RULE.md` (same directory)

## Quality Criteria

### For Rules
- ✅ Automatable (no human judgment needed)
- ✅ Deterministic (clear pass/fail)
- ✅ Valuable (prevents real problems)
- ✅ Maintainable (simple configuration)
- ✅ Performant (reasonable runtime)

### Red Flags (Don't Extract)
- ❌ One-time issue
- ❌ Style/formatting only (use Prettier)
- ❌ Too many false positives
- ❌ Requires human judgment
- ❌ Duplicates existing tools

## Integration with Workflow

After completing any task, Claudeception now evaluates:

1. Did we learn something worth saving? → Skill
2. Could this be prevented automatically? → Rule
3. Both? → Extract both with cross-references

This creates a self-improving system that not only remembers solutions but also prevents problems from recurring.

## Next Steps

Users can now:
1. Extract rules alongside skills
2. Build comprehensive prevention systems
3. Create skill+rule pairs for recurring problems
4. Establish project-specific quality gates
5. Automate their learnings

## Benefits

- **Prevention over cure**: Stop problems before they happen
- **Compound learning**: Each problem solved → skill + rule
- **Team scaling**: Rules enforce learnings across team
- **Continuous improvement**: System gets smarter over time
