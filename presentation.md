# Introduction to Testing Web Applications

---

## Slide 01 — Title

# Introduction to Testing Web Applications

**From Unit Tests to End-to-End — Building Confidence in Every Deploy**

Jest · Supertest · React Testing Library · Playwright · Cypress · TDD · CI/CD

---

## Slide 02 — Agenda

### Foundations
- Why test? Confidence & regression prevention
- The testing pyramid & trade-offs
- Unit testing with Jest
- Testing async code
- Mocking & spying strategies

### Integration & API
- Integration testing concepts
- API testing with Supertest
- Component testing with React Testing Library
- Snapshot testing techniques

### End-to-End
- E2E testing with Playwright
- E2E testing with Cypress
- Performance & load testing
- Test-Driven Development

### Production
- Code coverage & thresholds
- CI/CD integration
- Testing patterns & anti-patterns
- Summary & next steps

---

## Slide 03 — Why Test?

Testing is not optional overhead — it is a **core engineering practice** that pays dividends at every stage of the software lifecycle. Without tests, every deployment is a gamble.

Teams with comprehensive test suites deploy **more frequently**, recover **faster from failures**, and spend **less time debugging** production issues.

### The Cost of Bugs
- A bug caught in development costs **1x**
- A bug caught in QA costs **10x**
- A bug caught in production costs **100x**
- Automated tests catch bugs at **1x cost**

### Key Benefits
- **Confidence** — deploy knowing nothing is broken
- **Regression prevention** — bugs don't come back
- **Living documentation** — tests describe behaviour
- **Refactoring safety** — change code without fear
- **Faster onboarding** — new devs read tests to learn

### Testing is NOT
- A guarantee of zero bugs
- A replacement for code review
- Something only QA teams do
- Worth skipping to "save time"

---

## Slide 04 — The Testing Pyramid

The testing pyramid is a strategy for balancing **speed**, **cost**, and **confidence**. More tests at the base, fewer at the top.

```
         /  E2E  \          ~5-10%   — Slow, expensive, high confidence
        /----------\
       / Integration \      ~15-25%  — Moderate speed & cost
      /----------------\
     /   Unit Tests     \   ~65-80%  — Fast, cheap, many tests
    /____________________\
```

### Unit Tests
Test individual functions/classes in isolation. Run in milliseconds. Easiest to write and maintain. Mock external dependencies.

### Integration Tests
Test modules working together — API routes, database queries, middleware chains. Slower but catch wiring bugs unit tests miss.

### E2E Tests
Test the full application from a user's perspective — real browser, real network. Slowest and most brittle, but highest confidence.

---

## Slide 05 — Unit Testing with Jest

### describe / it / expect

```javascript
// math.js
function add(a, b) { return a + b; }
function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}
module.exports = { add, divide };

// math.test.js
const { add, divide } = require('./math');

describe('Math utilities', () => {
  describe('add', () => {
    it('adds two positive numbers', () => {
      expect(add(2, 3)).toBe(5);
    });

    it('handles negative numbers', () => {
      expect(add(-1, -2)).toBe(-3);
    });
  });

  describe('divide', () => {
    it('divides correctly', () => {
      expect(divide(10, 2)).toBe(5);
    });

    it('throws on division by zero', () => {
      expect(() => divide(10, 0))
        .toThrow('Division by zero');
    });
  });
});
```

### Common Matchers

| Matcher | Use Case |
|---------|----------|
| `toBe(val)` | Strict equality (`===`) |
| `toEqual(obj)` | Deep equality (objects/arrays) |
| `toBeTruthy()` | Truthy check |
| `toContain(item)` | Array/string includes |
| `toHaveLength(n)` | Array/string length |
| `toThrow(msg)` | Error thrown |
| `toMatch(/regex/)` | String regex match |
| `toHaveProperty(k)` | Object has key |
| `toBeNull()` | Strict null check |
| `toBeGreaterThan(n)` | Numeric comparison |

### Setup & Teardown

```javascript
beforeAll(() => { /* once before all tests */ });
afterAll(() =>  { /* once after all tests  */ });
beforeEach(() => { /* before each test */ });
afterEach(() =>  { /* after each test  */ });
```

---

## Slide 06 — Testing Async Code

### async/await (recommended)

```javascript
// userService.js
async function fetchUser(id) {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error('User not found');
  return res.json();
}

// userService.test.js
test('fetches user by ID', async () => {
  const user = await fetchUser(1);
  expect(user).toHaveProperty('name');
  expect(user.id).toBe(1);
});

test('throws for missing user', async () => {
  await expect(fetchUser(99999))
    .rejects.toThrow('User not found');
});
```

### Promise-based

```javascript
test('resolves user data', () => {
  return fetchUser(1).then(user => {
    expect(user.name).toBeDefined();
  });
});
```

### done callback (legacy)

```javascript
test('calls back with data', (done) => {
  fetchUserCallback(1, (err, user) => {
    try {
      expect(err).toBeNull();
      expect(user.name).toBeDefined();
      done();
    } catch (e) {
      done(e);
    }
  });
});
```

### Testing Timers

```javascript
jest.useFakeTimers();

test('debounce waits before calling', () => {
  const fn = jest.fn();
  const debounced = debounce(fn, 500);

  debounced();
  expect(fn).not.toHaveBeenCalled();

  jest.advanceTimersByTime(500);
  expect(fn).toHaveBeenCalledTimes(1);
});

afterEach(() => jest.useRealTimers());
```

### Common Pitfall
Forgetting to `return` or `await` a promise in a test — Jest won't wait for it and the test passes even if the assertion would fail.

---

## Slide 07 — Mocking & Spying

### jest.fn() — manual mock

```javascript
const callback = jest.fn();
callback('hello');

expect(callback).toHaveBeenCalledWith('hello');
expect(callback).toHaveBeenCalledTimes(1);

// Return values
const getPrice = jest.fn()
  .mockReturnValueOnce(9.99)
  .mockReturnValueOnce(19.99);

expect(getPrice()).toBe(9.99);
expect(getPrice()).toBe(19.99);
```

### jest.spyOn() — watch real methods

```javascript
const spy = jest.spyOn(console, 'log');

myFunction();

expect(spy).toHaveBeenCalledWith('Starting...');
spy.mockRestore(); // clean up
```

### jest.mock() — module mocks

```javascript
// Mock entire module
jest.mock('./database');
const db = require('./database');

db.findUser.mockResolvedValue({
  id: 1, name: 'Alice'
});

test('returns user from DB', async () => {
  const user = await getUser(1);
  expect(user.name).toBe('Alice');
  expect(db.findUser).toHaveBeenCalledWith(1);
});
```

### Manual Mock File

```javascript
// __mocks__/axios.js
module.exports = {
  get: jest.fn(() =>
    Promise.resolve({ data: {} })
  ),
  post: jest.fn(() =>
    Promise.resolve({ data: {} })
  ),
};
```

### Dependency Injection
Design functions to accept dependencies as parameters — makes mocking trivial without `jest.mock()`.

```javascript
// Easy to test: inject the DB
function getUser(db, id) {
  return db.findUser(id);
}
```

---

## Slide 08 — Integration Testing

Integration tests verify that **multiple modules work together correctly**. They test the wiring between components — database access, middleware chains, and service interactions.

### Database Integration Example

```javascript
const { Pool } = require('pg');
const { createUser, getUser } = require('./users');

let pool;

beforeAll(async () => {
  pool = new Pool({
    connectionString: process.env.TEST_DB_URL
  });
  await pool.query('BEGIN');
});

afterAll(async () => {
  await pool.query('ROLLBACK');
  await pool.end();
});

test('creates and retrieves a user', async () => {
  const created = await createUser(pool, {
    name: 'Alice', email: 'alice@test.com'
  });

  const found = await getUser(pool, created.id);

  expect(found.name).toBe('Alice');
  expect(found.email).toBe('alice@test.com');
});
```

### What to Integration Test
- API routes end-to-end (request → response)
- Database queries with real data
- Authentication & authorisation flows
- Middleware chains (validation, CORS, logging)
- Third-party service interactions

### Test Database Strategies
- **Transaction rollback** — wrap each test in a transaction
- **Docker containers** — spin up a fresh DB per run
- **In-memory DB** — SQLite for speed
- **Seeding** — load known fixtures before tests

### Unit vs Integration

| Unit | Integration |
|------|-------------|
| Single function | Multiple modules |
| Mocked deps | Real deps |
| Milliseconds | Seconds |
| Precise errors | Wiring errors |

---

## Slide 09 — API Testing with Supertest

### Basic HTTP Assertions

```javascript
const request = require('supertest');
const app = require('./app');

describe('GET /api/products', () => {
  test('returns 200 and JSON array', async () => {
    const res = await request(app)
      .get('/api/products')
      .expect('Content-Type', /json/)
      .expect(200);

    expect(res.body).toBeInstanceOf(Array);
    expect(res.body.length).toBeGreaterThan(0);
  });

  test('filters by category', async () => {
    const res = await request(app)
      .get('/api/products?category=electronics')
      .expect(200);

    res.body.forEach(product => {
      expect(product.category).toBe('electronics');
    });
  });
});
```

### POST with Auth Headers

```javascript
describe('POST /api/products', () => {
  const token = 'Bearer test-jwt-token';

  test('creates product with valid auth', async () => {
    const res = await request(app)
      .post('/api/products')
      .set('Authorization', token)
      .send({
        name: 'Widget',
        price: 29.99,
        category: 'electronics'
      })
      .expect(201);

    expect(res.body.id).toBeDefined();
    expect(res.body.name).toBe('Widget');
  });

  test('returns 401 without auth', async () => {
    await request(app)
      .post('/api/products')
      .send({ name: 'Widget' })
      .expect(401);
  });

  test('returns 400 for invalid body', async () => {
    await request(app)
      .post('/api/products')
      .set('Authorization', token)
      .send({ name: '' })  // validation fails
      .expect(400);
  });
});
```

### Tip
Export your Express `app` without calling `.listen()`. Supertest binds to an ephemeral port internally. This avoids port conflicts when running tests in parallel.

---

## Slide 10 — Component Testing with React Testing Library

### render, screen, queries

```javascript
import { render, screen } from '@testing-library/react';
import UserProfile from './UserProfile';

test('renders user name and email', () => {
  render(<UserProfile
    name="Alice"
    email="alice@example.com"
  />);

  expect(screen.getByText('Alice'))
    .toBeInTheDocument();
  expect(screen.getByText('alice@example.com'))
    .toBeInTheDocument();
});

test('shows loading state', () => {
  render(<UserProfile loading={true} />);

  expect(screen.getByRole('progressbar'))
    .toBeInTheDocument();
  expect(screen.queryByText('Alice'))
    .not.toBeInTheDocument();
});
```

### User Interaction with userEvent

```javascript
import userEvent from '@testing-library/user-event';
import LoginForm from './LoginForm';

test('submits login form', async () => {
  const onSubmit = jest.fn();
  const user = userEvent.setup();

  render(<LoginForm onSubmit={onSubmit} />);

  await user.type(
    screen.getByLabelText('Email'),
    'alice@test.com'
  );
  await user.type(
    screen.getByLabelText('Password'),
    'secret123'
  );
  await user.click(
    screen.getByRole('button', { name: /log in/i })
  );

  expect(onSubmit).toHaveBeenCalledWith({
    email: 'alice@test.com',
    password: 'secret123',
  });
});
```

### Query Priority
`getByRole` > `getByLabelText` > `getByText` > `getByTestId`. Prefer accessible queries — they test what users see.

### Async Queries
Use `findBy*` for elements that appear after async operations. Uses `waitFor` under the hood with a default timeout.

### Guiding Principle
"The more your tests resemble the way your software is used, the more confidence they can give you." — Kent C. Dodds

---

## Slide 11 — E2E Testing with Playwright

### Browser Automation

```javascript
// tests/login.spec.js
import { test, expect } from '@playwright/test';

test('user can log in', async ({ page }) => {
  await page.goto('/login');

  await page.getByLabel('Email')
    .fill('alice@test.com');
  await page.getByLabel('Password')
    .fill('secret123');
  await page.getByRole('button', { name: 'Log in' })
    .click();

  // Wait for navigation
  await expect(page).toHaveURL('/dashboard');
  await expect(
    page.getByText('Welcome, Alice')
  ).toBeVisible();
});

test('shows error for bad credentials', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('bad@test.com');
  await page.getByLabel('Password').fill('wrong');
  await page.getByRole('button', { name: 'Log in' })
    .click();

  await expect(
    page.getByText('Invalid credentials')
  ).toBeVisible();
});
```

### Fixtures & Page Objects

```javascript
// fixtures.js
import { test as base } from '@playwright/test';

class LoginPage {
  constructor(page) { this.page = page; }

  async login(email, password) {
    await this.page.goto('/login');
    await this.page.getByLabel('Email').fill(email);
    await this.page.getByLabel('Password')
      .fill(password);
    await this.page.getByRole('button',
      { name: 'Log in' }).click();
  }
}

export const test = base.extend({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  },
});

// Usage in tests
test('dashboard loads', async ({ loginPage, page }) => {
  await loginPage.login('alice@test.com', 'secret');
  await expect(page).toHaveURL('/dashboard');
});
```

### Key Features
- **Multi-browser** — Chromium, Firefox, WebKit
- **Auto-wait** — waits for elements to be actionable
- **Trace viewer** — step-by-step debugging
- **Parallel execution** — out of the box

---

## Slide 12 — E2E Testing with Cypress

### cy Commands

```javascript
// cypress/e2e/products.cy.js
describe('Product page', () => {
  beforeEach(() => {
    cy.visit('/products');
  });

  it('displays product list', () => {
    cy.get('[data-testid="product-card"]')
      .should('have.length.greaterThan', 0);
  });

  it('filters by search', () => {
    cy.get('[data-testid="search-input"]')
      .type('laptop');

    cy.get('[data-testid="product-card"]')
      .each(($card) => {
        cy.wrap($card)
          .should('contain.text', 'laptop');
      });
  });

  it('adds item to cart', () => {
    cy.get('[data-testid="add-to-cart"]')
      .first().click();
    cy.get('[data-testid="cart-count"]')
      .should('have.text', '1');
  });
});
```

### Network Intercepts

```javascript
it('handles API failure gracefully', () => {
  cy.intercept('GET', '/api/products', {
    statusCode: 500,
    body: { error: 'Server error' }
  }).as('getProducts');

  cy.visit('/products');
  cy.wait('@getProducts');

  cy.get('[data-testid="error-message"]')
    .should('contain', 'Something went wrong');
});
```

### Custom Commands

```javascript
// cypress/support/commands.js
Cypress.Commands.add('login', (email, pw) => {
  cy.session([email, pw], () => {
    cy.request('POST', '/api/auth/login', {
      email, password: pw
    }).then(({ body }) => {
      window.localStorage.setItem(
        'token', body.token
      );
    });
  });
});

// Usage
beforeEach(() => cy.login('alice@test.com', 'pw'));
```

### Playwright vs Cypress

| Feature | Playwright | Cypress |
|---------|-----------|---------|
| Browsers | All three | Chromium-focused |
| Language | JS/TS/Python/C# | JS/TS only |
| Tabs/windows | Yes | Limited |
| Dev UX | Good | Excellent |

---

## Slide 13 — Test-Driven Development

TDD is a development workflow where you write a **failing test first**, then write the **minimum code to pass**, then **refactor**. This cycle is called **Red-Green-Refactor**.

```
    RED               REFACTOR            GREEN
 Write failing  →   Clean up code   ←   Make it pass
    test              (tests stay         (simplest
                       green)             implementation)
```

### Practical TDD Workflow

```javascript
// 1. RED: write the test first
test('validates email format', () => {
  expect(isValidEmail('bad')).toBe(false);
  expect(isValidEmail('a@b.com')).toBe(true);
});

// 2. GREEN: simplest implementation
function isValidEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

// 3. REFACTOR: improve without breaking tests
```

### When TDD Shines
- Well-defined business logic & algorithms
- Bug fixes (write test that reproduces the bug)
- API design — tests clarify the interface
- Complex validation rules
- Data transformations & parsing

### When TDD is Harder
- Exploratory UI prototyping
- Rapidly changing requirements
- Heavy third-party integrations
- Visual/CSS-focused work

### TDD Benefits
- **Better design** — testable code is modular code
- **100% test coverage** from the start (for new code)
- **Instant feedback loop** — catch issues immediately
- **Confidence** — every line of code has a reason to exist

---

## Slide 14 — Code Coverage

### What Coverage Measures

| Metric | Meaning |
|--------|---------|
| **Statement** | % of statements executed |
| **Branch** | % of if/else paths taken |
| **Function** | % of functions called |
| **Line** | % of lines executed |

### Running Coverage

```bash
# Jest (built-in Istanbul)
npx jest --coverage

# Vitest
npx vitest --coverage

# c8 (native V8 coverage)
npx c8 node my-script.js
npx c8 report --reporter=html
```

### Sample Output

```text
------------|---------|----------|---------|
File        | % Stmts | % Branch | % Funcs |
------------|---------|----------|---------|
math.js     |   100   |   85.71  |   100   |
utils.js    |   92.3  |   75.00  |   100   |
auth.js     |   78.5  |   60.00  |   83.3  |
------------|---------|----------|---------|
All files   |   89.2  |   72.41  |   94.1  |
------------|---------|----------|---------|
```

### Configuring Thresholds

```javascript
// jest.config.js
module.exports = {
  coverageThreshold: {
    global: {
      statements: 80,
      branches: 75,
      functions: 80,
      lines: 80,
    },
    './src/critical/': {
      statements: 95,
      branches: 90,
    },
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx}',
    '!src/**/*.test.js',
    '!src/index.js',
  ],
};
```

### Coverage Pitfalls
- **100% coverage does not mean zero bugs** — it means every line ran, not that every edge case was verified
- Coverage without meaningful assertions is a **vanity metric**
- **Branch coverage** is more valuable than statement coverage
- Don't chase 100% — aim for **80-90%** with good assertions

---

## Slide 15 — Snapshot Testing

### How Snapshots Work

```javascript
import { render } from '@testing-library/react';
import UserCard from './UserCard';

test('renders correctly', () => {
  const { container } = render(
    <UserCard name="Alice" role="admin" />
  );

  // First run: creates .snap file
  // Next runs: compares against snapshot
  expect(container).toMatchSnapshot();
});

// To update snapshots after intentional changes:
// npx jest --updateSnapshot
// or press 'u' in watch mode
```

### Inline Snapshots

```javascript
test('formats currency', () => {
  expect(formatCurrency(1299))
    .toMatchInlineSnapshot(`"$12.99"`);

  expect(formatCurrency(0))
    .toMatchInlineSnapshot(`"$0.00"`);
});
// Jest writes the expected value into the source
```

### When to Use Snapshots

| Good For | Bad For |
|----------|---------|
| Serialisable output (JSON, HTML) | Frequently changing UI |
| API response shapes | Large DOM trees |
| Config objects | Replacing all other assertions |
| Error messages | Dynamic data (dates, IDs) |

### Snapshot Anti-Patterns
- **Blindly updating** snapshots without reviewing the diff
- **Huge snapshots** (1000+ lines) — nobody reviews them
- Snapshotting **entire pages** instead of specific components
- Using snapshots as a **lazy substitute** for explicit assertions

### Property Matchers

```javascript
test('user object shape', () => {
  expect(createUser('Alice')).toMatchSnapshot({
    id: expect.any(String),
    createdAt: expect.any(Date),
    // name: 'Alice' checked via snapshot
  });
});
```

---

## Slide 16 — Performance & Load Testing

### k6 — Load Testing

```javascript
// load-test.js (run with: k6 run load-test.js)
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 50 },  // ramp up
    { duration: '1m',  target: 50 },  // sustain
    { duration: '10s', target: 0 },   // ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% < 500ms
    http_req_failed: ['rate<0.01'],   // <1% errors
  },
};

export default function () {
  const res = http.get('http://localhost:3000/api');
  check(res, {
    'status 200': (r) => r.status === 200,
    'body has data': (r) => r.json().data !== null,
  });
  sleep(1);
}
```

### Artillery — API Load Testing

```yaml
# artillery-config.yml
config:
  target: "http://localhost:3000"
  phases:
    - duration: 60
      arrivalRate: 20
      name: "Sustained load"

scenarios:
  - name: "Browse products"
    flow:
      - get:
          url: "/api/products"
          expect:
            - statusCode: 200
      - think: 2
      - get:
          url: "/api/products/1"
```

### Lighthouse CI

```bash
# Install and run
npm i -g @lhci/cli
lhci autorun --config=lighthouserc.json

# Assert performance budgets
# performance >= 90, accessibility >= 95
```

### Core Web Vitals
- **LCP** — Largest Contentful Paint < 2.5s
- **INP** — Interaction to Next Paint < 200ms
- **CLS** — Cumulative Layout Shift < 0.1

---

## Slide 17 — CI/CD Integration

### GitHub Actions Example

```yaml
# .github/workflows/test.yml
name: Test Suite
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: test
          POSTGRES_PASSWORD: test
        ports: ['5432:5432']

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - run: npm ci
      - run: npm run lint
      - run: npm test -- --coverage
      - run: npx playwright install --with-deps
      - run: npm run test:e2e

      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: test-results
          path: test-results/
```

### Parallel Execution

```yaml
# Split tests across shards
jobs:
  test:
    strategy:
      matrix:
        shard: [1/4, 2/4, 3/4, 4/4]
    steps:
      - run: npx jest --shard=${{ matrix.shard }}

  e2e:
    strategy:
      matrix:
        shard: [1/3, 2/3, 3/3]
    steps:
      - run: npx playwright test --shard=${{ matrix.shard }}
```

### Test Reporting
- **JUnit XML** — universal CI format
- **jest-junit** — Jest reporter for CI
- **Coverage badges** — Codecov, Coveralls
- **Playwright HTML report** — artifact upload

### Best Practices
- Run **fast tests first** (lint → unit → integration → E2E)
- **Fail fast** — stop pipeline on first failure
- Cache `node_modules` and Playwright browsers
- Use **matrix builds** for multi-version testing
- Require **passing tests** for PR merge

---

## Slide 18 — Testing Patterns & Anti-Patterns

### Patterns (Do This)

#### Arrange-Act-Assert (AAA)

```javascript
test('calculates order total', () => {
  // Arrange
  const items = [
    { price: 10, qty: 2 },
    { price: 5,  qty: 3 },
  ];

  // Act
  const total = calculateTotal(items);

  // Assert
  expect(total).toBe(35);
});
```

#### Test Isolation
- Each test is independent — no shared state
- Tests can run in **any order**
- Use `beforeEach` to reset, not `beforeAll`
- Clean up side effects in `afterEach`

#### Descriptive Test Names

```javascript
// Bad
test('works', () => { ... });

// Good
test('returns 404 when user ID does not exist', () => { ... });
```

### Anti-Patterns (Avoid This)

#### Testing Implementation Details

```javascript
// BAD: tests internal state
test('sets isLoading to true', () => {
  const wrapper = shallow(<App />);
  wrapper.instance().fetchData();
  expect(wrapper.state('isLoading')).toBe(true);
});

// GOOD: tests observable behaviour
test('shows loading spinner', async () => {
  render(<App />);
  expect(screen.getByRole('progressbar'))
    .toBeInTheDocument();
});
```

#### Flaky Tests
- Tests that **sometimes pass, sometimes fail**
- Causes: race conditions, shared state, time-dependent logic, network calls
- Fix: mock time, isolate state, use `waitFor`, avoid `sleep`

#### More Anti-Patterns
- **No assertions** — test passes but verifies nothing
- **Multiple concerns** — one test testing five things
- **Conditional logic** in tests (`if`/`switch`)
- **Ignoring failing tests** with `.skip`

---

## Slide 19 — Summary & Next Steps

### Key Takeaways
- The testing pyramid: **many unit**, some integration, **few E2E**
- **Jest** for unit tests — matchers, mocks, snapshots
- **Supertest** for API integration testing
- **React Testing Library** — test behaviour, not implementation
- **Playwright** & **Cypress** for end-to-end browser testing
- **TDD** — Red-Green-Refactor drives better design
- Code coverage is a guide, not a goal — **80-90%** is healthy
- CI/CD integration makes testing automatic and reliable
- Arrange-Act-Assert, test isolation, no flaky tests

### Recommended Reading
- **Testing JavaScript** — Kent C. Dodds (testingjavascript.com)
- **Jest Documentation** — jestjs.io
- **Playwright Documentation** — playwright.dev
- **Testing Library Docs** — testing-library.com

### Try These
- Add Jest unit tests to an existing project
- Write Supertest integration tests for your API
- Set up Playwright E2E tests with CI
- Practice TDD on a small utility library
- Configure coverage thresholds in your CI pipeline
- Convert a flaky test into a reliable one

**Thank you!** — Built with Reveal.js
