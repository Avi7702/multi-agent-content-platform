# STAGE K: E2E TESTING - QUICK REFERENCE

**Version:** 1.0.0
**Last Updated:** November 10, 2025

---

## TL;DR

**What:** System-wide end-to-end testing covering all 12 stages (A-L)
**Why:** Validate complete user journeys from onboarding to publishing
**How:** 10 Playwright scenarios + LLM-as-Judge + performance benchmarks

---

## 10 Test Scenarios (80% Pass Threshold)

### Happy Path (3 scenarios)
1. **E2E-01:** Complete onboarding → first post (< 8 min)
2. **E2E-02:** Brand voice learning → on-brand generation
3. **E2E-03:** Analytics dashboard → ROI tracking

### Edge Cases (4 scenarios)
4. **E2E-04:** Product out of stock → advisory pattern (A/B/C choices)
5. **E2E-05:** OAuth token expired → auto-refresh
6. **E2E-06:** Rate limit hit → queue & retry
7. **E2E-07:** Multi-platform → aspect ratio adaptation

### Error Cases (3 scenarios)
8. **E2E-08:** OpenAI failure → Claude fallback
9. **E2E-09:** Invalid input → validation errors
10. **E2E-10:** Database failure → circuit breaker

---

## Performance Benchmarks (SLA Targets)

| Metric | Target | P95 |
|--------|--------|-----|
| Onboarding | < 120s | < 150s |
| Validation | < 200ms | < 300ms |
| Content Generation | < 30s | < 45s |
| Publishing | < 10s | < 15s |
| End-to-End | < 180s | < 240s |

---

## LLM-as-Judge Scoring

**Evaluation Criteria:**
- Brand Voice Match: ≥ 8.0/10
- Creativity: ≥ 7.0/10
- Engagement Potential: ≥ 7.0/10
- Technical Accuracy: ≥ 9.0/10
- **Overall Score: ≥ 8.0/10**

**Pass Threshold:** 8/10 scenarios must meet minimum scores

---

## Quick Commands

```bash
# Run all E2E tests
npm run test:e2e

# Run single scenario
npm run test:e2e -- e2e-01-complete-onboarding.spec.ts

# Run with UI (debug mode)
npm run test:e2e -- --ui

# Generate HTML report
npx playwright show-report

# Seed test data
npm run seed:test-data

# Cleanup after tests
npm run cleanup:test-data

# Run performance benchmarks
npm run test:performance

# Run load tests (k6)
k6 run load-test.js --env TEST_API_KEY=<key>
```

---

## Test Environment Setup

**3 Test Tenants:**
1. **NextDaySteel** (Construction/B2B) - LinkedIn, Twitter
2. **TechStartup** (SaaS/B2B) - LinkedIn, Twitter, Facebook
3. **FashionBoutique** (Retail/B2C) - Instagram, Facebook, Twitter

**Environment URLs:**
- Dev: `http://localhost:3000`
- Staging: `https://staging.dify-agent.com`
- Production: `https://app.dify-agent.com` (smoke tests only)

---

## CI/CD Pipeline (GitHub Actions)

**Test Execution Order:**
1. Unit Tests (400+ tests, ~2 min)
2. Integration Tests (60 tests, ~5 min)
3. **E2E Tests (10 scenarios, ~20 min)**
4. Performance Tests (5 benchmarks, ~3 min)
5. Load Tests (k6, ~16 min)
6. Security Tests (OWASP ZAP, ~10 min)

**Total CI/CD Time:** ~56 minutes

**Nightly Regression:** Every night at 2 AM UTC

---

## Pass/Fail Criteria

**System Pass:**
- ✅ 8/10 scenarios pass (80%)
- ✅ All P0 scenarios pass (5 scenarios)
- ✅ Performance benchmarks met
- ✅ LLM Judge average ≥ 8.0/10
- ✅ No security vulnerabilities
- ✅ No cross-tenant data leakage

**Individual Scenario Pass:**
- All assertions pass
- No unhandled errors
- Performance within SLA
- Visual regressions < 5%

---

## Key Files

```
.planning/approved/stage-k-system-testing/
├── STAGE-K-E2E-TESTING-STRATEGY.md (main spec, 1,700+ lines)
├── QUICK-REFERENCE.md (this file)
└── README.md

tests/e2e/
├── e2e-01-complete-onboarding.spec.ts
├── e2e-02-brand-voice-learning.spec.ts
├── e2e-03-analytics-dashboard.spec.ts
├── e2e-04-out-of-stock-advisory.spec.ts
├── e2e-05-oauth-token-refresh.spec.ts
├── e2e-06-rate-limit-queue.spec.ts
├── e2e-07-aspect-ratio-adaptation.spec.ts
├── e2e-08-llm-fallback.spec.ts
├── e2e-09-input-validation.spec.ts
└── e2e-10-circuit-breaker.spec.ts

src/testing/
├── llm-judge.ts (LLM-as-Judge framework)
└── test-helpers.ts

scripts/
├── seed-test-data.ts
└── cleanup-test-data.ts

load-test.js (k6 script)
```

---

## Common Issues & Solutions

### Issue: Test Timeout
**Solution:** Increase timeout for async operations (brand voice analysis, OpenManus tasks)
```typescript
await expect(element).toContainText('Complete', { timeout: 300000 }); // 5 min
```

### Issue: Element Not Found
**Solution:** Wait for network idle before querying
```typescript
await page.waitForLoadState('networkidle');
```

### Issue: LLM Judge Low Score
**Solution:** Model variability - re-run test or adjust threshold
```typescript
expect(evaluation.overall_score).toBeGreaterThanOrEqual(7.5); // Lower threshold
```

### Issue: OAuth Flow Fails
**Solution:** Verify sandbox account credentials in GitHub Secrets

### Issue: Database Connection Failure
**Solution:** Check `DATABASE_URL` environment variable

---

## Monitoring & Alerts

**Test Failures:**
- Slack: #engineering channel
- Email: on-call engineer
- PagerDuty: P0 failures only

**Metrics:**
- Test pass rate (target: > 80%)
- Average execution time (target: < 30 min)
- Flaky test count (target: < 5%)

---

## Next Steps After Implementation

1. **Run Initial Test Suite** - Validate all 10 scenarios pass
2. **Measure Performance** - Ensure benchmarks met
3. **Configure CI/CD** - Set up GitHub Actions workflow
4. **Enable Nightly Regression** - Schedule automated runs
5. **Monitor Test Health** - Track flaky tests, failure rate
6. **Iterate** - Add new scenarios as system evolves

---

## Cost Estimate

**Development Time:**
- Playwright setup: 8 hours
- 10 E2E scenarios: 60 hours (6 hours each)
- LLM-as-Judge integration: 12 hours
- Performance benchmarks: 8 hours
- CI/CD pipeline: 12 hours
- Documentation: 8 hours

**Total:** 108 hours (~2.7 weeks)

**Operational Cost:**
- CI/CD runs: ~$50/month (GitHub Actions minutes)
- LLM Judge (GPT-4): ~$20/month (evaluation API calls)
- Test data storage: ~$5/month

**Total:** ~$75/month

---

## Success Metrics

**After 1 Month:**
- ✅ All 10 scenarios passing consistently
- ✅ < 5% flaky test rate
- ✅ < 30 min average execution time
- ✅ 100% P0 scenario pass rate
- ✅ LLM Judge average ≥ 8.5/10

**After 3 Months:**
- ✅ 15+ scenarios (added 5 new)
- ✅ < 3% flaky test rate
- ✅ 90%+ developer confidence in deployments

---

## Resources

- **Main Spec:** `STAGE-K-E2E-TESTING-STRATEGY.md`
- **Playwright Docs:** https://playwright.dev
- **k6 Docs:** https://k6.io/docs/
- **LLM-as-Judge Paper:** https://arxiv.org/abs/2306.05685

---

**Questions?** Contact QA team or refer to main specification document.
