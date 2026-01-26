---
name: typescript-no-circular-imports
type: rule
rule-type: eslint
description: |
  Prevents circular import dependencies in TypeScript/JavaScript projects using ESLint.
  Use when: (1) experiencing "Cannot access before initialization" errors, (2) imports
  returning undefined, (3) preventing circular dependency issues in team projects,
  (4) enforcing clean architecture patterns. Detects cycles at development time before
  they cause runtime errors.
author: Claude Code
version: 1.0.0
date: 2024-03-10
---

# ESLint Rule: No Circular Imports

## Problem

Circular dependencies in TypeScript/JavaScript cause runtime errors where imports evaluate
to `undefined` because modules haven't finished initializing. This rule detects circular
imports during development, preventing them from reaching production.

Related to the `typescript-circular-dependency` skill, this rule provides automated
prevention rather than manual detection and resolution.

## Context / When to Apply

Add this rule when:

- **Project type**: TypeScript or JavaScript codebases (any size)
- **Team structure**: Multiple developers or when onboarding new team members
- **Architecture**: Projects using modular architecture with multiple service/component layers
- **Problem frequency**: After experiencing circular dependency issues (reactive) or
  proactively to prevent them
- **Development phase**: Early in project lifecycle for prevention, or after identifying
  circular dependency patterns

Best suited for:
- Monorepos with shared packages
- Service-oriented architectures
- Component libraries
- Projects with complex dependency graphs

## Rule Configuration

### For ESLint (Flat Config - ESLint 9+)

```javascript
// eslint.config.js
import importPlugin from 'eslint-plugin-import';

export default [
  {
    plugins: {
      import: importPlugin
    },
    rules: {
      'import/no-cycle': ['error', { 
        maxDepth: 10,           // How deep to traverse imports (default: Infinity)
        ignoreExternal: true,   // Don't check node_modules
        allowUnsafeDynamicCyclicDependency: false
      }]
    },
    settings: {
      'import/resolver': {
        typescript: true,       // Enable TypeScript resolution
        node: true
      }
    }
  }
];
```

### For ESLint (Legacy Config - ESLint 8 and below)

```javascript
// .eslintrc.js
module.exports = {
  plugins: ['import'],
  rules: {
    'import/no-cycle': ['error', { 
      maxDepth: 10,
      ignoreExternal: true,
      allowUnsafeDynamicCyclicDependency: false
    }]
  },
  settings: {
    'import/resolver': {
      typescript: true,
      node: true
    }
  }
};
```

### Customizing Severity

```javascript
// Warning instead of error (useful during migration)
'import/no-cycle': ['warn', { maxDepth: 10 }]

// Only check direct cycles (depth 1)
'import/no-cycle': ['error', { maxDepth: 1 }]

// Check all depths (can be slow on large projects)
'import/no-cycle': ['error', { maxDepth: Infinity }]
```

## Installation

### Step 1: Install Dependencies

```bash
# For JavaScript projects
npm install --save-dev eslint eslint-plugin-import

# For TypeScript projects (additional packages)
npm install --save-dev eslint-plugin-import eslint-import-resolver-typescript
```

### Step 2: Configure ESLint

Choose your config format (flat config for ESLint 9+, legacy for ESLint 8-):

```bash
# Create config file if it doesn't exist
touch eslint.config.js  # or .eslintrc.js for legacy
```

Add the configuration shown above.

### Step 3: Add to package.json Scripts

```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "check:circular": "eslint . --rule 'import/no-cycle: error'"
  }
}
```

### Step 4: Add to CI/CD

```yaml
# .github/workflows/lint.yml
name: Lint
on: [pull_request, push]

jobs:
  eslint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
```

### Step 5: Add Pre-commit Hook (Optional)

Using Husky:

```bash
npm install --save-dev husky
npx husky init
echo "npm run lint" > .husky/pre-commit
```

Or using lint-staged for faster checks:

```bash
npm install --save-dev husky lint-staged
```

```json
// package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": "eslint --rule 'import/no-cycle: error'"
  }
}
```

## What It Catches

### Example 1: Direct Circular Dependency

**Violation:**

```typescript
// services/userService.ts
import { OrderService } from './orderService';

export class UserService {
  async getUserOrders(userId: string) {
    const orderService = new OrderService();
    return orderService.getOrdersByUser(userId);
  }
}

// services/orderService.ts
import { UserService } from './userService';  // ❌ ESLint error here

export class OrderService {
  async getOrdersByUser(userId: string) {
    const userService = new UserService();
    // ...
  }
}
```

**ESLint Output:**
```
services/orderService.ts
  1:1  error  Dependency cycle detected  import/no-cycle

✖ 1 problem (1 error, 0 warnings)
```

**Fixed:**

```typescript
// types/services.ts (new file)
export interface IUserService {
  getUserOrders(userId: string): Promise<Order[]>;
}

export interface IOrderService {
  getOrdersByUser(userId: string): Promise<Order[]>;
}

// services/userService.ts
import type { IOrderService } from '../types/services';

export class UserService implements IUserService {
  constructor(private orderService: IOrderService) {}
  // ...
}

// services/orderService.ts  
import type { IUserService } from '../types/services';

export class OrderService implements IOrderService {
  constructor(private userService: IUserService) {}
  // ...
}
```

### Example 2: Barrel File Circular Dependency

**Violation:**

```typescript
// components/index.ts
export * from './Button';
export * from './Modal';

// components/Modal.tsx
import { Button } from './index';  // ❌ ESLint error: cycle via index.ts

export function Modal() {
  return <Button>Close</Button>;
}
```

**Fixed:**

```typescript
// components/Modal.tsx
import { Button } from './Button';  // ✅ Direct import, no cycle

export function Modal() {
  return <Button>Close</Button>;
}
```

### Example 3: Transitive Circular Dependency

**Violation (3-module cycle):**

```typescript
// A.ts
import { B } from './B';

// B.ts
import { C } from './C';

// C.ts
import { A } from './A';  // ❌ ESLint error: cycle A → B → C → A
```

## Verification

### Test the Rule is Active

1. Create a test file with a circular dependency:

```typescript
// test-circle-a.ts
import { funcB } from './test-circle-b';
export const funcA = () => funcB();

// test-circle-b.ts
import { funcA } from './test-circle-a';
export const funcB = () => funcA();
```

2. Run ESLint:

```bash
npm run lint
```

3. Expected output:

```
test-circle-b.ts
  1:1  error  Dependency cycle detected  import/no-cycle
```

4. Clean up test files:

```bash
rm test-circle-*.ts
```

### Integration Test in CI

Ensure your CI pipeline fails when circular dependencies are introduced:

1. Create a PR with a circular dependency
2. CI should fail with ESLint error
3. Fix the cycle
4. CI should pass

## False Positives

### Type-Only Imports

If you only need types (not runtime values), the cycle is harmless:

```typescript
// This is safe but might trigger the rule
import type { User } from './userService';  // Type-only import
```

**Solution**: The rule should handle `import type` correctly, but if issues persist:

```typescript
// Disable for specific line
// eslint-disable-next-line import/no-cycle
import type { User } from './userService';
```

### Dynamic Imports

Dynamic imports don't create circular dependencies at module evaluation time:

```typescript
// This is safe
async function loadUser() {
  const { UserService } = await import('./userService');
  return new UserService();
}
```

The rule might still flag this. If intentional, disable for that line.

### External Library Cycles

Sometimes third-party libraries have internal cycles that you can't fix:

```javascript
// .eslintrc.js
rules: {
  'import/no-cycle': ['error', { 
    ignoreExternal: true  // Ignore node_modules
  }]
}
```

## Performance Considerations

- **Large projects**: Set `maxDepth` to a reasonable value (10-20) to avoid slow linting
- **CI optimization**: Cache `node_modules` and ESLint cache
- **Pre-commit**: Use `lint-staged` to only check changed files

```json
// .eslintrc.js - optimize for large projects
{
  "rules": {
    "import/no-cycle": ["error", { 
      "maxDepth": 10,  // Limit search depth
      "ignoreExternal": true  // Skip node_modules
    }]
  }
}
```

## Notes

- **Type imports are safe**: `import type` doesn't create runtime cycles, but the rule may still flag them
- **Dynamic imports**: `await import()` breaks cycles but may still be flagged
- **Barrel files**: Be cautious with `index.ts` re-exports as they're a common source of cycles
- **Gradual adoption**: Start with `'warn'` severity during migration, then upgrade to `'error'`
- **Complementary tools**: Combine with `madge` for visualization during investigation
- **ESLint performance**: On very large codebases, this rule can slow down linting; consider adjusting `maxDepth`

## Migration Strategy

If adding to an existing project with cycles:

1. **Audit**: Run `madge --circular src/` to find all existing cycles
2. **Document**: Create a list of known cycles and their remediation plan
3. **Start with warnings**: Set rule to `'warn'` initially
4. **Fix incrementally**: Address one cycle at a time
5. **Prevent new ones**: Use `overrides` to enforce `'error'` on new code:

```javascript
module.exports = {
  rules: {
    'import/no-cycle': 'warn'  // Existing code
  },
  overrides: [{
    files: ['src/new-feature/**/*.ts'],
    rules: {
      'import/no-cycle': 'error'  // New code must be clean
    }
  }]
};
```

6. **Upgrade**: Once all cycles are fixed, change global rule to `'error'`

## Related

- **Skill**: [typescript-circular-dependency](../typescript-circular-dependency/SKILL.md) - Manual detection and resolution strategies
- **Tool**: `madge` - Visualize dependency graphs and find cycles
- **Other Rules**: 
  - `import/no-self-import` - Prevents a module importing itself
  - `import/no-relative-parent-imports` - Enforces architectural boundaries

## References

- [ESLint Plugin Import Documentation](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-cycle.md)
- [TypeScript Circular Dependencies Explained](https://medium.com/visual-development/how-to-fix-nasty-circular-dependency-issues-once-and-for-all-in-javascript-typescript-a04c987cf0de)
