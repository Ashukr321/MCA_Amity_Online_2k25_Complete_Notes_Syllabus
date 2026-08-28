# Module 04: Legacy & Enterprise E2E Testing with Cypress

## 1. Executive Summary & Architecture

Cypress is an established, widely adopted JavaScript end-to-end testing framework. Unlike traditional Selenium or Playwright, Cypress executes **inside the browser instance itself** in the same run-loop as the application under test.

### Cypress In-Browser Execution Model
```
┌──────────────────────────────────────────────────────────┐
│                   Browser Window                         │
│  ┌──────────────────────────┐  ┌──────────────────────┐  │
│  │   Cypress Test Runner    │  │  Application Under   │  │
│  │    (Chai, Mocha, Sinon)  │  │   Test (React/App)   │  │
│  └─────────────┬────────────┘  └──────────▲───────────┘  │
│                │ Direct JS Access         │              │
│                └──────────────────────────┘              │
└─────────────────────────────┬────────────────────────────┘
                              │ Node.js Proxy Process
                              ▼ (Intercepts HTTP Traffic)
```

### Key Architectural Differences: Cypress vs Playwright
- **Execution Context**: Cypress runs *inside* the browser DOM alongside your app; Playwright runs *outside* controlling the browser over WebSockets.
- **Async Execution**: Cypress queues commands into a custom promise-like chain; Playwright uses standard native `async/await`.
- **Multi-Tab / Multi-Domain**: Cypress enforces strict single-tab and same-origin policies (mitigated via `cy.origin()`); Playwright supports multi-tab natively.

---

## 2. Configuration & Custom Commands Setup

```typescript
// cypress.config.ts
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
    defaultCommandTimeout: 10000,
    requestTimeout: 10000,
    video: false,
    screenshotOnRunFailure: true,
    setupNodeEvents(on, config) {
      // Node event listeners (e.g. database seeds, task hooks)
    },
  },
});
```

### Session Caching & Custom Auth Command (`cypress/support/commands.ts`)

```typescript
// Fast Auth Session Bypass in Cypress
Cypress.Commands.add('loginViaAPI', (email: string, pass: string) => {
  cy.session([email, pass], () => {
    cy.request({
      method: 'POST',
      url: '/api/auth/login',
      body: { email, password: pass },
    }).then((response) => {
      expect(response.status).to.eq(200);
      window.localStorage.setItem('token', response.body.token);
    });
  });
});
```

---

## 3. Real-World E2E Test Suite (`cypress/e2e/orders.cy.ts`)

```typescript
describe('Orders Management E2E Suite', () => {
  beforeEach(() => {
    // Intercept network request before navigating
    cy.intercept('GET', '/api/orders', { fixture: 'orders.json' }).as('getOrders');
    cy.intercept('POST', '/api/orders/cancel/*', { statusCode: 200, body: { success: true } }).as('cancelOrder');

    cy.loginViaAPI('user@example.com', 'Password123');
    cy.visit('/orders');
    cy.wait('@getOrders');
  });

  it('displays active orders and cancels an order successfully', () => {
    cy.get('[data-testid="order-row"]').should('have.length', 2);
    cy.contains('[data-testid="order-row"]', 'ORD-99').within(() => {
      cy.get('button.cancel-btn').click();
    });

    // Confirm modal dialog
    cy.get('.modal-confirm').should('be.visible');
    cy.get('.modal-confirm button.confirm').click();

    cy.wait('@cancelOrder').its('request.url').should('include', 'ORD-99');
    cy.get('.toast-notification').should('contain.text', 'Order canceled');
  });
});
```

---

## 4. Cypress Component Testing (CT) vs. E2E

Cypress supports testing React components rendered directly into a component canvas without spinning up a full dev server backend:

```tsx
// src/components/Button.cy.tsx
import React from 'react';
import { Button } from './Button';

describe('<Button /> Component Test', () => {
  it('renders label and handles click event', () => {
    const onClickSpy = cy.spy().as('clickSpy');

    cy.mount(<Button label="Submit Form" onClick={onClickSpy} />);

    cy.get('button').should('have.text', 'Submit Form').click();
    cy.get('@clickSpy').should('have.been.calledOnce');
  });
});
```

---

## 5. Senior Frontend Engineer Interview Questions & Answers

### Q1: Why can't you mix standard JavaScript `async/await` with Cypress `cy` commands directly?
**Answer**:
Cypress commands (`cy.get()`, `cy.click()`) do **not** return native JavaScript Promises. Instead, they enqueue work into an internal asynchronous execution queue that runs sequentially after the spec file script finishes loading. Using native `async/await` breaks the execution order of Cypress's command queue. To access values, developers must chain commands using `.then(($el) => { ... })`.

### Q2: How does `cy.intercept()` differ from traditional HTTP server mocks?
**Answer**: `cy.intercept()` acts as a network proxy layer at the browser boundary. It can passively monitor real network traffic, stub responses with fixture data, modify request headers on the fly, or introduce simulated network latency without altering client code.

### Q3: How do you handle cross-origin navigation in Cypress?
**Answer**: Prior to v12, Cypress threw errors when navigating to a second domain (e.g., Auth0 or PayPal gateway). Modern Cypress handles multi-domain flows using `cy.origin('https://auth0.com', () => { ... })`, which runs context commands inside the target domain origin sandbox.
