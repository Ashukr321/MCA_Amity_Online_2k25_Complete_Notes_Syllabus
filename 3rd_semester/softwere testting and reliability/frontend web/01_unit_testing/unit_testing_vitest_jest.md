# Module 01: Unit Testing with Vitest & Jest

## 1. Executive Summary & Core Principles

Unit testing in modern web applications isolates individual units of code—such as pure functions, domain models, utility helpers, custom hooks logic, and state reducers—to verify that specific inputs deterministically produce expected outputs without external side effects.

### The F.I.R.S.T. Principles of Unit Testing
- **Fast**: Unit tests must execute in milliseconds. A suite of 1,000 unit tests should run in under 3 seconds.
- **Independent/Isolated**: Tests must not depend on each other or share mutable global state. Any test should pass regardless of execution order.
- **Repeatable**: Results must be deterministic across environments (local dev, CI/CD server, Docker containers).
- **Self-Validating**: Tests yield a pass/fail boolean result automatically with clear error diffs.
- **Thorough & Timely**: Tests cover happy paths, edge cases, boundary values, null/undefined inputs, and async error rejections.

---

## 2. Jest vs. Vitest: Architectural Deep Dive

| Architectural Feature | **Jest** | **Vitest** |
| :--- | :--- | :--- |
| **Module System** | CommonJS native (ESM requires experimental flags / Babel / ts-jest transform step) | ESM-first native engine powered by Vite |
| **Bundler Integration** | Custom module resolution pipeline (requires re-configuring aliases in `jest.config.js`) | Reuses `vite.config.ts` setup (plugins, path aliases, CSS handling) |
| **Speed & Performance** | Heavy cold startup time due to custom file transformations | Instant HMR watch mode; fast execution using Worker Threads (`tinypool`) |
| **DOM Simulation** | Defaults to `jsdom` | Supports `jsdom` and lightweight `happy-dom` |
| **API Compatibility** | De-facto standard matcher API | 100% Jest-compatible API drop-in (`vi` spy replacement for `jest`) |

### When to Choose Which?
- **Choose Vitest**: For modern Vite, Next.js (App Router with ESM), Remix, Nuxt, or TypeScript projects.
- **Choose Jest**: For legacy Webpack/Babel React codebases, Create React App legacy projects, or enterprise codebases bound to existing Jest plugins.

---

## 3. Production Configuration Setup

### Vitest Production Setup (`vitest.config.ts`)

```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'happy-dom', // Faster than jsdom for pure unit & component tests
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.{test,spec}.{ts,tsx}'],
    exclude: ['node_modules', 'dist', 'e2e'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      thresholds: {
        lines: 85,
        functions: 85,
        branches: 80,
        statements: 85,
      },
    },
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### Setup Script (`src/test/setup.ts`)

```typescript
import '@testing-library/jest-dom/vitest';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';

// Automatically cleanup DOM after each test case
afterEach(() => {
  cleanup();
});

// Global Window Mocks for Browser APIs
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation((query) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});
```

---

## 4. Writing Senior-Grade Unit Tests

### Scenario A: Testing Pure Complex Business Logic (Currency & Tax Calculator)

```typescript
// src/utils/calculator.ts
export interface CartItem {
  id: string;
  price: number;
  quantity: number;
  taxExempt?: boolean;
}

export function calculateOrderTotal(items: CartItem[], taxRate: number, discountCode?: string): {
  subtotal: number;
  tax: number;
  discount: number;
  total: number;
} {
  if (taxRate < 0) throw new Error('Tax rate cannot be negative');

  const subtotal = items.reduce((sum, item) => sum + item.price * item.quantity, 0);

  let discount = 0;
  if (discountCode === 'SAVE20') {
    discount = subtotal * 0.2;
  } else if (discountCode === 'FLAT10') {
    discount = 10;
  }

  const taxableSubtotal = items
    .filter((item) => !item.taxExempt)
    .reduce((sum, item) => sum + item.price * item.quantity, 0);

  const discountedTaxable = Math.max(0, taxableSubtotal - discount);
  const tax = Number((discountedTaxable * taxRate).toFixed(2));
  const total = Number(Math.max(0, subtotal - discount + tax).toFixed(2));

  return { subtotal, tax, discount, total };
}
```

```typescript
// src/utils/calculator.test.ts
import { describe, it, expect } from 'vitest';
import { calculateOrderTotal, CartItem } from './calculator';

describe('calculateOrderTotal Unit Tests', () => {
  const sampleItems: CartItem[] = [
    { id: '1', price: 100, quantity: 2 }, // 200
    { id: '2', price: 50, quantity: 1, taxExempt: true }, // 50 (Exempt)
  ];

  it('should calculate subtotal, tax, and total correctly without discount', () => {
    const result = calculateOrderTotal(sampleItems, 0.1); // 10% tax
    expect(result).toEqual({
      subtotal: 250,
      tax: 20, // 10% of taxable 200
      discount: 0,
      total: 270,
    });
  });

  it('should apply percentage discount correctly', () => {
    const result = calculateOrderTotal(sampleItems, 0.1, 'SAVE20');
    expect(result.discount).toBe(50); // 20% of 250
    expect(result.subtotal).toBe(250);
  });

  it('should throw an explicit error when tax rate is negative', () => {
    expect(() => calculateOrderTotal(sampleItems, -0.05)).toThrowError('Tax rate cannot be negative');
  });

  it.each([
    { items: [], taxRate: 0.1, expectedTotal: 0 },
    { items: [{ id: '1', price: 10, quantity: 0 }], taxRate: 0.1, expectedTotal: 0 },
  ])('should handle boundary case when items are empty or zero quantity: %j', ({ items, taxRate, expectedTotal }) => {
    const result = calculateOrderTotal(items, taxRate);
    expect(result.total).toBe(expectedTotal);
  });
});
```

---

## 5. Advanced Mocking Techniques & Isolation

### Mocking Modules, Timers, and Environment Variables

```typescript
// src/services/analytics.ts
import axios from 'axios';

export async function sendTelemetryEvent(eventName: string, payload: Record<string, unknown>) {
  if (process.env.NODE_ENV === 'test-disabled') return false;
  const response = await axios.post('/api/telemetry', { eventName, payload, timestamp: Date.now() });
  return response.status === 200;
}
```

```typescript
// src/services/analytics.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import axios from 'axios';
import { sendTelemetryEvent } from './analytics';

// Mock module axios
vi.mock('axios');
const mockedAxios = vi.mocked(axios, true);

describe('sendTelemetryEvent Isolation Tests', () => {
  beforeEach(() => {
    vi.useFakeTimers();
    vi.setSystemTime(new Date(2026, 0, 1, 12, 0, 0)); // Fixed timestamp
  });

  afterEach(() => {
    vi.useRealTimers();
    vi.restoreAllMocks();
  });

  it('should send payload with current timestamp', async () => {
    mockedAxios.post.mockResolvedValueOnce({ status: 200, data: { success: true } });

    const success = await sendTelemetryEvent('user_signup', { userId: '123' });

    expect(success).toBe(true);
    expect(mockedAxios.post).toHaveBeenCalledWith('/api/telemetry', {
      eventName: 'user_signup',
      payload: { userId: '123' },
      timestamp: new Date(2026, 0, 1, 12, 0, 0).getTime(),
    });
  });
});
```

---

## 6. Senior Frontend Engineer Interview Questions & Answers

### Q1: Why should you avoid testing internal implementation details in unit tests?
**Answer**: Testing implementation details (e.g., asserting private method calls, checking internal variable names, or inspecting state structures) makes tests fragile. When code is refactored without altering public behavior, implementation-dependent tests break, causing false negatives. Senior engineers test contracts (inputs vs outputs/behavior) rather than internal mechanics.

### Q2: What is the difference between `vi.fn()`, `vi.spyOn()`, and `vi.mock()`?
**Answer**:
- `vi.fn()`: Creates a standalone mock function to track calls and returns mock values.
- `vi.spyOn(object, 'method')`: Wraps an existing method on an object/module without replacing it by default, allowing observation or optional implementation overrides while retaining the original method restoration capability via `.mockRestore()`.
- `vi.mock('modulePath')`: Hoists module resolution to replace an entire module dependency with a auto-mocked or explicitly mocked object.

### Q3: How do you handle non-deterministic code (e.g., `Date.now()`, `Math.random()`, `setTimeout`) in unit tests?
**Answer**: We make non-deterministic code deterministic using fake timers (`vi.useFakeTimers()`), setting explicit system clock times (`vi.setSystemTime()`), and stubbing random generators (`vi.spyOn(Math, 'random').mockReturnValue(0.5)`).
