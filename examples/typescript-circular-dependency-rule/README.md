# TypeScript Circular Dependency - Skill & Rule Pair

This directory demonstrates the **skill + rule** pattern in Claudeception:

## The Problem
Circular dependencies in TypeScript/JavaScript cause runtime errors where imports evaluate to `undefined`.

## Two Forms of Knowledge

### 1. The Skill (in `../typescript-circular-dependency/`)
**How to solve it when it happens**
- Manual detection using tools like `madge`
- Resolution strategies (extract interfaces, dependency injection, etc.)
- Verification steps

### 2. The Rule (in this directory)
**How to prevent it from happening**
- ESLint configuration using `import/no-cycle`
- Automated checking in CI/CD
- Prevention at development time

## When to Use Each

| Use the Skill when: | Use the Rule when: |
|---------------------|-------------------|
| You have an existing circular dependency to fix | Setting up a new project |
| Investigating why imports are undefined | Adding automated checks to prevent issues |
| Learning resolution patterns | Enforcing code quality standards |

## Best Practice
Use **both**:
1. Apply the **rule** to prevent new circular dependencies
2. Keep the **skill** for when you need to resolve existing cycles or understand the problem
