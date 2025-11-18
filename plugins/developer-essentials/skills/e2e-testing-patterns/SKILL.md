---
name: e2e-testing-patterns
description: 精通使用 Playwright 和 Cypress 的端對端測試，建立可靠的測試套件，以捕捉錯誤、提高信心並實現快速部署。適用於實作 E2E 測試、除錯不穩定測試或建立測試標準。
---

# E2E 測試模式

建立可靠、快速且可維護的端對端測試套件，提供快速交付程式碼的信心，並在使用者發現之前捕捉回歸問題。

## 何時使用此技能

- 實作端對端測試自動化
- 除錯不穩定或不可靠的測試
- 測試關鍵使用者工作流程
- 設定 CI/CD 測試管線
- 跨多個瀏覽器測試
- 驗證無障礙需求
- 測試響應式設計
- 建立 E2E 測試標準

## 核心概念

### 1. E2E 測試基礎

**使用 E2E 測試的項目：**
- 關鍵使用者旅程（登入、結帳、註冊）
- 複雜互動（拖放、多步驟表單）
- 跨瀏覽器相容性
- 真實 API 整合
- 身份驗證流程

**不使用 E2E 測試的項目：**
- 單元級別邏輯（使用單元測試）
- API 契約（使用整合測試）
- 邊緣案例（太慢）
- 內部實作細節

### 2. 測試哲學

**測試金字塔：**
```
        /\
       /E2E\         ← 少量，專注於關鍵路徑
      /─────\
     /Integr\        ← 更多，測試元件互動
    /────────\
   /Unit Tests\      ← 許多，快速，隔離
  /────────────\
```

**最佳實務：**
- 測試使用者行為，而非實作
- 保持測試獨立
- 使測試確定性
- 優化速度
- 使用 data-testid，而非 CSS 選擇器

## Playwright 模式

### 設定和配置

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
    testDir: './e2e',
    timeout: 30000,
    expect: {
        timeout: 5000,
    },
    fullyParallel: true,
    forbidOnly: !!process.env.CI,
    retries: process.env.CI ? 2 : 0,
    workers: process.env.CI ? 1 : undefined,
    reporter: [
        ['html'],
        ['junit', { outputFile: 'results.xml' }],
    ],
    use: {
        baseURL: 'http://localhost:3000',
        trace: 'on-first-retry',
        screenshot: 'only-on-failure',
        video: 'retain-on-failure',
    },
    projects: [
        { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
        { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
        { name: 'webkit', use: { ...devices['Desktop Safari'] } },
        { name: 'mobile', use: { ...devices['iPhone 13'] } },
    ],
});
```

### 模式 1：頁面物件模型

```typescript
// pages/LoginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
    readonly page: Page;
    readonly emailInput: Locator;
    readonly passwordInput: Locator;
    readonly loginButton: Locator;
    readonly errorMessage: Locator;

    constructor(page: Page) {
        this.page = page;
        this.emailInput = page.getByLabel('Email');
        this.passwordInput = page.getByLabel('Password');
        this.loginButton = page.getByRole('button', { name: 'Login' });
        this.errorMessage = page.getByRole('alert');
    }

    async goto() {
        await this.page.goto('/login');
    }

    async login(email: string, password: string) {
        await this.emailInput.fill(email);
        await this.passwordInput.fill(password);
        await this.loginButton.click();
    }

    async getErrorMessage(): Promise<string> {
        return await this.errorMessage.textContent() ?? '';
    }
}

// Test using Page Object
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('successful login', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login('user@example.com', 'password123');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByRole('heading', { name: 'Dashboard' }))
        .toBeVisible();
});

test('failed login shows error', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login('invalid@example.com', 'wrong');

    const error = await loginPage.getErrorMessage();
    expect(error).toContain('Invalid credentials');
});
```

### 模式 2：測試資料的 Fixture

```typescript
// fixtures/test-data.ts
import { test as base } from '@playwright/test';

type TestData = {
    testUser: {
        email: string;
        password: string;
        name: string;
    };
    adminUser: {
        email: string;
        password: string;
    };
};

export const test = base.extend<TestData>({
    testUser: async ({}, use) => {
        const user = {
            email: `test-${Date.now()}@example.com`,
            password: 'Test123!@#',
            name: 'Test User',
        };
        // Setup: Create user in database
        await createTestUser(user);
        await use(user);
        // Teardown: Clean up user
        await deleteTestUser(user.email);
    },

    adminUser: async ({}, use) => {
        await use({
            email: 'admin@example.com',
            password: process.env.ADMIN_PASSWORD!,
        });
    },
});

// Usage in tests
import { test } from './fixtures/test-data';

test('user can update profile', async ({ page, testUser }) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill(testUser.email);
    await page.getByLabel('Password').fill(testUser.password);
    await page.getByRole('button', { name: 'Login' }).click();

    await page.goto('/profile');
    await page.getByLabel('Name').fill('Updated Name');
    await page.getByRole('button', { name: 'Save' }).click();

    await expect(page.getByText('Profile updated')).toBeVisible();
});
```

### 模式 3：等待策略

```typescript
// ❌ 不好：固定逾時
await page.waitForTimeout(3000);  // Flaky!

// ✅ 好：等待特定條件
await page.waitForLoadState('networkidle');
await page.waitForURL('/dashboard');
await page.waitForSelector('[data-testid="user-profile"]');

// ✅ 更好：使用斷言的自動等待
await expect(page.getByText('Welcome')).toBeVisible();
await expect(page.getByRole('button', { name: 'Submit' }))
    .toBeEnabled();

// Wait for API response
const responsePromise = page.waitForResponse(
    response => response.url().includes('/api/users') && response.status() === 200
);
await page.getByRole('button', { name: 'Load Users' }).click();
const response = await responsePromise;
const data = await response.json();
expect(data.users).toHaveLength(10);

// Wait for multiple conditions
await Promise.all([
    page.waitForURL('/success'),
    page.waitForLoadState('networkidle'),
    expect(page.getByText('Payment successful')).toBeVisible(),
]);
```

### 模式 4：網路 Mock 和攔截

```typescript
// Mock API responses
test('displays error when API fails', async ({ page }) => {
    await page.route('**/api/users', route => {
        route.fulfill({
            status: 500,
            contentType: 'application/json',
            body: JSON.stringify({ error: 'Internal Server Error' }),
        });
    });

    await page.goto('/users');
    await expect(page.getByText('Failed to load users')).toBeVisible();
});

// Intercept and modify requests
test('can modify API request', async ({ page }) => {
    await page.route('**/api/users', async route => {
        const request = route.request();
        const postData = JSON.parse(request.postData() || '{}');

        // Modify request
        postData.role = 'admin';

        await route.continue({
            postData: JSON.stringify(postData),
        });
    });

    // Test continues...
});

// Mock third-party services
test('payment flow with mocked Stripe', async ({ page }) => {
    await page.route('**/api/stripe/**', route => {
        route.fulfill({
            status: 200,
            body: JSON.stringify({
                id: 'mock_payment_id',
                status: 'succeeded',
            }),
        });
    });

    // Test payment flow with mocked response
});
```

## Cypress 模式

### 設定和配置

```typescript
// cypress.config.ts
import { defineConfig } from 'cypress';

export default defineConfig({
    e2e: {
        baseUrl: 'http://localhost:3000',
        viewportWidth: 1280,
        viewportHeight: 720,
        video: false,
        screenshotOnRunFailure: true,
        defaultCommandTimeout: 10000,
        requestTimeout: 10000,
        setupNodeEvents(on, config) {
            // Implement node event listeners
        },
    },
});
```

### 模式 1：自訂命令

```typescript
// cypress/support/commands.ts
declare global {
    namespace Cypress {
        interface Chainable {
            login(email: string, password: string): Chainable<void>;
            createUser(userData: UserData): Chainable<User>;
            dataCy(value: string): Chainable<JQuery<HTMLElement>>;
        }
    }
}

Cypress.Commands.add('login', (email: string, password: string) => {
    cy.visit('/login');
    cy.get('[data-testid="email"]').type(email);
    cy.get('[data-testid="password"]').type(password);
    cy.get('[data-testid="login-button"]').click();
    cy.url().should('include', '/dashboard');
});

Cypress.Commands.add('createUser', (userData: UserData) => {
    return cy.request('POST', '/api/users', userData)
        .its('body');
});

Cypress.Commands.add('dataCy', (value: string) => {
    return cy.get(`[data-cy="${value}"]`);
});

// Usage
cy.login('user@example.com', 'password');
cy.dataCy('submit-button').click();
```

### 模式 2：Cypress 攔截

```typescript
// Mock API calls
cy.intercept('GET', '/api/users', {
    statusCode: 200,
    body: [
        { id: 1, name: 'John' },
        { id: 2, name: 'Jane' },
    ],
}).as('getUsers');

cy.visit('/users');
cy.wait('@getUsers');
cy.get('[data-testid="user-list"]').children().should('have.length', 2);

// Modify responses
cy.intercept('GET', '/api/users', (req) => {
    req.reply((res) => {
        // Modify response
        res.body.users = res.body.users.slice(0, 5);
        res.send();
    });
});

// Simulate slow network
cy.intercept('GET', '/api/data', (req) => {
    req.reply((res) => {
        res.delay(3000);  // 3 second delay
        res.send();
    });
});
```

## 進階模式

### 模式 1：視覺回歸測試

```typescript
// With Playwright
import { test, expect } from '@playwright/test';

test('homepage looks correct', async ({ page }) => {
    await page.goto('/');
    await expect(page).toHaveScreenshot('homepage.png', {
        fullPage: true,
        maxDiffPixels: 100,
    });
});

test('button in all states', async ({ page }) => {
    await page.goto('/components');

    const button = page.getByRole('button', { name: 'Submit' });

    // Default state
    await expect(button).toHaveScreenshot('button-default.png');

    // Hover state
    await button.hover();
    await expect(button).toHaveScreenshot('button-hover.png');

    // Disabled state
    await button.evaluate(el => el.setAttribute('disabled', 'true'));
    await expect(button).toHaveScreenshot('button-disabled.png');
});
```

### 模式 2：使用分片的並行測試

```typescript
// playwright.config.ts
export default defineConfig({
    projects: [
        {
            name: 'shard-1',
            use: { ...devices['Desktop Chrome'] },
            grepInvert: /@slow/,
            shard: { current: 1, total: 4 },
        },
        {
            name: 'shard-2',
            use: { ...devices['Desktop Chrome'] },
            shard: { current: 2, total: 4 },
        },
        // ... more shards
    ],
});

// Run in CI
// npx playwright test --shard=1/4
// npx playwright test --shard=2/4
```

### 模式 3：無障礙測試

```typescript
// Install: npm install @axe-core/playwright
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('page should not have accessibility violations', async ({ page }) => {
    await page.goto('/');

    const accessibilityScanResults = await new AxeBuilder({ page })
        .exclude('#third-party-widget')
        .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
});

test('form is accessible', async ({ page }) => {
    await page.goto('/signup');

    const results = await new AxeBuilder({ page })
        .include('form')
        .analyze();

    expect(results.violations).toEqual([]);
});
```

## 最佳實務

1. **使用資料屬性**：`data-testid` 或 `data-cy` 以獲得穩定的選擇器
2. **避免脆弱的選擇器**：不要依賴 CSS 類別或 DOM 結構
3. **測試使用者行為**：點擊、輸入、查看 - 而非實作細節
4. **保持測試獨立**：每個測試應該能獨立執行
5. **清理測試資料**：在每個測試中建立和銷毀測試資料
6. **使用頁面物件**：封裝頁面邏輯
7. **有意義的斷言**：檢查實際使用者可見的行為
8. **優化速度**：盡可能 mock，並行執行

```typescript
// ❌ 不好的選擇器
cy.get('.btn.btn-primary.submit-button').click();
cy.get('div > form > div:nth-child(2) > input').type('text');

// ✅ 好的選擇器
cy.getByRole('button', { name: 'Submit' }).click();
cy.getByLabel('Email address').type('user@example.com');
cy.get('[data-testid="email-input"]').type('user@example.com');
```

## 常見陷阱

- **不穩定的測試**：使用適當的等待，而非固定逾時
- **測試緩慢**：Mock 外部 API，使用並行執行
- **過度測試**：不要用 E2E 測試每個邊緣案例
- **耦合測試**：測試不應相互依賴
- **糟糕的選擇器**：避免 CSS 類別和 nth-child
- **沒有清理**：每個測試後清理測試資料
- **測試實作**：測試使用者行為，而非內部

## 除錯失敗的測試

```typescript
// Playwright debugging
// 1. Run in headed mode
npx playwright test --headed

// 2. Run in debug mode
npx playwright test --debug

// 3. Use trace viewer
await page.screenshot({ path: 'screenshot.png' });
await page.video()?.saveAs('video.webm');

// 4. Add test.step for better reporting
test('checkout flow', async ({ page }) => {
    await test.step('Add item to cart', async () => {
        await page.goto('/products');
        await page.getByRole('button', { name: 'Add to Cart' }).click();
    });

    await test.step('Proceed to checkout', async () => {
        await page.goto('/cart');
        await page.getByRole('button', { name: 'Checkout' }).click();
    });
});

// 5. Inspect page state
await page.pause();  // Pauses execution, opens inspector
```

## 資源

- **references/playwright-best-practices.md**：Playwright 特定模式
- **references/cypress-best-practices.md**：Cypress 特定模式
- **references/flaky-test-debugging.md**：除錯不可靠的測試
- **assets/e2e-testing-checklist.md**：使用 E2E 測試的項目
- **assets/selector-strategies.md**：尋找可靠的選擇器
- **scripts/test-analyzer.ts**：分析測試不穩定性和持續時間
