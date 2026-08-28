# Module 07: Visual Regression Testing with Chromatic & Storybook

## 1. Executive Summary & Core Concepts

Visual Regression Testing captures automated visual snapshots of UI components or entire web pages to compare them pixel-by-pixel against approved baseline snapshots. It catches unintended layout breaks, CSS regressions, font rendering bugs, and z-index issues that standard functional DOM tests miss.

### Image Snapshot vs DOM Snapshot (Chromatic Model)

```
┌─────────────────────────────────────────────────────────────┐
│                    Chromatic Architecture                   │
│  ┌───────────────────────────┐   ┌───────────────────────┐  │
│  │   Storybook Test Suite    │   │  DOM Snapshot Engine  │  │
│  │  (Captures HTML + CSS)    │───▶   (Uploads DOM to     │  │
│  └───────────────────────────┘   │    Chromatic Cloud)   │  │
│                                  └───────────┬───────────┘  │
│                                              │              │
│                                  Render Cloud Browsers      │
│                                  (Chrome, Firefox, Safari)  │
│                                              │              │
│                                  ┌───────────▼──────────┐  │
│                                  │ Pixel Diff Engine    │  │
│                                  │ Baseline vs Candidate│  │
│                                  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

1. **Pixel-by-Pixel Diffing**: Takes PNG screenshots of both baseline and candidate UI state, highlighting changed pixels in magenta.
2. **DOM Snapshotting (Chromatic approach)**: Captures the serialized DOM structure, CSS styles, and assets in Storybook, sending them to cloud browsers to render clean pixel-perfect snapshots across multiple viewports and browsers in parallel.

---

## 2. Writing Stories with Storybook (CSF 3.0) & Interaction Play Functions

```tsx
// src/components/Badge.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { userEvent, within, expect } from '@storybook/test';
import { Badge } from './Badge';

const meta: Meta<typeof Badge> = {
  title: 'Components/Badge',
  component: Badge,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['success', 'warning', 'danger', 'info'],
    },
  },
};

export default meta;
type Story = StoryObj<typeof Badge>;

export const SuccessVariant: Story = {
  args: {
    label: 'Active Subscription',
    variant: 'success',
  },
};

export const InteractiveTooltipBadge: Story = {
  args: {
    label: 'Hover Me',
    variant: 'info',
    tooltip: 'Detailed metric information',
  },
  // Play function executes real user interaction before capturing visual snapshot
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const badge = canvas.getByText('Hover Me');
    await userEvent.hover(badge);
    await expect(canvas.getByRole('tooltip')).toBeVisible();
  },
};
```

---

## 3. Chromatic Cloud Integration & CI Pipeline

Run Chromatic directly inside GitHub Actions on every Pull Request:

```yaml
# .github/workflows/chromatic.yml
name: 'Chromatic Visual Regression'

on: push

jobs:
  chromatic:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Full git history required for baseline comparison

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Publish to Chromatic
        uses: chromaui/action@v1
        with:
          projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          token: ${{ secrets.GITHUB_TOKEN }}
          onlyChanged: true # Smart component optimization
```

---

## 4. Playwright Visual Snapshot Testing

For full-page visual regression without Storybook:

```typescript
// e2e/specs/visual-dashboard.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Visual Snapshot Regression', () => {
  test('matches dashboard visual layout baseline', async ({ page }) => {
    await page.goto('/dashboard');

    // Freeze dynamic content (e.g. current date or clock)
    await page.evaluate(() => {
      const dateEl = document.querySelector('[data-testid="current-date"]');
      if (dateEl) dateEl.textContent = '2026-01-01';
    });

    // Take visual snapshot of specific element or full page
    await expect(page).toHaveScreenshot('dashboard-baseline.png', {
      maxDiffPixelRatio: 0.01, // Allow 1% pixel threshold for anti-aliasing
      fullPage: true,
      mask: [page.getByTestId('live-chart')], // Mask unpredictable dynamic elements
    });
  });
});
```

---

## 5. Preventing Flakiness in Visual Tests

### Common Sources of Visual Flakiness & Solutions
1. **Font Loading Delays**: Ensure custom web fonts (`WOFF2`) are fully loaded before capturing snapshots (`await document.fonts.ready`).
2. **CSS Animations & Transitions**: Disable all CSS animations automatically during testing:
   ```css
   /* Global test CSS override */
   *, *::before, *::after {
     animation-duration: 0s !important;
     transition-duration: 0s !important;
   }
   ```
3. **Caret & Cursor Blinking**: Hide blinking text cursors in inputs using CSS (`caret-color: transparent`).

---

## 6. Senior Frontend Engineer Interview Questions & Answers

### Q1: How does Chromatic's DOM snapshot approach differ from Playwright's canvas screenshot approach?
**Answer**:
Playwright renders the page locally and captures a rasterized PNG image. Differences in local OS font rendering (macOS vs Linux in CI) can cause false visual diffs.
Chromatic captures the DOM, CSS rules, and static assets into an archive format and uploads it to Chromatic's cloud infrastructure, where standardized headless browsers render snapshots under identical GPU/font environments, eliminating cross-OS rendering flakiness.

### Q2: What strategies do you use to manage Chromatic snapshot build costs for large design systems (e.g., 500+ component stories)?
**Answer**:
1. Enable `onlyChanged: true` in the Chromatic GitHub Action so only stories affected by git commit diffs are re-captured.
2. Group variant matrix stories using Storybook decorators instead of separate stories where appropriate.
3. Skip snapshotting purely structural layout stories using `parameters: { chromatic: { disableSnapshot: true } }`.
