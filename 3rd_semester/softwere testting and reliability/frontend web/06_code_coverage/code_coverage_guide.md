# Module 06: Code Coverage Analysis & Enforcement

## 1. Executive Summary & Core Metrics

Code coverage measures the percentage of source code executed while running automated test suites. It provides quantitative metrics to identify untested code paths, dead code, and uncovered logical branches.

### The 4 Pillars of Code Coverage Metrics

```
                 Source Code Coverage Metrics
  ┌──────────────────┬──────────────────┬──────────────────┬──────────────────┐
  │    Statement     │      Branch      │     Function     │       Line       │
  │     Coverage     │     Coverage     │     Coverage     │     Coverage     │
  └────────┬─────────┴────────┬─────────┴────────┬─────────┴────────┬─────────┘
           │                  │                  │                  │
   Executing single    Evaluating all    Calling defined    Executing numbered
   JS statements       if/else/switch    functions at       physical lines of
   in AST              decision paths    least once         source file
```

1. **Statement Coverage**: Percentage of executable AST statements executed.
2. **Branch Coverage**: Percentage of conditional control flow branches (`if/else`, ternary `? :`, `switch` cases, logical short-circuits `&&` / `||`) tested under both `true` and `false` evaluations.
3. **Function Coverage**: Percentage of declared functions or methods invoked.
4. **Line Coverage**: Percentage of physical lines containing executable code hit.

---

## 2. Coverage Engines: V8 vs. Istanbul

| Feature | **V8 Coverage Engine** | **Istanbul (Babel / c8)** |
| :--- | :--- | :--- |
| **Mechanism** | Uses Node.js / Chromium native V8 engine byte-code execution counters | Instruments source code at build time by wrapping every statement/branch in counter functions |
| **Performance Overhead** | Near-zero runtime overhead | 20% to 50% slower test execution due to AST transformation overhead |
| **Accuracy** | Precise line & block execution data | Can report false branch misses on transpiled TS code |
| **Configuration** | Default in Vitest (`provider: 'v8'`) | Default in Jest (`provider: 'babel'`) |

---

## 3. Strict Production Coverage Configuration

```typescript
// vitest.config.ts (Coverage Section)
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'text-summary', 'json', 'lcov', 'html'],
      reportsDirectory: './coverage',
      include: ['src/**/*.{ts,tsx}'],
      exclude: [
        'src/**/*.d.ts',
        'src/**/*.stories.tsx',
        'src/test/**',
        'src/main.tsx',
        'src/**/index.ts',
      ],
      // Enforce strict build failure if thresholds are missed
      thresholds: {
        lines: 85,
        functions: 85,
        branches: 80,
        statements: 85,
        // Per-file threshold enforcement
        autoUpdate: false,
      },
    },
  },
});
```

---

## 4. The 100% Coverage Myth: A Concrete Counter-Example

High code coverage does **not** guarantee code correctness or freedom from bugs!

### Example: 100% Statement & Line Covered Code with Critical Bug

```typescript
// src/utils/divider.ts
export function divideAndFormat(a: number, b: number): string {
  const result = a / b;
  return result.toFixed(2);
}
```

```typescript
// src/utils/divider.test.ts
import { test, expect } from 'vitest';
import { divideAndFormat } from './divider';

test('divides numbers', () => {
  // Executes 100% of statements, lines, functions, and branches!
  expect(divideAndFormat(10, 2)).toBe('5.00');
});
```

### Why this is dangerous:
The test achieves **100% statement & branch coverage**, but fails to test boundary conditions:
1. `divideAndFormat(10, 0)` returns `"Infinity"` (Unhandled boundary condition).
2. `divideAndFormat(NaN, 2)` throws a runtime TypeError: `result.toFixed is not a function` or `"NaN"`.

---

## 5. Ignoring Intentional Untestable Blocks

When dealing with third-party error boundaries or platform-specific fallbacks:

```typescript
/* v8 ignore start */
if (process.env.NODE_ENV === 'production') {
  setupProductionMonitoring();
}
/* v8 ignore stop */

// Or single line ignore:
const legacyId = window.legacyProp /* v8 ignore next */ ?? 'default_id';
```

---

## 6. Senior Frontend Engineer Interview Questions & Answers

### Q1: Why is Branch Coverage generally considered more critical than Statement Coverage?
**Answer**: Statement coverage only proves that a line of code executed, whereas branch coverage proves that logic decision branches (`if (A && B)`, ternary operators, default parameters) were evaluated under all boolean states. A function can achieve 100% statement coverage while missing half of its logical control flow scenarios.

### Q2: How do you integrate coverage reports into CI/CD Pull Request workflows?
**Answer**:
1. Generate LCOV coverage files (`coverage/lcov.info`) during CI test runs.
2. Upload LCOV artifacts to tools like **Codecov** or **Coveralls** using GitHub Actions.
3. Configure PR bots to post automated diff comments showing coverage deltas (e.g., "+0.4% coverage overall, but 2 lines missed in `src/auth.ts`").
4. Block PR merges if coverage drops below specified project thresholds.
