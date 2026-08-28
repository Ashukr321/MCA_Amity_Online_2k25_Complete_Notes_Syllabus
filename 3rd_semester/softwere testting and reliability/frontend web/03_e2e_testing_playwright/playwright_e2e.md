# Module 03: End-to-End (E2E) Testing with Playwright

## 1. Architecture & Core Capabilities

Playwright is Microsoft's modern, open-source automation framework designed for fast, reliable, and headless/headed End-to-End browser testing across Chromium, Firefox, and WebKit engines.

### Key Architectural Advantages
- **Out-of-Process Automation**: Communicates directly with browser debugging protocols (CDP for Chrome, Firefox Inspector, WebKit Protocol) over a single WebSocket connection without WebDriver HTTP polling latency.
- **Auto-Waiting**: Automatically waits for elements to meet actionability criteria (visible, enabled, stable position, receiving events) before performing actions.
- **True Multi-Browser & Multi-Tab Support**: Native execution across Chrome, Edge, Firefox, Safari (WebKit), and mobile device viewports (iPhone, Pixel).
- **Isolated Browser Contexts**: Each test runs in a pristine `BrowserContext` (equivalent to an isolated incognito profile) creating near-zero creation overhead (< 10ms per test).

---

## 2. Production Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 4 : undefined,
  reporter: [
    ['html', { open: 'never' }],
    ['list'],
    ['json', { outputFile: 'playwright-results.json' }],
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    actionTimeout: 10000,
    navigationTimeout: 15000,
  },
  projects: [
    // Setup Project: Authenticate once before test suite
    { name: 'setup', testMatch: /.*\.setup\.ts/ },
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
        // Reuse authenticated state from setup project
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
    {
      name: 'mobile-safari',
      use: {
        ...devices['iPhone 14'],
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
});
```

---

## 3. High-Speed Auth Reuse (`auth.setup.ts`)

Bypass UI login forms in 99% of tests by persisting browser cookies & localStorage state once:

```typescript
// e2e/auth.setup.ts
import { test as setup, expect } from '@playwright/test';

const authFile = 'playwright/.auth/user.json';

setup('authenticate user via API/UI', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel(/email/i).fill('admin@company.com');
  await page.getByLabel(/password/i).fill('SuperSecret123!');
  await page.getByRole('button', { name: /sign in/i }).click();

  await page.waitForURL('/dashboard');
  await expect(page.getByRole('heading', { name: /dashboard/i })).toBeVisible();

  // Save auth storage state to file
  await page.context().storageState({ path: authFile });
});
```

---

## 4. Page Object Model (POM) Design Pattern

The Page Object Model encapsulates page locators and interaction logic away from individual test assertions to enhance maintainability.

```typescript
// e2e/pages/CheckoutPage.ts
import { Page, Locator, expect } from '@playwright/test';

export class CheckoutPage {
  readonly page: Page;
  readonly cartItemCount: Locator;
  readonly promoInput: Locator;
  readonly applyPromoButton: Locator;
  readonly checkoutButton: Locator;
  readonly successMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.cartItemCount = page.getByTestId('cart-count');
    this.promoInput = page.getByPlaceholder(/promo code/i);
    this.applyPromoButton = page.getByRole('button', { name: /apply/i });
    this.checkoutButton = page.getByRole('button', { name: /proceed to payment/i });
    this.successMessage = page.getByRole('alert');
  }

  async goto() {
    await this.page.goto('/checkout');
  }

  async applyDiscount(code: string) {
    await this.promoInput.fill(code);
    await this.applyPromoButton.click();
  }

  async submitOrder() {
    await this.checkoutButton.click();
  }
}
```

```typescript
// e2e/specs/checkout.spec.ts
import { test, expect } from '@playwright/test';
import { CheckoutPage } from '../pages/CheckoutPage';

test.describe('E2E Checkout Flow', () => {
  test('should apply promo code and complete purchase', async ({ page }) => {
    const checkout = new CheckoutPage(page);
    await checkout.goto();

    await checkout.applyDiscount('SAVE20');
    await expect(page.getByText('20% Discount Applied')).toBeVisible();

    await checkout.submitOrder();
    await expect(checkout.successMessage).toHaveText(/Order #.* placed successfully/);
  });
});
```

---

## 5. Network Interception & Route Mocking

```typescript
test('simulates 500 internal server error on checkout API', async ({ page }) => {
  // Mock API network route before triggering UI action
  await page.route('/api/checkout', async (route) => {
    await route.fulfill({
      status: 500,
      contentType: 'application/json',
      body: JSON.stringify({ error: 'Payment service gateway unreachable' }),
    });
  });

  await page.goto('/checkout');
  await page.getByRole('button', { name: /proceed to payment/i }).click();

  await expect(page.getByRole('alert')).toHaveText(/Payment service gateway unreachable/);
});
```

---

## 6. Senior Frontend Engineer Interview Questions & Answers

### Q1: What makes Playwright immune to arbitrary sleep (`page.waitForTimeout`) calls?
**Answer**: Playwright uses an internal **Actionability Engine** before every action (click, fill, hover). It automatically waits for elements to be attached to the DOM, visible, stable in bounding box, enabled, and not covered by overlay elements. Explicit hardcoded timeouts like `setTimeout` or `waitForTimeout` are considered anti-patterns.

### Q2: How does Playwright execute tests in parallel across large suites without resource contention?
**Answer**:
1. **Context Isolation**: Each test runs in its own lightweight `BrowserContext` inside worker threads.
2. **CI Sharding**: Playwright supports CLI test sharding out of the box (`npx playwright test --shard=1/4`, `--shard=2/4`), enabling 4 parallel CI instances to run 25% of the test suite each simultaneously.

### Q3: How do you capture visual regression snapshots in Playwright?
**Answer**: Using `await expect(page).toHaveScreenshot('dashboard-page.png', { maxDiffPixelRatio: 0.02 });`. Playwright renders pixels, compares them against baseline snapshots stored in git, and highlights visual diffs upon mismatch.
