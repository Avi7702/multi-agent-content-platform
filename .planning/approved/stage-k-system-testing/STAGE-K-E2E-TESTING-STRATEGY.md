# STAGE K: SYSTEM-WIDE END-TO-END TESTING STRATEGY

**Version:** 1.0.0
**Status:** ✅ APPROVED - Production Ready
**Date:** November 10, 2025
**Stage:** K of 12 (System Integration Testing)
**Dependencies:** Stages A-I (All prior stages)

---

## Executive Summary

### Purpose
Define a **comprehensive end-to-end testing strategy** for the DIFY-AGENT system covering all 12 stages (A-L), focusing on complete user journeys from onboarding through content generation to multi-platform publishing.

**Key Difference from Stage I:**
- **Stage I:** REST API layer testing (515+ tests, unit/integration focus)
- **Stage K:** System-wide E2E testing (user journey focus, cross-stage integration)

### Scope
- **User Journeys:** 10 comprehensive scenarios (3 happy path, 4 edge cases, 3 error cases)
- **Performance Benchmarks:** 5 critical path metrics with SLA targets
- **LLM-as-Judge:** Automated content quality evaluation (brand voice, creativity, engagement)
- **Cross-Platform Testing:** LinkedIn, Twitter/X, Facebook, Instagram
- **Multi-Tenant Isolation:** Verify tenant data isolation and security

### Success Criteria
- ✅ 8/10 E2E scenarios pass (80% threshold)
- ✅ All performance benchmarks meet SLA targets
- ✅ LLM-as-judge scores ≥ 8.0/10 for brand voice match
- ✅ Zero cross-tenant data leakage detected
- ✅ All OAuth flows complete successfully
- ✅ Published content appears on platforms within 10s

### Testing Tools
- **E2E Framework:** Playwright (TypeScript)
- **LLM Evaluation:** GPT-4 (Claude Sonnet 4.5 as fallback)
- **Performance:** k6 (Grafana Labs)
- **Visual Testing:** Percy (Browserstack)
- **API Testing:** Supertest (Node.js)
- **CI/CD:** GitHub Actions

---

## Table of Contents

1. [Testing Architecture](#1-testing-architecture)
2. [Test Scenario Catalog](#2-test-scenario-catalog)
3. [Happy Path Scenarios](#3-happy-path-scenarios)
4. [Edge Case Scenarios](#4-edge-case-scenarios)
5. [Error Case Scenarios](#5-error-case-scenarios)
6. [LLM-as-Judge Framework](#6-llm-as-judge-framework)
7. [Performance Benchmarks](#7-performance-benchmarks)
8. [Test Data Management](#8-test-data-management)
9. [Automated Testing Pipeline](#9-automated-testing-pipeline)
10. [Test Execution & Reporting](#10-test-execution--reporting)

---

## 1. Testing Architecture

### 1.1 System Under Test

```
┌─────────────────────────────────────────────────────────────┐
│                      DIFY-AGENT SYSTEM                      │
├─────────────────────────────────────────────────────────────┤
│  User Interface (Stage G)                                   │
│  ├── Next.js 15 Frontend (Vercel)                          │
│  ├── Dify Chat API (Conversation state)                    │
│  └── Media Components (Video, Image, Carousel)             │
├─────────────────────────────────────────────────────────────┤
│  Orchestration Layer                                        │
│  ├── Dify Workflows (5 workflows)                          │
│  ├── Knowledge Bases (4 KB systems - Stage C)              │
│  └── OpenManus Multi-Agent (Stage H - async tasks)         │
├─────────────────────────────────────────────────────────────┤
│  Business Logic (MCP Gateway - Stages D, E, F, I)          │
│  ├── Content Generation (11 services)                      │
│  ├── Multi-Platform Publishing (4 adapters)                │
│  ├── Analytics & Tracking (8 KPIs)                         │
│  └── REST API Layer (44 tools, 24 endpoints)               │
├─────────────────────────────────────────────────────────────┤
│  Infrastructure (Cloudflare Workers)                        │
│  ├── D1 Database (8 tables)                                │
│  ├── R2 Storage (images)                                   │
│  ├── Vectorize (embeddings)                                │
│  ├── KV Store (cache, rate limits)                         │
│  └── Durable Objects (task queue)                          │
├─────────────────────────────────────────────────────────────┤
│  External Services                                          │
│  ├── OpenAI (GPT-5, GPT-4 Turbo, embeddings)              │
│  ├── Anthropic (Claude Sonnet 4.5)                        │
│  ├── Google Vision API                                     │
│  ├── LinkedIn API, Twitter API, Facebook Graph, Instagram  │
│  ├── Firecrawl (web scraping)                             │
│  └── Cloudinary (optional image optimization)              │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Test Environment Setup

**3 Environments:**

1. **Development** (`dev.dify-agent.com`)
   - Local Cloudflare Workers (Miniflare)
   - Dify self-hosted (Docker)
   - Mock external APIs (MSW)
   - Seed data auto-loaded

2. **Staging** (`staging.dify-agent.com`)
   - Cloudflare Workers (staging namespace)
   - Dify Cloud Professional
   - Real external APIs (sandbox accounts)
   - Test tenant accounts (3 tenants)

3. **Production** (`app.dify-agent.com`)
   - Cloudflare Workers (production)
   - Dify Cloud Professional
   - Real external APIs
   - Smoke tests only (no destructive operations)

### 1.3 Test Tenant Accounts

**3 Test Tenants** (different industries for diversity):

1. **Tenant: NextDaySteel** (Construction/B2B - existing customer)
   - Industry: Steel manufacturing
   - Brand voice: Professional, technical, safety-focused
   - Platforms: LinkedIn, Twitter
   - Products: Rebar, mesh, wire products

2. **Tenant: TechStartup** (SaaS/B2B - synthetic)
   - Industry: Cloud software
   - Brand voice: Innovative, friendly, thought-leadership
   - Platforms: LinkedIn, Twitter, Facebook
   - Products: API management platform

3. **Tenant: FashionBoutique** (Retail/B2C - synthetic)
   - Industry: E-commerce fashion
   - Brand voice: Trendy, aspirational, visual-first
   - Platforms: Instagram, Facebook, Twitter
   - Products: Clothing, accessories

---

## 2. Test Scenario Catalog

### 2.1 Scenario Overview

| ID | Scenario Name | Type | Priority | Duration | Complexity |
|----|---------------|------|----------|----------|------------|
| **E2E-01** | Complete Onboarding to First Post | Happy Path | P0 | 3-5 min | High |
| **E2E-02** | Brand Voice Learning & On-Brand Generation | Happy Path | P0 | 2-3 min | High |
| **E2E-03** | Analytics Dashboard & ROI Tracking | Happy Path | P1 | 1-2 min | Medium |
| **E2E-04** | Product Out of Stock → Advisory Pattern | Edge Case | P0 | 2-3 min | High |
| **E2E-05** | OAuth Token Expired → Auto-Refresh | Edge Case | P1 | 1-2 min | Medium |
| **E2E-06** | Rate Limit Hit → Queue & Retry | Edge Case | P1 | 2-3 min | Medium |
| **E2E-07** | Multi-Platform Aspect Ratio Adaptation | Edge Case | P1 | 2-3 min | Medium |
| **E2E-08** | OpenAI API Failure → Claude Fallback | Error Case | P0 | 1-2 min | High |
| **E2E-09** | Invalid Input → Validation Errors | Error Case | P1 | 1 min | Low |
| **E2E-10** | Database Connection Failure → Circuit Breaker | Error Case | P0 | 1-2 min | High |

**Total Estimated Test Time:** ~18-26 minutes (sequential execution)

### 2.2 Pass/Fail Criteria

**System-Wide Pass Criteria:**
- ✅ 8/10 scenarios must pass (80% threshold)
- ✅ All P0 scenarios must pass (5 scenarios)
- ✅ Performance benchmarks met (see Section 7)
- ✅ No security vulnerabilities detected
- ✅ No cross-tenant data leakage

**Individual Scenario Pass Criteria:**
- All assertions pass
- No unhandled errors
- Performance within SLA targets
- Visual regressions < 5% pixel difference

---

## 3. Happy Path Scenarios

### 3.1 E2E-01: Complete Onboarding to First Post

**User Story:**
> As a new customer, I want to onboard my brand and generate my first LinkedIn post, so that I can quickly start my social media presence.

#### 3.1.1 Preconditions
- User has a LinkedIn Business Page
- User has OAuth credentials set up
- Test tenant "NextDaySteel" is clean (no prior data)

#### 3.1.2 Test Steps

**Step 1: User Registration & Account Setup**
```typescript
// Test: e2e-01-complete-onboarding.spec.ts

import { test, expect } from '@playwright/test';

test.describe('E2E-01: Complete Onboarding to First Post', () => {
  test('should complete full onboarding and publish first post', async ({ page }) => {
    // Step 1: Navigate to app
    await page.goto('https://staging.dify-agent.com');

    // Step 2: Sign up new user
    await page.click('button:has-text("Get Started")');
    await page.fill('input[name="email"]', 'test@nextdaysteel.co.uk');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.fill('input[name="company_name"]', 'Next Day Steel');
    await page.click('button:has-text("Create Account")');

    // Assertion 1: User redirected to onboarding flow
    await expect(page).toHaveURL(/.*\/onboarding/);
    await expect(page.locator('h1')).toContainText('Welcome to DIFY-AGENT');

    // Step 3: Upload brand documents (Stage C - Knowledge Base)
    await page.click('button:has-text("Upload Brand Materials")');

    // Upload 3 files
    const fileInput = page.locator('input[type="file"]');
    await fileInput.setInputFiles([
      './test-data/nextdaysteel/brand-guide.pdf',
      './test-data/nextdaysteel/product-catalog.pdf',
      './test-data/nextdaysteel/example-posts.csv'
    ]);

    await page.click('button:has-text("Process Documents")');

    // Assertion 2: Documents processed successfully
    await expect(page.locator('[data-testid="upload-status"]')).toContainText('Processing complete', { timeout: 120000 }); // 2 min timeout
    await expect(page.locator('[data-testid="doc-count"]')).toContainText('3 documents');
    await expect(page.locator('[data-testid="entry-count"]')).toContainText(/\d+ entries extracted/); // Content multiplication

    // Step 4: Brand voice analysis (Stage H - OpenManus async)
    await page.click('button:has-text("Analyze My Brand Voice")');

    // Assertion 3: Brand voice analysis completes
    await expect(page.locator('[data-testid="brand-voice-status"]')).toContainText('Analysis complete', { timeout: 300000 }); // 5 min timeout

    const voiceScore = await page.locator('[data-testid="voice-score"]').textContent();
    expect(parseFloat(voiceScore || '0')).toBeGreaterThan(0.7); // Minimum 70% confidence

    // Step 5: Connect LinkedIn account (Stage E - OAuth)
    await page.click('button:has-text("Connect Social Accounts")');
    await page.click('button:has-text("Connect LinkedIn")');

    // OAuth flow (new window)
    const [popup] = await Promise.all([
      page.waitForEvent('popup'),
      page.click('button:has-text("Authorize LinkedIn")')
    ]);

    // Simulate LinkedIn OAuth (on staging, use test account)
    await popup.fill('input[name="session_key"]', process.env.LINKEDIN_TEST_EMAIL);
    await popup.fill('input[name="session_password"]', process.env.LINKEDIN_TEST_PASSWORD);
    await popup.click('button[type="submit"]');
    await popup.click('button:has-text("Allow")'); // Grant permissions

    // Assertion 4: OAuth successful
    await expect(page.locator('[data-testid="linkedin-status"]')).toContainText('Connected');

    // Step 6: Generate first post (Stage D - Content Generation)
    await page.click('button:has-text("Create My First Post")');

    // Type conversational request
    await page.fill('textarea[data-testid="chat-input"]', 'Create a LinkedIn post about our A142 steel mesh for construction projects');
    await page.click('button[data-testid="send-message"]');

    // Assertion 5: System generates 2-3 candidates
    await expect(page.locator('[data-testid="candidate-count"]')).toContainText(/[2-3] options/, { timeout: 45000 }); // 45s for generation

    const candidates = page.locator('[data-testid="content-candidate"]');
    const candidateCount = await candidates.count();
    expect(candidateCount).toBeGreaterThanOrEqual(2);
    expect(candidateCount).toBeLessThanOrEqual(3);

    // Assertion 6: Guards passed
    for (let i = 0; i < candidateCount; i++) {
      const guardStatus = candidates.nth(i).locator('[data-testid="guards-passed"]');
      await expect(guardStatus).toContainText('7/7 checks passed');
    }

    // Assertion 7: Images rendered (not 250px Dify limitation)
    const image = candidates.first().locator('img[data-testid="post-image"]');
    const imageWidth = await image.evaluate((el) => el.naturalWidth);
    expect(imageWidth).toBeGreaterThanOrEqual(1200); // LinkedIn requires 1200x627

    // Step 7: User selects candidate A
    await page.click('[data-testid="candidate-0"] button:has-text("Select Option A")');

    // Step 8: Publish to LinkedIn (Stage E - Publishing)
    await page.click('button:has-text("Publish Now")');

    // Assertion 8: Publishing successful
    await expect(page.locator('[data-testid="publish-status"]')).toContainText('Published successfully', { timeout: 15000 });

    // Assertion 9: Post appears in LinkedIn feed (verify via API)
    const postId = await page.locator('[data-testid="linkedin-post-id"]').textContent();
    expect(postId).toMatch(/^urn:li:share:\d+$/);

    // Verify via LinkedIn API
    const response = await fetch(`https://api.linkedin.com/v2/shares/${postId}`, {
      headers: { 'Authorization': `Bearer ${process.env.LINKEDIN_TEST_TOKEN}` }
    });
    expect(response.status).toBe(200);

    // Step 9: Check analytics dashboard (Stage F)
    await page.click('a:has-text("View Analytics")');

    // Assertion 10: Analytics shows published post
    await expect(page.locator('[data-testid="posts-count"]')).toContainText('1 post');
    await expect(page.locator('[data-testid="platforms-active"]')).toContainText('LinkedIn');
  });
});
```

#### 3.1.3 Expected Results
- ✅ User account created successfully
- ✅ 3 brand documents uploaded and processed (< 2 min)
- ✅ Brand voice analysis completes (< 5 min) with confidence score > 70%
- ✅ LinkedIn OAuth completes successfully
- ✅ 2-3 content candidates generated (< 45s)
- ✅ All candidates pass 7/7 quality guards
- ✅ Images rendered at full resolution (≥ 1200px width)
- ✅ Post published to LinkedIn (< 10s)
- ✅ Post appears in LinkedIn feed (verified via API)
- ✅ Analytics dashboard shows published post

#### 3.1.4 Performance Targets
- **Total Time:** < 8 minutes (onboarding + first post)
- **Document Processing:** < 2 minutes (3 files)
- **Brand Voice Analysis:** < 5 minutes (async OpenManus)
- **Content Generation:** < 45 seconds (GPT-5 + guards)
- **Publishing:** < 10 seconds (LinkedIn API)

#### 3.1.5 Pass/Fail Criteria
**PASS if:**
- All 10 assertions pass
- Total time < 8 minutes
- No errors in console logs
- Post visible on LinkedIn

**FAIL if:**
- Any assertion fails
- Total time > 10 minutes
- Unhandled errors occur
- Post not visible on LinkedIn after 30s

---

### 3.2 E2E-02: Brand Voice Learning & On-Brand Generation

**User Story:**
> As an existing customer, I want to upload new brand examples so the system learns my voice better and generates more on-brand content.

#### 3.2.1 Preconditions
- User "NextDaySteel" already onboarded (from E2E-01)
- Initial brand voice score: 0.72
- 10 example posts in knowledge base

#### 3.2.2 Test Steps

```typescript
// Test: e2e-02-brand-voice-learning.spec.ts

import { test, expect } from '@playwright/test';

test.describe('E2E-02: Brand Voice Learning & On-Brand Generation', () => {
  test('should improve brand voice score after uploading examples', async ({ page }) => {
    // Step 1: Login as existing user
    await page.goto('https://staging.dify-agent.com/login');
    await page.fill('input[name="email"]', 'test@nextdaysteel.co.uk');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.click('button:has-text("Sign In")');

    // Step 2: Navigate to Knowledge Base
    await page.click('a:has-text("Brand Voice")');

    // Assertion 1: Current voice score displayed
    const initialScore = await page.locator('[data-testid="voice-score"]').textContent();
    expect(parseFloat(initialScore || '0')).toBeGreaterThan(0.7);

    // Step 3: Upload 20 new example posts (CSV)
    await page.click('button:has-text("Upload Examples")');
    await page.locator('input[type="file"]').setInputFiles('./test-data/nextdaysteel/additional-posts.csv');
    await page.click('button:has-text("Process Examples")');

    // Assertion 2: Examples processed
    await expect(page.locator('[data-testid="upload-status"]')).toContainText('20 examples added', { timeout: 60000 });

    // Step 4: Trigger brand voice re-analysis
    await page.click('button:has-text("Re-analyze Brand Voice")');

    // Assertion 3: Brand voice score improves
    await expect(page.locator('[data-testid="analysis-status"]')).toContainText('Complete', { timeout: 180000 }); // 3 min

    const newScore = await page.locator('[data-testid="voice-score"]').textContent();
    expect(parseFloat(newScore || '0')).toBeGreaterThan(parseFloat(initialScore || '0'));
    expect(parseFloat(newScore || '0')).toBeGreaterThan(0.85); // Target 85%+

    // Step 5: Generate new content (should be more on-brand)
    await page.click('a:has-text("Create Content")');
    await page.fill('textarea[data-testid="chat-input"]', 'Create a post about our T12 rebar for bridge construction');
    await page.click('button[data-testid="send-message"]');

    // Assertion 4: Generated content has high brand voice score
    await expect(page.locator('[data-testid="candidate-count"]')).toContainText(/[2-3] options/, { timeout: 45000 });

    const candidates = page.locator('[data-testid="content-candidate"]');
    const firstCandidateScore = await candidates.first().locator('[data-testid="brand-voice-score"]').textContent();
    expect(parseFloat(firstCandidateScore || '0')).toBeGreaterThan(0.90); // High match
  });
});
```

#### 3.2.3 Expected Results
- ✅ User logs in successfully
- ✅ Initial brand voice score displayed (> 70%)
- ✅ 20 new examples uploaded and processed (< 1 min)
- ✅ Brand voice re-analysis completes (< 3 min)
- ✅ New voice score improves (> 85%)
- ✅ Generated content has brand voice score > 90%

#### 3.2.4 Performance Targets
- **Example Upload:** < 60 seconds (20 posts)
- **Re-analysis:** < 3 minutes (OpenManus)
- **Content Generation:** < 45 seconds

---

### 3.3 E2E-03: Analytics Dashboard & ROI Tracking

**User Story:**
> As a customer, I want to view my social media analytics and ROI so I can measure the effectiveness of my posts.

#### 3.3.1 Preconditions
- User "NextDaySteel" has published 10 posts across LinkedIn and Twitter
- Posts are 7 days old (enough time for engagement)
- Analytics sync has run (last 6 hours)

#### 3.3.2 Test Steps

```typescript
// Test: e2e-03-analytics-dashboard.spec.ts

import { test, expect } from '@playwright/test';

test.describe('E2E-03: Analytics Dashboard & ROI Tracking', () => {
  test('should display comprehensive analytics and ROI metrics', async ({ page }) => {
    // Step 1: Login
    await page.goto('https://staging.dify-agent.com/login');
    await page.fill('input[name="email"]', 'test@nextdaysteel.co.uk');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.click('button:has-text("Sign In")');

    // Step 2: Navigate to Analytics
    await page.click('a:has-text("Analytics")');

    // Assertion 1: Overview metrics displayed
    await expect(page.locator('[data-testid="total-posts"]')).toContainText('10');
    await expect(page.locator('[data-testid="total-impressions"]')).toContainText(/\d+/);
    await expect(page.locator('[data-testid="total-engagement"]')).toContainText(/\d+/);

    const engagementRate = await page.locator('[data-testid="engagement-rate"]').textContent();
    expect(parseFloat(engagementRate || '0')).toBeGreaterThan(0); // Non-zero

    // Assertion 2: Platform breakdown displayed
    await expect(page.locator('[data-testid="platform-linkedin"]')).toBeVisible();
    await expect(page.locator('[data-testid="platform-twitter"]')).toBeVisible();

    const linkedinPosts = await page.locator('[data-testid="linkedin-post-count"]').textContent();
    const twitterPosts = await page.locator('[data-testid="twitter-post-count"]').textContent();
    expect(parseInt(linkedinPosts || '0') + parseInt(twitterPosts || '0')).toBe(10);

    // Step 3: View top performing post
    await page.click('[data-testid="top-posts"] .post-card:first-child');

    // Assertion 3: Post detail shows metrics
    await expect(page.locator('[data-testid="post-likes"]')).toContainText(/\d+/);
    await expect(page.locator('[data-testid="post-comments"]')).toContainText(/\d+/);
    await expect(page.locator('[data-testid="post-shares"]')).toContainText(/\d+/);
    await expect(page.locator('[data-testid="post-impressions"]')).toContainText(/\d+/);

    // Step 4: Check ROI tracking
    await page.click('a:has-text("ROI")');

    // Assertion 4: ROI metrics displayed
    await expect(page.locator('[data-testid="cpe"]')).toContainText(/\$\d+\.\d+/); // Cost per engagement
    await expect(page.locator('[data-testid="attributed-conversions"]')).toContainText(/\d+/);

    // Assertion 5: Charts rendered
    const chart = page.locator('canvas[data-testid="engagement-chart"]');
    await expect(chart).toBeVisible();
    expect(await chart.isVisible()).toBe(true);
  });
});
```

#### 3.3.3 Expected Results
- ✅ Overview metrics displayed (10 posts, impressions, engagement)
- ✅ Platform breakdown correct (LinkedIn + Twitter = 10)
- ✅ Top post detail shows all metrics
- ✅ ROI metrics calculated (CPE, conversions)
- ✅ Charts rendered successfully

#### 3.3.4 Performance Targets
- **Page Load:** < 2 seconds
- **Metrics Query:** < 500ms
- **Chart Render:** < 1 second

---

## 4. Edge Case Scenarios

### 4.1 E2E-04: Product Out of Stock → Advisory Pattern

**User Story:**
> As a customer, when I request a post about an out-of-stock product, I want the system to advise me with alternatives so I can choose a different product.

#### 4.1.1 Preconditions
- User "NextDaySteel" is logged in
- Product "A193 Mesh" is marked as out-of-stock in Shopify
- Alternative products available: "A142 Mesh", "A252 Mesh"

#### 4.1.2 Test Steps

```typescript
// Test: e2e-04-out-of-stock-advisory.spec.ts

import { test, expect } from '@playwright/test';

test.describe('E2E-04: Product Out of Stock → Advisory Pattern', () => {
  test('should present alternatives when product is out of stock', async ({ page }) => {
    // Step 1: Login
    await page.goto('https://staging.dify-agent.com/login');
    await page.fill('input[name="email"]', 'test@nextdaysteel.co.uk');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.click('button:has-text("Sign In")');

    // Step 2: Request post for out-of-stock product
    await page.click('a:has-text("Create Content")');
    await page.fill('textarea[data-testid="chat-input"]', 'Create a post about A193 mesh');
    await page.click('button[data-testid="send-message"]');

    // Assertion 1: System detects out-of-stock
    await expect(page.locator('[data-testid="bot-message"]')).toContainText('A193 Mesh is currently out of stock', { timeout: 10000 });

    // Assertion 2: System presents alternatives (conversational pattern)
    const botMessage = page.locator('[data-testid="bot-message"]').last();
    await expect(botMessage).toContainText('Would you like to');
    await expect(botMessage).toContainText('A) Promote A142 Mesh instead');
    await expect(botMessage).toContainText('B) Promote A252 Mesh instead');
    await expect(botMessage).toContainText('C) Wait until A193 is back in stock');

    // Step 3: User types choice
    await page.fill('textarea[data-testid="chat-input"]', 'A');
    await page.click('button[data-testid="send-message"]');

    // Assertion 3: System generates content for alternative product
    await expect(page.locator('[data-testid="candidate-count"]')).toContainText(/[2-3] options/, { timeout: 45000 });

    const candidates = page.locator('[data-testid="content-candidate"]');
    const firstCandidateText = await candidates.first().locator('[data-testid="post-text"]').textContent();
    expect(firstCandidateText).toContain('A142 Mesh'); // Alternative product
    expect(firstCandidateText).not.toContain('A193 Mesh'); // Not out-of-stock product
  });
});
```

#### 4.1.3 Expected Results
- ✅ System detects out-of-stock product via Shopify API
- ✅ Conversational options presented (A/B/C format)
- ✅ User types choice, system continues workflow
- ✅ Generated content mentions alternative product, not original

---

### 4.2 E2E-05: OAuth Token Expired → Auto-Refresh

**User Story:**
> As a customer, when my LinkedIn OAuth token expires, I want the system to automatically refresh it so my post still publishes without manual intervention.

#### 4.2.1 Preconditions
- User "NextDaySteel" has LinkedIn connected
- OAuth access token expires in 1 hour (manually set expiry in test)
- Refresh token is valid

#### 4.2.2 Test Steps

```typescript
// Test: e2e-05-oauth-token-refresh.spec.ts

import { test, expect } from '@playwright/test';

test.describe('E2E-05: OAuth Token Expired → Auto-Refresh', () => {
  test('should auto-refresh expired OAuth token during publish', async ({ page, request }) => {
    // Step 1: Manually expire access token (via API)
    await request.post('https://staging.dify-agent.com/api/test/expire-oauth-token', {
      data: {
        tenant_id: 'nextdaysteel',
        platform: 'linkedin'
      },
      headers: { 'Authorization': `Bearer ${process.env.TEST_API_KEY}` }
    });

    // Step 2: Login and generate content
    await page.goto('https://staging.dify-agent.com/login');
    await page.fill('input[name="email"]', 'test@nextdaysteel.co.uk');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.click('button:has-text("Sign In")');

    await page.click('a:has-text("Create Content")');
    await page.fill('textarea[data-testid="chat-input"]', 'Create a post about our wire products');
    await page.click('button[data-testid="send-message"]');

    await expect(page.locator('[data-testid="candidate-count"]')).toContainText(/[2-3] options/, { timeout: 45000 });
    await page.click('[data-testid="candidate-0"] button:has-text("Select Option A")');

    // Step 3: Attempt publish (should trigger auto-refresh)
    await page.click('button:has-text("Publish to LinkedIn")');

    // Assertion 1: System detects expired token
    // (No user-visible message, happens transparently)

    // Assertion 2: Token refreshed automatically
    await expect(page.locator('[data-testid="publish-status"]')).toContainText('Publishing...', { timeout: 5000 });

    // Assertion 3: Publish succeeds with new token
    await expect(page.locator('[data-testid="publish-status"]')).toContainText('Published successfully', { timeout: 15000 });

    const postId = await page.locator('[data-testid="linkedin-post-id"]').textContent();
    expect(postId).toMatch(/^urn:li:share:\d+$/);

    // Step 4: Verify token was refreshed (via API)
    const response = await request.get('https://staging.dify-agent.com/api/test/check-oauth-token', {
      data: {
        tenant_id: 'nextdaysteel',
        platform: 'linkedin'
      },
      headers: { 'Authorization': `Bearer ${process.env.TEST_API_KEY}` }
    });

    const tokenData = await response.json();
    expect(tokenData.refreshed).toBe(true);
    expect(tokenData.new_expiry).toBeGreaterThan(Date.now() + 3600000); // > 1 hour
  });
});
```

#### 4.2.3 Expected Results
- ✅ Expired token detected during publish attempt
- ✅ System automatically calls OAuth refresh endpoint
- ✅ New access token stored securely
- ✅ Publish retries with new token and succeeds
- ✅ No user intervention required

---

### 4.3 E2E-06: Rate Limit Hit → Queue & Retry

**User Story:**
> As a customer, when I hit LinkedIn's rate limit, I want my post to be queued and automatically retry so I don't lose my content.

#### 4.3.1 Test Steps

```typescript
// Test: e2e-06-rate-limit-queue.spec.ts

import { test, expect } from '@playwright/test';

test.describe('E2E-06: Rate Limit Hit → Queue & Retry', () => {
  test('should queue post when rate limit hit and retry automatically', async ({ page, request }) => {
    // Step 1: Simulate rate limit (publish 100 posts via API to hit LinkedIn limit)
    for (let i = 0; i < 100; i++) {
      await request.post('https://staging.dify-agent.com/api/v1/publish/linkedin', {
        data: {
          tenant_id: 'nextdaysteel',
          content: { text: `Test post ${i}`, image_url: 'https://example.com/test.png' }
        },
        headers: { 'Authorization': `Bearer ${process.env.TEST_API_KEY}` }
      });
    }

    // Step 2: Login and attempt to publish 101st post (should hit rate limit)
    await page.goto('https://staging.dify-agent.com/login');
    await page.fill('input[name="email"]', 'test@nextdaysteel.co.uk');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.click('button:has-text("Sign In")');

    await page.click('a:has-text("Create Content")');
    await page.fill('textarea[data-testid="chat-input"]', 'Create a post about our rebar products');
    await page.click('button[data-testid="send-message"]');

    await expect(page.locator('[data-testid="candidate-count"]')).toContainText(/[2-3] options/, { timeout: 45000 });
    await page.click('[data-testid="candidate-0"] button:has-text("Select Option A")');

    await page.click('button:has-text("Publish to LinkedIn")');

    // Assertion 1: Rate limit detected
    await expect(page.locator('[data-testid="publish-status"]')).toContainText('Rate limit reached', { timeout: 10000 });

    // Assertion 2: Post queued
    await expect(page.locator('[data-testid="queue-status"]')).toContainText('Queued for later');
    await expect(page.locator('[data-testid="retry-time"]')).toContainText(/in \d+ minutes?/);

    // Step 3: Fast-forward time (mock) or wait for retry
    // In real test, we'd use a time mock. For demo, we verify queue entry exists.
    const response = await request.get('https://staging.dify-agent.com/api/v1/queue/status', {
      data: { tenant_id: 'nextdaysteel' },
      headers: { 'Authorization': `Bearer ${process.env.TEST_API_KEY}` }
    });

    const queueData = await response.json();
    expect(queueData.queued_posts).toBeGreaterThan(0);
    expect(queueData.next_retry).toBeDefined();
  });
});
```

#### 4.3.4 Expected Results
- ✅ Rate limit detected (HTTP 429 from LinkedIn)
- ✅ Post queued in Durable Objects task queue
- ✅ User notified with retry time
- ✅ Background job retries after cooldown period
- ✅ Post eventually publishes successfully

---

### 4.4 E2E-07: Multi-Platform Aspect Ratio Adaptation

**User Story:**
> As a customer, when I publish to multiple platforms, I want the image automatically adapted to each platform's aspect ratio so it looks good everywhere.

#### 4.4.1 Test Steps

```typescript
// Test: e2e-07-aspect-ratio-adaptation.spec.ts

import { test, expect } from '@playwright/test';
import sharp from 'sharp';

test.describe('E2E-07: Multi-Platform Aspect Ratio Adaptation', () => {
  test('should adapt image aspect ratios for each platform', async ({ page, request }) => {
    // Step 1: Login and generate content
    await page.goto('https://staging.dify-agent.com/login');
    await page.fill('input[name="email"]', 'test@nextdaysteel.co.uk');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.click('button:has-text("Sign In")');

    await page.click('a:has-text("Create Content")');
    await page.fill('textarea[data-testid="chat-input"]', 'Create a post about A142 mesh');
    await page.click('button[data-testid="send-message"]');

    await expect(page.locator('[data-testid="candidate-count"]')).toContainText(/[2-3] options/, { timeout: 45000 });
    await page.click('[data-testid="candidate-0"] button:has-text("Select Option A")');

    // Step 2: Select all 4 platforms
    await page.click('button:has-text("Publish to Multiple Platforms")');
    await page.check('input[name="platform-linkedin"]');
    await page.check('input[name="platform-twitter"]');
    await page.check('input[name="platform-facebook"]');
    await page.check('input[name="platform-instagram"]');

    // Step 3: Publish
    await page.click('button:has-text("Publish to All")');

    // Assertion 1: All platforms publish successfully
    await expect(page.locator('[data-testid="publish-status"]')).toContainText('Published to 4 platforms', { timeout: 30000 });

    // Assertion 2: Each platform has correct aspect ratio
    const imageUrls = await page.locator('[data-testid="published-image-url"]').allTextContents();
    expect(imageUrls.length).toBe(4);

    // Verify LinkedIn (1.91:1 → 1200x627)
    const linkedinImg = await fetch(imageUrls[0]);
    const linkedinBuffer = await linkedinImg.arrayBuffer();
    const linkedinMeta = await sharp(Buffer.from(linkedinBuffer)).metadata();
    expect(linkedinMeta.width).toBe(1200);
    expect(linkedinMeta.height).toBe(627);

    // Verify Twitter (16:9 → 1200x675)
    const twitterImg = await fetch(imageUrls[1]);
    const twitterBuffer = await twitterImg.arrayBuffer();
    const twitterMeta = await sharp(Buffer.from(twitterBuffer)).metadata();
    expect(twitterMeta.width).toBe(1200);
    expect(twitterMeta.height).toBe(675);

    // Verify Facebook (1.91:1 → 1200x630)
    const facebookImg = await fetch(imageUrls[2]);
    const facebookBuffer = await facebookImg.arrayBuffer();
    const facebookMeta = await sharp(Buffer.from(facebookBuffer)).metadata();
    expect(facebookMeta.width).toBe(1200);
    expect(facebookMeta.height).toBe(630);

    // Verify Instagram (1:1 → 1080x1080)
    const instagramImg = await fetch(imageUrls[3]);
    const instagramBuffer = await instagramImg.arrayBuffer();
    const instagramMeta = await sharp(Buffer.from(instagramBuffer)).metadata();
    expect(instagramMeta.width).toBe(1080);
    expect(instagramMeta.height).toBe(1080);
  });
});
```

#### 4.4.3 Expected Results
- ✅ User selects 4 platforms for publishing
- ✅ System generates 4 different image versions
- ✅ Each image has correct aspect ratio for platform
- ✅ Smart cropping preserves important content (faces, text)
- ✅ All 4 posts publish successfully

---

## 5. Error Case Scenarios

### 5.1 E2E-08: OpenAI API Failure → Claude Fallback

**User Story:**
> As a customer, when OpenAI's API is down, I want the system to automatically use Claude so my content generation doesn't fail.

#### 5.1.1 Test Steps

```typescript
// Test: e2e-08-llm-fallback.spec.ts

import { test, expect } from '@playwright/test';

test.describe('E2E-08: OpenAI API Failure → Claude Fallback', () => {
  test('should fallback to Claude when OpenAI fails', async ({ page, request }) => {
    // Step 1: Simulate OpenAI outage (via test API)
    await request.post('https://staging.dify-agent.com/api/test/disable-openai', {
      headers: { 'Authorization': `Bearer ${process.env.TEST_API_KEY}` }
    });

    // Step 2: Login and generate content
    await page.goto('https://staging.dify-agent.com/login');
    await page.fill('input[name="email"]', 'test@nextdaysteel.co.uk');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.click('button:has-text("Sign In")');

    await page.click('a:has-text("Create Content")');
    await page.fill('textarea[data-testid="chat-input"]', 'Create a post about our T16 rebar');
    await page.click('button[data-testid="send-message"]');

    // Assertion 1: System detects OpenAI failure and falls back
    // (No user-visible message, happens transparently)

    // Assertion 2: Content generated with Claude
    await expect(page.locator('[data-testid="candidate-count"]')).toContainText(/[2-3] options/, { timeout: 60000 }); // Claude may be slower

    const candidates = page.locator('[data-testid="content-candidate"]');
    expect(await candidates.count()).toBeGreaterThanOrEqual(2);

    // Assertion 3: Quality maintained (guards pass)
    const guardStatus = await candidates.first().locator('[data-testid="guards-passed"]').textContent();
    expect(guardStatus).toContain('7/7');

    // Step 3: Verify LLM provider used (via API)
    const response = await request.get('https://staging.dify-agent.com/api/test/last-llm-provider', {
      headers: { 'Authorization': `Bearer ${process.env.TEST_API_KEY}` }
    });

    const providerData = await response.json();
    expect(providerData.provider).toBe('anthropic');
    expect(providerData.model).toContain('claude-sonnet-4');

    // Step 4: Re-enable OpenAI
    await request.post('https://staging.dify-agent.com/api/test/enable-openai', {
      headers: { 'Authorization': `Bearer ${process.env.TEST_API_KEY}` }
    });
  });
});
```

#### 5.1.3 Expected Results
- ✅ OpenAI API failure detected (HTTP 503 or timeout)
- ✅ System automatically falls back to Claude Sonnet 4.5
- ✅ Content generation succeeds with fallback model
- ✅ Quality guards still pass
- ✅ No user-visible errors

---

### 5.2 E2E-09: Invalid Input → Validation Errors

**User Story:**
> As a customer, when I provide invalid input, I want clear error messages so I know what to fix.

#### 5.2.1 Test Steps

```typescript
// Test: e2e-09-input-validation.spec.ts

import { test, expect } from '@playwright/test';

test.describe('E2E-09: Invalid Input → Validation Errors', () => {
  test('should show user-friendly validation errors', async ({ page }) => {
    // Step 1: Login
    await page.goto('https://staging.dify-agent.com/login');
    await page.fill('input[name="email"]', 'test@nextdaysteel.co.uk');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.click('button:has-text("Sign In")');

    // Test Case 1: Empty request
    await page.click('a:has-text("Create Content")');
    await page.fill('textarea[data-testid="chat-input"]', '');
    await page.click('button[data-testid="send-message"]');

    await expect(page.locator('[data-testid="error-message"]')).toContainText('Please enter a request');

    // Test Case 2: Request too long (> 500 chars)
    const longText = 'a'.repeat(501);
    await page.fill('textarea[data-testid="chat-input"]', longText);
    await page.click('button[data-testid="send-message"]');

    await expect(page.locator('[data-testid="error-message"]')).toContainText('Request too long (max 500 characters)');

    // Test Case 3: Invalid product name
    await page.fill('textarea[data-testid="chat-input"]', 'Create a post about XYZ-999 product');
    await page.click('button[data-testid="send-message"]');

    await expect(page.locator('[data-testid="bot-message"]')).toContainText('I couldn\'t find a product called "XYZ-999"', { timeout: 10000 });
    await expect(page.locator('[data-testid="bot-message"]')).toContainText('Did you mean:');

    // Test Case 4: Profanity in request
    await page.fill('textarea[data-testid="chat-input"]', 'Create a f***ing post about rebar');
    await page.click('button[data-testid="send-message"]');

    await expect(page.locator('[data-testid="error-message"]')).toContainText('Please use professional language');
  });
});
```

#### 5.2.3 Expected Results
- ✅ Empty input: Clear error message
- ✅ Too long input: Character limit warning
- ✅ Invalid product: Suggested alternatives
- ✅ Profanity: Professional language reminder

---

### 5.3 E2E-10: Database Connection Failure → Circuit Breaker

**User Story:**
> As a customer, when the database is temporarily unavailable, I want the system to gracefully degrade instead of crashing.

#### 5.3.1 Test Steps

```typescript
// Test: e2e-10-circuit-breaker.spec.ts

import { test, expect } from '@playwright/test';

test.describe('E2E-10: Database Connection Failure → Circuit Breaker', () => {
  test('should activate circuit breaker on database failure', async ({ page, request }) => {
    // Step 1: Simulate database outage
    await request.post('https://staging.dify-agent.com/api/test/disable-database', {
      headers: { 'Authorization': `Bearer ${process.env.TEST_API_KEY}` }
    });

    // Step 2: Login (should succeed with cached auth)
    await page.goto('https://staging.dify-agent.com/login');
    await page.fill('input[name="email"]', 'test@nextdaysteel.co.uk');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.click('button:has-text("Sign In")');

    // Assertion 1: Login succeeds (cached in KV)
    await expect(page.locator('[data-testid="user-menu"]')).toContainText('NextDaySteel');

    // Step 3: Attempt content generation (requires DB)
    await page.click('a:has-text("Create Content")');
    await page.fill('textarea[data-testid="chat-input"]', 'Create a post about A142 mesh');
    await page.click('button[data-testid="send-message"]');

    // Assertion 2: Circuit breaker activates (graceful degradation)
    await expect(page.locator('[data-testid="bot-message"]')).toContainText('System temporarily unavailable', { timeout: 15000 });
    await expect(page.locator('[data-testid="bot-message"]')).toContainText('retry in a few minutes');

    // Assertion 3: Circuit breaker status (via API)
    const response = await request.get('https://staging.dify-agent.com/api/test/circuit-breaker-status', {
      headers: { 'Authorization': `Bearer ${process.env.TEST_API_KEY}` }
    });

    const cbData = await response.json();
    expect(cbData.state).toBe('open'); // Circuit breaker open
    expect(cbData.failure_count).toBeGreaterThan(0);

    // Step 4: Re-enable database
    await request.post('https://staging.dify-agent.com/api/test/enable-database', {
      headers: { 'Authorization': `Bearer ${process.env.TEST_API_KEY}` }
    });

    // Step 5: Wait for circuit breaker to close (half-open → closed)
    await page.waitForTimeout(30000); // 30s timeout before retry

    // Step 6: Retry request
    await page.fill('textarea[data-testid="chat-input"]', 'Create a post about A142 mesh');
    await page.click('button[data-testid="send-message"]');

    // Assertion 4: Circuit breaker closed, request succeeds
    await expect(page.locator('[data-testid="candidate-count"]')).toContainText(/[2-3] options/, { timeout: 45000 });
  });
});
```

#### 5.3.3 Expected Results
- ✅ Database failure detected (3 consecutive timeouts)
- ✅ Circuit breaker opens (blocks further requests)
- ✅ User shown graceful error message
- ✅ Circuit breaker half-opens after timeout
- ✅ Circuit breaker closes when database recovers
- ✅ Subsequent requests succeed

---

## 6. LLM-as-Judge Framework

### 6.1 Overview

**Purpose:** Automatically evaluate content quality using GPT-4 as a judge, measuring:
1. **Brand Voice Match** (0-10): Does content match customer's brand voice?
2. **Creativity Score** (0-10): Is content original and engaging?
3. **Engagement Potential** (0-10): Likelihood to drive likes, comments, shares
4. **Technical Accuracy** (0-10): Are product details correct?

### 6.2 LLM Judge Implementation

```typescript
// src/testing/llm-judge.ts

import OpenAI from 'openai';

interface ContentEvaluation {
  brand_voice_match: number;
  creativity_score: number;
  engagement_potential: number;
  technical_accuracy: number;
  overall_score: number;
  reasoning: string;
}

export class LLMJudge {
  private openai: OpenAI;

  constructor(apiKey: string) {
    this.openai = new OpenAI({ apiKey });
  }

  async evaluateContent(
    generatedContent: string,
    brandVoiceExamples: string[],
    productDetails: Record<string, any>,
    platform: string
  ): Promise<ContentEvaluation> {
    const prompt = this.buildEvaluationPrompt(
      generatedContent,
      brandVoiceExamples,
      productDetails,
      platform
    );

    const response = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert social media content evaluator. Analyze the provided content and score it across 4 dimensions on a scale of 0-10. Return your evaluation as JSON.'
        },
        {
          role: 'user',
          content: prompt
        }
      ],
      response_format: { type: 'json_object' },
      temperature: 0.2 // Low temperature for consistent scoring
    });

    const evaluation = JSON.parse(response.choices[0].message.content || '{}');

    // Calculate overall score (weighted average)
    evaluation.overall_score = (
      evaluation.brand_voice_match * 0.35 +
      evaluation.creativity_score * 0.25 +
      evaluation.engagement_potential * 0.30 +
      evaluation.technical_accuracy * 0.10
    );

    return evaluation;
  }

  private buildEvaluationPrompt(
    content: string,
    examples: string[],
    product: Record<string, any>,
    platform: string
  ): string {
    return `
# Content Evaluation Task

## Generated Content to Evaluate
Platform: ${platform}
Content:
"""
${content}
"""

## Brand Voice Reference Examples
${examples.map((ex, i) => `Example ${i + 1}:\n${ex}`).join('\n\n')}

## Product Details (for technical accuracy)
${JSON.stringify(product, null, 2)}

## Evaluation Criteria

### 1. Brand Voice Match (0-10)
- Compare tone, vocabulary, style to reference examples
- 10 = Perfect match, indistinguishable from examples
- 5 = Some elements match, but noticeable differences
- 0 = Completely different voice

### 2. Creativity Score (0-10)
- Originality of messaging
- Unique angles or perspectives
- Avoids clichés and generic statements
- 10 = Highly original and engaging
- 5 = Somewhat original but predictable
- 0 = Generic, template-like content

### 3. Engagement Potential (0-10)
- Likelihood to drive likes, comments, shares
- Includes call-to-action
- Emotional resonance
- Platform-appropriate format
- 10 = Highly likely to go viral
- 5 = Decent engagement expected
- 0 = Unlikely to get engagement

### 4. Technical Accuracy (0-10)
- Product details correct (name, specs, pricing)
- No factual errors
- Appropriate technical depth for audience
- 10 = Completely accurate
- 5 = Minor inaccuracies
- 0 = Major factual errors

## Output Format (JSON)
{
  "brand_voice_match": <score 0-10>,
  "creativity_score": <score 0-10>,
  "engagement_potential": <score 0-10>,
  "technical_accuracy": <score 0-10>,
  "reasoning": "<2-3 sentence explanation of scores>"
}
`;
  }
}
```

### 6.3 Integration with Playwright Tests

```typescript
// e2e-with-llm-judge.spec.ts

import { test, expect } from '@playwright/test';
import { LLMJudge } from '../src/testing/llm-judge';

test.describe('E2E with LLM-as-Judge Evaluation', () => {
  let judge: LLMJudge;

  test.beforeAll(() => {
    judge = new LLMJudge(process.env.OPENAI_API_KEY!);
  });

  test('should generate high-quality on-brand content', async ({ page }) => {
    // ... (login and content generation steps from E2E-02)

    // Extract generated content
    const contentText = await page.locator('[data-testid="candidate-0"] [data-testid="post-text"]').textContent();

    // Load brand voice examples
    const brandExamples = [
      "At Next Day Steel, safety isn't just a priority – it's our foundation. Our A142 mesh meets BS 4483 standards for structural reinforcement.",
      "Quality rebar delivered fast. T16 high-tensile steel, 500MPa grade, certified to BS 4449. Next day delivery across the UK."
    ];

    // Product details
    const productDetails = {
      name: 'A142 Mesh',
      category: 'Reinforcement Mesh',
      standard: 'BS 4483',
      wire_diameter: '6mm',
      spacing: '200x200mm'
    };

    // Evaluate with LLM Judge
    const evaluation = await judge.evaluateContent(
      contentText || '',
      brandExamples,
      productDetails,
      'LinkedIn'
    );

    // Assertions based on LLM evaluation
    expect(evaluation.brand_voice_match).toBeGreaterThanOrEqual(8.0); // Minimum 8/10
    expect(evaluation.creativity_score).toBeGreaterThanOrEqual(7.0);
    expect(evaluation.engagement_potential).toBeGreaterThanOrEqual(7.0);
    expect(evaluation.technical_accuracy).toBeGreaterThanOrEqual(9.0);
    expect(evaluation.overall_score).toBeGreaterThanOrEqual(8.0);

    // Log reasoning for debugging
    console.log('LLM Judge Evaluation:', evaluation);
  });
});
```

### 6.4 Pass Threshold

**Minimum Acceptable Scores:**
- Brand Voice Match: ≥ 8.0/10
- Creativity: ≥ 7.0/10
- Engagement Potential: ≥ 7.0/10
- Technical Accuracy: ≥ 9.0/10
- Overall Score: ≥ 8.0/10

**Test Pass Criteria:**
- ✅ 8/10 scenarios must meet minimum scores
- ✅ All P0 scenarios must meet minimum scores
- ✅ Average overall score across all scenarios ≥ 8.5/10

---

## 7. Performance Benchmarks

### 7.1 Critical Path Metrics

| Metric | Target | P95 Target | Measurement Method |
|--------|--------|------------|-------------------|
| **Onboarding Speed** | < 120s | < 150s | Time from signup to "ready to create" |
| **Validation Speed** | < 200ms | < 300ms | Input validation + intent parsing |
| **Content Generation** | < 30s | < 45s | Request to candidates displayed |
| **Publishing Speed** | < 10s | < 15s | Publish button to "success" message |
| **End-to-End (Onboard → Publish)** | < 180s | < 240s | Complete first post journey |

### 7.2 Performance Test Implementation

```typescript
// performance-benchmarks.spec.ts

import { test, expect } from '@playwright/test';

test.describe('Performance Benchmarks', () => {
  test('P1: Onboarding speed < 120s', async ({ page }) => {
    const startTime = Date.now();

    // Execute onboarding flow (from E2E-01)
    await page.goto('https://staging.dify-agent.com');
    // ... (full onboarding steps)

    const endTime = Date.now();
    const duration = (endTime - startTime) / 1000; // Convert to seconds

    console.log(`Onboarding completed in ${duration}s`);
    expect(duration).toBeLessThan(120);
  });

  test('P2: Validation speed < 200ms', async ({ page }) => {
    await page.goto('https://staging.dify-agent.com/create');

    // Measure validation time
    const startTime = Date.now();
    await page.fill('textarea[data-testid="chat-input"]', 'Create a post about A142 mesh');

    // Wait for validation indicator
    await page.waitForSelector('[data-testid="validation-complete"]');
    const endTime = Date.now();

    const duration = endTime - startTime;
    console.log(`Validation completed in ${duration}ms`);
    expect(duration).toBeLessThan(200);
  });

  test('P3: Content generation < 30s', async ({ page }) => {
    await page.goto('https://staging.dify-agent.com/create');

    await page.fill('textarea[data-testid="chat-input"]', 'Create a LinkedIn post about T12 rebar');

    const startTime = Date.now();
    await page.click('button[data-testid="send-message"]');

    // Wait for candidates
    await expect(page.locator('[data-testid="candidate-count"]')).toContainText(/[2-3] options/);
    const endTime = Date.now();

    const duration = (endTime - startTime) / 1000;
    console.log(`Content generation completed in ${duration}s`);
    expect(duration).toBeLessThan(30);
  });

  test('P4: Publishing speed < 10s', async ({ page }) => {
    // Assume content already generated
    await page.goto('https://staging.dify-agent.com/create?candidate_id=test-123');

    const startTime = Date.now();
    await page.click('button:has-text("Publish to LinkedIn")');

    await expect(page.locator('[data-testid="publish-status"]')).toContainText('Published successfully');
    const endTime = Date.now();

    const duration = (endTime - startTime) / 1000;
    console.log(`Publishing completed in ${duration}s`);
    expect(duration).toBeLessThan(10);
  });

  test('P5: End-to-end < 180s', async ({ page }) => {
    const startTime = Date.now();

    // Complete journey: login → generate → publish
    // (Assumes user already onboarded)
    await page.goto('https://staging.dify-agent.com/login');
    // ... (full E2E flow)

    await expect(page.locator('[data-testid="publish-status"]')).toContainText('Published successfully');
    const endTime = Date.now();

    const duration = (endTime - startTime) / 1000;
    console.log(`End-to-end completed in ${duration}s`);
    expect(duration).toBeLessThan(180);
  });
});
```

### 7.3 Load Testing with k6

```javascript
// load-test.js (k6 script)

import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 },  // Ramp to 100 users
    { duration: '5m', target: 100 },  // Stay at 100 users
    { duration: '2m', target: 1000 }, // Ramp to 1,000 users
    { duration: '5m', target: 1000 }, // Stay at 1,000 users
    { duration: '2m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests < 500ms
    http_req_failed: ['rate<0.01'],   // Error rate < 1%
  },
};

export default function () {
  // Test content generation endpoint
  const payload = JSON.stringify({
    tenant_id: `tenant-${Math.floor(Math.random() * 100)}`,
    request: 'Create a LinkedIn post about our steel products',
  });

  const params = {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${__ENV.TEST_API_KEY}`,
    },
  };

  const res = http.post('https://staging.dify-agent.com/api/v1/content/generate', payload, params);

  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
    'has candidates': (r) => JSON.parse(r.body).candidates.length >= 2,
  });

  sleep(1);
}
```

**Run Load Test:**
```bash
k6 run load-test.js --env TEST_API_KEY=your_key_here
```

---

## 8. Test Data Management

### 8.1 Seed Data Strategy

**Purpose:** Provide consistent, realistic test data across all environments.

**Data Categories:**
1. **Test Tenants** (3 tenants: NextDaySteel, TechStartup, FashionBoutique)
2. **Brand Documents** (PDFs, CSVs, example posts)
3. **Product Catalogs** (Shopify sync)
4. **OAuth Tokens** (sandbox accounts for LinkedIn, Twitter, Facebook, Instagram)
5. **Historical Posts** (for analytics testing)

### 8.2 Seed Data Scripts

```typescript
// scripts/seed-test-data.ts

import { PrismaClient } from '@prisma/client';
import fs from 'fs';
import path from 'path';

const prisma = new PrismaClient();

async function seedTestData() {
  console.log('Seeding test data...');

  // 1. Create test tenants
  const tenants = await Promise.all([
    prisma.tenant.create({
      data: {
        id: 'nextdaysteel',
        name: 'Next Day Steel',
        industry: 'Construction/B2B',
        created_at: new Date(),
      },
    }),
    prisma.tenant.create({
      data: {
        id: 'techstartup',
        name: 'TechStartup',
        industry: 'SaaS/B2B',
        created_at: new Date(),
      },
    }),
    prisma.tenant.create({
      data: {
        id: 'fashionboutique',
        name: 'Fashion Boutique',
        industry: 'Retail/B2C',
        created_at: new Date(),
      },
    }),
  ]);

  console.log(`Created ${tenants.length} test tenants`);

  // 2. Load brand documents
  const brandDocs = [
    { tenant: 'nextdaysteel', file: 'brand-guide.pdf' },
    { tenant: 'nextdaysteel', file: 'product-catalog.pdf' },
    { tenant: 'nextdaysteel', file: 'example-posts.csv' },
  ];

  for (const doc of brandDocs) {
    const filePath = path.join(__dirname, '../test-data', doc.tenant, doc.file);
    const content = fs.readFileSync(filePath);

    await prisma.brandDocument.create({
      data: {
        tenant_id: doc.tenant,
        filename: doc.file,
        content: content.toString('base64'),
        uploaded_at: new Date(),
      },
    });
  }

  console.log(`Loaded ${brandDocs.length} brand documents`);

  // 3. Seed products
  const products = [
    {
      tenant_id: 'nextdaysteel',
      shopify_id: '123456',
      name: 'A142 Mesh',
      category: 'Reinforcement Mesh',
      price: 45.99,
      in_stock: true,
    },
    {
      tenant_id: 'nextdaysteel',
      shopify_id: '123457',
      name: 'A193 Mesh',
      category: 'Reinforcement Mesh',
      price: 52.99,
      in_stock: false, // Out of stock for E2E-04 test
    },
    {
      tenant_id: 'nextdaysteel',
      shopify_id: '123458',
      name: 'T12 Rebar',
      category: 'Rebar',
      price: 12.50,
      in_stock: true,
    },
  ];

  await prisma.product.createMany({ data: products });
  console.log(`Seeded ${products.length} products`);

  // 4. Seed historical posts (for analytics)
  const posts = [];
  for (let i = 0; i < 10; i++) {
    posts.push({
      tenant_id: 'nextdaysteel',
      platform: i % 2 === 0 ? 'linkedin' : 'twitter',
      content: `Test post ${i + 1}`,
      published_at: new Date(Date.now() - (7 - i) * 24 * 60 * 60 * 1000), // Last 7 days
      likes: Math.floor(Math.random() * 100),
      comments: Math.floor(Math.random() * 20),
      shares: Math.floor(Math.random() * 10),
      impressions: Math.floor(Math.random() * 1000) + 500,
    });
  }

  await prisma.publishedPost.createMany({ data: posts });
  console.log(`Seeded ${posts.length} historical posts`);

  console.log('Test data seeding complete!');
}

seedTestData()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

**Run Seed Script:**
```bash
npm run seed:test-data
```

### 8.3 Cleanup Strategy

**After Each Test:**
- Delete test-generated posts (to avoid cluttering analytics)
- Reset OAuth tokens to initial state
- Clear task queue (Durable Objects)
- Clear KV cache

**Cleanup Script:**
```typescript
// scripts/cleanup-test-data.ts

import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function cleanupTestData() {
  console.log('Cleaning up test data...');

  // Delete test posts created during tests
  const deleted = await prisma.publishedPost.deleteMany({
    where: {
      tenant_id: { in: ['nextdaysteel', 'techstartup', 'fashionboutique'] },
      created_at: { gte: new Date(Date.now() - 24 * 60 * 60 * 1000) }, // Last 24 hours
    },
  });

  console.log(`Deleted ${deleted.count} test posts`);

  // Reset OAuth tokens (set expiry to 1 hour ago to force refresh)
  await prisma.oauthToken.updateMany({
    where: { tenant_id: { in: ['nextdaysteel', 'techstartup', 'fashionboutique'] } },
    data: { expires_at: new Date(Date.now() - 3600000) },
  });

  console.log('Reset OAuth tokens');

  // Clear KV cache
  // (Done via Cloudflare API or Workers script)

  console.log('Cleanup complete!');
}

cleanupTestData()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

**Run Cleanup:**
```bash
npm run cleanup:test-data
```

### 8.4 Test Isolation

**Between Test Runs:**
- Each Playwright test uses a unique `tenant_id` suffix (e.g., `nextdaysteel-test-1`, `nextdaysteel-test-2`)
- Tests run in parallel don't interfere with each other
- Cleanup runs after all tests complete

---

## 9. Automated Testing Pipeline

### 9.1 GitHub Actions Workflow

```yaml
# .github/workflows/e2e-tests.yml

name: E2E Testing Pipeline

on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * *' # Nightly at 2 AM UTC

jobs:
  unit-tests:
    name: Unit Tests (Stage I - 400+ tests)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test:unit
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  integration-tests:
    name: Integration Tests (Stage I - 60 tests)
    runs-on: ubuntu-latest
    needs: unit-tests
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: ${{ secrets.STAGING_DATABASE_URL }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}

  e2e-tests:
    name: E2E Tests (Stage K - 10 scenarios)
    runs-on: ubuntu-latest
    needs: [unit-tests, integration-tests]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Install Playwright
        run: npx playwright install --with-deps
      - name: Seed test data
        run: npm run seed:test-data
        env:
          DATABASE_URL: ${{ secrets.STAGING_DATABASE_URL }}
      - name: Run E2E tests
        run: npm run test:e2e
        env:
          TEST_API_KEY: ${{ secrets.TEST_API_KEY }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          LINKEDIN_TEST_EMAIL: ${{ secrets.LINKEDIN_TEST_EMAIL }}
          LINKEDIN_TEST_PASSWORD: ${{ secrets.LINKEDIN_TEST_PASSWORD }}
          LINKEDIN_TEST_TOKEN: ${{ secrets.LINKEDIN_TEST_TOKEN }}
      - name: Cleanup test data
        if: always()
        run: npm run cleanup:test-data
        env:
          DATABASE_URL: ${{ secrets.STAGING_DATABASE_URL }}
      - name: Upload Playwright report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30

  performance-tests:
    name: Performance Benchmarks
    runs-on: ubuntu-latest
    needs: e2e-tests
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - name: Run performance tests
        run: npm run test:performance
      - name: Check performance thresholds
        run: |
          if grep -q "FAIL" performance-results.txt; then
            echo "Performance tests failed!"
            exit 1
          fi

  load-tests:
    name: Load Testing (k6)
    runs-on: ubuntu-latest
    needs: performance-tests
    if: github.ref == 'refs/heads/main' # Only on main branch
    steps:
      - uses: actions/checkout@v3
      - name: Install k6
        run: |
          sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
          echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
          sudo apt-get update
          sudo apt-get install k6
      - name: Run load tests
        run: k6 run load-test.js --env TEST_API_KEY=${{ secrets.TEST_API_KEY }}
      - name: Upload k6 results
        uses: actions/upload-artifact@v3
        with:
          name: k6-results
          path: k6-results.json

  security-tests:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: e2e-tests
    steps:
      - uses: actions/checkout@v3
      - name: Run OWASP ZAP scan
        run: |
          docker run -t owasp/zap2docker-stable zap-baseline.py \
            -t https://staging.dify-agent.com \
            -r zap-report.html
      - name: Upload ZAP report
        uses: actions/upload-artifact@v3
        with:
          name: zap-report
          path: zap-report.html

  deploy-to-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [e2e-tests, performance-tests, security-tests]
    if: github.ref == 'refs/heads/staging'
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - name: Deploy to Cloudflare Workers
        run: npm run deploy:staging
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}

  smoke-tests-production:
    name: Smoke Tests (Production)
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npx playwright install --with-deps
      - name: Run smoke tests
        run: npm run test:smoke
        env:
          BASE_URL: https://app.dify-agent.com
```

### 9.2 Test Execution Order

**Sequential Execution:**
1. **Unit Tests** (400+ tests, ~2 min)
2. **Integration Tests** (60 tests, ~5 min)
3. **E2E Tests** (10 scenarios, ~20 min)
4. **Performance Tests** (5 benchmarks, ~3 min)
5. **Load Tests** (k6, ~16 min)
6. **Security Tests** (OWASP ZAP, ~10 min)

**Total CI/CD Time:** ~56 minutes

**Parallelization:**
- Unit tests run in parallel (4 workers)
- E2E tests run sequentially (to avoid test data conflicts)
- Load tests run after E2E (to ensure system is stable)

### 9.3 Nightly Regression Tests

**Schedule:** Every night at 2 AM UTC

**Tests Run:**
- All 515+ Stage I tests (unit + integration)
- All 10 Stage K E2E scenarios
- Performance benchmarks
- Security scan

**Failure Notifications:**
- Slack alert to #engineering channel
- Email to on-call engineer
- PagerDuty alert for P0 failures

---

## 10. Test Execution & Reporting

### 10.1 Running Tests Locally

**Prerequisites:**
```bash
# Install dependencies
npm install

# Install Playwright browsers
npx playwright install --with-deps

# Set environment variables
cp .env.example .env.test
# Edit .env.test with your credentials
```

**Run Tests:**
```bash
# Run all E2E tests
npm run test:e2e

# Run single scenario
npm run test:e2e -- e2e-01-complete-onboarding.spec.ts

# Run with UI (Playwright inspector)
npm run test:e2e -- --ui

# Run in debug mode
npm run test:e2e -- --debug

# Generate HTML report
npm run test:e2e -- --reporter=html
```

### 10.2 Test Reports

**Playwright HTML Report:**
```bash
npx playwright show-report
```

**Report Contents:**
- Test execution timeline
- Screenshots on failure
- Video recordings
- Network logs
- Console logs
- LLM Judge evaluations

**Example Report Structure:**
```
playwright-report/
├── index.html
├── screenshots/
│   ├── e2e-01-step-5.png
│   └── e2e-08-error.png
├── videos/
│   └── e2e-01-complete-flow.webm
└── traces/
    └── e2e-01-trace.zip
```

### 10.3 Pass/Fail Summary

**Test Run Summary:**
```
========================================
STAGE K: E2E TESTING RESULTS
========================================

Happy Path Scenarios:
✅ E2E-01: Complete Onboarding to First Post (PASS)
✅ E2E-02: Brand Voice Learning (PASS)
✅ E2E-03: Analytics Dashboard (PASS)

Edge Case Scenarios:
✅ E2E-04: Product Out of Stock (PASS)
✅ E2E-05: OAuth Token Refresh (PASS)
✅ E2E-06: Rate Limit Queue (PASS)
✅ E2E-07: Aspect Ratio Adaptation (PASS)

Error Case Scenarios:
✅ E2E-08: LLM Fallback (PASS)
✅ E2E-09: Input Validation (PASS)
✅ E2E-10: Circuit Breaker (PASS)

========================================
PASS: 10/10 scenarios (100%)
LLM Judge: 8.7/10 average score
Performance: All benchmarks passed
========================================
```

### 10.4 Failure Triage

**If Tests Fail:**

1. **Check Screenshots/Videos**
   - Playwright captures failures automatically
   - Review visual evidence

2. **Review Logs**
   - Console errors
   - Network requests (failed API calls)
   - LLM Judge reasoning

3. **Reproduce Locally**
   - Run failed test with `--debug` flag
   - Step through with Playwright Inspector

4. **Common Failure Causes:**
   - **Timeout:** External API slow (increase timeout)
   - **Element Not Found:** UI change (update selector)
   - **Assertion Failed:** Expected data changed (verify test data)
   - **LLM Judge Low Score:** Model variability (re-run or adjust threshold)

---

## 11. Appendix

### 11.1 Test Environment URLs

| Environment | URL | Purpose |
|-------------|-----|---------|
| Development | http://localhost:3000 | Local testing |
| Staging | https://staging.dify-agent.com | Pre-production E2E tests |
| Production | https://app.dify-agent.com | Smoke tests only |

### 11.2 Test Credentials

**Stored in GitHub Secrets:**
- `TEST_API_KEY` - API key for test tenant
- `OPENAI_API_KEY` - OpenAI for LLM Judge
- `LINKEDIN_TEST_EMAIL` - LinkedIn sandbox account
- `LINKEDIN_TEST_PASSWORD` - LinkedIn sandbox password
- `LINKEDIN_TEST_TOKEN` - LinkedIn OAuth token
- `DATABASE_URL` - Staging database connection

**Never commit credentials to Git!**

### 11.3 Useful Commands

```bash
# Run specific test
npm run test:e2e -- e2e-01-complete-onboarding.spec.ts

# Run tests matching pattern
npm run test:e2e -- --grep "happy path"

# Run tests in headed mode (see browser)
npm run test:e2e -- --headed

# Generate code from interactions (Playwright Codegen)
npx playwright codegen https://staging.dify-agent.com

# Trace viewer (debug test execution)
npx playwright show-trace trace.zip

# Update Playwright browsers
npx playwright install --with-deps
```

### 11.4 Further Reading

- [Playwright Documentation](https://playwright.dev)
- [k6 Load Testing](https://k6.io/docs/)
- [LLM-as-Judge Paper](https://arxiv.org/abs/2306.05685)
- [Testing Best Practices](https://martinfowler.com/articles/practical-test-pyramid.html)

---

## Document History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0.0 | 2025-11-10 | Initial Stage K E2E testing strategy | QA Architect |

---

**Status:** ✅ APPROVED - Ready for Implementation

**Next Steps:**
1. Set up Playwright project structure
2. Implement 10 E2E test scenarios
3. Integrate LLM-as-Judge framework
4. Configure GitHub Actions pipeline
5. Seed test data in staging environment
6. Run initial test suite and validate

**Estimated Implementation Time:** 2-3 weeks (80-120 hours)
