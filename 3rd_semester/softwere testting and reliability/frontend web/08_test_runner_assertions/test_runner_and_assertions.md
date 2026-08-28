# Module 08: Test Runners, Assertions, Spies & Mocks Deep Dive

## 1. Test Runner Lifecycle & Hook Scope Inheritance

Modern test runners (Vitest, Jest) execute test suites using hierarchical block scopes defined by `describe` and `it`/`test` blocks.

```
┌─────────────────────────────────────────────────────────────┐
│ Root Describe Block                                         │
│  ├── beforeAll  (Runs ONCE before all child blocks)         │
│  ├── beforeEach (Runs BEFORE EVERY test in suite)           │
│  │                                                          │
│  ├── Nested Describe Scope A                                │
│  │    ├── beforeEach (Runs BEFORE tests in Scope A only)    │
│  │    ├── test A1                                           │
│  │    └── afterEach  (Runs AFTER tests in Scope A only)     │
│  │                                                          │
│  ├── afterEach  (Runs AFTER EVERY test in suite)            │
│  └── afterAll   (Runs ONCE after all tests complete)        │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Assertion Equality Spectrum

Choosing the correct matcher prevents subtle bugs in test suites:

| Matcher | Equality Type | Prototype & Undefined Check | Best Use Case |
| :--- | :--- | :--- | :--- |
| `toBe(val)` | Primitive Strict Equality (`Object.is`) | Checks strict reference (`===`). Fails for object literals with identical values. | Strings, numbers, booleans, reference identity checks. |
| `toEqual(val)` | Deep Structural Value Equality | Recursively checks object properties; **ignores `undefined` values & class prototypes**. | Plain JSON payloads, array contents. |
| `toStrictEqual(val)` | Deep Strict Type & Prototype Equality | **Checks object prototypes**, class instances, and explicit `undefined` properties. | Domain models, class instance validations. |
| `toMatchObject(val)` | Partial Object Match | Asserts that target object contains a subset of key-value pairs. | Testing large API responses where only 2 fields matter. |

### Equality Code Comparison

```typescript
import { describe, it, expect } from 'vitest';

class User {
  constructor(public name: string) {}
}

describe('Assertion Deep Dive', () => {
  it('demonstrates equality differences', () => {
    const objA = { name: 'Alice', age: undefined };
    const objB = { name: 'Alice' };

    expect(objA).toEqual(objB); // PASSES! (toEqual ignores undefined)
    // expect(objA).toStrictEqual(objB); // FAILS! (toStrictEqual checks explicit undefined)

    const userInstance = new User('Alice');
    const plainLiteral = { name: 'Alice' };

    expect(userInstance).toEqual(plainLiteral); // PASSES!
    // expect(userInstance).toStrictEqual(plainLiteral); // FAILS! (Prototypes do not match)
  });
});
```

---

## 3. Spies, Mocks, and Stubs: The Clearing Spectrum

Understanding mock cleanup methods is a mandatory requirement for senior engineers to prevent cross-test state leakages.

```typescript
import { vi } from 'vitest';

const spy = vi.fn();
```

| Cleanup Method | Clears Call History? | Resets Implementation / Return Values? | Restores Original Un-mocked Method? |
| :--- | :---: | :---: | :---: |
| `mockClear()` | ✅ YES | ❌ NO | ❌ NO |
| `mockReset()` | ✅ YES | ✅ YES (Resets to empty `noop`) | ❌ NO |
| `mockRestore()` | ✅ YES | ✅ YES | ✅ YES (Restores original method wrapped via `spyOn`) |

### Standard `afterEach` Mock Cleanup

```typescript
import { afterEach, vi } from 'vitest';

afterEach(() => {
  // Always restore original implementations after each test run
  vi.restoreAllMocks();
});
```

---

## 4. Parameterized Testing (`test.each`)

Avoid repetitive test code by leveraging data-driven parameterized tables:

```typescript
import { describe, it, expect } from 'vitest';
import { validateEmail } from './validator';

describe('validateEmail Parameterized Suite', () => {
  it.each([
    { email: 'user@example.com', expected: true, reason: 'standard email format' },
    { email: 'user.name+tag@sub.domain.co', expected: true, reason: 'subdomain and tags' },
    { email: 'invalid-email', expected: false, reason: 'missing @ symbol' },
    { email: 'user@', expected: false, reason: 'missing domain TLD' },
    { email: '', expected: false, reason: 'empty string input' },
  ])('returns $expected for $email ($reason)', ({ email, expected }) => {
    expect(validateEmail(email)).toBe(expected);
  });
});
```

---

## 5. Custom Matchers (`expect.extend`)

Create domain-specific custom assertions for cleaner test suites:

```typescript
// src/test/custom-matchers.ts
import { expect } from 'vitest';

expect.extend({
  toBeWithinRange(received: number, floor: number, ceiling: number) {
    const pass = received >= floor && received <= ceiling;
    if (pass) {
      return {
        message: () => `expected ${received} not to be within range [${floor}, ${ceiling}]`,
        pass: true,
      };
    } else {
      return {
        message: () => `expected ${received} to be within range [${floor}, ${ceiling}]`,
        pass: false,
      };
    }
  },
});
```

---

## 6. Senior Frontend Engineer Interview Questions & Answers

### Q1: What is the difference between `mockReset()` and `mockRestore()`?
**Answer**:
- `mockReset()` clears all call history recorded by a mock AND resets its implementation back to an empty function returning `undefined`.
- `mockRestore()` clears call history, resets implementations, AND restores the original, un-mocked JavaScript method on the host object (only applies to mocks created via `vi.spyOn`).

### Q2: Why can `toEqual()` be dangerous when asserting domain entity objects?
**Answer**: `toEqual()` ignores `undefined` properties and prototype class instances. Two objects with different prototype chains (e.g., a custom `UserEntity` instance vs a plain JS `{}`) will evaluate as equal under `toEqual()`. Senior engineers use `toStrictEqual()` when prototype type safety is required.
