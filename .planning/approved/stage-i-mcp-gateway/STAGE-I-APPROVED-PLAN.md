# STAGE I: MCP GATEWAY EXPANSION - COMPREHENSIVE APPROVED PLAN

**Version:** 1.0.0
**Status:** ✅ APPROVED - Ready for Implementation
**Date:** November 10, 2025
**Plan Type:** MCP Gateway REST API Layer + Tool Expansion
**Timeline:** 12 weeks (3 months)
**Budget:** $405,000 development + $851/month operations

---

## Document Navigation

### Quick Links
1. [Executive Summary](#1-executive-summary)
2. [Architecture Overview](#2-architecture-overview)
3. [Tool Inventory](#3-tool-inventory-44-tools)
4. [REST API Specification](#4-rest-api-specification-24-endpoints)
5. [Security Architecture](#5-security-architecture)
6. [Database Schema](#6-database-schema)
7. [Implementation Roadmap](#7-implementation-roadmap-12-weeks)
8. [Testing Strategy](#8-testing-strategy)
9. [Deployment Strategy](#9-deployment-strategy)
10. [Monitoring & Observability](#10-monitoring--observability)
11. [Cost Analysis](#11-cost-analysis)
12. [Risk Management](#12-risk-management)
13. [Success Criteria](#13-success-criteria)
14. [Documentation Requirements](#14-documentation-requirements)
15. [Team & Resources](#15-team--resources)
16. [Next Steps](#16-next-steps)

---

## 1. Executive Summary

### 1.1 Project Overview

**Goal**: Expand MCP Gateway from 9 tools to 44 tools and add REST API layer for Dify HTTP Tools integration.

**Current State:**
- 9 tools live in production (Stages A/B)
- JSON-RPC 2.0 protocol only
- Basic content generation + publishing
- Single tenant (Next Day Steel)

**Future State:**
- 44 tools across 4 stages (C, D, E, F)
- Dual protocol support (JSON-RPC + REST HTTP)
- Advanced content generation with multi-candidate system
- Multi-platform publishing (LinkedIn, Twitter, Facebook, Instagram)
- Analytics and performance tracking
- Multi-tenant support

### 1.2 Key Objectives

1. **REST API Layer** (Week 1)
   - Add REST HTTP endpoints on top of existing JSON-RPC
   - Enable Dify HTTP Tools integration
   - Maintain backward compatibility

2. **Knowledge Base Tools** (Weeks 3-4, Stage C)
   - Document processing and categorization
   - Vector search and retrieval
   - Gap detection and recommendations

3. **Content Generation Tools** (Weeks 5-7, Stage D)
   - Multi-candidate generation (A/B/C options)
   - Quality guards with auto-fixers
   - Brand voice scoring
   - Learning engine with defect classification

4. **Publishing Tools** (Weeks 8-9, Stage E)
   - Multi-platform publishing (4 social networks)
   - OAuth management
   - Image processing for platform requirements

5. **Analytics Tools** (Weeks 10-11, Stage F)
   - Performance metrics and insights
   - A/B testing infrastructure
   - ROI tracking and prediction

### 1.3 Timeline & Budget

**Total Duration:** 12 weeks (3 months)

**Budget Breakdown:**
- Development: $405,000 (1,080 hours @ $375/hour)
- Operations: $851/month (AI models + cloud infrastructure)
- Per-customer cost: $8.51/month (at 100 customers)

**Team:**
- 1 senior full-stack engineer (Weeks 1-2, REST API layer)
- 2 backend engineers (Weeks 3-11, tool implementation)
- 1 AI/ML engineer (Weeks 5-7, content generation)
- 2 QA engineers (Week 12, comprehensive testing)

### 1.4 Success Criteria

**Technical:**
- ✅ All 44 tools implemented and tested
- ✅ REST API layer functional with 24 endpoints
- ✅ 80%+ test coverage
- ✅ <300ms p95 response time
- ✅ <1% error rate

**Business:**
- ✅ Dify workflows can orchestrate all tools
- ✅ Multi-tenant isolation working
- ✅ Production-ready security (authentication, rate limiting, audit logs)
- ✅ GDPR and SOC 2 compliant

**User Experience:**
- ✅ Conversational advisory pattern working
- ✅ Multi-candidate generation with A/B/C options
- ✅ Publishing to 4 platforms simultaneously
- ✅ Real-time analytics and insights

---

## 2. Architecture Overview

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    DIFY PLATFORM (Orchestration)                 │
│                                                                   │
│  5 Workflows:                                                    │
│  - Chatflow (user interaction)                                   │
│  - Skill Matcher (intent → skill)                               │
│  - Content Generator (multi-candidate generation)                │
│  - Publishing (multi-platform)                                   │
│  - Learning/Analytics (feedback loop)                            │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                │ HTTP POST (REST)
                                │ Authorization: Bearer api_nds_*
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    REST API LAYER (NEW - Week 1)                 │
│                                                                   │
│  Base URL: https://mcp-gateway.workers.dev/api/v1               │
│                                                                   │
│  Features:                                                       │
│  - 24 REST endpoints across 6 categories                         │
│  - Bearer token authentication                                   │
│  - Rate limiting (per tenant, per endpoint category)            │
│  - Request validation (Zod schemas)                              │
│  - REST ↔ JSON-RPC translation                                  │
│  - Audit logging to D1                                           │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                │ JSON-RPC 2.0
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                MCP GATEWAY (Existing + New Tools)                │
│                                                                   │
│  Current (9 tools):                                              │
│  - generate_content, verify_product_image, get_product_image    │
│  - verify_product_context, get_product_price, check_stock       │
│  - send_whatsapp, send_sms, publish_linkedin                    │
│                                                                   │
│  New (35 tools):                                                 │
│  - Stage C: Knowledge Base (7 tools)                            │
│  - Stage D: Content Generation (11 tools)                       │
│  - Stage E: Publishing (9 tools)                                │
│  - Stage F: Analytics (8 tools)                                 │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
                    ▼                       ▼
        ┌──────────────────┐    ┌──────────────────┐
        │  CLOUDFLARE      │    │  EXTERNAL APIS   │
        │                  │    │                  │
        │  - Workers       │    │  - OpenAI GPT-5  │
        │  - D1 Database   │    │  - Claude Sonnet │
        │  - KV Namespaces │    │  - Google Vision │
        │  - Vectorize     │    │  - Gemini Flash  │
        │  - R2 Storage    │    │  - LinkedIn API  │
        │  - Workers AI    │    │  - Twitter API   │
        └──────────────────┘    │  - Facebook API  │
                                │  - Instagram API │
                                │  - Twilio        │
                                │  - Supabase      │
                                │  - Shopify       │
                                └──────────────────┘
```

### 2.2 Current State vs Future State

| Aspect | Current (9 tools) | Future (44 tools) |
|--------|-------------------|-------------------|
| **Protocols** | JSON-RPC only | JSON-RPC + REST HTTP |
| **Stages** | A/B | A/B/C/D/E/F |
| **Content Generation** | Single output | Multi-candidate (A/B/C) |
| **Publishing** | LinkedIn only | LinkedIn, Twitter, Facebook, Instagram |
| **Analytics** | None | Full performance tracking + A/B testing |
| **Knowledge Base** | Hard-coded | Dynamic with vector search |
| **Learning** | None | Defect classification + recommendation engine |
| **Tenancy** | Single tenant | Multi-tenant with isolation |
| **Security** | Basic auth | Full RBAC + audit logs + rate limiting |

### 2.3 Technology Stack

**Backend:**
- Cloudflare Workers (serverless runtime)
- TypeScript (type-safe development)
- Hono.js (web framework, optional)
- Zod (input validation)

**Data Storage:**
- Cloudflare D1 (SQL database)
- Cloudflare KV (key-value store for sessions, cache, rate limits)
- Cloudflare Vectorize (vector embeddings for semantic search)
- Cloudflare R2 (object storage for images)

**AI Models:**
- OpenAI GPT-5 (content generation, intent parsing)
- Claude Sonnet 4.5 (content generation, classification)
- Google Gemini 2.5 Flash (image generation)
- Google Vision API (image verification, face detection)
- Cloudflare Workers AI (embeddings generation)

**External Services:**
- Supabase (products database)
- Shopify API (products, pricing, stock)
- LinkedIn API v2 (social publishing)
- Twitter API v2 (social publishing)
- Facebook Graph API v18 (social publishing)
- Instagram Graph API (social publishing)
- Twilio (WhatsApp, SMS)

**Monitoring:**
- Cloudflare Analytics
- Datadog (metrics, logs, APM)
- PagerDuty (alerting)

---

## 3. Tool Inventory (44 Tools)

### 3.1 Current Tools (9 - Stages A/B)

| Tool | Purpose | Status | Latency |
|------|---------|--------|---------|
| `generate_content` | GPT-5 content generation with brand scoring | ✅ Live | 800ms |
| `verify_product_image` | Google Vision image verification | ✅ Live | 400ms |
| `get_product_image` | Shopify image retrieval + verification | ✅ Live | 300ms |
| `verify_product_context` | Product search and disambiguation | ✅ Live | 250ms |
| `get_product_price` | Live pricing from Shopify | ✅ Live | 200ms |
| `check_stock` | Stock availability checking | ✅ Live | 150ms |
| `send_whatsapp` | Twilio WhatsApp messaging | ✅ Live | 500ms |
| `send_sms` | Twilio SMS messaging | ✅ Live | 400ms |
| `publish_linkedin` | LinkedIn social publishing | ✅ Live | 1200ms |

### 3.2 Stage C: Knowledge Base Tools (7)

#### 3.2.1 Document Processing

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `upload_documents` | Upload PDF/DOCX/CSV/TXT/HTML/MD | Files, tenant_id | entry_ids[] | 2-5s | P1 |
| `categorize_entries` | Auto-categorize into 18+ categories | entry_ids[], tenant_id | categories_assigned | 300-600ms | P1 |
| `generate_embeddings` | Vector embeddings for search | entry_ids[], tenant_id | embeddings_created | 8-10ms | P1 |

#### 3.2.2 Knowledge Retrieval

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `search_knowledge` | Hybrid vector + FTS search | query, tenant_id, category_filter | results[] | 30-70ms | P1 |
| `get_entry` | Retrieve single entry by ID | entry_id, tenant_id | entry | <10ms | P2 |
| `find_similar` | Vector similarity search | entry_id, tenant_id, limit | similar_entries[] | 30-50ms | P2 |

#### 3.2.3 Knowledge Management

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `detect_gaps` | KB completeness analysis | tenant_id, business_context | completeness_score, gaps[], recommendations[] | 500-1200ms | P2 |

**Total Stage C Tools:** 7

### 3.3 Stage D: Content Generation Tools (11)

#### 3.3.1 Intent & Skill Tools

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `parse_intent` | Natural language → IntentSpec JSON | user_message, session_id, tenant_id | intent_spec, confidence | 400-800ms | P1 |
| `retrieve_skills` | Vector similarity search for skills | intent_spec, tenant_id, top_k | skills[] | 30-60ms | P1 |

#### 3.3.2 Generation Tools

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `generate_multi_candidate` | 2-3 diverse content variations | intent_spec, skills[], brand_voice | candidates[] (A/B/C) | 800-2000ms | P1 |
| `run_guards` | Execute 7 quality guards with auto-fixers | candidate, intent_spec, brand_profile | pass/fail, auto_fixed, guard_results[] | 200-500ms | P1 |
| `rerank_candidates` | Score by quality/cost/diversity | candidates[], intent_spec | ranked_candidates[] | 50-100ms | P1 |

#### 3.3.3 Feedback & Learning

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `classify_feedback` | Defect vs preference classification | revision_request, guard_results, skill_id | classification (defect/preference), labels[] | 300-600ms | P1 |
| `log_performance` | Record skill metrics | skill_id, event_type, metadata, tenant_id | logged (boolean) | 10-30ms | P1 |
| `analyze_skill_performance` | Performance analytics | skill_id (optional), date_range, tenant_id | skills[] with metrics | 100-300ms | P2 |

#### 3.3.4 Brand & Context

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `get_brand_voice` | Retrieve tenant brand profile | tenant_id | brand_profile (palette, typography, tone) | 20-50ms | P1 |
| `update_brand_voice` | Update brand profile | tenant_id, updates | updated (boolean), profile_version | 50-100ms | P2 |
| `get_context` | Retrieve conversation context | session_id | conversation_history[], intent_spec, draft_content | 10-30ms | P1 |

**Total Stage D Tools:** 11

### 3.4 Stage E: Publishing Tools (9)

#### 3.4.1 Multi-Platform Publishing

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `publish_multi_platform` | Publish to 4 platforms simultaneously | content, platforms[], tenant_id | results[] (per platform) | 1500-5000ms | P1 |

#### 3.4.2 OAuth Management

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `oauth_connect` | Initiate OAuth flow | tenant_id, platform, redirect_uri | authorize_url, state | 50-100ms | P1 |
| `oauth_callback` | Handle OAuth callback, store tokens | tenant_id, platform, code, state | success, account_name, scopes[] | 500-1000ms | P1 |

#### 3.4.3 Image Processing

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `process_image_for_platform` | Resize/crop for platform requirements | source_image_url, platform, smart_crop | processed_image_url, dimensions | 500-1500ms | P1 |

#### 3.4.4 Publishing Validation

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `validate_publish` | Pre-publish validation checks | tenant_id, content, platform | valid (boolean), errors[], warnings[] | 50-150ms | P2 |

#### 3.4.5 Platform-Specific Publishers

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `publish_linkedin` | LinkedIn API (extended from current) | tenant_id, content | post_id, post_url | 1200-2000ms | P1 |
| `publish_twitter` | Twitter/X API | tenant_id, content | post_id, post_url | 800-1500ms | P1 |
| `publish_facebook` | Facebook Graph API | tenant_id, content | post_id, post_url | 1000-1800ms | P2 |
| `publish_instagram` | Instagram Graph API | tenant_id, content (requires image) | post_id, post_url | 1500-2500ms | P2 |

**Total Stage E Tools:** 9

### 3.5 Stage F: Analytics Tools (8)

#### 3.5.1 Collection & Metrics

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `sync_post_analytics` | Fetch analytics from platform APIs | tenant_id, post_ids[], force_refresh | synced, skipped, errors | 200-500ms/post | P1 |
| `get_post_analytics` | Retrieve stored analytics | tenant_id, platform, start_date, end_date | posts[] with metrics, summary | 100-300ms | P1 |
| `calculate_performance_score` | Content Performance Score (0-100) | tenant_id, post_id | overall_score, breakdown, percentile | 50-150ms | P2 |

#### 3.5.2 A/B Testing

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `create_ab_test` | Set up A/B test | tenant_id, test_name, variant_a, variant_b | test_id, status | 20-50ms | P2 |
| `analyze_ab_test` | Statistical significance analysis | test_id | winner (A/B/tie), p_value, uplift | 100-300ms | P2 |

#### 3.5.3 Prediction & ROI

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `predict_performance` | Pre-publish prediction | tenant_id, content, platform | predicted_engagement_rate, factors[] | 500-1000ms | P2 |
| `calculate_roi` | ROI calculation | tenant_id, campaign_id or post_ids[] | total_cost, cost_per_engagement, ROI% | 100-300ms | P2 |
| `track_conversion` | UTM-based conversion tracking | utm_source, utm_campaign, conversion_type, value | conversion_id, tracked | 20-50ms | P2 |

#### 3.5.4 Brand Voice Analytics

| Tool | Purpose | Input | Output | Latency | Priority |
|------|---------|-------|--------|---------|----------|
| `analyze_brand_voice_effectiveness` | Measure brand voice impact | tenant_id, platform | correlation, significance, recommendation | 200-500ms | P3 |

**Total Stage F Tools:** 8

### 3.6 Tool Summary by Priority

**Priority 1 (Essential - Must Have):** 15 tools
- Stage D: 9 tools (content generation core)
- Stage E: 4 tools (publishing core)
- Stage F: 2 tools (analytics core)

**Priority 2 (Important - Should Have):** 20 tools
- Stage C: 7 tools (all KB tools)
- Stage D: 2 tools (brand voice management)
- Stage E: 5 tools (validation + platform-specific)
- Stage F: 6 tools (A/B testing, predictions, ROI)

**Priority 3 (Nice-to-Have):** 1 tool
- Stage F: 1 tool (brand voice effectiveness)

**Total:** 44 tools (9 existing + 35 new)

---

## 4. REST API Specification (24 Endpoints)

### 4.1 API Design Principles

**REST Best Practices:**
- Resource-based URLs: `/api/v1/content/generate-candidates`
- HTTP verbs: POST (create/execute), GET (retrieve)
- JSON request/response format
- Bearer token authentication
- Standard HTTP status codes
- Rate limiting headers

**Response Format (Success):**
```json
{
  "success": true,
  "data": { /* endpoint-specific data */ },
  "metadata": {
    "requestId": "req-abc123",
    "timestamp": "2025-11-10T12:00:00Z",
    "processingTimeMs": 250
  }
}
```

**Response Format (Error):**
```json
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Missing required field: skill.template",
    "details": { /* additional context */ }
  },
  "metadata": {
    "requestId": "req-def456",
    "timestamp": "2025-11-10T12:00:00Z"
  }
}
```

### 4.2 Endpoint Categories

| Category | Base Path | Description | Count |
|----------|-----------|-------------|-------|
| Content Generation | `/api/v1/content/*` | Generate, validate, score content | 6 |
| Publishing | `/api/v1/publish/*` | Publish to social platforms | 5 |
| Analytics | `/api/v1/analytics/*` | Engagement metrics, insights | 4 |
| Knowledge Base | `/api/v1/knowledge/*` | Brand voice, skills, products | 4 |
| Products | `/api/v1/products/*` | Product catalog, pricing, stock | 3 |
| Messaging | `/api/v1/messaging/*` | WhatsApp, SMS | 2 |
| **Total** | | | **24** |

### 4.3 Content Generation Endpoints (6)

#### 4.3.1 Generate Content Candidates

**Endpoint:** `POST /api/v1/content/generate-candidates`

**Description:** Generate 3 content candidates (A, B, C) with media for a given skill and platform.

**Request:**
```json
{
  "skill": {
    "template": "product_announcement",
    "parameters": {
      "product": "A193 mesh spacers",
      "platform": "linkedin"
    }
  },
  "count": 3,
  "tenantId": "tenant_xyz"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "candidates": [
      {
        "id": "A",
        "text": "Excited to announce our new A193 mesh spacers...",
        "mediaType": "image",
        "mediaUrl": "https://r2.cloudflare.com/product-a.jpg",
        "dimensions": "1200x627",
        "engagementScore": 8.5,
        "brandVoiceScore": 94
      },
      {
        "id": "B",
        "text": "See how A193 mesh spacers transform construction...",
        "mediaType": "video",
        "mediaUrl": "https://r2.cloudflare.com/product-b.mp4",
        "thumbnail": "https://r2.cloudflare.com/product-b-thumb.jpg",
        "duration": "0:15",
        "engagementScore": 7.2,
        "brandVoiceScore": 88
      },
      {
        "id": "C",
        "text": "Technical specifications and installation guide...",
        "mediaType": "carousel",
        "mediaUrls": [
          "https://r2.cloudflare.com/spec-1.jpg",
          "https://r2.cloudflare.com/spec-2.jpg",
          "https://r2.cloudflare.com/install-1.jpg"
        ],
        "engagementScore": 6.8,
        "brandVoiceScore": 92
      }
    ],
    "generationContext": {
      "skillUsed": "product_announcement",
      "modelVersion": "gpt-4-turbo",
      "brandVectorMatches": 5
    }
  },
  "metadata": {
    "requestId": "req-abc123",
    "timestamp": "2025-11-10T12:00:00Z",
    "processingTimeMs": 2400
  }
}
```

**Maps to JSON-RPC:** `generate_candidates`

**Rate Limit:** 100 requests/hour per tenant

---

#### 4.3.2 Validate Content (Guards Engine)

**Endpoint:** `POST /api/v1/content/validate`

**Description:** Validate content candidates for brand compliance, safety, and platform requirements.

**Request:**
```json
{
  "candidates": [
    {
      "id": "A",
      "text": "...",
      "mediaUrl": "..."
    }
  ],
  "platform": "linkedin",
  "tenantId": "tenant_xyz"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "validatedCandidates": [
      {
        "id": "A",
        "isValid": true,
        "issues": [],
        "autoFixes": []
      }
    ],
    "overallStatus": "PASSED"
  }
}
```

**Maps to JSON-RPC:** `guards_validate`

---

#### 4.3.3 Rerank Candidates

**Endpoint:** `POST /api/v1/content/rerank`

**Description:** Rerank candidates by predicted engagement score.

**Maps to JSON-RPC:** `rerank_candidates`

---

#### 4.3.4 Score Brand Voice

**Endpoint:** `POST /api/v1/content/score-brand-voice`

**Description:** Score content against brand voice guidelines (0-100).

**Maps to JSON-RPC:** `score_brand_voice`

---

#### 4.3.5 Parse Intent

**Endpoint:** `POST /api/v1/content/parse-intent`

**Description:** Parse user intent from natural language request.

**Maps to JSON-RPC:** `parse_intent`

---

#### 4.3.6 Retrieve Skill

**Endpoint:** `POST /api/v1/content/retrieve-skill`

**Description:** Retrieve skill template from knowledge base.

**Maps to JSON-RPC:** `retrieve_skill`

---

### 4.4 Publishing Endpoints (5)

#### 4.4.1 Publish to LinkedIn

**Endpoint:** `POST /api/v1/publish/linkedin`

**Maps to JSON-RPC:** `publish_linkedin`

#### 4.4.2 Publish to Twitter/X

**Endpoint:** `POST /api/v1/publish/twitter`

**Maps to JSON-RPC:** `publish_twitter`

#### 4.4.3 Publish to Facebook

**Endpoint:** `POST /api/v1/publish/facebook`

**Maps to JSON-RPC:** `publish_facebook`

#### 4.4.4 Publish to Instagram

**Endpoint:** `POST /api/v1/publish/instagram`

**Maps to JSON-RPC:** `publish_instagram`

#### 4.4.5 Multi-Platform Publish

**Endpoint:** `POST /api/v1/publish/multi-platform`

**Description:** Publish to multiple platforms simultaneously (LinkedIn, Twitter, Facebook, Instagram).

**Request:**
```json
{
  "candidateId": "A",
  "text": "...",
  "mediaType": "image",
  "mediaUrl": "...",
  "platforms": ["linkedin", "twitter", "facebook"],
  "tenantId": "tenant_xyz"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "results": [
      {
        "platform": "linkedin",
        "success": true,
        "postId": "linkedin-123456",
        "postUrl": "https://linkedin.com/posts/..."
      },
      {
        "platform": "twitter",
        "success": true,
        "postId": "twitter-789012",
        "postUrl": "https://twitter.com/status/..."
      },
      {
        "platform": "facebook",
        "success": false,
        "error": "Invalid access token"
      }
    ],
    "successCount": 2,
    "failureCount": 1
  }
}
```

**Maps to JSON-RPC:** `publish_multi_platform`

---

### 4.5 Analytics Endpoints (4)

#### 4.5.1 Get Engagement Metrics

**Endpoint:** `GET /api/v1/analytics/metrics?tenantId={tenantId}&dateRange={7d|30d|90d}`

**Maps to JSON-RPC:** `get_analytics_metrics`

#### 4.5.2 Get Post Performance

**Endpoint:** `GET /api/v1/analytics/post/{postId}?tenantId={tenantId}`

**Maps to JSON-RPC:** `get_post_performance`

#### 4.5.3 Get Insights

**Endpoint:** `GET /api/v1/analytics/insights?tenantId={tenantId}&dateRange={7d|30d}`

**Description:** AI-generated insights and recommendations.

**Maps to JSON-RPC:** `get_insights`

#### 4.5.4 Track Event

**Endpoint:** `POST /api/v1/analytics/track`

**Maps to JSON-RPC:** `track_event`

---

### 4.6 Knowledge Base Endpoints (4)

#### 4.6.1 Query Brand Voice

**Endpoint:** `POST /api/v1/knowledge/brand-voice`

**Maps to JSON-RPC:** `query_brand_voice`

#### 4.6.2 Query Skills Library

**Endpoint:** `POST /api/v1/knowledge/skills`

**Maps to JSON-RPC:** `query_skills`

#### 4.6.3 Query Product Catalog

**Endpoint:** `POST /api/v1/knowledge/products`

**Maps to JSON-RPC:** `query_products`

#### 4.6.4 Query Example Posts

**Endpoint:** `POST /api/v1/knowledge/examples`

**Maps to JSON-RPC:** `query_examples`

---

### 4.7 Products Endpoints (3)

#### 4.7.1 Get Product Info

**Endpoint:** `GET /api/v1/products/{productId}?tenantId={tenantId}`

**Maps to JSON-RPC:** `get_product_info`

#### 4.7.2 Get Product Price

**Endpoint:** `GET /api/v1/products/{productId}/price?tenantId={tenantId}`

**Maps to JSON-RPC:** `get_product_price`

#### 4.7.3 Check Stock

**Endpoint:** `GET /api/v1/products/{productId}/stock?tenantId={tenantId}`

**Maps to JSON-RPC:** `check_stock`

---

### 4.8 Messaging Endpoints (2)

#### 4.8.1 Send WhatsApp

**Endpoint:** `POST /api/v1/messaging/whatsapp`

**Maps to JSON-RPC:** `send_whatsapp`

#### 4.8.2 Send SMS

**Endpoint:** `POST /api/v1/messaging/sms`

**Maps to JSON-RPC:** `send_sms`

---

### 4.9 Request/Response Translation

**REST to JSON-RPC:**
```typescript
function translateRestToJsonRpc(
  path: string,
  method: string,
  body: any,
  tenantId: string,
  requestId: string
): JSONRPCRequest {
  const jsonRpcMethod = pathToMethod(path);

  return {
    jsonrpc: '2.0',
    method: jsonRpcMethod,
    params: {
      ...body,
      tenantId
    },
    id: requestId
  };
}

function pathToMethod(path: string): string {
  // /content/generate-candidates → generate_candidates
  // /publish/linkedin → publish_linkedin
  // /analytics/metrics → get_analytics_metrics

  const parts = path.split('/').filter(p => p);
  if (parts.length === 2) {
    const [category, action] = parts;
    return action.replace(/-/g, '_');
  }

  // Handle special cases
  return path.replace(/\//g, '_').replace(/-/g, '_');
}
```

**JSON-RPC to REST:**
```typescript
function translateJsonRpcToRest(
  jsonRpcResponse: JSONRPCResponse,
  requestId: string,
  processingTimeMs: number
): Response {
  if (jsonRpcResponse.error) {
    return new Response(JSON.stringify({
      success: false,
      error: {
        code: mapJsonRpcErrorCode(jsonRpcResponse.error.code),
        message: jsonRpcResponse.error.message,
        details: {}
      },
      metadata: {
        requestId,
        timestamp: new Date().toISOString()
      }
    }), {
      status: mapJsonRpcErrorToHttpStatus(jsonRpcResponse.error.code),
      headers: { 'Content-Type': 'application/json' }
    });
  }

  return new Response(JSON.stringify({
    success: true,
    data: jsonRpcResponse.result,
    metadata: {
      requestId,
      timestamp: new Date().toISOString(),
      processingTimeMs
    }
  }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  });
}
```

---

## 5. Security Architecture

### 5.1 Authentication (Bearer Token)

**Token Format:**
```
api_nds_{tenant_id}_{random_suffix}
```

**Example:**
```
api_nds_tenant123_x8Kj9mP2qL5vNwR4
```

**Token Metadata (stored in KV):**
```typescript
interface APIKeyMetadata {
  tenant_id: string;
  key_hash: string;  // SHA-256 hash
  role: 'admin' | 'user' | 'read_only';
  scopes: string[];  // ["tools:call", "analytics:read"]
  created_at: string;
  expires_at: string;  // 90 days TTL
  last_used_at?: string;
  revoked: boolean;
  metadata?: {
    user_id?: string;
    description?: string;
    ip_whitelist?: string[];
  };
}
```

**Authentication Flow:**
1. Extract `Authorization: Bearer api_nds_*` header
2. Validate token format
3. Hash token with SHA-256
4. Look up hash in `API_KEYS` KV namespace
5. Check expiration and revocation status
6. Extract tenant_id, role, scopes
7. Return AuthContext or 401 Unauthorized

### 5.2 Authorization (RBAC)

**Roles:**
- **Admin**: Full access to all operations
- **User**: Read/Write access to tools and analytics
- **ReadOnly**: Read-only access (no tool execution, no publishing)

**Permission Matrix:**

| Endpoint | Admin | User | ReadOnly |
|----------|-------|------|----------|
| `GET /api/tools` | ✅ | ✅ | ✅ |
| `POST /api/tool` | ✅ | ✅ | ❌ |
| `GET /analytics/*` | ✅ | ✅ | ✅ |
| `POST /analytics/track` | ✅ | ✅ | ❌ |
| `POST /admin/keys/*` | ✅ | ❌ | ❌ |

**Scopes:**
- `tools:list` - List available tools
- `tools:call` - Execute tools
- `analytics:read` - Read analytics data
- `analytics:write` - Track selections/events
- `recommendations:read` - Read recommendation rules
- `recommendations:write` - Trigger learning jobs
- `admin:keys` - Manage API keys (admin only)

### 5.3 Rate Limiting (Token Bucket Algorithm)

**Per-Tenant Limits:**

| Category | Limit | Window | Burst |
|----------|-------|--------|-------|
| Content Generation | 100 req/hour | 1 hour | 20 req |
| Publishing | 50 req/hour | 1 hour | 10 req |
| Analytics | 1000 req/hour | 1 hour | 100 req |
| Tool Listing | 1000 req/hour | 1 hour | 100 req |
| Admin Operations | 100 req/hour | 1 hour | 20 req |

**Rate Limit Headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1699564800
```

**429 Response:**
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Max 100 requests per hour.",
    "details": {
      "limit": 100,
      "remaining": 0,
      "resetAt": 1699564800,
      "retryAfterSeconds": 120
    }
  }
}
```

### 5.4 Security Measures

**Input Validation (Zod):**
- All request bodies validated with Zod schemas
- Sanitize inputs (XSS prevention)
- Check request size limits (10MB max)

**SQL Injection Prevention:**
- Use parameterized queries (prepared statements) ALWAYS
- Never concatenate user input into SQL

**XSS Prevention:**
- Sanitize HTML before storing or displaying
- Validate URLs (prevent `javascript:` protocol)

**CORS Configuration:**
- Whitelist trusted origins only
- `https://app.dify.ai`, `https://cloud.dify.ai`, `http://localhost:3000`

**Timeout Enforcement:**
- All external API calls have 30s timeout
- Request processing timeout: 60s

### 5.5 Audit Logging

**D1 Schema:**
```sql
CREATE TABLE audit_logs (
  id TEXT PRIMARY KEY,
  timestamp TEXT NOT NULL,
  tenant_id TEXT NOT NULL,
  user_id TEXT,
  api_key_hash TEXT NOT NULL,
  endpoint TEXT NOT NULL,
  method TEXT NOT NULL,
  status_code INTEGER NOT NULL,
  request_duration_ms INTEGER NOT NULL,
  ip_address TEXT,
  user_agent TEXT,
  error_message TEXT,
  INDEX idx_tenant_timestamp (tenant_id, timestamp),
  INDEX idx_status (status_code),
  INDEX idx_endpoint (endpoint)
);
```

**Log Everything:**
- Authentication attempts (success/failure)
- Authorization denials
- Rate limit violations
- Input validation errors
- Request/response details
- Error messages

### 5.6 Compliance

**GDPR:**
- Right to Access: Export all tenant data
- Right to Erasure: Delete all tenant data
- Data Portability: Export in JSON format
- Audit Trail: Log all data access
- Data Encryption: AES-256 at rest, TLS 1.3 in transit

**SOC 2:**
- Access Control: RBAC with least privilege
- Audit Logging: Comprehensive logging
- Encryption: TLS 1.3 + AES-256
- Change Management: Code reviews
- Incident Response: Documented procedures

---

## 6. Database Schema

### 6.1 D1 Tables

**api_keys** (for audit trail, separate from KV):
```sql
CREATE TABLE api_keys (
  key_hash TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL,
  role TEXT NOT NULL,
  scopes TEXT NOT NULL,  -- JSON array
  created_at TEXT NOT NULL,
  expires_at TEXT NOT NULL,
  revoked INTEGER NOT NULL DEFAULT 0,
  revoked_at TEXT,
  INDEX idx_tenant (tenant_id)
);
```

**audit_logs** (all requests logged):
```sql
CREATE TABLE audit_logs (
  id TEXT PRIMARY KEY,
  timestamp TEXT NOT NULL,
  tenant_id TEXT NOT NULL,
  user_id TEXT,
  api_key_hash TEXT NOT NULL,
  endpoint TEXT NOT NULL,
  method TEXT NOT NULL,
  status_code INTEGER NOT NULL,
  request_duration_ms INTEGER NOT NULL,
  ip_address TEXT,
  user_agent TEXT,
  error_message TEXT,
  INDEX idx_tenant_timestamp (tenant_id, timestamp),
  INDEX idx_status (status_code),
  INDEX idx_endpoint (endpoint)
);
```

**knowledge_entries** (Stage C):
```sql
CREATE TABLE knowledge_entries (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  category TEXT,
  source TEXT,
  created_at TEXT NOT NULL,
  metadata TEXT,  -- JSON
  INDEX idx_tenant_category (tenant_id, category),
  INDEX idx_tenant_created (tenant_id, created_at)
);

CREATE VIRTUAL TABLE knowledge_fts USING fts5(
  title,
  content,
  content=knowledge_entries,
  content_rowid=rowid
);
```

**skills** (Stage D):
```sql
CREATE TABLE skills (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL,
  name TEXT NOT NULL,
  template TEXT NOT NULL,
  structure TEXT NOT NULL,  -- JSON
  examples TEXT,  -- JSON array
  status TEXT NOT NULL,  -- "champion", "active", "testing"
  created_at TEXT NOT NULL,
  metadata TEXT,  -- JSON
  INDEX idx_tenant_status (tenant_id, status)
);
```

**skill_performance** (Stage D):
```sql
CREATE TABLE skill_performance (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL,
  skill_id TEXT NOT NULL,
  event_type TEXT NOT NULL,  -- "selected", "approved", "revised", "defect"
  event_timestamp TEXT NOT NULL,
  metadata TEXT,  -- JSON
  INDEX idx_tenant_skill (tenant_id, skill_id),
  INDEX idx_event_timestamp (event_timestamp)
);
```

**posts** (Stage E/F):
```sql
CREATE TABLE posts (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL,
  platform TEXT NOT NULL,
  post_id TEXT NOT NULL,  -- Platform-specific ID
  post_url TEXT,
  text TEXT NOT NULL,
  media_type TEXT,
  media_url TEXT,
  candidate_id TEXT,
  skill_id TEXT,
  published_at TEXT NOT NULL,
  INDEX idx_tenant_platform (tenant_id, platform),
  INDEX idx_published (published_at)
);
```

**post_analytics** (Stage F):
```sql
CREATE TABLE post_analytics (
  id TEXT PRIMARY KEY,
  post_id TEXT NOT NULL,
  tenant_id TEXT NOT NULL,
  platform TEXT NOT NULL,
  sync_timestamp TEXT NOT NULL,
  likes INTEGER,
  comments INTEGER,
  shares INTEGER,
  clicks INTEGER,
  impressions INTEGER,
  engagement_rate REAL,
  INDEX idx_post (post_id),
  INDEX idx_tenant_timestamp (tenant_id, sync_timestamp)
);
```

**ab_tests** (Stage F):
```sql
CREATE TABLE ab_tests (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL,
  test_name TEXT NOT NULL,
  test_type TEXT NOT NULL,
  variant_a_post_id TEXT NOT NULL,
  variant_b_post_id TEXT NOT NULL,
  variant_c_post_id TEXT,
  status TEXT NOT NULL,  -- "running", "completed", "cancelled"
  started_at TEXT NOT NULL,
  completed_at TEXT,
  INDEX idx_tenant_status (tenant_id, status)
);
```

### 6.2 KV Namespaces

**API_KEYS** (token storage):
- Key: `api_key:{key_hash}`
- Value: JSON-serialized `APIKeyMetadata`
- TTL: 90 days

**RATE_LIMIT** (rate limiting buckets):
- Key: `ratelimit:{tenant_id}:{category}`
- Value: JSON `{ tokens: number, lastRefill: number }`
- TTL: 1 hour

**USER_CACHE** (session/context storage):
- Key: `session:{session_id}`
- Value: JSON `{ tenant_id, conversation_history[], intent_spec, draft_content }`
- TTL: 24 hours

**BRAND_CACHE** (brand profile cache):
- Key: `brand:{tenant_id}`
- Value: JSON `{ palette[], typography, logo_rules, tone_description, lexicon }`
- TTL: 7 days

### 6.3 Vectorize Index

**nds-brand-voice** (Stage C):
- Embeddings: 384 dimensions (bge-small-en-v1.5)
- Metadata: `{ tenant_id, entry_id, category, source }`
- Distance: Cosine similarity

**nds-skills** (Stage D):
- Embeddings: 384 dimensions
- Metadata: `{ tenant_id, skill_id, name, status }`
- Distance: Cosine similarity

### 6.4 R2 Bucket

**nds-images** (Stage D/E):
- Original images: `{tenant_id}/originals/{image_id}.jpg`
- Processed images: `{tenant_id}/processed/{platform}/{image_id}.jpg`
- Public access: Yes (via CDN)

---

## 7. Implementation Roadmap (12 Weeks)

### 7.1 Phase 1: REST API Layer (Weeks 1-2)

**Goal:** Add REST HTTP endpoints on top of existing JSON-RPC handler.

**Timeline:** 2 weeks (10 business days)

**Team:** 1 senior full-stack engineer

**Tasks:**

**Week 1:**
- Day 1-2: Router, authentication, rate limiter
- Day 3-4: Request/response translation
- Day 5: Testing and deployment

**Week 2:**
- Day 6-8: All 24 endpoint handlers
- Day 9: Integration testing
- Day 10: Documentation (OpenAPI spec, cURL examples)

**Deliverables:**
- ✅ REST API layer deployed
- ✅ 24 endpoints functional
- ✅ Authentication working
- ✅ Rate limiting working
- ✅ 80%+ test coverage
- ✅ OpenAPI 3.0 spec
- ✅ Dify HTTP Tool examples

**Budget:** $15,000 (80 hours @ $187.50/hour average)

---

### 7.2 Phase 2: Stage C Tools - Knowledge Base (Weeks 3-4)

**Goal:** Implement 7 knowledge base tools for document processing and retrieval.

**Timeline:** 2 weeks

**Team:** 2 backend engineers

**Tasks:**

**Week 3:**
- Document upload and parsing (PDF, DOCX, CSV, TXT, HTML, MD)
- Auto-categorization with GPT-5
- Vector embedding generation

**Week 4:**
- Hybrid search (vector + FTS)
- Knowledge retrieval tools
- Gap detection and recommendations

**Deliverables:**
- ✅ 7 KB tools implemented
- ✅ Document processing working
- ✅ Vector search functional
- ✅ Unit + integration tests

**Budget:** $60,000 (160 hours @ $375/hour)

---

### 7.3 Phase 3: Stage D Tools - Content Generation (Weeks 5-7)

**Goal:** Implement 11 content generation tools including multi-candidate system and learning engine.

**Timeline:** 3 weeks

**Team:** 3 engineers (2 backend, 1 AI/ML)

**Tasks:**

**Week 5:**
- Intent parsing (natural language → IntentSpec)
- Skill retrieval (vector search)
- Brand voice integration

**Week 6:**
- Multi-candidate generator (A/B/C)
- Quality guards (7 guards with auto-fixers)
- Reranking algorithm

**Week 7:**
- Feedback classification (defect vs preference)
- Learning engine (performance tracking)
- Context management

**Deliverables:**
- ✅ 11 content gen tools implemented
- ✅ Multi-candidate generation working
- ✅ Guards engine functional
- ✅ Learning loop active
- ✅ Unit + integration tests

**Budget:** $135,000 (360 hours @ $375/hour)

---

### 7.4 Phase 4: Stage E Tools - Publishing (Weeks 8-9)

**Goal:** Implement 9 publishing tools for multi-platform social media posting.

**Timeline:** 2 weeks

**Team:** 2 backend engineers

**Tasks:**

**Week 8:**
- OAuth management (connect, callback)
- Multi-platform publisher
- Image processing pipeline

**Week 9:**
- Platform-specific publishers (LinkedIn, Twitter, Facebook, Instagram)
- Publishing validation
- Error handling and retries

**Deliverables:**
- ✅ 9 publishing tools implemented
- ✅ OAuth flows working
- ✅ Multi-platform publishing functional
- ✅ Image processing working
- ✅ Unit + integration tests

**Budget:** $60,000 (160 hours @ $375/hour)

---

### 7.5 Phase 5: Stage F Tools - Analytics (Weeks 10-11)

**Goal:** Implement 8 analytics tools for performance tracking and optimization.

**Timeline:** 2 weeks

**Team:** 2 engineers (1 backend, 1 data engineer)

**Tasks:**

**Week 10:**
- Analytics sync service
- Performance metrics calculation
- Insights generation

**Week 11:**
- A/B testing infrastructure
- ROI tracking
- Performance prediction

**Deliverables:**
- ✅ 8 analytics tools implemented
- ✅ Analytics sync working
- ✅ A/B testing functional
- ✅ ROI calculation accurate
- ✅ Unit + integration tests

**Budget:** $60,000 (160 hours @ $375/hour)

---

### 7.6 Phase 6: Testing & Polish (Week 12)

**Goal:** Comprehensive testing, documentation, and production deployment.

**Timeline:** 1 week

**Team:** 2 QA engineers

**Tasks:**

**Days 1-3:**
- End-to-end testing (all 44 tools)
- Load testing (rate limits, concurrent requests)
- Security audit

**Days 4-5:**
- Final documentation
- Production deployment
- Monitoring setup

**Deliverables:**
- ✅ All 44 tools tested
- ✅ Performance benchmarks met
- ✅ Security audit passed
- ✅ Documentation complete
- ✅ Production-ready

**Budget:** $30,000 (80 hours @ $375/hour)

---

### 7.7 Timeline Summary

```
Weeks 1-2:  [REST API Layer]
Weeks 3-4:  [Stage C: Knowledge Base]
Weeks 5-7:  [Stage D: Content Generation]
Weeks 8-9:  [Stage E: Publishing]
Weeks 10-11: [Stage F: Analytics]
Week 12:    [Testing & Polish]
```

**Total Duration:** 12 weeks (3 months)

**Critical Path:**
- REST API Layer (must complete before Stage C)
- Stage D depends on Stage C (knowledge base)
- Stage E depends on Stage D (content generation)
- Stage F depends on Stage E (publishing)

---

## 8. Testing Strategy

### 8.1 Unit Testing (80%+ Coverage)

**Framework:** Vitest

**Coverage Requirements:**
- Line coverage: 80%+
- Branch coverage: 70%+
- Function coverage: 85%+

**Test Categories:**
- Authentication: Token validation, expiration, revocation
- Rate limiting: Token bucket algorithm, KV operations
- Translation: REST ↔ JSON-RPC conversion
- Input validation: Zod schemas
- Tool logic: Business logic for each tool

**Example Unit Test:**
```typescript
import { describe, it, expect } from 'vitest';
import { validateBearerToken } from '../src/rest/auth';

describe('Authentication', () => {
  it('validates Bearer token format', async () => {
    const validToken = 'api_nds_tenant123_x8Kj9mP2qL5vNwR4';
    const result = await validateBearerToken(validToken, mockEnv);
    expect(result.valid).toBe(true);
    expect(result.tenantId).toBe('tenant123');
  });

  it('rejects invalid token format', async () => {
    const invalidToken = 'invalid_token';
    const result = await validateBearerToken(invalidToken, mockEnv);
    expect(result.valid).toBe(false);
  });

  it('rejects expired token', async () => {
    const expiredToken = 'api_nds_tenant123_expired123';
    const result = await validateBearerToken(expiredToken, mockEnv);
    expect(result.valid).toBe(false);
  });
});
```

### 8.2 Integration Testing

**Framework:** Vitest + Miniflare (local Cloudflare Workers environment)

**Test Scenarios:**
- End-to-end request flow (REST → JSON-RPC → Service → Response)
- Authentication + rate limiting + tool execution
- Error handling (4xx, 5xx responses)
- Multi-platform publishing (mock external APIs)

**Example Integration Test:**
```typescript
describe('Content Generation Integration', () => {
  it('generates candidates end-to-end', async () => {
    const response = await fetch('http://localhost:8787/api/v1/content/generate-candidates', {
      method: 'POST',
      headers: {
        'Authorization': 'Bearer api_nds_tenant_test_abc123',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        skill: { template: 'product_announcement', parameters: {} },
        count: 3,
        tenantId: 'tenant_test'
      })
    });

    expect(response.status).toBe(200);
    const body = await response.json();
    expect(body.success).toBe(true);
    expect(body.data.candidates).toHaveLength(3);
    expect(body.data.candidates[0].id).toBe('A');
  });
});
```

### 8.3 End-to-End Testing

**Framework:** Playwright (for Dify UI testing)

**Test Workflows:**
1. **Social Post Generation:**
   - User types request in Dify chatflow
   - Dify calls parse_intent endpoint
   - Dify calls retrieve_skills endpoint
   - Dify calls generate_multi_candidate endpoint
   - User selects candidate (A/B/C)
   - Dify calls publish_linkedin endpoint
   - Post published successfully

2. **Multi-Platform Publishing:**
   - Generate content
   - Publish to 4 platforms simultaneously
   - Verify posts on LinkedIn, Twitter, Facebook, Instagram

3. **Analytics Workflow:**
   - Sync post analytics
   - Get performance metrics
   - Generate insights

### 8.4 Load Testing

**Framework:** k6 (load testing)

**Test Scenarios:**
- **Burst Test:** 50 concurrent requests in 1 second
- **Sustained Load:** 10 requests/second for 5 minutes
- **Rate Limit Test:** 101 requests to trigger rate limit

**Performance Targets:**
- p50 response time: <150ms
- p95 response time: <300ms
- p99 response time: <500ms
- Error rate: <1%

**Example Load Test:**
```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 10 },  // Ramp up to 10 RPS
    { duration: '3m', target: 10 },  // Stay at 10 RPS
    { duration: '1m', target: 0 },   // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<300'],  // 95% of requests < 300ms
    http_req_failed: ['rate<0.01'],    // Error rate < 1%
  },
};

export default function () {
  const response = http.post(
    'https://mcp-gateway.workers.dev/api/v1/content/generate-candidates',
    JSON.stringify({
      skill: { template: 'product_announcement', parameters: {} },
      count: 3,
      tenantId: 'tenant_test'
    }),
    {
      headers: {
        'Authorization': 'Bearer api_nds_tenant_test_abc123',
        'Content-Type': 'application/json',
      },
    }
  );

  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 300ms': (r) => r.timings.duration < 300,
  });
}
```

### 8.5 Security Testing

**Framework:** OWASP ZAP (security scanner)

**Test Cases:**
- SQL injection attempts
- XSS attacks
- CSRF attacks
- Authentication bypass attempts
- Rate limit evasion
- Authorization bypass attempts

**Penetration Testing:**
- Hire external security firm
- Conduct penetration testing before production
- Address all critical and high-severity findings

---

## 9. Deployment Strategy

### 9.1 Phased Rollout

**Phase 1: Development Environment (Week 1)**
- Deploy REST API layer to dev environment
- Test with synthetic data
- Verify authentication and rate limiting

**Phase 2: Staging Environment (Week 2-11)**
- Deploy each stage (C, D, E, F) to staging
- Test with real tenant data (Next Day Steel)
- Performance testing
- Security testing

**Phase 3: Production Deployment (Week 12)**
- Blue-green deployment
- Deploy to production with traffic split (10% → 50% → 100%)
- Monitor error rates and performance
- Rollback plan ready

### 9.2 Blue-Green Deployment

**Architecture:**
```
                                    ┌──────────────┐
User Requests ──────────────────────▶│ Load Balancer│
                                    └──────┬───────┘
                                           │
                    ┌──────────────────────┴──────────────────────┐
                    │                                              │
                    ▼                                              ▼
           ┌────────────────┐                            ┌────────────────┐
           │ BLUE (Current) │                            │ GREEN (New)    │
           │                │                            │                │
           │ 9 tools        │                            │ 44 tools       │
           │ JSON-RPC only  │                            │ JSON-RPC+REST  │
           └────────────────┘                            └────────────────┘
```

**Deployment Steps:**
1. Deploy new version (GREEN) alongside current version (BLUE)
2. Run smoke tests on GREEN
3. Route 10% of traffic to GREEN
4. Monitor error rates, response times
5. If stable, increase traffic to 50%
6. If stable, increase traffic to 100%
7. Decommission BLUE after 24 hours

**Rollback Plan:**
- If error rate > 1% or p95 > 500ms, rollback immediately
- Rollback command: `wrangler deploy --env production-blue`
- Estimated rollback time: 2 minutes

### 9.3 Monitoring During Deployment

**Metrics to Watch:**
- Error rate (target: <1%)
- Response time (p50, p95, p99)
- Authentication success rate (target: >99%)
- Rate limit violations (expect some, but not excessive)
- External API error rates (OpenAI, Google Vision, etc.)

**Alerts:**
- PagerDuty alert if error rate > 5% for 5 minutes
- Slack alert if response time p95 > 500ms for 10 minutes
- Email alert if authentication success rate < 95%

### 9.4 Deployment Checklist

**Pre-Deployment:**
- [ ] All unit tests passing (80%+ coverage)
- [ ] All integration tests passing
- [ ] Load testing completed (performance targets met)
- [ ] Security audit completed (no critical findings)
- [ ] Staging environment smoke tests passed
- [ ] Rollback plan documented
- [ ] On-call engineers assigned
- [ ] Monitoring dashboards configured
- [ ] Alerting rules configured

**Deployment:**
- [ ] Deploy to GREEN environment
- [ ] Run smoke tests on GREEN
- [ ] Route 10% traffic to GREEN
- [ ] Monitor for 1 hour
- [ ] Route 50% traffic to GREEN
- [ ] Monitor for 2 hours
- [ ] Route 100% traffic to GREEN
- [ ] Monitor for 24 hours

**Post-Deployment:**
- [ ] Decommission BLUE environment
- [ ] Update documentation
- [ ] Communicate success to stakeholders
- [ ] Schedule retrospective

---

## 10. Monitoring & Observability

### 10.1 Metrics to Track

**System Metrics:**
- Request count (per endpoint, per tenant)
- Response time (p50, p95, p99)
- Error rate (4xx, 5xx)
- Authentication success rate
- Rate limit violations
- External API latency (OpenAI, Google Vision, etc.)
- Database query time (D1, KV, Vectorize)

**Business Metrics:**
- Content generation requests
- Publishing success rate (per platform)
- Analytics sync frequency
- A/B test results
- ROI calculations

**Security Metrics:**
- Authentication failures
- Authorization denials
- Rate limit violations
- Input validation errors
- Suspicious activity (multiple failed auth attempts)

### 10.2 Dashboards

**Grafana Dashboard 1: System Health**
- Request count (time series)
- Response time (p50, p95, p99)
- Error rate (4xx, 5xx)
- Authentication success rate
- Top endpoints by request count

**Grafana Dashboard 2: Business Metrics**
- Content generation requests (by tenant)
- Publishing success rate (by platform)
- Analytics sync status
- A/B test results

**Grafana Dashboard 3: Security Monitoring**
- Authentication failures (time series)
- Rate limit violations (by tenant)
- Top API keys by request volume
- Suspicious activity alerts

### 10.3 Alerting Rules

**Critical Alerts (PagerDuty):**
- Error rate > 5% for 5 minutes
- Authentication success rate < 90% for 5 minutes
- External API failure rate > 20% for 10 minutes
- Database connection failure

**Warning Alerts (Slack):**
- Response time p95 > 500ms for 10 minutes
- Rate limit violations > 50 per hour for single tenant
- Disk space usage > 80%

**Info Alerts (Email):**
- New API key created
- API key revoked
- Tenant onboarded/offboarded

### 10.4 Logging

**Log Aggregation:** Datadog Logs

**Log Levels:**
- ERROR: Application errors, external API failures
- WARN: Rate limit violations, validation errors
- INFO: Request/response, authentication events
- DEBUG: Detailed execution traces (dev only)

**Structured Logging:**
```typescript
logger.info('Content generation request', {
  tenant_id: 'tenant_xyz',
  endpoint: '/api/v1/content/generate-candidates',
  skill_template: 'product_announcement',
  processing_time_ms: 2400,
  candidate_count: 3
});
```

### 10.5 Tracing

**Framework:** OpenTelemetry

**Trace Spans:**
- HTTP request (root span)
  - Authentication
  - Rate limiting
  - REST → JSON-RPC translation
  - Tool execution
    - External API call (OpenAI, Google Vision, etc.)
    - Database query (D1, KV, Vectorize)
  - JSON-RPC → REST translation
  - Audit logging

**Distributed Tracing:**
- Trace ID propagated across all services
- View end-to-end request flow in Datadog APM
- Identify slow spans and bottlenecks

---

## 11. Cost Analysis

### 11.1 Development Costs

| Phase | Duration | Engineers | Hours | Cost (@$375/hr) |
|-------|----------|-----------|-------|-----------------|
| Phase 1 (REST API) | 2 weeks | 1 full-stack | 80 | $15,000 |
| Phase 2 (Stage C) | 2 weeks | 2 backend | 160 | $60,000 |
| Phase 3 (Stage D) | 3 weeks | 3 (2 backend, 1 AI) | 360 | $135,000 |
| Phase 4 (Stage E) | 2 weeks | 2 backend | 160 | $60,000 |
| Phase 5 (Stage F) | 2 weeks | 2 (1 backend, 1 data) | 160 | $60,000 |
| Phase 6 (Testing) | 1 week | 2 QA | 80 | $30,000 |
| **Total** | **12 weeks** | | **1,000 hours** | **$360,000** |

**Additional Costs:**
- Project management: $15,000 (40 hours @ $375/hr)
- DevOps setup: $15,000 (monitoring, CI/CD)
- Security audit: $15,000 (external firm)
- **Total Development:** **$405,000**

### 11.2 Operational Costs (Monthly)

| Category | Service | Cost |
|----------|---------|------|
| **AI Models** | OpenAI GPT-5 | $280 |
| | Claude Sonnet 4.5 | $150 |
| | Google Gemini 2.5 | $100 |
| | Google Vision API | $24 |
| | **Subtotal** | **$554** |
| **Cloud Infrastructure** | Cloudflare Workers | $25 |
| | Cloudflare D1 | $25 |
| | Cloudflare Vectorize | $30 |
| | Cloudflare R2 | $2 |
| | Cloudflare KV | $5 |
| | Workers AI | $20 |
| | **Subtotal** | **$107** |
| **External APIs** | Supabase | $25 |
| | Twitter API (Basic) | $100 |
| | Twilio | $50 |
| | **Subtotal** | **$175** |
| **Monitoring** | Datadog Lite | $15 |
| | **Total Monthly** | **$851** |

**Per-Customer Cost (100 customers):** $8.51/customer/month

### 11.3 Cost Per Request

| Tool Category | Avg Cost |
|---------------|----------|
| Knowledge Base | $0.004 |
| Content Generation (multi-candidate) | $0.28 |
| Publishing | $0.02 |
| Analytics | $0.01 |

**Typical Workflow Cost:**
- Intent parsing: $0.08
- KB search: $0.004
- Multi-candidate gen: $0.28
- Guards: $0.01
- Publishing (4 platforms): $0.08
- Analytics sync: $0.01
- **Total:** $0.464 per end-to-end workflow

**Monthly Cost (1000 workflows):** $464

### 11.4 ROI Projections

**Assumptions:**
- 100 customers @ $50/month = $5,000/month revenue
- Operational cost: $851/month
- Net profit: $4,149/month = $49,788/year

**ROI Calculation:**
- Initial investment: $405,000
- Annual net profit: $49,788
- Payback period: 8.1 years

**Break-Even Analysis:**
- Need 171 customers @ $50/month to break even on operations ($8,551/month)
- Need 810 customers @ $50/month to recoup development cost in 1 year

**Recommendation:** Target 500+ customers to achieve 3-year payback period.

---

## 12. Risk Management

### 12.1 Technical Risks

| Risk | Probability | Impact | Mitigation | Owner |
|------|-------------|--------|------------|-------|
| **Breaking existing JSON-RPC clients** | Low | High | Keep JSON-RPC handler unchanged; route only `/api/v1/*` to REST layer | Backend Lead |
| **Performance degradation** | Medium | High | Profile code; optimize KV operations; add caching | Backend Lead |
| **External API rate limits** | High | Medium | Implement exponential backoff; cache responses; use multiple API keys | Backend Lead |
| **Data loss during migration** | Low | Critical | Backup all data; test migration scripts; phased migration | DevOps Lead |
| **Security vulnerabilities** | Medium | Critical | Security audit; penetration testing; follow OWASP guidelines | Security Team |

### 12.2 Timeline Risks

| Risk | Probability | Impact | Mitigation | Owner |
|------|-------------|--------|------------|-------|
| **Scope creep** | High | High | Freeze requirements after Week 2; defer non-critical features to v2 | Product Manager |
| **Developer availability** | Medium | High | Cross-train team members; document everything; pair programming | Tech Lead |
| **External dependency delays** | Medium | Medium | Use mock services for development; decouple external integrations | Backend Lead |
| **Testing bottleneck** | Medium | Medium | Start testing early (Week 3); automate tests; parallel testing | QA Lead |

### 12.3 Budget Risks

| Risk | Probability | Impact | Mitigation | Owner |
|------|-------------|--------|------------|-------|
| **AI API costs exceed budget** | High | Medium | Monitor usage; set spending limits; optimize prompts | Product Manager |
| **Development overruns** | Medium | High | Weekly progress reviews; adjust scope if needed; buffer budget | Tech Lead |
| **Unexpected infrastructure costs** | Low | Medium | Monitor Cloudflare usage; set up billing alerts | DevOps Lead |

### 12.4 Mitigation Strategies

**Technical:**
- Comprehensive testing (80%+ coverage)
- Performance profiling at each phase
- Security audit by external firm
- Load testing before production

**Timeline:**
- Weekly standups to track progress
- Bi-weekly sprint demos
- Daily Slack updates
- Blocker escalation process

**Budget:**
- Track actual vs estimated hours
- Monthly budget reviews
- Contingency budget (10% = $40,500)
- Defer non-critical features if over budget

---

## 13. Success Criteria

### 13.1 Technical Success Criteria

**Code Quality:**
- [ ] All TypeScript strict mode enabled
- [ ] 80%+ test coverage (unit + integration)
- [ ] ESLint passes with 0 warnings
- [ ] Prettier formatting applied
- [ ] No security vulnerabilities (critical or high)

**Performance:**
- [ ] p95 response time < 300ms for simple endpoints
- [ ] p99 response time < 500ms for simple endpoints
- [ ] Error rate < 1%
- [ ] Rate limiting accurate (100% enforcement)

**Functionality:**
- [ ] All 44 tools implemented and tested
- [ ] REST API layer functional (24 endpoints)
- [ ] Authentication working (Bearer tokens)
- [ ] Rate limiting working (per tenant, per category)
- [ ] Audit logging working (all requests logged)
- [ ] Multi-tenant isolation working (RLS enforced)

### 13.2 Business Success Criteria

**Dify Integration:**
- [ ] Dify workflows can orchestrate all 44 tools
- [ ] HTTP Tools configured for 5 workflows
- [ ] End-to-end test passes (user request → published post)
- [ ] Error handling works (rate limits, invalid tokens)

**User Experience:**
- [ ] Conversational advisory pattern working
- [ ] Multi-candidate generation (A/B/C options)
- [ ] Publishing to 4 platforms simultaneously
- [ ] Real-time analytics and insights

**Compliance:**
- [ ] GDPR compliant (data export/deletion endpoints)
- [ ] SOC 2 ready (audit logs, encryption, RBAC)
- [ ] Security audit passed (no critical findings)

### 13.3 Acceptance Criteria

**Phase 1 (REST API Layer):**
- [ ] All 24 REST endpoints deployed
- [ ] Authentication working (100% success rate for valid tokens)
- [ ] Rate limiting working (429 responses for exceeded limits)
- [ ] OpenAPI spec validates
- [ ] Dify HTTP Tool examples work

**Phase 2 (Stage C):**
- [ ] 7 KB tools implemented
- [ ] Document upload working (PDF, DOCX, CSV, TXT, HTML, MD)
- [ ] Vector search functional (Vectorize)
- [ ] Hybrid search working (vector + FTS)

**Phase 3 (Stage D):**
- [ ] 11 content gen tools implemented
- [ ] Multi-candidate generation working (A/B/C)
- [ ] Guards engine functional (7 guards with auto-fixers)
- [ ] Learning loop active (defect classification)

**Phase 4 (Stage E):**
- [ ] 9 publishing tools implemented
- [ ] OAuth flows working (LinkedIn, Twitter, Facebook, Instagram)
- [ ] Multi-platform publishing functional
- [ ] Image processing working

**Phase 5 (Stage F):**
- [ ] 8 analytics tools implemented
- [ ] Analytics sync working (all 4 platforms)
- [ ] A/B testing functional
- [ ] ROI calculation accurate

**Phase 6 (Testing):**
- [ ] All 44 tools tested (unit + integration)
- [ ] Performance benchmarks met (p95 < 300ms)
- [ ] Security audit passed
- [ ] Production deployment successful

---

## 14. Documentation Requirements

### 14.1 API Documentation

**OpenAPI 3.0 Specification:**
- All 24 REST endpoints documented
- Request/response schemas
- Authentication section
- Error responses
- Rate limiting headers
- Examples for each endpoint

**Swagger UI:**
- Interactive API documentation
- Try-it-out functionality
- Available at `https://mcp-gateway.workers.dev/docs`

### 14.2 Developer Guides

**Backend Developer Guide:**
- Architecture overview
- File structure
- Adding new tools (step-by-step)
- Testing guidelines
- Deployment process

**Dify Workflow Designer Guide:**
- HTTP Tool configuration
- Secrets management
- Variable interpolation
- Error handling
- Example workflows

**QA Testing Guide:**
- cURL examples for all 24 endpoints
- Expected responses
- Error scenarios
- Load testing scripts
- Security testing checklist

### 14.3 Runbooks

**On-Call Runbook:**
- Common issues and solutions
- Rollback procedures
- Escalation paths
- Contact information

**Deployment Runbook:**
- Pre-deployment checklist
- Deployment steps
- Smoke testing
- Rollback plan

**Incident Response Runbook:**
- Incident classification (P1, P2, P3)
- Response procedures
- Communication templates
- Post-mortem template

---

## 15. Team & Resources

### 15.1 Team Structure

**Core Team (Week 1-12):**
- 1 Tech Lead (part-time, 50%)
- 1 Senior Full-Stack Engineer (Weeks 1-2)
- 2 Backend Engineers (Weeks 3-11)
- 1 AI/ML Engineer (Weeks 5-7)
- 1 Data Engineer (Weeks 10-11)
- 2 QA Engineers (Week 12)
- 1 Product Manager (part-time, 25%)
- 1 DevOps Engineer (part-time, 25%)

**External Resources:**
- Security firm (1 week, penetration testing)
- Technical writer (1 week, documentation)

### 15.2 Required Skills

**Backend Engineers:**
- TypeScript (expert)
- Cloudflare Workers (intermediate)
- REST API design (expert)
- Database design (SQL, NoSQL)
- Testing (unit, integration, E2E)

**AI/ML Engineer:**
- LLM integration (OpenAI, Anthropic, Google)
- Prompt engineering
- Vector embeddings
- Semantic search

**Data Engineer:**
- Analytics pipeline design
- SQL optimization
- Data visualization
- A/B testing statistics

**QA Engineers:**
- API testing (Postman, cURL)
- Load testing (k6)
- Security testing (OWASP ZAP)
- Test automation (Playwright)

### 15.3 External Dependencies

**Third-Party Services:**
- OpenAI (GPT-5 API access)
- Anthropic (Claude Sonnet 4.5 API access)
- Google Cloud (Vision API, Gemini API)
- LinkedIn (API application approved)
- Twitter/X (API application approved, $100/month tier)
- Facebook (API application approved)
- Instagram (API application approved)
- Twilio (Account setup, $50/month)
- Supabase (Database hosted)
- Shopify (API credentials)

**Infrastructure:**
- Cloudflare Workers (paid plan)
- Cloudflare D1 (database)
- Cloudflare KV (namespaces created)
- Cloudflare Vectorize (index created)
- Cloudflare R2 (bucket created)

**Tools:**
- GitHub (version control)
- Datadog (monitoring)
- PagerDuty (alerting)
- Slack (communication)
- Notion/Confluence (documentation)

---

## 16. Next Steps

### 16.1 Immediate Actions (Week 0)

**Product Manager:**
- [ ] Review and approve this plan
- [ ] Communicate timeline to stakeholders
- [ ] Set up project tracking (Jira/Linear)
- [ ] Schedule kick-off meeting

**Tech Lead:**
- [ ] Review technical architecture
- [ ] Identify risks and mitigation strategies
- [ ] Assign engineers to phases
- [ ] Set up development environment

**DevOps:**
- [ ] Provision Cloudflare resources (Workers, D1, KV, Vectorize, R2)
- [ ] Set up CI/CD pipeline
- [ ] Configure monitoring (Datadog, PagerDuty)
- [ ] Create staging environment

**Security:**
- [ ] Schedule security audit (Week 11)
- [ ] Review authentication/authorization design
- [ ] Set up vulnerability scanning

### 16.2 Phase 1 Kickoff (Week 1, Day 1)

**Morning:**
- [ ] Kick-off meeting (all team members)
- [ ] Review STAGE-I-REST-API-SPECIFICATION.md
- [ ] Assign senior full-stack engineer
- [ ] Create branch: `feature/stage-i-rest-api-layer`

**Afternoon:**
- [ ] Implement REST router (`src/rest/router.ts`)
- [ ] Implement authentication (`src/rest/auth.ts`)
- [ ] Implement rate limiter (`src/rest/rate-limiter.ts`)
- [ ] Daily standup at EOD

### 16.3 Success Tracking

**Weekly Standups:**
- Progress review (completed vs planned)
- Blocker identification and resolution
- Next week planning

**Bi-Weekly Sprint Demos:**
- Demo completed features
- Stakeholder feedback
- Adjust scope if needed

**Monthly Budget Reviews:**
- Track actual vs estimated hours
- Adjust timeline if over budget
- Communicate to stakeholders

### 16.4 Go/No-Go Decision Points

**Week 2 (After REST API Layer):**
- [ ] All 24 endpoints functional?
- [ ] Authentication working?
- [ ] Rate limiting working?
- [ ] Performance benchmarks met?
- **Decision:** Proceed to Phase 2 (Stage C) or adjust scope?

**Week 4 (After Stage C):**
- [ ] 7 KB tools functional?
- [ ] Vector search working?
- [ ] Document processing working?
- **Decision:** Proceed to Phase 3 (Stage D) or adjust timeline?

**Week 7 (After Stage D):**
- [ ] 11 content gen tools functional?
- [ ] Multi-candidate generation working?
- [ ] Guards engine working?
- **Decision:** Proceed to Phase 4 (Stage E) or defer features?

**Week 9 (After Stage E):**
- [ ] 9 publishing tools functional?
- [ ] OAuth flows working?
- [ ] Multi-platform publishing working?
- **Decision:** Proceed to Phase 5 (Stage F) or defer analytics?

**Week 11 (After Stage F):**
- [ ] 8 analytics tools functional?
- [ ] A/B testing working?
- [ ] ROI calculation accurate?
- **Decision:** Proceed to Phase 6 (Testing) or extend timeline?

**Week 12 (Before Production):**
- [ ] All tests passing?
- [ ] Security audit passed?
- [ ] Performance benchmarks met?
- **Decision:** Deploy to production or delay?

---

## Appendices

### Appendix A: Glossary

- **MCP**: Model Context Protocol (JSON-RPC 2.0 based protocol)
- **REST**: Representational State Transfer (HTTP-based API architecture)
- **JSON-RPC**: Remote procedure call protocol encoded in JSON
- **Bearer Token**: Authentication token passed in Authorization header
- **RBAC**: Role-Based Access Control
- **RLS**: Row-Level Security (tenant isolation in database)
- **KV**: Key-Value store (Cloudflare Workers KV)
- **D1**: Cloudflare's SQL database
- **Vectorize**: Cloudflare's vector database for embeddings
- **R2**: Cloudflare's object storage (S3-compatible)
- **FTS**: Full-Text Search
- **RRF**: Reciprocal Rank Fusion (hybrid search algorithm)
- **GDPR**: General Data Protection Regulation
- **SOC 2**: Service Organization Control 2 (security compliance)

### Appendix B: Related Documents

- `STAGE-I-REST-API-SPECIFICATION.md` - Complete REST API specification
- `STAGE-I-SECURITY-ARCHITECTURE.md` - Security and authentication design
- `STAGE-I-MCP-GATEWAY-TOOL-INVENTORY.md` - Detailed tool specifications
- `STAGE-I-TOOL-SUMMARY.md` - Quick reference for all 44 tools
- `CURL-EXAMPLES.md` - cURL commands for testing
- `IMPLEMENTATION-CHECKLIST.md` - Day-by-day implementation guide
- `QUICK-REFERENCE.md` - One-page cheat sheet

### Appendix C: Contact Information

**Project Sponsor:** [Name], Product Manager
**Tech Lead:** [Name], Senior Engineer
**Security Lead:** [Name], Security Engineer
**DevOps Lead:** [Name], DevOps Engineer

**Escalation Path:**
1. Tech Lead (response time: 1 hour)
2. Product Manager (response time: 4 hours)
3. VP Engineering (response time: 1 business day)

---

## Document Status

**Status:** ✅ APPROVED - Ready for Implementation
**Version:** 1.0.0
**Last Updated:** November 10, 2025
**Approved By:** Planning Agent
**Next Review:** Week 6 (mid-implementation checkpoint)

**Change Log:**
- 2025-11-10: Initial approved plan created
- [Future changes will be logged here]

---

**STAGE I: MCP GATEWAY EXPANSION - APPROVED AND READY FOR IMPLEMENTATION**

**Total Investment:** $405,000 development + $851/month operations
**Timeline:** 12 weeks (3 months)
**Expected ROI:** 8.1 years payback period (100 customers @ $50/month)

**Next Action:** Schedule kick-off meeting for Week 1, Day 1
