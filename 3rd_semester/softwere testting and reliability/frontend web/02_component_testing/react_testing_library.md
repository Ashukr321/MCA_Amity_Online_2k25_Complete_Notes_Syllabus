# Module 02: Component Testing with React Testing Library (RTL)

## 1. Executive Summary & Guiding Philosophy

React Testing Library (RTL) provides light utility functions on top of `DOM Testing Library` to query and interact with DOM nodes in a way that mimics how actual end-users interact with your UI.

> **Guiding Principle**: *"The more your tests resemble the way your software is used, the more confidence they can give you."* — Kent C. Dodds

### Core Rule: Do NOT Test Implementation Details
Avoid asserting component state (`wrapper.state()`), prop passing, or internal component class instances. Instead, assert DOM nodes, accessible roles, rendered text, and user-perceived state changes.

---

## 2. Accessible Query Priority Hierarchy

Always prefer queries in the following order to encourage built-in accessibility (a11y):

| Priority | Query Type | Purpose & Best Practice |
| :--- | :--- | :--- |
| **1 (Highest)** | `getByRole` / `findByRole` | Reflects how screen readers parse the accessibility tree. E.g., `getByRole('button', { name: /submit/i })`. |
| **2** | `getByLabelText` | Essential for form controls connected to `<label>` tags. |
| **3** | `getByPlaceholderText` | Backup for form inputs lacking explicitly linked labels. |
| **4** | `getByText` | Useful for non-interactive elements (paragraphs, headings, divs). |
| **5 (Lowest)** | `getByTestId` | Last resort when dynamic text or dynamic roles cannot be queried clean (`data-testid="submit-btn"`). |

### Query Prefix Decision Tree
- `getBy...`: Returns the matching node; **throws immediate error** if 0 or >1 nodes found. (Use for synchronous assertions).
- `queryBy...`: Returns matching node or `null`; does **not** throw. (Use exclusively to assert non-existence: `expect(queryByText('Error')).toBeNull()`).
- `findBy...`: Returns a Promise that resolves when element appears; **times out** if element not found. (Use for async elements: `await findByText('Success')`).

---

## 3. User Interactions: `userEvent` vs `fireEvent`

Never use `fireEvent` for user interaction testing in modern RTL!
- `fireEvent`: Triggers raw dispatch events directly on target elements without bubbling or trigger secondary events (e.g., clicking a checkbox with `fireEvent` does not focus the element or trigger keydown/keyup).
- `userEvent` (`@testing-library/user-event` v14+): Simulates full realistic browser interaction flows (hover -> focus -> mousedown -> mouseup -> click -> input).

```typescript
import userEvent from '@testing-library/user-event';

test('submitting a form with userEvent', async () => {
  const user = userEvent.setup(); // Always call setup() before rendering
  render(<LoginForm />);

  await user.type(screen.getByLabelText(/email/i), 'user@example.com');
  await user.type(screen.getByLabelText(/password/i), 'Secret123!');
  await user.click(screen.getByRole('button', { name: /login/i }));
});
```

---

## 4. Production Component Testing Patterns

### Custom Render Wrapper (`renderWithProviders`)

Real-world enterprise components require Context Providers (Redux Store, React Query Client, Theme Provider, React Router).

```typescript
// src/test/test-utils.tsx
import React, { ReactElement } from 'react';
import { render, RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { MemoryRouter } from 'react-router-dom';

interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  initialEntries?: string[];
}

export function renderWithProviders(
  ui: ReactElement,
  { initialEntries = ['/'], ...renderOptions }: CustomRenderOptions = {}
) {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false, gcTime: 0 },
    },
  });

  function AllTheProviders({ children }: { children: React.ReactNode }) {
    return (
      <QueryClientProvider client={queryClient}>
        <MemoryRouter initialEntries={initialEntries}>{children}</MemoryRouter>
      </QueryClientProvider>
    );
  }

  return {
    ...render(ui, { wrapper: AllTheProviders, ...renderOptions }),
    queryClient,
  };
}

export * from '@testing-library/react';
export { default as userEvent } from '@testing-library/user-event';
```

---

### Scenario B: Testing Complex Async Component State Transitions

```tsx
// src/components/UserProfileCard.tsx
import React, { useState } from 'react';

export function UserProfileCard({ userId }: { userId: string }) {
  const [user, setUser] = useState<{ name: string; email: string } | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchProfile = async () => {
    setLoading(true);
    setError(null);
    try {
      const res = await fetch(`/api/users/${userId}`);
      if (!res.ok) throw new Error('User not found');
      const data = await res.json();
      setUser(data);
    } catch (err: any) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <h2>User Details</h2>
      <button onClick={fetchProfile}>Load Profile</button>
      {loading && <p role="status">Loading profile...</p>}
      {error && <p role="alert">{error}</p>}
      {user && (
        <div>
          <p>Name: {user.name}</p>
          <p>Email: {user.email}</p>
        </div>
      )}
    </div>
  );
}
```

```tsx
// src/components/UserProfileCard.test.tsx
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen, userEvent } from '../test/test-utils';
import { UserProfileCard } from './UserProfileCard';

describe('UserProfileCard Component Tests', () => {
  beforeEach(() => {
    vi.restoreAllMocks();
  });

  it('renders initial state and fetches user profile successfully', async () => {
    const user = userEvent.setup();
    const mockUser = { name: 'Alice Smith', email: 'alice@example.com' };

    global.fetch = vi.fn().mockResolvedValueOnce({
      ok: true,
      json: async () => mockUser,
    });

    render(<UserProfileCard userId="usr_123" />);

    expect(screen.getByRole('heading', { name: /user details/i })).toBeInTheDocument();
    expect(screen.queryByRole('status')).not.toBeInTheDocument();

    await user.click(screen.getByRole('button', { name: /load profile/i }));

    // Async waiting for spinner and resolved content
    expect(screen.getByRole('status')).toHaveTextContent('Loading profile...');
    expect(await screen.findByText('Name: Alice Smith')).toBeInTheDocument();
    expect(screen.getByText('Email: alice@example.com')).toBeInTheDocument();
    expect(screen.queryByRole('status')).not.toBeInTheDocument();
  });

  it('displays error message when API fails', async () => {
    const user = userEvent.setup();

    global.fetch = vi.fn().mockResolvedValueOnce({
      ok: false,
      status: 404,
    });

    render(<UserProfileCard userId="usr_invalid" />);

    await user.click(screen.getByRole('button', { name: /load profile/i }));

    const alert = await screen.findByRole('alert');
    expect(alert).toHaveTextContent('User not found');
  });
});
```

---

## 5. Custom Hook Testing with `renderHook`

```typescript
// src/hooks/useCounter.ts
import { useState, useCallback } from 'react';

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  const increment = useCallback(() => setCount((c) => c + 1), []);
  const decrement = useCallback(() => setCount((c) => c - 1), []);
  return { count, increment, decrement };
}
```

```typescript
// src/hooks/useCounter.test.ts
import { describe, it, expect } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter Custom Hook', () => {
  it('should initialize count and mutate state using act', () => {
    const { result } = renderHook(() => useCounter(10));

    expect(result.current.count).toBe(10);

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(11);
  });
});
```

---

## 6. Senior Frontend Engineer Interview Questions & Answers

### Q1: What causes the infamous `An update to Component inside a test was not wrapped in act(...)` warning, and how do you resolve it properly?
**Answer**:
The `act(...)` warning occurs when a React state update happens outside RTL's tracked event loop (e.g., an un-awaited promise resolves, an async timer fires, or WebSocket data arrives after a user interaction).
**Proper Resolution**:
1. Do **not** wrap every test assertion manually in `act()`.
2. Use async RTL utilities like `await screen.findBy...()` or `await waitFor(() => ...)` to await the DOM updates resulting from the async state transition.
3. Ensure all pending microtasks / user event promises are awaited before test teardown.

### Q2: How do you perform accessibility (a11y) testing inside React Component tests?
**Answer**: By integrating `jest-axe` (or `axe-core`). We pass the container returned from `render(<Component />)` to `axe(container)` and assert `expect(results).toHaveNoViolations()`.

### Q3: When should you use `waitFor` vs `findBy`?
**Answer**: Use `findBy...` whenever querying a single element that will appear asynchronously. Use `waitFor` when asserting non-element conditions (e.g., checking that a callback function was called, or asserting that an element disappears using `queryBy`).
