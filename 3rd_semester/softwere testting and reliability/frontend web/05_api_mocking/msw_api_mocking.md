# Module 05: API Mocking with Mock Service Worker (MSW)

## 1. Executive Summary & Architecture

Mock Service Worker (MSW) is the industry standard for API mocking in modern JavaScript applications. Instead of monkey-patching native `fetch`/`axios` modules, MSW intercepts HTTP requests at the network layer using:
1. **Browser**: Service Worker API (`msw worker`).
2. **Node.js**: Intercepts native HTTP/HTTPS sockets using `@mswjs/interceptors` (`setupServer`).

```
┌─────────────────────────────────────────────────────────────┐
│                       Browser / Node                        │
│  ┌────────────────────────┐        ┌─────────────────────┐  │
│  │ Application Code       │        │  MSW Service Worker │  │
│  │ (fetch / axios / RTK)  │        │  or Node Interceptor│  │
│  └───────────┬────────────┘        └──────────▲──────────┘  │
│              │ Outgoing HTTP Request          │             │
│              └────────────────────────────────┘             │
│                                               │             │
│                               Intercepts & Match Handlers   │
│                                               │             │
│                                   ┌───────────▼──────────┐  │
│                                   │ Mocked Response      │  │
│                                   └──────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Why MSW Superior to Module Mocking?
- **Zero Client Code Modification**: Works seamlessly regardless of data fetching library (`fetch`, `axios`, `React Query`, `Apollo Client`, `RTK Query`).
- **Single Source of Truth**: Share identical mock handlers across Unit Tests (Vitest/Jest), Component Tests (RTL), Visual Stories (Storybook), and local browser development!

---

## 2. Defining Centralized API Handlers (`src/mocks/handlers.ts`)

```typescript
import { http, HttpResponse, delay } from 'msw';

export interface User {
  id: string;
  name: string;
  role: 'admin' | 'user';
}

const mockUsers: User[] = [
  { id: 'usr_1', name: 'Alex Johnson', role: 'admin' },
  { id: 'usr_2', name: 'Maria Garcia', role: 'user' },
];

export const handlers = [
  // REST Handler: GET /api/users
  http.get('/api/users', async () => {
    await delay(100); // Simulate network latency
    return HttpResponse.json(mockUsers, { status: 200 });
  }),

  // REST Handler: POST /api/users
  http.post('/api/users', async ({ request }) => {
    const newUser = (await request.json()) as Omit<User, 'id'>;
    if (!newUser.name) {
      return HttpResponse.json({ message: 'Name field required' }, { status: 400 });
    }
    const createdUser = { id: `usr_${Date.now()}`, ...newUser };
    return HttpResponse.json(createdUser, { status: 201 });
  }),
];
```

---

## 3. Node.js Setup for Vitest / RTL (`src/mocks/node.ts`)

```typescript
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

### Hooking MSW into Global Test Suite (`src/test/setup.ts`)

```typescript
import { beforeAll, afterEach, afterAll } from 'vitest';
import { server } from '../mocks/node';

// Start server before all tests run
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));

// Reset runtime dynamic handler overrides after each test case
afterEach(() => server.resetHandlers());

// Clean up server after suite completes
afterAll(() => server.close());
```

---

## 4. Dynamic One-Off Test Overrides (`server.use`)

Test boundary conditions, server crashes, and network timeouts directly inside individual test files without polluting global handlers:

```typescript
import { describe, it, expect } from 'vitest';
import { http, HttpResponse } from 'msw';
import { server } from '../mocks/node';
import { render, screen, userEvent } from '../test/test-utils';
import { UserList } from '../components/UserList';

describe('UserList Resilience Tests', () => {
  it('handles 500 server error gracefully', async () => {
    // Override global GET handler for this test specifically
    server.use(
      http.get('/api/users', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );

    render(<UserList />);

    expect(await screen.findByRole('alert')).toHaveTextContent('Failed to fetch users');
  });

  it('handles network authorization error 401', async () => {
    server.use(
      http.get('/api/users', () => {
        return HttpResponse.json({ message: 'Unauthorized session' }, { status: 401 });
      })
    );

    render(<UserList />);

    expect(await screen.findByText('Please log in to view users')).toBeInTheDocument();
  });
});
```

---

## 5. GraphQL API Mocking Example

```typescript
import { graphql, HttpResponse } from 'msw';

export const graphqlHandlers = [
  graphql.query('GetUserProfile', ({ variables }) => {
    const { userId } = variables;
    return HttpResponse.json({
      data: {
        user: {
          id: userId,
          username: 'johndoe',
          email: 'john@example.com',
        },
      },
    });
  }),
];
```

---

## 6. Senior Frontend Engineer Interview Questions & Answers

### Q1: How does MSW compare to mocking libraries like `axios-mock-adapter` or `jest.mock('axios')`?
**Answer**:
- `axios-mock-adapter` or `jest.mock()` couples tests to a specific networking client implementation (`axios`). If the team switches to `window.fetch`, `React Query`, or GraphQL `Apollo`, all existing mocks break.
- MSW operates at the network socket layer. Client application code executes authentic `fetch`/`XMLHttpRequest` network requests that are caught by the browser worker / Node interceptor, making mocks entirely framework-agnostic.

### Q2: What does `server.resetHandlers()` do in `afterEach`?
**Answer**: `server.resetHandlers()` removes any dynamic runtime handler overrides introduced by `server.use(...)` inside specific tests, restoring the mock server back to its initial baseline handlers array. This prevents test state leakage across spec runs.
