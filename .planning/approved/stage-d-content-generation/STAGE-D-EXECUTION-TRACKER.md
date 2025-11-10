# STAGE D EXECUTION TRACKER - Content Generation Workflow

**Version**: 1.0.0
**Status**: ⏸️ NOT STARTED (Pending LinkedIn Approval + Resources)
**Framework**: Milestone-Kanban Hybrid with Weekly OKRs
**Timeline**: 5 weeks (25 working days)
**Start Date**: TBD (upon approval)
**Expected Completion**: Week 5 (Day 25)

---

## QUICK STATUS

| Metric | Status |
|--------|--------|
| **Overall Progress** | 0% (0/25 days) |
| **Current Week** | Not started |
| **Current Phase** | Pre-implementation |
| **Blockers** | LinkedIn approval pending |
| **On Track?** | ⏸️ Awaiting start |

---

## WEEK 1: REQUEST PROCESSING (Days 1-5)

**Week 1 OKRs**:
- **O1**: Implement intelligent user request processing
  - KR1: Intent classification accuracy ≥85%
  - KR2: Entity extraction accuracy ≥90%
  - KR3: Platform detection accuracy ≥90%
  - KR4: Conversation state management working

**Milestone**: User Request Processing Complete ✅

### Day 1-2: Intent Classification

**Tasks**:
- [ ] Set up hybrid NLU + LLM architecture
- [ ] Integrate spaCy `en_core_web_trf` model
- [ ] Implement confidence scoring (0-1 scale)
- [ ] Create intent decision tree (≥0.90 execute, 0.60-0.89 confirm, <0.60 clarify)
- [ ] Test with 20 sample user inputs
- [ ] Measure accuracy (target: ≥85%)

**Deliverables**:
- `intent-classifier.ts` service
- Unit tests (95%+ coverage)
- Accuracy report

**Dependencies**: None

**Status**: ⬜ Not started

---

### Day 3-4: Entity Extraction + Fuzzy Matching

**Tasks**:
- [ ] Implement product entity extraction (spaCy NER)
- [ ] Add fuzzy matching with RapidFuzz (handle variants)
- [ ] Integrate Supabase product search
- [ ] Add semantic search fallback (pgvector)
- [ ] Handle disambiguation ("Which product? A/B/C")
- [ ] Test with 30 product queries
- [ ] Measure extraction accuracy (target: ≥90%)

**Deliverables**:
- `entity-extractor.ts` service
- Product disambiguation workflow
- Test results

**Dependencies**: Supabase connection

**Status**: ⬜ Not started

---

### Day 5: Platform Detection + State Management

**Tasks**:
- [ ] Implement platform preference detection
- [ ] Add explicit keyword matching ("LinkedIn post")
- [ ] Add context-based inference ("professional update")
- [ ] Set up Redis for conversation state
- [ ] Implement context carryover (entity salience tracking)
- [ ] Create session management (30min TTL, extend on activity)
- [ ] Test multi-turn conversations (10 scenarios)

**Deliverables**:
- `platform-detector.ts` service
- `conversation-state.ts` manager
- Redis integration
- Multi-turn test results

**Dependencies**: Redis deployment

**Status**: ⬜ Not started

---

**Week 1 Success Criteria**:
- ✅ Intent classification ≥85% accurate
- ✅ Entity extraction ≥90% accurate
- ✅ Platform detection ≥90% accurate
- ✅ Conversation state persists across turns
- ✅ All unit tests passing

---

## WEEK 2: CONTENT GENERATION (Days 6-10)

**Week 2 OKRs**:
- **O1**: Generate high-quality text and images
  - KR1: Brand voice score ≥90% average
  - KR2: Image quality score ≥85% average
  - KR3: Generation time <15 seconds (text + images)
  - KR4: User approval rate ≥80%

**Milestone**: Content Generation Complete ✅

### Day 6-7: Text Generation with Few-Shot Prompting

**Tasks**:
- [ ] Enhance `generate-with-quality-gate.ts` with few-shot prompting
- [ ] Implement example retrieval from Vectorize (top-5 similar posts)
- [ ] Build platform-specific prompt templates
- [ ] Add character count validation (per platform)
- [ ] Implement hashtag generation (3-5 for LinkedIn, 1-2 for Twitter)
- [ ] Add CTA requirement enforcement
- [ ] Test with 50 generation requests
- [ ] Measure brand score distribution (target: 90%+ average)

**Deliverables**:
- Enhanced `generate-with-quality-gate.ts`
- Few-shot prompt templates
- Brand score analysis report

**Dependencies**: Stage C brand voice Vectorize index

**Status**: ⬜ Not started

---

### Day 8-9: Image Generation (Gemini 2.5 Flash)

**Tasks**:
- [ ] Set up Gemini 2.5 Flash API integration
- [ ] Implement batch generation (4 variants per request)
- [ ] Add platform-specific aspect ratios (1.91:1, 16:9, 1:1, 9:16)
- [ ] Create prompt templates for B2B construction products
- [ ] Implement quality verification (Google Vision API)
- [ ] Add fallback to Cloudflare Workers AI (Leonardo) if Gemini fails
- [ ] Test generation speed (target: <10 seconds for 4 images)
- [ ] Test quality scores (target: ≥85% average)

**Deliverables**:
- `image-generator.ts` service
- Batch processing implementation
- Quality verification pipeline
- Performance benchmarks

**Dependencies**: Gemini API access, Google Vision API

**Status**: ⬜ Not started

---

### Day 10: Multi-Dimensional Brand Scoring

**Tasks**:
- [ ] Enhance BrandScorer with dimensional analysis
- [ ] Implement LLM-based scoring (tone, terminology, structure, readability)
- [ ] Create composite score calculation (vector 50% + dimensions 50%)
- [ ] Add recommendation generation (below-threshold feedback)
- [ ] Build scoring dashboard (real-time metrics)
- [ ] Test scoring accuracy (compare with manual ratings)
- [ ] Document scoring rubric

**Deliverables**:
- Enhanced `brand-scorer.ts`
- Dimensional scoring implementation
- Scoring documentation

**Dependencies**: Week 1 Day 6-7 completion

**Status**: ⬜ Not started

---

**Week 2 Success Criteria**:
- ✅ Text generation with 90%+ brand score
- ✅ Image generation <10 seconds (4 variants)
- ✅ Image quality ≥85% average
- ✅ Multi-dimensional scoring operational
- ✅ All quality gates functional

---

## WEEK 3: PREVIEW & APPROVAL (Days 11-15)

**Week 3 OKRs**:
- **O1**: Build intuitive preview and approval workflow
  - KR1: Preview renders correctly in Dify chat
  - KR2: Multi-variant selection working (A/B/C/D)
  - KR3: Revision workflow functional
  - KR4: User feedback collection implemented

**Milestone**: Preview & Approval System Complete ✅

### Day 11-12: Conversational Preview Format

**Tasks**:
- [ ] Design markdown preview template (comprehensive format)
- [ ] Implement inline image rendering
- [ ] Add metadata display (character count, quality scores, hashtags)
- [ ] Create quality verification summary (✓/⚠️ indicators)
- [ ] Build recommendation engine ("I recommend Option A because...")
- [ ] Test preview rendering in Dify
- [ ] Ensure mobile-friendly display

**Deliverables**:
- `preview-formatter.ts` service
- Markdown preview templates
- Dify rendering tests

**Dependencies**: Week 2 completion (content + images ready)

**Status**: ⬜ Not started

---

### Day 13-14: Multi-Option Presentation + Selection

**Tasks**:
- [ ] Implement A/B/C/D variant presentation
- [ ] Add style descriptions ("Split-screen professional", "Technical blueprint")
- [ ] Add "Best for" recommendations per variant
- [ ] Build user choice parser (accept "A", "option A", "first one")
- [ ] Create "Request changes" workflow
- [ ] Add "Regenerate all" option
- [ ] Test selection with 20 user inputs (various phrasings)

**Deliverables**:
- `option-presenter.ts` service
- Choice parsing logic
- User input flexibility tests

**Dependencies**: Day 11-12 preview format

**Status**: ⬜ Not started

---

### Day 15: Revision Workflow + Change Tracking

**Tasks**:
- [ ] Implement revision request handling
- [ ] Add change tracking (before/after comparison)
- [ ] Create version history storage (D1 database)
- [ ] Build feedback collection (track user edits)
- [ ] Add revision quality re-check
- [ ] Implement diff display (show what changed)
- [ ] Test revision cycles (1-3 iterations)

**Deliverables**:
- `revision-handler.ts` service
- D1 schema for version history
- Feedback tracking system

**Dependencies**: Content generation + brand scoring

**Status**: ⬜ Not started

---

**Week 3 Success Criteria**:
- ✅ Preview displays correctly with all metadata
- ✅ 4 image variants presented clearly
- ✅ User selection parsing works ≥95% of time
- ✅ Revision workflow functional
- ✅ Change tracking operational

---

## WEEK 4: MULTI-PLATFORM PUBLISHING (Days 16-20)

**Week 4 OKRs**:
- **O1**: Implement reliable multi-platform publishing
  - KR1: LinkedIn publishing working (post approval obtained)
  - KR2: Twitter publishing working (hybrid v1.1 + v2)
  - KR3: Facebook publishing working
  - KR4: Error rate <5%, retry logic functional

**Milestone**: Publishing Infrastructure Complete ✅

### Day 16-17: LinkedIn API Integration

**Tasks**:
- [ ] ⚠️ **BLOCKER**: Obtain LinkedIn Marketing Developer approval
- [ ] Implement OAuth 2.0 with PKCE flow
- [ ] Build 3-step publishing (register upload → upload image → create post)
- [ ] Add access token management (refresh, expiry handling)
- [ ] Implement rate limit monitoring (X-RateLimit headers)
- [ ] Add idempotency protection (Redis caching)
- [ ] Test with 10 sample posts
- [ ] Extract and return post URL

**Deliverables**:
- `publish-linkedin.ts` service
- OAuth token manager
- Rate limit monitor
- Test results

**Dependencies**: LinkedIn approval (CRITICAL PATH)

**Status**: ⬜ Not started - **BLOCKED by approval**

---

### Day 18: Twitter/X Hybrid API Integration

**Tasks**:
- [ ] Implement OAuth 2.0 with PKCE (v2 API)
- [ ] Implement OAuth 1.0a for media upload (v1.1 API)
- [ ] Build 3-step media upload (INIT → APPEND → FINALIZE)
- [ ] Build tweet creation (v2 API with media_id)
- [ ] Add rate limit handling (1,500 posts/month free tier)
- [ ] Implement error code handling (187 duplicate, 186 too long)
- [ ] Test with 10 sample tweets

**Deliverables**:
- `publish-twitter.ts` service
- Hybrid API implementation
- Error handling tests

**Dependencies**: Twitter Developer account

**Status**: ⬜ Not started

---

### Day 19: Facebook Graph API Integration

**Tasks**:
- [ ] Implement OAuth 2.0 flow
- [ ] Build page access token retrieval
- [ ] Implement photo upload (binary or URL)
- [ ] Add caption/message support
- [ ] Handle publishing status
- [ ] Test with 10 sample posts
- [ ] Extract and return post URL

**Deliverables**:
- `publish-facebook.ts` service
- Page token management
- Test results

**Dependencies**: Facebook App approval (if needed)

**Status**: ⬜ Not started

---

### Day 20: Error Handling + Retry Logic

**Tasks**:
- [ ] Implement exponential backoff with jitter
- [ ] Add retry logic (max 5 attempts)
- [ ] Build error categorization (rate limit, auth, server, client)
- [ ] Implement circuit breaker pattern
- [ ] Add dead letter queue for failures
- [ ] Build monitoring and alerting
- [ ] Test all error scenarios (rate limit, timeout, 500 error, auth failure)

**Deliverables**:
- `retry-handler.ts` service
- Circuit breaker implementation
- Error monitoring dashboard
- Test suite for error scenarios

**Dependencies**: All publishing services (LinkedIn, Twitter, Facebook)

**Status**: ⬜ Not started

---

**Week 4 Success Criteria**:
- ✅ LinkedIn publishing working (if approved)
- ✅ Twitter publishing working
- ✅ Facebook publishing working
- ✅ Error rate <5% in testing
- ✅ Retry logic handling 429, 5xx errors
- ✅ Post URLs returned successfully

---

## WEEK 5: TESTING & DEPLOYMENT (Days 21-25)

**Week 5 OKRs**:
- **O1**: Ensure production readiness
  - KR1: 10/10 test scenarios pass
  - KR2: Performance targets met (<30s total)
  - KR3: Error rate <5%
  - KR4: User acceptance ≥80%

**Milestone**: Stage D Complete & Deployed ✅

### Day 21-22: End-to-End Testing

**Tasks**:
- [ ] Run all 10 test scenarios (3 happy path, 4 edge case, 3 error)
- [ ] Measure performance (request processing, generation, publishing)
- [ ] Test multi-turn conversations (context preservation)
- [ ] Verify quality gates (brand score, image quality)
- [ ] Test platform-specific features (character limits, image sizes)
- [ ] Document test results
- [ ] Fix critical bugs

**Test Scenarios**:
1. ✅ Happy: "Create LinkedIn post about A142 mesh"
2. ✅ Happy: "Quick tweet about next-day delivery for T12 rebar"
3. ✅ Happy: "Professional Facebook post for new mesh products"
4. ⚠️ Edge: Non-existent product
5. ⚠️ Edge: Ambiguous request (no platform, no product)
6. ⚠️ Edge: 3-turn conversation with context
7. ⚠️ Edge: 3 revision requests
8. ❌ Error: Rate limit on LinkedIn
9. ❌ Error: Image generation timeout
10. ❌ Error: User rejects all variants

**Deliverables**:
- Test execution report
- Performance benchmarks
- Bug fix log

**Dependencies**: All Week 1-4 components complete

**Status**: ⬜ Not started

---

### Day 23-24: Dify Integration + Configuration

**Tasks**:
- [ ] Configure Dify chatflow (nodes, conditions, tools)
- [ ] Set up conversation variables
- [ ] Connect all MCP Gateway tools
- [ ] Test complete workflow in Dify
- [ ] Configure error handling in chatflow
- [ ] Add user feedback collection
- [ ] Test in Dify development environment
- [ ] Document chatflow structure

**Deliverables**:
- Dify chatflow configuration (JSON export)
- Integration documentation
- Test results in Dify

**Dependencies**: MCP Gateway tools operational

**Status**: ⬜ Not started

---

### Day 25: Production Deployment + Monitoring

**Tasks**:
- [ ] Deploy MCP Gateway to Cloudflare Workers (production)
- [ ] Deploy Dify chatflow (staging → production)
- [ ] Configure monitoring (Cloudflare Analytics, error tracking)
- [ ] Set up alerting (>5% error rate, >30s latency)
- [ ] Create user documentation
- [ ] Conduct internal beta test (5 users)
- [ ] Monitor for 24 hours
- [ ] Document known issues + roadmap

**Deliverables**:
- Production deployment
- Monitoring dashboard
- User documentation
- Stage D completion report

**Dependencies**: All testing passed

**Status**: ⬜ Not started

---

**Week 5 Success Criteria**:
- ✅ All 10 test scenarios pass
- ✅ Performance <30 seconds total
- ✅ Error rate <5%
- ✅ Dify integration working
- ✅ Deployed to production
- ✅ Monitoring operational

---

## OVERALL MILESTONES

| Milestone | Target Date | Status | Dependencies |
|-----------|-------------|--------|--------------|
| **M1: Request Processing** | Week 1 (Day 5) | ⬜ Not started | None |
| **M2: Content Generation** | Week 2 (Day 10) | ⬜ Not started | Stage C KB |
| **M3: Preview & Approval** | Week 3 (Day 15) | ⬜ Not started | M2 complete |
| **M4: Publishing** | Week 4 (Day 20) | ⬜ Not started | LinkedIn approval |
| **M5: Production Ready** | Week 5 (Day 25) | ⬜ Not started | M1-M4 complete |

---

## CRITICAL PATH

```
LinkedIn Approval (BLOCKER)
    ↓
Week 1: Request Processing
    ↓
Week 2: Content Generation (requires Stage C brand voice)
    ↓
Week 3: Preview & Approval
    ↓
Week 4: Publishing (requires LinkedIn approval)
    ↓
Week 5: Testing & Deployment
```

**CRITICAL**: LinkedIn Marketing Developer approval can take 2-4 weeks. Apply immediately to avoid blocking Week 4.

---

## BLOCKERS & RISKS

### Active Blockers

| Blocker | Impact | Mitigation | Status |
|---------|--------|------------|--------|
| **LinkedIn Approval** | High - Blocks Week 4 | Apply immediately, use Twitter/Facebook first | 🔴 Not applied |
| **Resource Allocation** | High - Blocks start | Assign 1 full-stack + 1 AI/ML dev | 🟡 Pending |
| **Stage C Completion** | Medium - Need brand voice | Complete Stage C planning first | 🟢 Done |

### Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| LinkedIn approval delayed | High | High | Focus on Twitter/Facebook Week 4 |
| Image quality issues | Medium | Medium | Multi-model fallback (DALL-E 3, Workers AI) |
| Performance <30s | Low | Medium | Optimize batch processing, caching |
| User dissatisfaction | Medium | High | Advisory pattern (user control) |

---

## VELOCITY TRACKING

**Planned vs Actual**:

| Week | Planned Tasks | Completed | % Complete | On Track? |
|------|---------------|-----------|------------|-----------|
| Week 1 | 10 | 0 | 0% | ⏸️ Not started |
| Week 2 | 12 | 0 | 0% | ⏸️ Not started |
| Week 3 | 10 | 0 | 0% | ⏸️ Not started |
| Week 4 | 12 | 0 | 0% | ⏸️ Not started |
| Week 5 | 11 | 0 | 0% | ⏸️ Not started |
| **Total** | **55** | **0** | **0%** | ⏸️ **Pending start** |

---

## DAILY STANDUP FORMAT

**What did we accomplish yesterday?**
- [List completed tasks]

**What are we working on today?**
- [Current day's tasks from tracker]

**Any blockers?**
- [List blockers with mitigation plan]

**Are we on track for weekly OKRs?**
- [Yes/No with explanation]

---

## WEEKLY REVIEW FORMAT

**Week X Summary**:
- Planned tasks: X
- Completed: Y (Z%)
- Carry-over: A tasks
- Blockers resolved: B
- New blockers: C

**OKR Achievement**:
- O1: [Status] - KR1 [✅/❌], KR2 [✅/❌], KR3 [✅/❌]

**Next Week Preview**:
- Focus: [Main objectives]
- Critical tasks: [Must-complete items]
- Dependencies: [What we need]

---

## SUCCESS METRICS (FINAL)

**At Stage D Completion**:

✅ **Functional**:
- Intent classification: ≥85% accuracy
- Entity extraction: ≥90% accuracy
- Brand voice score: ≥90% average
- Image quality: ≥85% average
- Publishing success rate: ≥95%

✅ **Performance**:
- Request processing: <3 seconds
- Content generation: <15 seconds
- Total workflow: <30 seconds
- Error rate: <5%

✅ **User Experience**:
- Approval rate: ≥80%
- Revision rate: <20%
- User satisfaction: ≥80%
- Conversation success: ≥90%

---

## HANDOFF TO STAGE E

**Prerequisites for Stage E (Multi-Platform Enhancement)**:
- ✅ Stage D content generation working
- ✅ Publishing to all 3 platforms operational
- ✅ Analytics tracking foundation (for Stage F)
- ✅ Performance optimizations complete
- ✅ User feedback collected and analyzed

**Stage E Focus Areas**:
1. Cross-platform content adaptation
2. Platform-specific optimization
3. Image aspect ratio conversion
4. Advanced scheduling
5. Multi-platform analytics

---

**Tracker Status**: ✅ READY FOR USE
**Version**: 1.0.0
**Framework**: Milestone-Kanban Hybrid with Weekly OKRs
**Last Updated**: November 9, 2025
**Next Action**: Apply for LinkedIn approval, allocate resources, begin Week 1
