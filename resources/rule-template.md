---
name: [descriptive-kebab-case-name]
type: rule
rule-type: [eslint|prettier|stylelint|custom-linter|ci-check|git-hook|other]
description: |
  [REQUIRED: Precise description that enables semantic matching. Include:
  (1) What problem this rule prevents/detects
  (2) When this rule should trigger
  (3) Key technologies/frameworks involved
  Example: "Prevents circular imports in TypeScript projects"]
author: Claude Code
version: 1.0.0
date: YYYY-MM-DD
---

# [Rule Name - Human Readable Title]

## Problem

[Clear description of what issue this rule prevents or detects.
Why is this rule necessary? What pain does it solve?]

## Context / When to Apply

[When should this rule be added to a project? Be specific:]

- [Project type: e.g., "TypeScript monorepos"]
- [Framework: e.g., "Next.js applications"]
- [Problem domain: e.g., "When experiencing circular dependency issues"]
- [Team size/maturity: e.g., "Teams with multiple contributors"]

## Rule Configuration

### For ESLint

```javascript
// .eslintrc.js or eslint.config.js
module.exports = {
  plugins: ['plugin-name'],
  rules: {
    'rule-name': ['error', {
      // Rule options
    }]
  }
}
```

### For Prettier

```json
{
  "rule-name": "value"
}
```

### For CI/CD

```yaml
# .github/workflows/check.yml
name: Rule Check
on: [pull_request]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm run check:rule
```

### For Git Hooks

```bash
#!/bin/bash
# .git/hooks/pre-commit or use husky

# Rule check command
```

### For Custom Scripts

```bash
# package.json
{
  "scripts": {
    "check:rule-name": "command-to-check-rule"
  }
}
```

## Installation

[Step-by-step instructions to install and configure the rule:]

### Step 1: Install Dependencies

```bash
npm install --save-dev package-name
```

### Step 2: Configure

[Configuration steps]

### Step 3: Add to CI/Build

[How to integrate into development workflow]

## What It Catches

[Specific examples of what this rule detects:]

**Example 1: [Description]**

```language
// Bad - what the rule catches
```

```language
// Good - what passes the rule
```

**Example 2: [Description]**

```language
// Bad
```

```language
// Good
```

## Verification

[How to verify the rule is working:]

1. [Test step 1]
2. [Test step 2]
3. [Expected outcome]

## False Positives

[Common false positives and how to handle them:]

- [False positive 1]: Use `// eslint-disable-next-line rule-name` or equivalent
- [False positive 2]: Configure exception in rule options

## Notes

[Important considerations:]

- [Performance impact if any]
- [When to disable the rule]
- [Related rules or tools]
- [Known limitations]
- [Alternative approaches]

## Related

- **Skill**: [link-to-related-skill] - The knowledge/solution this rule helps prevent
- **Other Rules**: [link-to-complementary-rules]

---

<!-- 
Extraction Checklist (remove before saving):
- [ ] Name is descriptive and uses kebab-case
- [ ] Rule type is specified correctly
- [ ] Description explains what the rule prevents/detects
- [ ] Context clearly states when to apply
- [ ] Configuration is complete and tested
- [ ] Installation steps are clear
- [ ] Examples show both violations and compliant code
- [ ] Verification steps are included
- [ ] False positive handling is documented
- [ ] Related skills/rules are cross-referenced
- [ ] No sensitive information included
-->
