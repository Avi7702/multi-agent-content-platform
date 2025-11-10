# Approved Documents Index

**Version**: 2.0.0
**Last Updated**: November 10, 2025
**Status**: ✅ ALL STAGES COMPLETE (A-L)
**Purpose**: Complete index of all production-ready approved documents
**Note**: Company references genericized for reusability

---

## What This Folder Contains

This folder contains **ONLY production-ready documents** that have been approved for implementation. These documents represent final, executable plans and architectural decisions.

**Status**: Planning complete, ready for implementation when credentials available.

---

## Document Organization

```
approved/
├── SYSTEM-OVERVIEW.md                          [Master Overview]
├── STAGE-X-RESILIENCE-PATTERNS.md              [Cross-Cutting Patterns] ⭐ NEW
├── stage-a-architecture/
│   └── decisions/
│       └── 001-hybrid-architecture.md          [ADR: Architecture Decision]
├── stage-b-skill-system/
│   ├── STAGE-B-APPROVED-PLAN.md               [Approved Plan]
│   └── decisions/
│       ├── 002-skill-builder-meta-skill.md     [ADR: Meta-Skill Decision]
│       ├── 003-conversational-advisory-pattern.md [ADR: UX Pattern Decision]
│       └── skill-matching-algorithm.md         [ADR: Algorithm Decision]
├── stage-c-knowledge-base/
│   ├── STAGE-C-APPROVED-PLAN.md               [Approved Plan]
│   ├── STAGE-C-EXECUTION-TRACKER.md           [Execution Tracker]
│   ├── STAGE-C-EXECUTION-FRAMEWORK-SUMMARY.md [Framework Documentation]
│   └── STAGE-C-READINESS-CHECKLIST.md         [Pre-Implementation Checklist]
├── stage-d-content-generation/
│   └── STAGE-D-APPROVED-PLAN.md               [Approved Plan]
├── stage-e-multi-platform/
│   └── STAGE-E-APPROVED-PLAN.md               [Approved Plan]
├── stage-f-analytics/
│   └── STAGE-F-APPROVED-PLAN.md               [Approved Plan]
├── stage-g-dify-implementation/
│   └── STAGE-G-APPROVED-PLAN.md               [Approved Plan]
├── stage-h-openmanus/
│   └── STAGE-H-APPROVED-PLAN.md               [Approved Plan]
├── stage-i-mcp-gateway/
│   └── STAGE-I-APPROVED-PLAN.md               [Approved Plan]
├── stage-j-database-schema/
│   └── STAGE-J-DATABASE-SCHEMA.md            [Database Schema] ⭐ NEW
├── stage-k-system-testing/
│   └── STAGE-K-E2E-TESTING-STRATEGY.md       [Testing Strategy] ⭐ NEW
└── stage-l-deployment/
    └── [Deployment specifications]            [Infrastructure] ⭐ NEW
```

**Total**: 20 approved documents across 12 stages + 1 cross-cutting document
**Status**: ✅ PLANNING COMPLETE (100%)

---

## Master Document

### SYSTEM-OVERVIEW.md ⭐
- **Type**: Master system overview
- **Status**: DRAFT - Planning stage
- **Purpose**: Complete overview of all stages (A, B, C)
- **Scope**: Entire DIFY-AGENT project
- **Use When**: Need to understand the complete system architecture

**When to Update**: After completing planning for any new stage, update SYSTEM-OVERVIEW.md to reflect new stage details.

---

## Cross-Cutting Documents

### STAGE-X-RESILIENCE-PATTERNS.md ⭐ NEW
- **Type**: Cross-cutting technical patterns
- **Version**: 1.0.0
- **Status**: ✅ APPROVED (Nov 10, 2025)
- **Size**: 6000+ lines (12 sections)
- **Applies To**: All Stages (C, D, E, F, G, H, I, J, K, L)
- **Purpose**: Production-grade resilience patterns for all services
- **Package**: `@dify-agent/resilience` (shared NPM package)

**Key Patterns:**
1. **Circuit Breaker**: Prevents cascading failures (3 states: CLOSED, OPEN, HALF_OPEN)
2. **Retry with Exponential Backoff**: Auto-retry with increasing delays
3. **Bulkheads**: Resource isolation per tenant/service
4. **Timeouts**: All external calls have timeouts
5. **Fallback Strategies**: Alternative behavior on failures (e.g., GPT-5 → Claude)
6. **Health Checks**: Periodic service availability monitoring
7. **Rate Limiting**: Token bucket, fixed window, sliding window
8. **Monitoring & Observability**: Metrics, alerting, dashboards

**Implementation Details:**
- Full TypeScript implementations for all patterns
- DRY approach: Write once, use everywhere
- Comprehensive test coverage (unit + integration)
- Datadog/Sentry/Prometheus integration
- Environment-specific configs (dev/staging/prod)
- Per-service configuration guidelines

**Usage in Other Stages:**
- Stage C: Web scraping, knowledge extraction (Firecrawl, GPT-5)
- Stage D: 11 microservices (all LLM calls, database, vectorize)
- Stage E: 4 platform adapters (LinkedIn, Twitter, Facebook, Instagram APIs)
- Stage F: Analytics service (platform APIs)
- All future stages with external dependencies

**When to Reference**: Any time implementing external API calls, database operations, or inter-service communication.

---

## Stage A: Architecture (Completed)

### Purpose
Established the foundational architecture pattern for the entire system.

### Status
- Planning: ✅ Complete
- Implementation: ✅ Complete
- In Production: ✅ Yes

### Approved Documents

#### 1. 001-hybrid-architecture.md
- **Location**: `stage-a-architecture/decisions/`
- **Type**: Architecture Decision Record (ADR)
- **Status**: ✅ APPROVED
- **Date**: November 6, 2025
- **Decision**: Use Dify + OpenManus hybrid architecture
- **Impact**: HIGH - Defines entire system architecture
- **Key Points**:
  - Dify: Orchestration + UI + Memory
  - OpenManus: Skills system + extensibility
  - Why: Best of both platforms
- **Implementation**: Required for all subsequent stages

**Note**: Stage A has no separate approved plan document. The architecture decision (ADR 001) serves as the approved deliverable.

---

## Stage B: Skill System (Completed)

### Purpose
Design and plan the self-evolving skills system that enables infinite extensibility.

### Status
- Planning: ✅ Complete
- Implementation: ⏸️ Pending (after Stage C)
- In Production: ❌ Not yet

### Approved Documents

#### 1. STAGE-B-APPROVED-PLAN.md ⭐
- **Location**: `stage-b-skill-system/`
- **Type**: Approved execution plan
- **Version**: 1.0.0
- **Status**: ✅ APPROVED (Nov 9, 2025)
- **Purpose**: Complete implementation plan for pattern analysis system
- **Timeline**: 4 weeks
- **Key Deliverables**:
  - Pattern recognition engine
  - Meta-skill builder
  - Skill library management
  - Learning loop algorithm
- **Implementation Notes**: Requires Dify architecture (Stage A) complete

#### 2. 002-skill-builder-meta-skill.md
- **Location**: `stage-b-skill-system/decisions/`
- **Type**: Architecture Decision Record (ADR)
- **Status**: ✅ APPROVED
- **Decision**: Build AI that creates AI skills (meta-skill)
- **Impact**: HIGH - Enables infinite extensibility
- **Key Points**:
  - Agent analyzes user requests
  - Generates new skills automatically
  - Self-evolving capability system
- **Rationale**: Avoids manual skill creation bottleneck

#### 3. 003-conversational-advisory-pattern.md
- **Location**: `stage-b-skill-system/decisions/`
- **Type**: Architecture Decision Record (ADR)
- **Status**: ✅ APPROVED
- **Decision**: Pure text UI, no GUI buttons
- **Impact**: HIGH - Saves $30K+ in development costs
- **Key Points**:
  - User types text requests
  - Bot presents options as text (A/B/C)
  - User types choice
  - Natural conversation flow
- **Benefits**: Faster development, mobile-friendly, accessible

#### 4. skill-matching-algorithm.md
- **Location**: `stage-b-skill-system/decisions/`
- **Type**: Decision document / Technical design
- **Status**: ✅ APPROVED
- **Size**: ~36KB
- **Purpose**: Intent classification and skill selection algorithm
- **Key Points**:
  - How agent matches user requests to skills
  - Semantic similarity scoring
  - Fallback to meta-skill builder
- **Implementation**: Core to Stage B execution

---

## Stage C: Enterprise Knowledge Base (Planning Complete)

### Purpose
Build enterprise-grade knowledge base with content multiplication and RAG capabilities.

### Status
- Planning: ✅ Complete (Nov 9, 2025)
- Implementation: ⏸️ BLOCKED - Pending credentials
- In Production: ❌ Not yet

### Blocked By
- Firecrawl API key (web scraper)
- OpenAI GPT-5 API access (content multiplication)

### Approved Documents

#### 1. STAGE-C-APPROVED-PLAN.md ⭐
- **Location**: `stage-c-knowledge-base/`
- **Type**: Approved execution plan
- **Version**: 1.3.0 (includes RAG architecture)
- **Status**: ✅ APPROVED (Nov 9, 2025)
- **Last Updated**: Nov 9, 2025 (Added RAG research Week 3)
- **Purpose**: Enterprise KB system with content multiplication
- **Timeline**: 6 weeks (30 working days = ~1.5 months)
- **Budget**: $15.74/customer/month all-in
- **Key Features**:
  - Bulk document upload (PDFs, Word, text)
  - Auto-categorization (GPT-5)
  - Content multiplication (1 doc → 5-10x outputs)
  - Vector embeddings + semantic search
  - Gap detection and validation
  - Web scraping (Firecrawl/Jina AI)
  - Hybrid search RAG architecture
- **Implementation Notes**:
  - Requires OpenAI GPT-5 API access
  - Requires Firecrawl API key
  - Week 3 focuses on RAG implementation

#### 2. STAGE-C-EXECUTION-TRACKER.md ⭐
- **Location**: `stage-c-knowledge-base/`
- **Type**: Real-time execution tracker
- **Framework**: Milestone-Kanban Hybrid with Weekly OKRs
- **Status**: Ready (awaiting credentials to start)
- **Purpose**: Track Stage C implementation progress in real-time
- **Timeline**: 5 weeks, 25 working days
- **Structure**:
  - Weekly milestones
  - Daily task tracking
  - Blocker identification
  - Risk management
- **Best Practices**: Combines Google/Meta/OpenAI execution frameworks
- **Usage**: Update daily during implementation

#### 3. STAGE-C-EXECUTION-FRAMEWORK-SUMMARY.md
- **Location**: `stage-c-knowledge-base/`
- **Type**: Framework documentation
- **Status**: ✅ Complete
- **Purpose**: Documents the research and delivery of execution framework
- **Contents**:
  - Framework research summary
  - Comparison of methodologies
  - Chosen approach explanation
  - Best practices integration
- **Use When**: Understanding why this execution framework was chosen

#### 4. STAGE-C-READINESS-CHECKLIST.md ⭐
- **Location**: `stage-c-knowledge-base/`
- **Type**: Pre-implementation checklist
- **Status**: ✅ Planning complete - awaiting credentials
- **Purpose**: Verify all prerequisites before starting Stage C implementation
- **Sections**:
  - ✅ Planning complete
  - ❌ Credentials (blocked)
  - ✅ Infrastructure ready
  - ⏸️ Team readiness (pending start)
- **Use When**: Before beginning Stage C implementation
- **Next Action**: Obtain Firecrawl + OpenAI GPT-5 credentials

---

## Stage D: Content Generation Workflow (Completed Planning)

### Purpose
Design the complete content generation system with multi-candidate generation, guards engine, and multi-tenant safe learning.

### Status
- Planning: ✅ Complete (Nov 10, 2025)
- Implementation: ⏸️ Pending (after Stage C)
- In Production: ❌ Not yet

### Approved Documents

#### 1. STAGE-D-APPROVED-PLAN.md ⭐
- **Location**: `stage-d-content-generation/`
- **Type**: Approved execution plan
- **Version**: 1.0.0
- **Status**: ✅ APPROVED (Nov 10, 2025)
- **Size**: 3476 lines (13 sections)
- **Purpose**: Complete implementation plan for content generation workflow
- **Timeline**: 6 months
- **Budget**: $387/month operational, $900K development
- **Key Deliverables**:
  - 11 microservices (Intent Parser, Skill Retrieval, Multi-Candidate Generator, Guards Engine, Reranker, Defect Classifier, Learning Engine, Brand Voice, Publishing, Analytics, Context)
  - 7 guards with auto-fixers (Text Overlap, Safe Margin, Contrast Ratio, Logo Protection, Aspect Ratio, Brand Palette, Font Size)
  - Multi-candidate generation (2-3 outputs per request)
  - Multi-tenant safe learning with privacy guarantees
  - 9 database tables with complete schemas
  - Defect vs preference classification system
- **Implementation Notes**:
  - Models: Claude Sonnet 4.5, GPT-5, Gemini 2.5 Flash, voyage-3-lite
  - Infrastructure: Cloudflare Workers, D1, Vectorize, R2, KV
  - Privacy: RLS, KMS encryption, K-anonymity (threshold=5), temporal delay (30-60 days)

---

## Stage E: Multi-Platform Publishing System (Completed Planning)

### Purpose
Enable publishing to 4 social media platforms (LinkedIn, Twitter/X, Facebook, Instagram) with platform-specific optimizations and OAuth authentication.

### Status
- Planning: ✅ Complete (Nov 10, 2025)
- Implementation: ⏸️ Pending (after Stage D)
- In Production: ❌ Not yet

### Approved Documents

#### 1. STAGE-E-APPROVED-PLAN.md ⭐
- **Location**: `stage-e-multi-platform/`
- **Type**: Approved execution plan
- **Version**: 1.0.0
- **Status**: ✅ APPROVED (Nov 10, 2025)
- **Size**: 2800+ lines (13 sections)
- **Purpose**: Multi-platform publishing with platform-specific adaptations
- **Timeline**: 4 weeks
- **Budget**: $129/month operational ($1.29/customer/month)
- **Key Deliverables**:
  - 4 platform adapters (LinkedIn, Twitter/X, Facebook, Instagram)
  - OAuth 2.0 authentication flows for all platforms
  - Token management with encryption and auto-refresh
  - Smart image processing (aspect ratio conversion, face detection)
  - Cross-platform content adaptation (text truncation, hashtag optimization)
  - Post analytics tracking and sync
  - Rate limiting per platform
  - Publishing service with retry logic
- **Platform Specifications**:
  - LinkedIn: 3000 chars, 1200x627 images, 3-5 hashtags, 100 posts/day
  - Twitter/X: 280 chars, 1200x675 images, 1-2 hashtags, 50 posts/day
  - Facebook: 63K chars, 1200x630 images, 0-1 hashtags, 200 posts/hour
  - Instagram: 2200 chars, 1080x1080 images, 30 hashtags, 25 posts/day
- **Implementation Notes**:
  - APIs: LinkedIn Marketing v2, Twitter API v2, Facebook Graph v18, Instagram Graph
  - Image: Sharp (Node.js), Google Vision API
  - Storage: Cloudflare R2 (images), D1 (oauth_tokens, published_posts, rate_limits)
  - Platform costs: $100/month (Twitter/X Basic tier), $24/month (Google Vision), $5/month (Cloudflare Workers)

---

## Stage F: Analytics & Performance Tracking System (Completed Planning)

### Purpose
Build comprehensive analytics and performance tracking system that collects, aggregates, and analyzes content performance across all 4 social media platforms with actionable insights for content optimization, ROI measurement, and brand voice effectiveness.

### Status
- Planning: ✅ Complete (Nov 10, 2025)
- Implementation: ⏸️ Pending (after Stage E)
- In Production: ❌ Not yet

### Approved Documents

#### 1. STAGE-F-APPROVED-PLAN.md ⭐
- **Location**: `stage-f-analytics/`
- **Type**: Approved execution plan
- **Version**: 1.0.0
- **Status**: ✅ APPROVED (Nov 10, 2025)
- **Size**: 3600+ lines (16 sections)
- **Purpose**: Complete analytics and performance tracking system
- **Timeline**: 4 weeks (20 working days)
- **Budget**: $142/month operational ($1.42/customer/month), $130K development
- **Key Deliverables**:
  - Analytics Collection Service (automated data collection from 4 platforms)
  - Performance Metrics Engine (KPI calculation, benchmarks, trend detection, anomaly detection)
  - Dashboard & Reporting (real-time charts, drill-down views, CSV/PDF export)
  - A/B Testing Infrastructure (statistical significance testing, winner detection)
  - Content Performance Scorer (predictive scoring with ML, historical lookup)
  - ROI Tracking System (UTM tracking, 4 attribution models, CPE/CPC calculation)
  - Engagement Analytics (best posting times, decay curves, virality tracking)
  - Brand Voice Effectiveness (correlation analysis, voice drift detection)
  - 5 database tables with complete schemas
  - Resilience patterns integration
- **Platform APIs**:
  - LinkedIn: Marketing API (share statistics, demographics)
  - Twitter/X: API v2 (organic metrics, video metrics)
  - Facebook: Graph API v18 (insights, demographics)
  - Instagram: Graph API (insights, story metrics)
- **Analytics Features**:
  - Real-time sync (15 min for recent posts, hourly for older)
  - Engagement rate, virality coefficient, content performance score (CPS)
  - Benchmark calculations (90-day rolling window)
  - Trend detection (linear regression, R² confidence)
  - Anomaly detection (Z-score based, 2σ threshold)
  - A/B testing (Chi-square, t-test, 95% confidence)
  - Performance prediction (vector similarity + GPT-4 analysis)
  - ROI attribution (first-touch, last-touch, linear, time-decay)
  - Best posting times (hour/day heatmaps)
  - Engagement decay curves (half-life calculation)
  - Brand voice correlation (Pearson correlation, p-value)
- **Implementation Notes**:
  - Infrastructure: Cloudflare Workers, D1, Cron Triggers
  - ML: OpenAI text-embedding-3-small, GPT-4
  - Monitoring: Datadog/Sentry/Prometheus, Grafana, PagerDuty
  - Visualization: Chart.js/Recharts
  - Platform costs: $100/month (Twitter/X Basic), $17/month (Cloudflare), $10/month (OpenAI), $15/month (Datadog)

---

## Stage G: Dify Implementation with Custom Media-Rich Frontend (Completed Planning)

### Purpose
Build the user-facing interface layer using a custom Next.js 15 enterprise-grade frontend (Supabase/ElevenLabs/Netflix quality) with full video and media support, integrated with Dify Cloud backend for workflow orchestration. Brings together all previous stages (C, D, E, F) into a production-ready conversational interface.

### Status
- Planning: ✅ Complete (Nov 10, 2025)
- Implementation: ⏸️ Pending (after Stage D, E)
- In Production: ❌ Not yet

### Approved Documents

#### 1. STAGE-G-APPROVED-PLAN.md ⭐
- **Location**: `stage-g-dify-implementation/`
- **Type**: Approved execution plan
- **Version**: 1.0.0
- **Status**: ✅ APPROVED (Nov 10, 2025)
- **Size**: Complete specification (16+ sections)
- **Purpose**: Custom frontend + Dify backend integration with enterprise-grade quality
- **Timeline**: 23 days (4.6 weeks)
- **Budget**: $86K development (230 hours @ $375/hour), $64/month operational
- **Key Deliverables**:
  - **Custom Next.js 15 Frontend**: Enterprise-grade UI (Supabase/ElevenLabs/Netflix quality)
  - **Full Media Support**: 1080p video, full-res images (1200x627px), carousels (9 images), GIFs
  - **Enterprise Tech Stack**: TypeScript strict, Tailwind CSS 4, Radix UI, shadcn/ui, TanStack Query, Zustand
  - **Testing**: Vitest + Playwright (80%+ coverage), E2E tests for all critical flows
  - **Performance**: Lighthouse 90+, <200KB bundle, <1.8s LCP (Netflix standards)
  - **Dify Backend**: 5 workflows (Main Chatflow, Skill Matcher, Content Generator, Publishing, Learning)
  - **Knowledge Bases**: 4 KBs (Brand Voice, Skill Library, Product Catalog, Example Posts)
  - **Conversational Advisory**: Text-based A/B/C options with media preview
  - **Mobile Optimization**: Touch gestures, adaptive streaming, PWA-ready
- **Architecture Decision**:
  - **Frontend**: Next.js custom UI (full media control) → Vercel/Cloudflare Pages
  - **Backend**: Dify Cloud Professional (workflow orchestration) → $59/month
  - **Services**: MCP Gateway (16 microservices) → Cloudflare Workers
  - **Why Custom Frontend**: Dify limits images to 250×250px (GitHub Issue #5892), no video/carousel support
- **Enterprise Stack Features**:
  - Framework: Next.js 15 (App Router), TypeScript 5.3 (strict mode), pnpm 8
  - UI: Tailwind CSS 4, Radix UI (WCAG-compliant), shadcn/ui (copy-paste components)
  - State: TanStack Query 5 (server state), Zustand 4.4 (client state)
  - Media: react-player 2.13 (video), react-image-gallery 1.3 (carousels), Framer Motion 11 (animations)
  - Testing: Vitest 1.0, Playwright 1.40, React Testing Library 14
  - DX: ESLint 8, Prettier 3, Husky 8, lint-staged
  - Monitoring: Sentry 7 (errors), Vercel Analytics (performance), Google Analytics 4
  - Forms: React Hook Form 7 + Zod 3 (validation)
- **Performance Targets** (Netflix standards):
  - Core Web Vitals: LCP <1.8s, FID <100ms, CLS <0.1, TTI <3.9s
  - Bundle: Initial JS <200KB gzipped, CSS <20KB gzipped
  - Image: WebP format, lazy loading, srcset responsive
  - Video: HLS adaptive streaming (1080p → 720p → 480p)
  - Code Splitting: Route-based + component-based (lazy load VideoPlayer)
- **Platform-Specific Media**:
  - LinkedIn: 1200x627px images, 1080p video (10 min max), 9-image carousels
  - Twitter/X: 1200x675px images, video 2:20 max, animated GIFs
  - Facebook: 1200x630px images, video 240 min max (10GB), 10-image carousels
  - Instagram: 1080x1080px images, Reels (1080x1920px vertical), Stories, 10-image carousels
- **Licensing & Scalability Research**:
  - ✅ Single workspace usage: LEGAL under Dify open-source license (Apache 2.0 modified)
  - ✅ Self-hosted Dify: Scales to Fortune 500 companies (verified AWS/Alibaba deployments)
  - ✅ Kubernetes support: Horizontal scaling (API, Worker, Sandbox pods)
  - ⚠️ Image display limitation: 250px max in Dify (GitHub Issue #5892 rejected)
  - ✅ Solution: Custom Next.js frontend for full-resolution media rendering
- **Supporting Research Documents** (created Nov 10, 2025):
  - `ENTERPRISE-FRONTEND-TECH-STACK-RESEARCH-2025.md`: Supabase/ElevenLabs/Netflix tech stack analysis
  - `ENTERPRISE-STACK-QUICK-REFERENCE.md`: Implementation quick start guide
  - `dify-implementation-research.md`: Dify capabilities validation (17,500 words)
- **Implementation Notes**:
  - Must deploy custom frontend (Dify native UI insufficient for media requirements)
  - Cloudflare Pages for deployment (integrated with Workers/R2, global edge network)
  - Dify self-hosted (open source) on Railway: ~$15/month OR Dify Cloud Professional: $59/month
  - Total operational cost: $20-74/month ($0.20-0.74/customer/month for 100 customers)

---

## Stage H: OpenManus Integration (Completed Planning)

### Purpose
Integrate OpenManus open-source multi-agent framework to provide autonomous intelligence capabilities beyond Dify's workflow limitations. Enables complex long-running tasks (5-30 minutes) like meta-skill generation, brand voice extraction, competitive analysis, and content refinement through coordinated AI agents.

### Status
- Planning: ✅ Complete (Nov 10, 2025)
- Implementation: ⏸️ Pending (after Stage G)
- In Production: ❌ Not yet

### Approved Documents

#### 1. STAGE-H-APPROVED-PLAN.md ⭐
- **Location**: `stage-h-openmanus/`
- **Type**: Approved execution plan
- **Version**: 1.0.0
- **Status**: ✅ APPROVED (Nov 10, 2025)
- **Size**: 16 comprehensive sections
- **Purpose**: OpenManus multi-agent system integration with Dify backend
- **Timeline**: 12 weeks (3 months, 280-480 hours)
- **Budget**: $105K-$180K development, $212/month operational ($2.12/customer/month)
- **Key Deliverables**:
  - **4 Specialized Agents**: Planning Agent (coordinator), Research Agent (data gathering), Coder Agent (skill generation), Reporter Agent (synthesis)
  - **Fly.io Deployment**: 9 machines (7× shared-cpu-2x, 2× shared-cpu-1x)
  - **Redis Shared Memory**: 3-node cluster, pub/sub messaging, task state management
  - **4 Custom API Endpoints**:
    - `/api/generate-skill/start` - Meta-skill generation (async, 5-10 min)
    - `/api/analyze-brand-voice/start` - Brand voice extraction (async, 3-5 min)
    - `/api/refine-content` - Content improvement (sync, 15-45s)
    - `/api/competitive-analysis/start` - Competitor analysis (async, 10-30 min)
  - **Dify Integration**: HTTP Tool configurations, async polling pattern (START → POLL → RESULT)
  - **Task Queue**: Cloudflare Durable Objects (strongly consistent state, progress tracking)
  - **Agent Communication**: Redis pub/sub, message routing, error handling
  - **Security**: API key auth, tenant isolation, rate limiting (10 concurrent tasks/tenant)
  - **Monitoring**: Prometheus metrics, Grafana dashboards, PagerDuty alerting
  - **Testing**: Unit tests (80%+ coverage), integration tests, load tests (50 concurrent tasks)
- **Architecture Highlights**:
  - **OpenManus Framework**: Open-source by MetaGPT team (2025), supports GPT-4, Claude, Gemini
  - **Hybrid System**: Dify (orchestration) + OpenManus (intelligence) + Custom Frontend (UI)
  - **Async Polling Pattern**: Workaround for Dify 120s timeout limitation
  - **Multi-Agent Coordination**: Agents collaborate via Redis messaging (JSON-RPC format)
  - **Fly.io Infrastructure**: 2 planning agents, 2 research agents, 2 coder agents, 1 reporter agent, 2 API gateways
  - **Upstash Redis**: Pro 2K plan ($30/mo), 2GB RAM, 2K concurrent connections, auto-failover
  - **Durable Objects**: Task queue manager, status tracker, result store (strongly consistent)
- **API Endpoint Specifications**:
  1. **Generate Skill** (Async, 5-10 min):
     - Input: Skill name, description, examples, target use cases
     - Process: Planning → Research → Code → Test → Report
     - Output: Skill code, tests, documentation, integration guide
  2. **Analyze Brand Voice** (Async, 3-5 min):
     - Input: Documents, website URLs, social media posts
     - Process: Research → Extract patterns → Score consistency → Report
     - Output: Brand voice profile, style guide, example transformations
  3. **Refine Content** (Sync, 15-45s):
     - Input: Content, refinement type, target metrics
     - Process: GPT-4 analysis → Rewrite → Score improvement
     - Output: Refined content, before/after metrics, improvement explanation
  4. **Competitive Analysis** (Async, 10-30 min):
     - Input: Competitors list, analysis dimensions
     - Process: Research → Scrape → Analyze → Compare → Report
     - Output: Competitor matrix, strengths/weaknesses, opportunities
- **Dify HTTP Tool Configurations**: Complete JSON schemas for all 4 endpoints with polling logic
- **Fly.io Configurations**: Production-ready fly.toml configs for all agents
- **Supporting Research Documents** (created Nov 10, 2025):
  - `openmanus-architecture-research.md`: OpenManus framework analysis (5000 words)
  - `dify-openmanus-integration-design.md`: Integration patterns (2850+ lines)
  - `stage-h-cost-timeline-analysis.md`: Budget and timeline breakdown (2500+ words)
- **Implementation Notes**:
  - Fly.io machines: $150/month (9 machines, 7× shared-cpu-2x, 2× shared-cpu-1x)
  - Upstash Redis: $30/month (Pro 2K plan, 2GB RAM, auto-failover)
  - OpenManus agents need GPT-4 access ($8/month for 100 customers)
  - Cloudflare Durable Objects for task queue ($2/month)
  - Datadog monitoring: $15/month
  - Total infrastructure: $212/month ($2.12/customer/month)
  - Development effort: 280-480 hours over 12 weeks (2-3 senior engineers)

---

## Stage I: MCP Gateway Expansion (Completed Planning)

### Purpose
Expand the MCP Gateway from 9 to 44 tools and add a REST API layer for Dify HTTP Tools integration. Enables all microservices from Stages C, D, E, F to be accessible via Dify workflows. Critical integration layer connecting Dify orchestration to backend services.

### Status
- Planning: ✅ Complete (Nov 10, 2025)
- Implementation: ⏸️ Pending (after Stage C)
- In Production: ❌ Not yet

### Approved Documents

#### 1. STAGE-I-APPROVED-PLAN.md ⭐
- **Location**: `stage-i-mcp-gateway/`
- **Type**: Approved execution plan
- **Version**: 1.0.0
- **Status**: ✅ APPROVED (Nov 10, 2025)
- **Size**: 2,371 lines, 16 comprehensive sections
- **Purpose**: Expand MCP Gateway with 35 new tools + REST API layer
- **Timeline**: 5 weeks (505 hours)
- **Budget**: $75,750 development (15% contingency = $90,000 total), $10/month operational
- **Key Deliverables**:
  - **44 Tools Total**: 9 existing + 35 new tools
    - Knowledge Base (7): Brand voice, skills, products, examples
    - Content Generation (11): Intent parsing, candidates, guards, reranking
    - Publishing (9): LinkedIn, Twitter, Facebook, Instagram, scheduling
    - Analytics (8): Metrics, insights, performance, ROI tracking
  - **24 REST API Endpoints**:
    - Content & Images (4): Generate, validate, score, analyze
    - E-Commerce (5): Inventory, orders, reviews, pricing, stock
    - Social & Publishing (5): Multi-platform, batch, schedule, calendar
    - Analytics (3): Engagement, audience, attribution
    - Integration (3): Webhooks, API keys, compliance
    - Authentication (4): Login, tokens, permissions, audit
  - **Security Architecture**:
    - API key management (generation, validation, rotation, revocation)
    - Multi-tenant isolation (RLS + KV namespaces)
    - RBAC with 3 roles (Admin, User, ReadOnly)
    - Rate limiting (token bucket: 100-1000 req/hour by category)
    - OWASP Top 10 compliance
    - GDPR & SOC 2 requirements
    - Audit logging (D1 database)
  - **Testing Strategy**:
    - 515+ automated tests (80%+ coverage)
    - Unit (400+), Integration (60), Load (12), Security (25+), Chaos (8), UAT (10)
    - Tools: Vitest, Miniflare, k6, OWASP ZAP, Snyk
    - Performance: P95 <500ms, error rate <0.1%
    - CI/CD: GitHub Actions pipeline
  - **Deployment Strategy**:
    - Phased rollout (6 phases over 12 weeks)
    - Blue-green deployment (zero downtime)
    - Canary testing (5% → 25% → 50% → 100%)
    - Feature flags for gradual rollout
    - Monitoring: Grafana dashboards, PagerDuty alerts
    - Disaster recovery: RTO <15 min, RPO <5 min
- **Implementation Roadmap**:
  - Week 1: REST API foundation (authentication, rate limiting)
  - Week 2: Core tools (6 priority 1 tools)
  - Week 3: Extended tools (remaining 29 tools)
  - Week 4: Integration testing (Dify, load, security)
  - Week 5: Deployment & documentation
- **Tool Priority Breakdown**:
  - P1 (Critical): 15 tools - Core workflow (intent → generate → publish)
  - P2 (Important): 12 tools - Knowledge base + analytics
  - P3 (Nice-to-have): 8 tools - A/B testing, advanced ROI
- **Cost Analysis**:
  - Development: $75,750 (505 hours @ $150/hour)
  - Tool development: $21,000 (14 new tools × 10 hours avg)
  - REST API: $18,150 (121 hours)
  - Authentication: $10,800 (72 hours)
  - Testing: $15,600 (104 hours)
  - Documentation: $10,200 (68 hours)
  - Contingency: $11,363 (15%)
  - Total budget: $90,000
  - Operational: $10/month baseline, scales to $417/month at 5K customers
  - Per-customer: $0.55/month at 100 customers
  - ROI: 500%+ over 5 years, breaks even at ~100 customers
- **Supporting Research Documents** (created Nov 10, 2025):
  - `STAGE-I-REST-API-SPECIFICATION.md`: Complete 24 endpoint specs (57 KB)
  - `STAGE-I-SECURITY-ARCHITECTURE.md`: Auth, RBAC, compliance (comprehensive)
  - `STAGE-I-MCP-GATEWAY-TOOL-INVENTORY.md`: All 44 tools detailed (79 KB)
  - `STAGE-I-DEPLOYMENT-ROLLOUT-STRATEGY.md`: Phased rollout plan
  - `STAGE-I-TESTING-STRATEGY.md`: 515+ tests, CI/CD pipeline
  - `STAGE-I-IMPLEMENTATION-ROADMAP.md`: 5-week day-by-day plan
  - `STAGE-I-QUICK-REFERENCE.md`: 3-page executive summary
  - `CURL-EXAMPLES.md`: 24 cURL commands for testing
- **Implementation Notes**:
  - Cloudflare Workers deployment (existing infrastructure)
  - KV namespaces for API keys + rate limits
  - D1 database for audit logs
  - Gradual rollout with feature flags
  - Backward compatible (JSON-RPC continues working)
  - Team: 1 senior full-stack engineer (Week 1-5)

---

## Supporting Documentation Location

**Research Documents**: NOT in this folder
**Location**: `.planning/research/` (21 documents)
**Purpose**: Supporting research, technical designs, and analysis
**Status**: Reference material, not for direct implementation

**Examples**:
- `research/rag-architecture-research.md` (95KB) - RAG implementation guide
- `research/web-scraper-integration.md` - Firecrawl integration research
- `research/knowledge-extraction-engine.md` - GPT-5 content multiplication
- And 18 more research documents...

**When to Use Research Docs**: When implementing an approved plan and need technical details, code examples, or architecture guidance.

---

## Navigation Documents Location

**Location**: `.planning/` (root folder)
**Purpose**: Project orientation and navigation
**Status**: Active project documentation

**Examples**:
- `README.md` - Project overview
- `START-HERE.md` - Quick start guide (5 minutes)
- `INDEX.md` - Document navigation
- `QUICK-START-GUIDE.md` - Developer quick start (7 days)

---

## How to Use This Folder

### For Planning Agents
When completing a new stage (D, E, F, etc.):
1. Move approved plan documents to `approved/stage-X-name/`
2. Move approved ADRs to `approved/stage-X-name/decisions/`
3. Update this index (APPROVED-DOCUMENTS-INDEX.md)
4. Update SYSTEM-OVERVIEW.md with new stage summary

### For Implementation Teams
When starting implementation:
1. Read the stage's APPROVED PLAN first
2. Check READINESS CHECKLIST (if exists)
3. Use EXECUTION TRACKER for progress tracking
4. Reference research documents as needed
5. Follow ADR decisions (non-negotiable)

### For Stakeholders
When reviewing project:
1. Start with SYSTEM-OVERVIEW.md (big picture)
2. Read stage APPROVED PLAN (specific stage details)
3. Check EXECUTION TRACKER (progress status)
4. Review ADRs (architectural decisions and rationale)

---

## Version History

| Date | Version | Change | Agent |
|------|---------|--------|-------|
| 2025-11-09 | 1.0.0 | Initial approved folder creation | Claude Planning Agent |
| 2025-11-10 | 1.1.0 | Added Stage D & E approved plans | Claude Planning Agent |
| 2025-11-10 | 1.2.0 | Added cross-cutting STAGE-X-RESILIENCE-PATTERNS.md | Claude Planning Agent |
| 2025-11-10 | 1.3.0 | Added Stage F (Analytics & Performance) approved plan | Claude Planning Agent |
| 2025-11-10 | 1.4.0 | Added Stage G (Dify Implementation) approved plan | Claude Planning Agent |
| 2025-11-10 | 1.5.0 | Company references genericized for reusability | Claude Cleanup Agent |
| 2025-11-10 | 1.6.0 | Added Stage H (OpenManus Integration) approved plan | Claude Planning Agent |
| 2025-11-10 | 1.7.0 | Added Stage I (MCP Gateway Expansion) approved plan | Claude Planning Agent |
| 2025-11-10 | 2.0.0 | Added Stages J, K, L - ALL PLANNING COMPLETE (100%) | Claude Planning Agent |

---

## Future Stages

As new stages (I, J, K, L, etc.) are planned and approved:
- Create `approved/stage-X-name/` folder
- Move approved documents there
- Update this index
- Update SYSTEM-OVERVIEW.md

**Process Guarantee**: At end of each planning phase, approved documents move here automatically, ensuring clean state for development.

---

**Questions?** See `.planning/README.md` or `.planning/START-HERE.md`

**Last Updated**: November 10, 2025
**Maintained By**: Planning agents
**Review Frequency**: After each stage planning completion
