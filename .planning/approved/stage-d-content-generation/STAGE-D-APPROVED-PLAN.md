# STAGE D: CONTENT GENERATION WORKFLOW - APPROVED PLAN

**Version:** 1.0.0 (cleaned for public release)
**Status:** ✅ APPROVED - Production Ready
**Date:** November 10, 2025
**Project:** DIFY-AGENT - AI-Powered Social Media Content System
**Phase:** Planning Complete - Ready for Implementation

---

## Document Purpose

This is the **production-ready specification** for Stage D: Content Generation Workflow. This document is **self-contained** and includes all details needed for implementation. Development teams should NOT need to reference any planning or research documents.

**What this stage delivers:**
- Intelligent content generation workflow with multi-candidate generation
- Central skill bank with per-tenant brand voice isolation
- Automated quality guards and defect classification
- Multi-tenant safe learning system
- Production-ready APIs and data schemas

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Services Architecture](#2-services-architecture)
3. [Data Schemas](#3-data-schemas)
4. [Multi-Tenant Strategy](#4-multi-tenant-strategy)
5. [Multi-Candidate Generation](#5-multi-candidate-generation)
6. [Guards Engine](#6-guards-engine)
7. [Defect vs Preference Classification](#7-defect-vs-preference-classification)
8. [APIs](#8-apis)
9. [Cost Analysis](#9-cost-analysis)
10. [Implementation](#10-implementation)
11. [Security & Compliance](#11-security--compliance)
12. [Observability](#12-observability)
13. [Integration](#13-integration)

---

## 1. System Overview

### 1.1 End-to-End Flow

```
User Request (text or voice)
    ↓
Intent Parser → IntentSpec JSON
    ↓
Skill Retrieval (similarity search on skill embeddings)
    ↓
Multi-Candidate Generator (2-3 diverse outputs)
    ↓
Guards Engine (automated quality checks)
    ↓
Reranker (score + sort by quality/cost/diversity)
    ↓
User Preview & Selection
    ↓
User Feedback (approve/revise)
    ↓
Defect-vs-Preference Classifier
    ↓
Performance Logger → Skill Metrics
    ↓
Learning Loop (auto-promote/retire skills)
```

### 1.2 Core Philosophy

**Conversational Advisory Pattern:**
- User types requests in natural language
- System presents options as text (A/B/C choices)
- User selects via text response
- Multi-turn conversations handle complex workflows
- **NO GUI buttons required** - pure conversational text

**Multi-Tenant Safe Learning:**
- **Shared:** Structural patterns (layouts, guards, constraints) - generic best practices
- **Private:** Brand voice (tone, lexicon, colors, logos) - competitive secrets
- Result: System learns general patterns while preserving brand differentiation

### 1.3 Key Innovations

1. **Central Invisible Skill Bank** - Shared structural intelligence across all tenants
2. **Per-Tenant Brand Voice Layer** - Private, isolated brand customization
3. **Multi-Candidate Generation** - 2-3 diverse outputs with intelligent reranking
4. **Guards Engine** - Automated quality gates with auto-fixers
5. **Defect-vs-Preference Classification** - Distinguish failures from stylistic preferences
6. **Adaptive Exploration** - ε-greedy bandit for controlled experimentation
7. **Production Metrics** - End-to-end observability with auto-promotion/retirement

---

## 2. Services Architecture

### 2.1 Service Map (11 Services)

```
┌─────────────────────────────────────────────────────────────────┐
│                     DIFY ORCHESTRATION LAYER                     │
│              (Manages conversation state & workflow)             │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │   MCP Gateway (REST)  │
                    │  (Tool coordination)   │
                    └───────────┬───────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
┌───────▼────────┐  ┌──────────▼──────────┐  ┌────────▼────────┐
│ Intent Parser  │  │  Skill Retrieval    │  │ Multi-Candidate │
│   Service      │  │     Service         │  │   Generator     │
└───────┬────────┘  └──────────┬──────────┘  └────────┬────────┘
        │                      │                       │
        │           ┌──────────▼──────────┐           │
        │           │   Guards Engine     │◄──────────┘
        │           │    Service          │
        │           └──────────┬──────────┘
        │                      │
        │           ┌──────────▼──────────┐
        │           │    Reranker         │
        │           │    Service          │
        │           └──────────┬──────────┘
        │                      │
        │           ┌──────────▼──────────┐
        │           │ Defect Classifier   │
        │           │    Service          │
        │           └──────────┬──────────┘
        │                      │
        └──────────────────────┼──────────────────────┐
                               │                      │
                    ┌──────────▼──────────┐  ┌────────▼────────┐
                    │  Learning Engine    │  │ Brand Voice     │
                    │     Service         │  │   Service       │
                    └──────────┬──────────┘  └─────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Publishing         │
                    │    Service          │
                    └─────────────────────┘
```

### 2.2 Service Details

#### 2.2.1 Intent Parser Service
**Purpose:** Convert natural language requests to structured IntentSpec JSON

**Tech Stack:**
- OpenAI GPT-5 for intent extraction
- spaCy for NER (product names, platforms)
- RapidFuzz for fuzzy product matching
- Cloudflare Workers runtime

**Input:**
```json
{
  "user_message": "Create a LinkedIn post about our Product SKU-Product SKU-A142 with pricing",
  "tenant_id": "tenant-001",
  "conversation_history": [...]
}
```

**Output:** IntentSpec JSON (see Section 3.2)

**API:** `POST /api/v1/intent/parse`

**Dependencies:**
- OpenAI API (GPT-5)
- Supabase products database (product verification)
- Cloudflare KV (context cache)

---

#### 2.2.2 Skill Retrieval Service
**Purpose:** Find matching skills from central bank using similarity search

**Tech Stack:**
- Cloudflare Vectorize (vector similarity search)
- voyage-3-lite embeddings (1024 dimensions)
- Hybrid search (vector + metadata filters)

**Retrieval Strategy:**
```python
# Similarity thresholds
≥ 0.80: Reuse skill as-is (high confidence)
0.65-0.80: Adapt skill with minor modifications
< 0.65: Generate new skill from scratch

# Query composition
query_embedding = embed(IntentSpec.task + IntentSpec.platform + IntentSpec.modality)
filters = {
  "task": IntentSpec.task,
  "platform": IntentSpec.platform,
  "modality": IntentSpec.modality,
  "status": "active"  # Only retrieve non-retired skills
}
```

**API:** `POST /api/v1/skills/retrieve`

**Dependencies:**
- Cloudflare Vectorize (skill embeddings index)
- Cloudflare D1 (skill metadata)

---

#### 2.2.3 Multi-Candidate Generator Service
**Purpose:** Generate 2-3 diverse content variations for user selection

**Tech Stack:**
- Claude Sonnet 4.5 (primary - high quality text)
- GPT-5 Mini (budget alternative)
- Gemini 2.5 Flash (image generation)
- Temperature sampling for diversity

**Generation Strategy:**
```python
# Candidate A: Best matching skill (similarity-based)
candidate_a = apply_skill(best_skill, brand_voice, assets)

# Candidate B: Second-best skill OR adapted version
if second_skill.similarity >= 0.65:
    candidate_b = apply_skill(second_skill, brand_voice, assets)
else:
    candidate_b = adapt_skill(best_skill, temperature=0.8)

# Candidate C: Exploratory (10% probability via ε-greedy)
if random() < 0.1:  # ε = 0.1
    candidate_c = generate_novel(IntentSpec, brand_voice, temperature=1.0)
else:
    candidate_c = apply_skill(third_skill, brand_voice, assets)
```

**API:** `POST /api/v1/generate/multi-candidate`

**Dependencies:**
- Anthropic API (Claude)
- OpenAI API (GPT-5)
- Google API (Gemini)
- Brand Voice Service (tenant-specific voice)

---

#### 2.2.4 Guards Engine Service
**Purpose:** Automated quality checks with auto-fixers

**Tech Stack:**
- Python (guard logic)
- OpenCV (image analysis)
- Pillow (image manipulation)
- Cloudflare Workers

**7 Guards Implemented:**

1. **Text Overlap Guard**
   - Detects text overlapping product images or faces
   - Auto-fix: Reposition text to safe zones
   - Threshold: Max 5% overlap allowed

2. **Safe Margin Guard**
   - Ensures minimum margin from edges
   - Auto-fix: Add padding to layout
   - Threshold: Min 32px on all sides

3. **Contrast Guard**
   - Checks text/background contrast ratio
   - Auto-fix: Adjust text color or add shadow
   - Threshold: Min 4.5:1 (WCAG AA)

4. **Logo Protection Guard**
   - Prevents covering brand logos
   - Auto-fix: Move overlapping elements
   - Threshold: 0% logo coverage allowed

5. **Aspect Ratio Fit Guard**
   - Validates platform requirements
   - Auto-fix: Crop/resize to target aspect
   - Examples: LinkedIn 1200×628, Instagram 1080×1920

6. **Brand Palette Guard**
   - Ensures only approved colors used
   - Auto-fix: Snap to nearest brand color
   - Source: Brand profile palette

7. **Font Size Guard**
   - Validates minimum readable font sizes
   - Auto-fix: Scale text to minimum
   - Threshold: Min 22pt for body text

**API:** `POST /api/v1/guards/check`

**Output:**
```json
{
  "pass": false,
  "failed_guards": ["contrast", "logo_protection"],
  "auto_fixed": true,
  "fixed_output": "data:image/png;base64,...",
  "guard_results": [
    {
      "guard_id": "contrast",
      "pass": false,
      "metrics": {"measured_ratio": 3.2, "required": 4.5},
      "action_taken": "Added text shadow"
    }
  ]
}
```

---

#### 2.2.5 Reranker Service
**Purpose:** Score and sort candidates by quality, cost, diversity

**Tech Stack:**
- Python (scoring algorithm)
- Cloudflare Workers

**Scoring Formula:**
```python
Score(candidate) = (
    α · similarity(IntentSpec, skill)      # How well skill matches intent
  + β · guard_pass_rate                     # Quality (% guards passed)
  + γ · performance_prior(context)          # Historical success in similar contexts
  + δ · aesthetic_score                     # Visual appeal (if image)
  + ε · diversity_bonus                     # Novelty vs other candidates
  - ζ · cost_penalty                        # API cost normalization
)

# Default weights (tunable per tenant)
α = 0.30  # Intent match
β = 0.25  # Quality
γ = 0.20  # Historical performance
δ = 0.15  # Aesthetics
ε = 0.05  # Diversity
ζ = 0.05  # Cost
```

**API:** `POST /api/v1/rerank`

**Dependencies:**
- Guards Engine (guard results)
- Learning Engine (performance priors)

---

#### 2.2.6 Defect Classifier Service
**Purpose:** Distinguish skill failures (defects) from user style preferences

**Tech Stack:**
- Claude Sonnet 4.5 (classification reasoning)
- Rule-based heuristics (fast path)
- Cloudflare Workers

**Classification Logic:**

**DEFECT** (skill failure - impacts skill metrics):
- Failed guard checks (contrast, margins, overlap)
- Factual errors (wrong product, wrong price)
- Technical issues (broken images, missing assets)
- Brand guideline violations (wrong colors, wrong fonts)

**PREFERENCE** (user style choice - does NOT impact skill metrics):
- Tone adjustments ("make it more casual")
- Layout preferences ("swap headline and image")
- Content additions ("add a customer quote")
- Creative direction ("use different metaphor")

**Examples:**
```python
# DEFECT
"The text is unreadable due to low contrast" → defect (failed contrast guard)
"Wrong product image shown" → defect (factual error)

# PREFERENCE
"I want a more friendly tone" → preference (style choice)
"Can we use a different layout?" → preference (creative direction)
"Add our new tagline" → preference (content addition)
```

**API:** `POST /api/v1/classify-feedback`

**Output:**
```json
{
  "classification": "defect",
  "confidence": 0.92,
  "labels_defect": ["low_contrast", "text_overlap"],
  "labels_preference": [],
  "reasoning": "User reported unreadable text. Contrast guard failed (3.2:1 vs 4.5:1 required). This is a skill execution failure."
}
```

---

#### 2.2.7 Learning Engine Service
**Purpose:** Auto-promote/retire skills based on performance metrics

**Tech Stack:**
- Python (analytics)
- Cloudflare D1 (metrics storage)
- Cron Triggers (daily jobs)

**Skill Lifecycle (4 Tiers):**

```python
# Champions: Success rate ≥ 85%
- Status: "champion"
- Selection weight: 2.0x
- Auto-promote to global if: uses ≥ 100 AND defect_rate < 5%

# Active: Success rate 70-85%
- Status: "active"
- Selection weight: 1.0x
- Standard rotation

# Testing: Success rate 50-70%
- Status: "testing"
- Selection weight: 0.5x
- Higher exploration probability
- Auto-retire if: no improvement after 50 uses

# Failing: Success rate < 50%
- Status: "failing"
- Selection weight: 0.1x
- Auto-retire after 7 days if no improvement
```

**Promotion to Global Bank:**
```python
# Criteria for sharing skill across tenants
if skill.uses >= 100 AND \
   skill.defect_rate < 0.05 AND \
   skill.unique_tenants >= 5 AND \
   skill.age_days >= 30:
    promote_to_global(skill_id)
    log_audit_event("promotion", skill_id, reason="High performance across 5+ tenants")
```

**API:** `POST /api/v1/learning/analyze` (internal, cron-triggered)

---

#### 2.2.8 Brand Voice Service
**Purpose:** Retrieve tenant-specific brand voice (private, isolated)

**Tech Stack:**
- Cloudflare D1 (brand profiles)
- Cloudflare Vectorize (brand voice embeddings)
- Per-tenant KMS encryption

**Brand Profile Schema:**
```sql
CREATE TABLE brand_profiles (
  tenant_id TEXT PRIMARY KEY,
  embeddings BLOB,              -- Encrypted brand voice vectors
  palette JSON,                 -- ["#1a5f7a", "#57cc99", ...]
  typography JSON,              -- {"headline": "Montserrat Bold 48pt", ...}
  logo_rules JSON,              -- {"min_size_px": 120, "safe_zone_px": 40}
  tone_vectors BLOB,            -- Encrypted tone embeddings
  lexicon JSON,                 -- {"products": "industrial-grade rebar", ...}
  kms_key_id TEXT,              -- Per-tenant encryption key
  updated_at TIMESTAMP,
  FOREIGN KEY (tenant_id) REFERENCES tenants(tenant_id)
);
```

**Row-Level Security (RLS):**
```sql
-- Enforce tenant isolation
CREATE POLICY tenant_isolation ON brand_profiles
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant'));
```

**API:** `GET /api/v1/brand-voice/{tenant_id}`

**Privacy Guarantees:**
- ✅ Per-tenant KMS encryption keys
- ✅ RLS enforces data isolation
- ✅ Audit logs track all access
- ✅ Per-region data residency

---

#### 2.2.9 Publishing Service
**Purpose:** Publish approved content to social platforms

**Tech Stack:**
- Platform SDKs (LinkedIn, Twitter, Instagram, Facebook)
- OAuth 2.0 with PKCE
- Cloudflare Workers

**Supported Platforms:**
1. **LinkedIn** - API v2 (ugcPosts endpoint)
2. **Twitter/X** - API v2 (tweets endpoint)
3. **Instagram** - Graph API (media endpoint)
4. **Facebook** - Graph API (feed endpoint)

**Publishing Workflow:**
```python
1. Validate OAuth token (refresh if expired)
2. Upload media assets (if image/video)
3. Create post with platform-specific payload
4. Poll for publish confirmation
5. Store post_id in analytics
6. Log event for metrics
```

**API:** `POST /api/v1/publish`

---

#### 2.2.10 Analytics Service
**Purpose:** Store and query performance metrics

**Tech Stack:**
- Cloudflare D1 (metrics storage)
- Cloudflare Analytics Engine (real-time)
- Daily aggregation jobs

**Metrics Tracked:**
- Skill usage counts
- Guard pass/fail rates
- Defect vs preference ratios
- User approval rates
- Platform-specific engagement (if connected)
- Cost per request
- Latency (p50, p95, p99)

**API:** `POST /api/v1/analytics/log`

---

#### 2.2.11 Context Service
**Purpose:** Manage conversation context and memory

**Tech Stack:**
- Cloudflare KV (session storage)
- Dify conversation state
- TTL: 24 hours

**Context Storage:**
```json
{
  "session_id": "sess_abc123",
  "tenant_id": "tenant-001",
  "conversation_history": [
    {"role": "user", "content": "Create a LinkedIn post about Product SKU-A142"},
    {"role": "assistant", "content": "I found 3 variations..."}
  ],
  "intent_spec": {...},
  "selected_candidate": "candidate_a",
  "draft_content": {...},
  "ttl": 86400
}
```

**API:** `GET /api/v1/context/{session_id}`

---

## 3. Data Schemas

### 3.1 IntentSpec Schema

**Purpose:** Structured representation of user's content request

```json
{
  "task": "social_image_post | ad | banner | story | email_header | product_card | infographic",
  "platform": "linkedin | instagram_story | x | facebook | email | web",
  "modality": "image | image+text | video | carousel",
  "aspect_ratio": "1080x1920 | 1200x628 | 1:1 | 16:9",

  "must_have": [
    "product_image",
    "price_tag",
    "cta_button",
    "company_logo"
  ],

  "avoid": [
    "text_over_face",
    "cover_logo",
    "competitor_colors"
  ],

  "copy_blocks": [
    {
      "name": "headline",
      "text": "Premium Product SKU-Product SKU-A142 - Next Day Delivery",
      "max_chars": 60
    },
    {
      "name": "cta",
      "text": "Order Now",
      "max_chars": 20
    }
  ],

  "constraints": {
    "min_font_pt": 22,
    "safe_margin_px": 32,
    "brand_palette_only": true,
    "max_text_blocks": 3
  },

  "assets": [
    {
      "type": "logo",
      "uri": "tenant://logos/main.svg",
      "required": true
    },
    {
      "type": "product_image",
      "uri": "shopify://products/a142-rebar/image",
      "required": true
    }
  ],

  "context": {
    "product_id": "prod_a142_mesh",
    "campaign": "spring_2025_promo",
    "target_audience": "construction_managers"
  },

  "confidence": 0.0  // Populated by Intent Parser
}
```

**Validation Rules:**
- `task` must match supported task types
- `platform` must have valid aspect ratios
- `modality` must be compatible with platform
- `copy_blocks[].max_chars` enforced strictly
- `assets[].uri` must resolve to valid resource

---

### 3.2 Database Tables (9 Tables)

#### 3.2.1 Tenants Table
```sql
CREATE TABLE tenants (
  tenant_id TEXT PRIMARY KEY,           -- e.g., "tenant-001"
  name TEXT NOT NULL,                   -- "Company XYZ"
  region TEXT NOT NULL,                 -- "eu-west", "us-east"
  kms_key_id TEXT NOT NULL,             -- Per-tenant encryption key
  tier TEXT DEFAULT 'standard',         -- "free", "standard", "enterprise"
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status TEXT DEFAULT 'active'          -- "active", "suspended", "deleted"
);

CREATE INDEX idx_tenants_region ON tenants(region);
CREATE INDEX idx_tenants_status ON tenants(status);
```

---

#### 3.2.2 Brand Profiles Table (PER-TENANT PRIVATE)
```sql
CREATE TABLE brand_profiles (
  tenant_id TEXT PRIMARY KEY,
  embeddings BLOB,                      -- Encrypted brand voice vectors (1024-dim)
  palette JSON NOT NULL,                -- ["#1a5f7a", "#57cc99", "#f0f3bd"]
  typography JSON NOT NULL,             -- {"headline": "Montserrat Bold 48pt", "body": "Open Sans Regular 16pt"}
  logo_rules JSON NOT NULL,             -- {"min_size_px": 120, "safe_zone_px": 40, "backgrounds": ["light", "dark"]}
  tone_vectors BLOB,                    -- Encrypted tone embeddings
  lexicon JSON,                         -- {"products": "industrial-grade rebar", "values": "reliability, expertise"}
  kms_key_id TEXT NOT NULL,             -- Must match tenants.kms_key_id
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (tenant_id) REFERENCES tenants(tenant_id)
);

-- RLS Policy (Row-Level Security)
CREATE POLICY tenant_isolation ON brand_profiles
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant'));
```

**Encryption:** All BLOB fields encrypted with per-tenant KMS keys.

---

#### 3.2.3 Skills Table (SHARED - GLOBAL)
```sql
CREATE TABLE skills (
  skill_id TEXT PRIMARY KEY,            -- e.g., "skill_linkedin_3col_product"
  name TEXT NOT NULL,                   -- "3-Column Product Layout (LinkedIn)"
  task TEXT NOT NULL,                   -- "social_image_post"
  platform TEXT NOT NULL,               -- "linkedin"
  modality TEXT NOT NULL,               -- "image+text"
  visibility TEXT DEFAULT 'global',     -- "global" (shared) or "tenant:{id}" (private)
  parent_skill_id TEXT,                 -- For skill evolution/forking
  status TEXT DEFAULT 'active',         -- "active", "testing", "champion", "failing", "retired"
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (parent_skill_id) REFERENCES skills(skill_id)
);

CREATE INDEX idx_skills_task_platform ON skills(task, platform, modality);
CREATE INDEX idx_skills_status ON skills(status);
CREATE INDEX idx_skills_visibility ON skills(visibility);
```

**Visibility Rules:**
- `global`: Available to all tenants (shared structural patterns)
- `tenant:{id}`: Private to specific tenant (not shared)

---

#### 3.2.4 Skill Versions Table
```sql
CREATE TABLE skill_versions (
  skill_version_id TEXT PRIMARY KEY,    -- e.g., "skill_linkedin_3col_v12"
  skill_id TEXT NOT NULL,
  prompt_fragments_json JSON NOT NULL,  -- {"system": "...", "user_template": "..."}
  constraints_json JSON NOT NULL,       -- {"min_font_pt": 22, "safe_margin_px": 32}
  toolchain_json JSON NOT NULL,         -- {"image_model": "gemini-2.5-flash", "text_model": "claude-sonnet-4.5"}
  guards_json JSON NOT NULL,            -- ["contrast", "safe_margin", "logo_protection"]
  version_notes TEXT,                   -- "Fixed contrast issue in low-light product photos"
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (skill_id) REFERENCES skills(skill_id)
);

CREATE INDEX idx_skill_versions_skill ON skill_versions(skill_id);
```

---

#### 3.2.5 Skill Embeddings Table
```sql
CREATE TABLE skill_embeddings (
  embedding_id TEXT PRIMARY KEY,
  skill_id TEXT NOT NULL,
  skill_version_id TEXT NOT NULL,
  vector BLOB NOT NULL,                 -- 1024-dim embedding (voyage-3-lite)
  tags JSON,                            -- ["product_focus", "minimalist", "high_contrast"]
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (skill_id) REFERENCES skills(skill_id),
  FOREIGN KEY (skill_version_id) REFERENCES skill_versions(skill_version_id)
);

CREATE INDEX idx_skill_embeddings_skill ON skill_embeddings(skill_id);
```

**Vectorize Integration:**
- Embeddings also stored in Cloudflare Vectorize for fast similarity search
- This table serves as backup/audit trail

---

#### 3.2.6 Skill Metrics Daily Table
```sql
CREATE TABLE skill_metrics_daily (
  date DATE NOT NULL,
  skill_id TEXT NOT NULL,
  skill_version_id TEXT NOT NULL,
  context_hash TEXT NOT NULL,           -- Hash of (task + platform + modality) for context grouping

  -- Usage metrics
  uses INTEGER DEFAULT 0,               -- Times skill was selected
  pass_rate REAL DEFAULT 0.0,           -- % that passed all guards
  defect_rate REAL DEFAULT 0.0,         -- % marked as defects (not preferences)
  selection_rate REAL DEFAULT 0.0,      -- % user selected this candidate (if multi-gen)
  approval_rate REAL DEFAULT 0.0,       -- % user approved without revisions

  -- Performance metrics
  cost_avg REAL DEFAULT 0.0,            -- Avg API cost per use (USD)
  latency_p50 INTEGER DEFAULT 0,        -- Median latency (ms)
  latency_p95 INTEGER DEFAULT 0,        -- 95th percentile latency (ms)

  -- Aggregation metadata
  unique_tenants INTEGER DEFAULT 0,     -- Count of distinct tenants using this skill (for K-anonymity)

  PRIMARY KEY (date, skill_id, context_hash),
  FOREIGN KEY (skill_id) REFERENCES skills(skill_id)
);

CREATE INDEX idx_metrics_skill_date ON skill_metrics_daily(skill_id, date DESC);
CREATE INDEX idx_metrics_context ON skill_metrics_daily(context_hash, date DESC);
```

**Privacy:** Only aggregate metrics stored. Individual tenant performance kept separate.

---

#### 3.2.7 Revision Events Table
```sql
CREATE TABLE revision_events (
  event_id TEXT PRIMARY KEY,
  request_id TEXT NOT NULL,             -- Links to original generation request
  tenant_id TEXT NOT NULL,              -- Which tenant requested revision
  skill_id TEXT NOT NULL,
  skill_version_id TEXT NOT NULL,

  -- Classification
  labels_defect JSON,                   -- ["low_contrast", "text_overlap"]
  labels_preference JSON,               -- ["tone_adjustment", "layout_change"]

  -- User feedback
  user_notes TEXT,                      -- Optional user explanation
  revision_type TEXT NOT NULL,          -- "regenerate", "manual_edit", "approved"

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (skill_id) REFERENCES skills(skill_id),
  FOREIGN KEY (tenant_id) REFERENCES tenants(tenant_id)
);

CREATE INDEX idx_revision_events_skill ON revision_events(skill_id);
CREATE INDEX idx_revision_events_tenant ON revision_events(tenant_id);
```

**Privacy:** Tenant-specific revisions stored separately, NOT aggregated into global metrics without K-anonymity check.

---

#### 3.2.8 Guard Results Table
```sql
CREATE TABLE guard_results (
  result_id TEXT PRIMARY KEY,
  request_id TEXT NOT NULL,
  guard_id TEXT NOT NULL,               -- "contrast", "safe_margin", etc.
  pass BOOLEAN NOT NULL,
  metrics_json JSON,                    -- Guard-specific measurements
  auto_fixed BOOLEAN DEFAULT FALSE,
  action_taken TEXT,                    -- "Added text shadow", "Repositioned logo"
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_guard_results_request ON guard_results(request_id);
CREATE INDEX idx_guard_results_guard ON guard_results(guard_id, pass);
```

---

#### 3.2.9 Sharing Events Table (AUDIT)
```sql
CREATE TABLE sharing_events (
  event_id TEXT PRIMARY KEY,
  action TEXT NOT NULL,                 -- "promote_to_global", "fork_skill", "retire_skill"
  actor TEXT NOT NULL,                  -- "system_auto" or "admin:{id}"
  tenant_id TEXT,                       -- Source tenant (if applicable)
  skill_id TEXT NOT NULL,
  reason TEXT NOT NULL,                 -- "High performance across 5+ tenants"
  metadata JSON,                        -- Additional context
  ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (skill_id) REFERENCES skills(skill_id)
);

CREATE INDEX idx_sharing_events_skill ON sharing_events(skill_id, ts DESC);
CREATE INDEX idx_sharing_events_action ON sharing_events(action, ts DESC);
```

**Purpose:** Complete audit trail of all skill sharing/promotion activities for compliance and debugging.

---

## 4. Multi-Tenant Strategy

### 4.1 Core Principle

**Separation of Concerns:**
- **Shared (Safe):** Structural patterns, layout rules, guard logic, generic best practices
- **Private (Protected):** Brand voice, tone, lexicon, colors, logos, competitive messaging

**Result:** System learns "what works" (structure) while preserving "how we say it" (voice).

---

### 4.2 Privacy Architecture

#### What IS Shared (Global Skill Bank):
✅ **Structural Patterns:**
- "3-column layout with product left, text right"
- "Headline size 48pt, body 16pt, CTA button 18pt"
- "Safe margin 32px from all edges"
- "Logo in top-right, product image hero"

✅ **Guard Rules:**
- "Contrast ratio must be ≥ 4.5:1"
- "Text must not overlap faces"
- "Logo must be fully visible"

✅ **Performance Metrics (Aggregated):**
- "This layout has 87% approval rate across 12 tenants"
- "3-column performs better than 2-column for product posts"

#### What is NOT Shared (Private):
❌ **Brand Voice:**
- Specific tone ("We deliver excellence" vs "Fast. Reliable. Done.")
- Lexicon (competitor A says "industrial-grade", competitor B says "premium quality")
- Messaging ("family-owned for 40 years" vs "cutting-edge technology")

❌ **Brand Assets:**
- Logo files
- Color palettes (#1a5f7a vs #ff6b6b)
- Typography choices (Montserrat vs Roboto)

❌ **Competitive Data:**
- Which products are promoted
- Pricing strategies
- Campaign timing

❌ **Individual Performance:**
- Tenant X's approval rates
- Tenant Y's defect rates
- Success metrics per tenant

---

### 4.3 Privacy Guarantees

#### 4.3.1 Row-Level Security (RLS)
```sql
-- Enforce tenant isolation
CREATE POLICY tenant_isolation ON brand_profiles
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant'));

-- Prevent cross-tenant queries
SET app.current_tenant = 'tenant-001';
SELECT * FROM brand_profiles;  -- Returns ONLY tenant-001 data
```

#### 4.3.2 K-Anonymity (Suppress Small Groups)
```python
# Only aggregate metrics if ≥5 tenants contributed
if skill_metrics.unique_tenants < 5:
    suppress_metric()  # Do NOT share to global learning
    log_audit("suppressed", skill_id, reason="K-anonymity threshold not met")
```

#### 4.3.3 Per-Tenant KMS Encryption
```python
# Each tenant has unique encryption key
brand_profile.embeddings = encrypt(
    data=brand_voice_vectors,
    key=tenant.kms_key_id  # Unique per tenant
)

# Competitor CANNOT decrypt even if DB breached
competitor_key = "kms://tenant-002/key"
decrypt(brand_profile.embeddings, competitor_key)  # FAILS
```

#### 4.3.4 Audit Logging
```sql
-- All promotions logged
INSERT INTO sharing_events (action, skill_id, reason, metadata)
VALUES (
  'promote_to_global',
  'skill_linkedin_3col_v12',
  'High performance across 7 tenants',
  '{"tenants_count": 7, "avg_pass_rate": 0.89, "defect_rate": 0.04}'
);
```

#### 4.3.5 Temporal Delay (Anti-Correlation)
```python
# Wait 30-60 days before promoting skill to global
# Prevents competitor from correlating "new skill appeared" with "Tenant X ran campaign"
if skill.age_days >= 30 AND skill.performance_stable:
    promote_to_global(skill_id)
```

#### 4.3.6 Per-Region Data Residency
```python
# EU customers' brand data stays in EU
if tenant.region == "eu-west":
    store_in_region("eu-west")  # Cloudflare D1 regional replica
```

---

### 4.4 Competitive Scenario Test

**Scenario:** Competitor uses same system as Company XYZ

**What Competitor Learns:**
✅ "3-column layouts work well for product posts" (structural pattern)
✅ "Adding price tags increases engagement" (generic best practice)
✅ "Safe margins improve readability" (design principle)

**What Competitor CANNOT Learn:**
❌ Company XYZ's brand voice ("Built on trust, delivered with precision")
❌ Company XYZ's color palette (#1a5f7a teal)
❌ Company XYZ's product focus (Product SKU-Product SKU-A142)
❌ Company XYZ's campaign timing (spring promotions)
❌ Company XYZ's pricing strategy
❌ Company XYZ's messaging ("Family-owned for 40 years")

**Outcome:**
- Both companies use proven layouts (efficiency)
- Each maintains unique brand identity (differentiation)
- **No competitive leakage**

---

### 4.5 Compliance & Certifications

**GDPR (EU):**
- ✅ Data minimization (only structural patterns shared)
- ✅ Right to deletion (tenant can delete all private data)
- ✅ Data portability (export brand profiles)
- ✅ Consent (DPA signed before skill sharing)

**SOC 2 Type II:**
- ✅ Access controls (RLS)
- ✅ Encryption at rest (KMS)
- ✅ Audit logging (all sharing events)
- ✅ Incident response (breach notification)

**ISO 27001:**
- ✅ Risk assessment (multi-tenant isolation)
- ✅ Penetration testing (quarterly)
- ✅ Security awareness training

---

## 5. Multi-Candidate Generation

### 5.1 Strategy

**Why 2-3 Candidates:**
- Gives user meaningful choice
- Explores different skill approaches
- Balances quality vs diversity
- Keeps cost manageable (~$0.20 per request vs $0.08 single-gen)

**Candidate Selection:**

```python
def generate_candidates(intent_spec, top_skills, brand_voice):
    candidates = []

    # Candidate A: Best Match (Exploitation)
    # Use highest similarity skill with proven track record
    skill_a = top_skills[0]  # similarity >= 0.80
    candidates.append({
        "id": "candidate_a",
        "strategy": "best_match",
        "skill_id": skill_a.skill_id,
        "output": apply_skill(skill_a, brand_voice, intent_spec.assets),
        "metadata": {
            "similarity": skill_a.similarity,
            "historical_pass_rate": skill_a.pass_rate,
            "cost": 0.08
        }
    })

    # Candidate B: Alternative (Balanced)
    # Use second-best OR adapted version of best
    if len(top_skills) >= 2 AND top_skills[1].similarity >= 0.65:
        skill_b = top_skills[1]
        candidates.append({
            "id": "candidate_b",
            "strategy": "alternative",
            "skill_id": skill_b.skill_id,
            "output": apply_skill(skill_b, brand_voice, intent_spec.assets),
            "metadata": {
                "similarity": skill_b.similarity,
                "historical_pass_rate": skill_b.pass_rate,
                "cost": 0.08
            }
        })
    else:
        # Adapt best skill with higher temperature for diversity
        candidates.append({
            "id": "candidate_b",
            "strategy": "adapted",
            "skill_id": skill_a.skill_id + "_adapted",
            "output": apply_skill(skill_a, brand_voice, intent_spec.assets, temperature=0.8),
            "metadata": {
                "similarity": skill_a.similarity * 0.9,  # Slightly lower confidence
                "cost": 0.08
            }
        })

    # Candidate C: Exploratory (10% probability via ε-greedy)
    # Explore novel approaches to discover better skills
    if random.random() < 0.1:  # ε = 0.1 (10% exploration)
        candidates.append({
            "id": "candidate_c",
            "strategy": "exploratory",
            "skill_id": "generated_novel",
            "output": generate_novel(intent_spec, brand_voice, temperature=1.0),
            "metadata": {
                "similarity": 0.0,  # Unknown - exploratory
                "cost": 0.12,  # Slightly higher (fresh generation)
                "exploration": True
            }
        })
    else:
        # Use third-best skill
        if len(top_skills) >= 3:
            skill_c = top_skills[2]
            candidates.append({
                "id": "candidate_c",
                "strategy": "third_best",
                "skill_id": skill_c.skill_id,
                "output": apply_skill(skill_c, brand_voice, intent_spec.assets),
                "metadata": {
                    "similarity": skill_c.similarity,
                    "cost": 0.08
                }
            })

    return candidates
```

---

### 5.2 Reranking Formula

**Goal:** Order candidates by predicted user satisfaction

```python
def rerank_candidates(candidates, intent_spec, context):
    scores = []

    for candidate in candidates:
        # Component scores (0.0 to 1.0 normalized)
        similarity_score = candidate.metadata.get("similarity", 0.5)
        guard_score = calculate_guard_pass_rate(candidate.guard_results)
        performance_score = get_historical_performance(
            candidate.skill_id,
            context_hash(intent_spec)
        )
        aesthetic_score = evaluate_aesthetics(candidate.output)  # ML model or heuristic
        diversity_score = calculate_diversity(candidate, candidates)
        cost_normalized = 1.0 - (candidate.metadata["cost"] / 0.20)  # Max cost $0.20

        # Weighted sum
        total_score = (
            0.30 * similarity_score +      # How well skill matches intent
            0.25 * guard_score +            # Quality (% guards passed)
            0.20 * performance_score +      # Historical success
            0.15 * aesthetic_score +        # Visual appeal
            0.05 * diversity_score +        # Novelty vs other candidates
            0.05 * cost_normalized          # Cost efficiency
        )

        scores.append({
            "candidate_id": candidate.id,
            "score": total_score,
            "breakdown": {
                "similarity": similarity_score,
                "guards": guard_score,
                "performance": performance_score,
                "aesthetics": aesthetic_score,
                "diversity": diversity_score,
                "cost": cost_normalized
            }
        })

    # Sort descending by score
    ranked = sorted(scores, key=lambda x: x["score"], reverse=True)
    return ranked
```

---

### 5.3 Presentation to User

**Conversational Format:**

```
Assistant: I've created 3 variations for your LinkedIn post about Product SKU-Product SKU-A142:

**Option A** (Recommended)
[Image preview]
"Transform your construction projects with premium Product SKU-Product SKU-A142.
Next-day delivery across the UK. #Industry #Rebar"
Score: 94/100 (Best match, proven track record)

**Option B**
[Image preview]
"Product SKU-Product SKU-A142 - Industrial-grade strength, delivered fast.
Order today for next-day arrival. #BuildBetter #Steel"
Score: 88/100 (Alternative layout)

**Option C** (Experimental)
[Image preview]
"Precision-engineered Product SKU-Product SKU-A142 for demanding projects.
Built on trust, delivered with precision. #Engineering"
Score: 82/100 (New approach)

Which would you like to use? Type A, B, or C.
You can also ask me to regenerate or make adjustments.
```

**User Response:**
```
User: A looks great! Post it.
```

**System Action:**
```
1. Log selection (candidate_a selected)
2. Update skill metrics (selection_rate += 1)
3. Publish to LinkedIn via Publishing Service
4. Confirm to user: "Posted! View at: [link]"
```

---

### 5.4 Cost Analysis (Multi-Gen)

**Single Request Cost:**
```
Candidate A: $0.08 (Claude Sonnet 4.5, 500 tokens)
Candidate B: $0.08 (Claude Sonnet 4.5, 500 tokens)
Candidate C: $0.08 or $0.12 (exploratory)
Guards (3 runs): $0.03
Reranking: $0.01
Total: ~$0.28 per request (avg)
```

**Monthly Cost (1000 posts):**
```
1000 requests × $0.28 = $280/month
```

**vs Single-Candidate:**
```
1000 requests × $0.08 = $80/month
```

**Trade-off:**
- 3.5× higher cost
- ~15-20% higher user satisfaction (from research)
- Faster skill discovery (exploration)
- Better A/B testing data

**Optimization:** After 6 months, as skill bank matures:
- Reduce to 2 candidates (best + alternative)
- Cost drops to ~$0.18 per request ($180/month)
- Maintain quality with mature skill library

---

## 6. Guards Engine

### 6.1 Architecture

**Purpose:** Automated quality gates to catch common defects before user sees output

**Execution:** All guards run in parallel, results aggregated

```python
def run_guards(candidate, intent_spec, brand_profile):
    guard_checks = [
        check_text_overlap(candidate.output, intent_spec),
        check_safe_margins(candidate.output, intent_spec),
        check_contrast(candidate.output),
        check_logo_protection(candidate.output, brand_profile.logo_rules),
        check_aspect_ratio(candidate.output, intent_spec.aspect_ratio),
        check_brand_palette(candidate.output, brand_profile.palette),
        check_font_size(candidate.output, intent_spec.constraints)
    ]

    results = await asyncio.gather(*guard_checks)  # Parallel execution

    # Attempt auto-fixes for failures
    fixed_output = candidate.output
    auto_fix_applied = False

    for result in results:
        if not result.pass and result.auto_fixer:
            fixed_output = result.auto_fixer(fixed_output)
            auto_fix_applied = True

    return {
        "pass": all(r.pass for r in results),
        "results": results,
        "auto_fixed": auto_fix_applied,
        "fixed_output": fixed_output if auto_fix_applied else None
    }
```

---

### 6.2 Guard Specifications

#### Guard 1: Text Overlap Detection

**Purpose:** Prevent text from covering important image areas (faces, products, logos)

**Algorithm:**
```python
def check_text_overlap(output, intent_spec):
    # 1. Detect important regions using object detection
    important_regions = detect_objects(output.image, ["face", "product", "logo"])

    # 2. Extract text bounding boxes
    text_boxes = extract_text_boxes(output.layout)

    # 3. Calculate overlap
    overlap_pixels = 0
    for text_box in text_boxes:
        for region in important_regions:
            overlap_pixels += calculate_intersection(text_box, region)

    total_important_pixels = sum(r.area for r in important_regions)
    overlap_ratio = overlap_pixels / total_important_pixels if total_important_pixels > 0 else 0

    # 4. Threshold check
    threshold = intent_spec.constraints.get("max_text_overlap", 0.05)  # Default 5%
    pass_check = overlap_ratio <= threshold

    return GuardResult(
        guard_id="text_overlap",
        pass=pass_check,
        metrics={"overlap_ratio": overlap_ratio, "threshold": threshold},
        auto_fixer=reposition_text_to_safe_zones if not pass_check else None
    )
```

**Auto-Fixer:**
```python
def reposition_text_to_safe_zones(output):
    # Move text to corners or safe margins
    safe_zones = identify_empty_regions(output.image)
    for text_block in output.layout.text_blocks:
        if overlaps_important_region(text_block):
            new_position = find_nearest_safe_zone(text_block, safe_zones)
            text_block.position = new_position
    return output
```

---

#### Guard 2: Safe Margin Check

**Purpose:** Ensure minimum spacing from edges (prevents text cutoff on social platforms)

**Algorithm:**
```python
def check_safe_margins(output, intent_spec):
    min_margin_px = intent_spec.constraints.get("safe_margin_px", 32)

    violations = []
    for element in output.layout.all_elements:
        if element.left < min_margin_px:
            violations.append(f"{element.id}: left={element.left}px")
        if element.top < min_margin_px:
            violations.append(f"{element.id}: top={element.top}px")
        if (output.width - element.right) < min_margin_px:
            violations.append(f"{element.id}: right={output.width - element.right}px")
        if (output.height - element.bottom) < min_margin_px:
            violations.append(f"{element.id}: bottom={output.height - element.bottom}px")

    return GuardResult(
        guard_id="safe_margin",
        pass=len(violations) == 0,
        metrics={"violations": violations, "min_margin": min_margin_px},
        auto_fixer=add_padding if violations else None
    )
```

**Auto-Fixer:**
```python
def add_padding(output):
    # Scale down content to fit within safe margins
    scale_factor = calculate_safe_scale(output, min_margin=32)
    output.layout.scale(scale_factor)
    output.layout.center()
    return output
```

---

#### Guard 3: Contrast Ratio Check

**Purpose:** Ensure text is readable (WCAG AA compliance)

**Algorithm:**
```python
def check_contrast(output):
    violations = []

    for text_block in output.layout.text_blocks:
        # Sample background color behind text
        bg_color = sample_background(output.image, text_block.bbox)
        fg_color = text_block.color

        # Calculate contrast ratio
        contrast_ratio = calculate_contrast_ratio(fg_color, bg_color)

        # WCAG AA requires 4.5:1 for normal text, 3:1 for large text
        threshold = 3.0 if text_block.font_size >= 24 else 4.5

        if contrast_ratio < threshold:
            violations.append({
                "text_id": text_block.id,
                "measured": contrast_ratio,
                "required": threshold
            })

    return GuardResult(
        guard_id="contrast",
        pass=len(violations) == 0,
        metrics={"violations": violations},
        auto_fixer=improve_contrast if violations else None
    )

def calculate_contrast_ratio(fg_rgb, bg_rgb):
    # WCAG formula
    l1 = relative_luminance(fg_rgb)
    l2 = relative_luminance(bg_rgb)
    lighter = max(l1, l2)
    darker = min(l1, l2)
    return (lighter + 0.05) / (darker + 0.05)
```

**Auto-Fixer:**
```python
def improve_contrast(output):
    for text_block in output.layout.text_blocks:
        bg_color = sample_background(output.image, text_block.bbox)

        # Try 1: Darken/lighten text color
        adjusted_color = adjust_for_contrast(text_block.color, bg_color)
        if calculate_contrast_ratio(adjusted_color, bg_color) >= 4.5:
            text_block.color = adjusted_color
            continue

        # Try 2: Add text shadow or background
        text_block.add_effect("drop_shadow", opacity=0.8, blur=4)

    return output
```

---

#### Guard 4: Logo Protection

**Purpose:** Prevent any elements from covering brand logo

**Algorithm:**
```python
def check_logo_protection(output, logo_rules):
    logo_bbox = find_logo(output.image)
    if not logo_bbox:
        return GuardResult(
            guard_id="logo_protection",
            pass=True,
            metrics={"logo_found": False}
        )

    # Check if any elements overlap logo
    overlapping_elements = []
    for element in output.layout.all_elements:
        if element.id == "logo":
            continue  # Skip logo itself

        overlap_area = calculate_intersection(element.bbox, logo_bbox)
        if overlap_area > 0:
            overlapping_elements.append({
                "element_id": element.id,
                "overlap_px": overlap_area
            })

    return GuardResult(
        guard_id="logo_protection",
        pass=len(overlapping_elements) == 0,
        metrics={"overlapping_elements": overlapping_elements},
        auto_fixer=move_overlapping_elements if overlapping_elements else None
    )
```

---

#### Guard 5: Aspect Ratio Fit

**Purpose:** Validate output matches platform requirements

**Algorithm:**
```python
def check_aspect_ratio(output, target_aspect_ratio):
    # Parse target (e.g., "1200x628" or "16:9")
    if "x" in target_aspect_ratio:
        target_w, target_h = map(int, target_aspect_ratio.split("x"))
        target_ratio = target_w / target_h
    else:
        w, h = map(int, target_aspect_ratio.split(":"))
        target_ratio = w / h

    actual_ratio = output.width / output.height

    # Allow 2% tolerance
    tolerance = 0.02
    ratio_diff = abs(actual_ratio - target_ratio)

    return GuardResult(
        guard_id="aspect_ratio",
        pass=ratio_diff <= tolerance,
        metrics={
            "target": target_ratio,
            "actual": actual_ratio,
            "diff": ratio_diff
        },
        auto_fixer=crop_to_aspect if ratio_diff > tolerance else None
    )
```

**Auto-Fixer:**
```python
def crop_to_aspect(output, target_aspect_ratio):
    # Crop from center to match target aspect ratio
    target_w, target_h = parse_aspect_ratio(target_aspect_ratio)
    target_ratio = target_w / target_h

    if output.width / output.height > target_ratio:
        # Too wide - crop width
        new_width = int(output.height * target_ratio)
        crop_x = (output.width - new_width) // 2
        output.crop(crop_x, 0, crop_x + new_width, output.height)
    else:
        # Too tall - crop height
        new_height = int(output.width / target_ratio)
        crop_y = (output.height - new_height) // 2
        output.crop(0, crop_y, output.width, crop_y + new_height)

    return output
```

---

#### Guard 6: Brand Palette Check

**Purpose:** Ensure only approved brand colors are used

**Algorithm:**
```python
def check_brand_palette(output, approved_palette):
    if not output.layout.constraints.get("brand_palette_only"):
        return GuardResult(guard_id="brand_palette", pass=True)

    used_colors = extract_colors(output.image, output.layout)
    violations = []

    for color in used_colors:
        if not color_in_palette(color, approved_palette, tolerance=10):
            violations.append({
                "color": color,
                "nearest_approved": find_nearest_color(color, approved_palette)
            })

    return GuardResult(
        guard_id="brand_palette",
        pass=len(violations) == 0,
        metrics={"violations": violations},
        auto_fixer=snap_to_palette if violations else None
    )
```

**Auto-Fixer:**
```python
def snap_to_palette(output, approved_palette):
    # Replace non-brand colors with nearest approved color
    color_map = {}
    for element in output.layout.all_elements:
        if element.color not in approved_palette:
            if element.color not in color_map:
                color_map[element.color] = find_nearest_color(element.color, approved_palette)
            element.color = color_map[element.color]

    return output
```

---

#### Guard 7: Font Size Minimum

**Purpose:** Ensure text is readable on all devices

**Algorithm:**
```python
def check_font_size(output, constraints):
    min_font_pt = constraints.get("min_font_pt", 22)
    violations = []

    for text_block in output.layout.text_blocks:
        if text_block.font_size < min_font_pt:
            violations.append({
                "text_id": text_block.id,
                "current_size": text_block.font_size,
                "min_required": min_font_pt
            })

    return GuardResult(
        guard_id="font_size",
        pass=len(violations) == 0,
        metrics={"violations": violations},
        auto_fixer=scale_fonts if violations else None
    )
```

**Auto-Fixer:**
```python
def scale_fonts(output, min_font_pt):
    for text_block in output.layout.text_blocks:
        if text_block.font_size < min_font_pt:
            text_block.font_size = min_font_pt
            # May need to adjust layout to fit larger text
            reflow_layout(output.layout)

    return output
```

---

### 6.3 Guard Execution Flow

```python
async def execute_guards_for_candidate(candidate, intent_spec, brand_profile):
    # Run all guards in parallel
    guard_results = await asyncio.gather(
        check_text_overlap(candidate, intent_spec),
        check_safe_margins(candidate, intent_spec),
        check_contrast(candidate),
        check_logo_protection(candidate, brand_profile.logo_rules),
        check_aspect_ratio(candidate, intent_spec.aspect_ratio),
        check_brand_palette(candidate, brand_profile.palette),
        check_font_size(candidate, intent_spec.constraints)
    )

    # Aggregate results
    all_passed = all(r.pass for r in guard_results)
    auto_fixable = any(r.auto_fixer for r in guard_results if not r.pass)

    # Apply auto-fixes if available
    fixed_output = candidate.output
    if not all_passed and auto_fixable:
        for result in guard_results:
            if not result.pass and result.auto_fixer:
                fixed_output = result.auto_fixer(fixed_output)

        # Re-run guards on fixed output
        recheck_results = await execute_guards_for_candidate(
            Candidate(output=fixed_output, ...),
            intent_spec,
            brand_profile
        )

        return {
            "original_pass": False,
            "auto_fixed": True,
            "fixed_pass": recheck_results["all_passed"],
            "output": fixed_output,
            "guard_results": guard_results,
            "recheck_results": recheck_results["guard_results"]
        }

    return {
        "original_pass": all_passed,
        "auto_fixed": False,
        "output": candidate.output,
        "guard_results": guard_results
    }
```

---

## 7. Defect vs Preference Classification

### 7.1 Core Distinction

**DEFECT (Skill Failure):**
- System failed to meet objective quality standards
- Impacts skill performance metrics
- Requires skill improvement or retirement
- Examples: Failed guard, factual error, broken asset

**PREFERENCE (User Style Choice):**
- User wants different creative direction
- Does NOT indicate skill failure
- Should NOT impact skill metrics
- Examples: Tone adjustment, layout variation, content addition

---

### 7.2 Classification Algorithm

```python
async def classify_revision(revision_request):
    """
    Classify user revision as DEFECT or PREFERENCE

    Returns:
        {
            "classification": "defect" | "preference",
            "confidence": 0.0-1.0,
            "labels_defect": [...],
            "labels_preference": [...],
            "reasoning": "..."
        }
    """

    # Fast path: Rule-based heuristics (80% of cases)
    heuristic_result = apply_heuristics(revision_request)
    if heuristic_result.confidence >= 0.9:
        return heuristic_result

    # Slow path: LLM classification (20% of cases)
    llm_result = await llm_classify(revision_request)
    return llm_result
```

---

### 7.3 Rule-Based Heuristics

```python
def apply_heuristics(revision_request):
    defect_labels = []
    preference_labels = []

    # Check guard failures (DEFECT)
    if revision_request.guard_results:
        failed_guards = [g for g in revision_request.guard_results if not g.pass]
        if failed_guards:
            defect_labels.extend([f"guard_fail_{g.guard_id}" for g in failed_guards])

    # Check factual errors (DEFECT)
    factual_error_keywords = ["wrong product", "incorrect price", "wrong image", "broken link"]
    if any(kw in revision_request.user_notes.lower() for kw in factual_error_keywords):
        defect_labels.append("factual_error")

    # Check technical issues (DEFECT)
    technical_error_keywords = ["broken", "missing", "doesn't load", "error", "corrupt"]
    if any(kw in revision_request.user_notes.lower() for kw in technical_error_keywords):
        defect_labels.append("technical_error")

    # Check brand guideline violations (DEFECT)
    brand_error_keywords = ["wrong color", "wrong font", "logo incorrect", "off-brand"]
    if any(kw in revision_request.user_notes.lower() for kw in brand_error_keywords):
        defect_labels.append("brand_violation")

    # Check tone/style requests (PREFERENCE)
    tone_keywords = ["more casual", "more professional", "friendlier", "more technical"]
    if any(kw in revision_request.user_notes.lower() for kw in tone_keywords):
        preference_labels.append("tone_adjustment")

    # Check layout changes (PREFERENCE)
    layout_keywords = ["different layout", "swap", "move", "rearrange"]
    if any(kw in revision_request.user_notes.lower() for kw in layout_keywords):
        preference_labels.append("layout_change")

    # Check content additions (PREFERENCE)
    content_keywords = ["add", "include", "mention", "also say"]
    if any(kw in revision_request.user_notes.lower() for kw in content_keywords):
        preference_labels.append("content_addition")

    # Determine classification
    defect_score = len(defect_labels)
    preference_score = len(preference_labels)

    if defect_score > preference_score:
        classification = "defect"
        confidence = min(0.95, 0.7 + (defect_score * 0.1))
    elif preference_score > defect_score:
        classification = "preference"
        confidence = min(0.95, 0.7 + (preference_score * 0.1))
    else:
        classification = "preference"  # Default to preference if unclear
        confidence = 0.5

    return {
        "classification": classification,
        "confidence": confidence,
        "labels_defect": defect_labels,
        "labels_preference": preference_labels,
        "reasoning": f"Heuristic match: {defect_score} defect signals, {preference_score} preference signals"
    }
```

---

### 7.4 LLM Classification (Low Confidence Cases)

```python
async def llm_classify(revision_request):
    """
    Use Claude Sonnet 4.5 for nuanced classification
    """

    prompt = f"""
You are a classifier for content generation revisions.

DEFECT = Skill failure (failed guard, factual error, technical issue, brand violation)
PREFERENCE = User style choice (tone, layout, content additions)

User requested revision:
"{revision_request.user_notes}"

Original output metadata:
- Guard results: {json.dumps(revision_request.guard_results)}
- Skill used: {revision_request.skill_id}
- Platform: {revision_request.intent_spec.platform}

Classify as DEFECT or PREFERENCE with labels and reasoning.

Output JSON:
{{
  "classification": "defect" | "preference",
  "confidence": 0.0-1.0,
  "labels_defect": [...],
  "labels_preference": [...],
  "reasoning": "..."
}}
"""

    response = await call_claude(prompt, model="claude-sonnet-4.5")
    result = json.loads(response.content)

    return result
```

---

### 7.5 Impact on Skill Metrics

```python
def update_skill_metrics(revision_event):
    classification = revision_event.classification

    if classification == "defect":
        # Impacts skill performance
        skill_metrics[skill_id].defect_count += 1
        skill_metrics[skill_id].defect_rate = defect_count / total_uses

        # May trigger retirement if defect_rate > 0.50
        if skill_metrics[skill_id].defect_rate > 0.50:
            retire_skill(skill_id, reason="High defect rate")

    elif classification == "preference":
        # Does NOT impact skill performance
        # Log for learning but don't penalize skill
        skill_metrics[skill_id].preference_count += 1

        # May create variant skill if preference is common
        if preference_count_for_label("tone_casual") > 10:
            create_variant_skill(
                parent=skill_id,
                modification="casual_tone",
                reason="Frequent user preference"
            )
```

---

### 7.6 Examples

#### Example 1: DEFECT - Failed Contrast

**User Revision:**
> "The text is hard to read - can you make it darker?"

**Classification:**
```json
{
  "classification": "defect",
  "confidence": 0.95,
  "labels_defect": ["low_contrast", "readability_issue"],
  "labels_preference": [],
  "reasoning": "User reported readability issue. Contrast guard failed (3.2:1 vs 4.5:1 required). This is a skill execution failure."
}
```

**Impact:** Skill defect_rate increases, may be retired if pattern continues.

---

#### Example 2: PREFERENCE - Tone Adjustment

**User Revision:**
> "This is good but can you make it sound more friendly and approachable?"

**Classification:**
```json
{
  "classification": "preference",
  "confidence": 0.92,
  "labels_defect": [],
  "labels_preference": ["tone_adjustment", "casual_tone"],
  "reasoning": "User requested tone change to 'more friendly'. All guards passed. Original output met quality standards. This is a stylistic preference."
}
```

**Impact:** Skill metrics unchanged. Preference logged for potential variant creation.

---

#### Example 3: DEFECT - Wrong Product Image

**User Revision:**
> "That's not Product SKU-Product SKU-A142 - wrong product image"

**Classification:**
```json
{
  "classification": "defect",
  "confidence": 0.98,
  "labels_defect": ["wrong_product", "factual_error"],
  "labels_preference": [],
  "reasoning": "User reported factual error (wrong product image). This is a skill execution failure in asset retrieval."
}
```

**Impact:** Skill defect_rate increases significantly. Asset retrieval logic reviewed.

---

#### Example 4: PREFERENCE - Layout Change

**User Revision:**
> "Can we swap the headline and image positions?"

**Classification:**
```json
{
  "classification": "preference",
  "confidence": 0.90,
  "labels_defect": [],
  "labels_preference": ["layout_change", "element_reposition"],
  "reasoning": "User requested layout modification. Original layout met all guards and constraints. This is a creative direction choice."
}
```

**Impact:** Skill metrics unchanged. Layout preference tracked for future personalization.

---

## 8. APIs

### 8.1 API Contracts (11 Endpoints)

All APIs use **JSON-RPC 2.0** over HTTPS for consistency with MCP protocol.

**Base URL:** `https://api.dify-agent.com/v1`

**Authentication:** Bearer token in `Authorization` header

**Rate Limits:** 100 requests/minute per tenant (configurable by tier)

---

#### 8.1.1 Intent Parser API

**Endpoint:** `POST /api/v1/intent/parse`

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "parse_intent",
  "params": {
    "tenant_id": "tenant-001",
    "user_message": "Create a LinkedIn post about our Product SKU-Product SKU-A142 with pricing",
    "conversation_history": [
      {"role": "user", "content": "I need help with social media"},
      {"role": "assistant", "content": "I can help create social posts..."}
    ],
    "session_id": "sess_abc123"
  },
  "id": "req_001"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "intent_spec": {
      "task": "social_image_post",
      "platform": "linkedin",
      "modality": "image+text",
      "aspect_ratio": "1200x628",
      "must_have": ["product_image", "price_tag", "company_logo"],
      "copy_blocks": [
        {"name": "headline", "text": "Premium Product SKU-Product SKU-A142", "max_chars": 60}
      ],
      "context": {
        "product_id": "prod_a142_mesh"
      },
      "confidence": 0.92
    },
    "clarifications_needed": []
  },
  "id": "req_001"
}
```

---

#### 8.1.2 Skill Retrieval API

**Endpoint:** `POST /api/v1/skills/retrieve`

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "retrieve_skills",
  "params": {
    "tenant_id": "tenant-001",
    "intent_spec": {...},
    "top_k": 5,
    "min_similarity": 0.65
  },
  "id": "req_002"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "skills": [
      {
        "skill_id": "skill_linkedin_3col_v12",
        "name": "3-Column Product Layout (LinkedIn)",
        "similarity": 0.89,
        "performance": {
          "pass_rate": 0.87,
          "defect_rate": 0.04,
          "approval_rate": 0.82
        },
        "skill_version_id": "skill_linkedin_3col_v12"
      }
    ],
    "total_results": 47,
    "search_time_ms": 23
  },
  "id": "req_002"
}
```

---

#### 8.1.3 Multi-Candidate Generation API

**Endpoint:** `POST /api/v1/generate/multi-candidate`

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "generate_multi_candidate",
  "params": {
    "tenant_id": "tenant-001",
    "intent_spec": {...},
    "top_skills": [...],
    "num_candidates": 3,
    "include_exploratory": true
  },
  "id": "req_003"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "candidates": [
      {
        "candidate_id": "candidate_a",
        "strategy": "best_match",
        "skill_id": "skill_linkedin_3col_v12",
        "output": {
          "text": "Transform your construction projects with premium Product SKU-Product SKU-A142...",
          "image_url": "https://cdn.dify-agent.com/gen/abc123.jpg",
          "hashtags": ["#Industry", "#Rebar", "#CompanyXYZ"]
        },
        "metadata": {
          "similarity": 0.89,
          "cost": 0.08,
          "generation_time_ms": 1240
        }
      }
    ],
    "total_cost": 0.28,
    "generation_time_ms": 3420
  },
  "id": "req_003"
}
```

---

#### 8.1.4 Guards Engine API

**Endpoint:** `POST /api/v1/guards/check`

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "check_guards",
  "params": {
    "tenant_id": "tenant-001",
    "candidate": {...},
    "intent_spec": {...},
    "brand_profile": {...},
    "auto_fix": true
  },
  "id": "req_004"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "pass": false,
    "auto_fixed": true,
    "fixed_output": {
      "image_url": "https://cdn.dify-agent.com/gen/abc123_fixed.jpg"
    },
    "guard_results": [
      {
        "guard_id": "contrast",
        "pass": false,
        "metrics": {"measured_ratio": 3.2, "required": 4.5},
        "action_taken": "Added text shadow"
      }
    ]
  },
  "id": "req_004"
}
```

---

#### 8.1.5 Reranker API

**Endpoint:** `POST /api/v1/rerank`

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "rerank_candidates",
  "params": {
    "tenant_id": "tenant-001",
    "candidates": [...],
    "intent_spec": {...},
    "context": {...}
  },
  "id": "req_005"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "ranked_candidates": [
      {
        "candidate_id": "candidate_a",
        "rank": 1,
        "score": 0.94,
        "breakdown": {
          "similarity": 0.89,
          "guards": 1.0,
          "performance": 0.87,
          "aesthetics": 0.92,
          "diversity": 0.5,
          "cost": 0.95
        }
      }
    ]
  },
  "id": "req_005"
}
```

---

#### 8.1.6 Classify Feedback API

**Endpoint:** `POST /api/v1/classify-feedback`

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "classify_feedback",
  "params": {
    "tenant_id": "tenant-001",
    "request_id": "req_003",
    "candidate_id": "candidate_a",
    "user_notes": "The text is hard to read - can you make it darker?",
    "revision_type": "regenerate"
  },
  "id": "req_006"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "classification": "defect",
    "confidence": 0.95,
    "labels_defect": ["low_contrast", "readability_issue"],
    "labels_preference": [],
    "reasoning": "User reported readability issue. Contrast guard failed..."
  },
  "id": "req_006"
}
```

---

#### 8.1.7 Publish API

**Endpoint:** `POST /api/v1/publish`

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "publish_content",
  "params": {
    "tenant_id": "tenant-001",
    "platform": "linkedin",
    "content": {
      "text": "Transform your construction projects...",
      "media_urls": ["https://cdn.dify-agent.com/gen/abc123.jpg"],
      "hashtags": ["#Industry", "#Rebar"]
    },
    "schedule_time": null
  },
  "id": "req_007"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "post_id": "linkedin_post_xyz789",
    "platform_url": "https://linkedin.com/posts/companyxyz_xyz789",
    "published_at": "2025-11-10T14:30:00Z",
    "status": "published"
  },
  "id": "req_007"
}
```

---

#### 8.1.8 Analytics Log API

**Endpoint:** `POST /api/v1/analytics/log`

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "log_analytics",
  "params": {
    "tenant_id": "tenant-001",
    "event_type": "candidate_selected",
    "metadata": {
      "request_id": "req_003",
      "candidate_id": "candidate_a",
      "skill_id": "skill_linkedin_3col_v12",
      "guard_pass": true,
      "cost": 0.08,
      "latency_ms": 1240
    }
  },
  "id": "req_008"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "logged": true,
    "event_id": "evt_abc123"
  },
  "id": "req_008"
}
```

---

#### 8.1.9 Brand Voice Get API

**Endpoint:** `GET /api/v1/brand-voice/{tenant_id}`

**Request:**
```
GET /api/v1/brand-voice/tenant-001
Authorization: Bearer {token}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "tenant_id": "tenant-001",
    "palette": ["#1a5f7a", "#57cc99", "#f0f3bd"],
    "typography": {
      "headline": "Montserrat Bold 48pt",
      "body": "Open Sans Regular 16pt"
    },
    "logo_rules": {
      "min_size_px": 120,
      "safe_zone_px": 40
    },
    "tone": "professional_industrial",
    "lexicon": {
      "products": "industrial-grade rebar",
      "values": "reliability, expertise"
    }
  },
  "id": "req_009"
}
```

---

#### 8.1.10 Learning Analyze API (Internal)

**Endpoint:** `POST /api/v1/learning/analyze`

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "analyze_skill_performance",
  "params": {
    "date_range": {
      "start": "2025-11-01",
      "end": "2025-11-10"
    }
  },
  "id": "req_010"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "skills_analyzed": 143,
    "promotions": [
      {
        "skill_id": "skill_linkedin_3col_v12",
        "action": "promote_to_champion",
        "reason": "Pass rate 89%, defect rate 4%"
      }
    ],
    "retirements": [
      {
        "skill_id": "skill_instagram_old_v3",
        "action": "retire",
        "reason": "Defect rate 52%, no improvement in 50 uses"
      }
    ]
  },
  "id": "req_010"
}
```

---

#### 8.1.11 Context Get API

**Endpoint:** `GET /api/v1/context/{session_id}`

**Request:**
```
GET /api/v1/context/sess_abc123
Authorization: Bearer {token}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "session_id": "sess_abc123",
    "tenant_id": "tenant-001",
    "conversation_history": [...],
    "intent_spec": {...},
    "selected_candidate": "candidate_a",
    "ttl_seconds": 86400
  },
  "id": "req_011"
}
```

---

## 9. Cost Analysis

### 9.1 Operational Costs (Monthly)

**Assumptions:**
- 1000 content requests/month
- 3 candidates per request (multi-gen)
- 70% approval rate (30% require 1 revision)

**API Costs:**

| Service | Cost per Call | Calls/Month | Monthly Cost |
|---------|---------------|-------------|--------------|
| GPT-5 (Intent Parser) | $0.01 | 1000 | $10 |
| Claude Sonnet 4.5 (Content Gen) | $0.08 | 3000 (3×1000) | $240 |
| Gemini 2.5 Flash (Images) | $0.039 | 1000 | $39 |
| voyage-3-lite (Embeddings) | $0.001 | 1000 | $1 |
| Guards execution | $0.01 | 3000 | $30 |
| Reranking | $0.005 | 1000 | $5 |
| Defect Classifier | $0.02 | 300 (revisions) | $6 |
| Publishing API calls | $0.001 | 1000 | $1 |

**Total API Costs:** ~$332/month for 1000 posts

**Infrastructure Costs (Cloudflare):**

| Resource | Usage | Cost |
|----------|-------|------|
| Workers (compute) | 3M requests/month | $25 |
| D1 (database) | 10M reads, 1M writes | $5 |
| Vectorize (similarity search) | 3K searches/day | $15 |
| R2 (image storage) | 50 GB, 100K requests | $5 |
| KV (cache) | 10M reads, 100K writes | $5 |

**Total Infrastructure:** ~$55/month

**Grand Total:** $387/month (~$4,644/year)

**Per-Post Cost:** $0.387/post

---

### 9.2 Cost Optimization (6-Month Mature System)

After skill bank matures:

**Reduced Multi-Gen:**
- Reduce to 2 candidates (best + alternative)
- API costs drop to ~$220/month
- Infrastructure stable at ~$55/month
- **Total:** $275/month ($0.275/post)

**Skill Reuse:**
- 80% of requests reuse existing skills (vs 50% initial)
- Less exploratory generation needed
- Embedding lookups faster and cheaper

**Guard Efficiency:**
- Auto-fixers handle 90% of failures
- Fewer regeneration cycles
- Lower revision rate (30% → 15%)

---

### 9.3 Development Costs (6-Month Build)

**Team Structure (5 Teams):**

| Team | Size | Role | Monthly Cost |
|------|------|------|--------------|
| Backend Engineers | 3 | Core services, APIs, database | $45K |
| ML/AI Engineers | 2 | LLM integration, embeddings, guards | $40K |
| Frontend Engineers | 2 | Dify integration, UI/UX | $30K |
| DevOps Engineer | 1 | Infrastructure, deployment, monitoring | $18K |
| Product Manager | 1 | Coordination, testing, documentation | $15K |

**Total Monthly:** $148K

**6-Month Timeline:** $148K × 6 = $888K

**Additional Costs:**
- Cloud infrastructure (dev/staging): $5K
- Third-party APIs (testing): $2K
- Tools & licenses: $3K

**Total Development:** ~$900K

---

### 9.4 ROI Analysis

**Value Delivered:**
- Automated content generation (saves 10 hours/week per customer)
- Multi-platform publishing (LinkedIn, Twitter, Instagram, Facebook)
- Brand-safe quality gates
- Multi-tenant scalable architecture

**Break-Even (SaaS Model):**
- If sold at $99/month per tenant
- Need 45 customers to cover operational costs ($4,644/year)
- Need 250 customers over 3 years to cover development ($900K)

**Conservative:** 100 customers × $99/month = $118,800/year revenue
**Optimistic:** 500 customers × $99/month = $594,000/year revenue

---

## 10. Implementation

### 10.1 Team Structure (5 Teams)

#### Team 1: Backend Services (3 Engineers)
**Responsibilities:**
- Intent Parser Service
- Skill Retrieval Service
- Context Service
- Analytics Service
- API Gateway & authentication

**Tech Stack:**
- Cloudflare Workers (TypeScript)
- Cloudflare D1 (SQL database)
- Cloudflare KV (cache)
- REST/JSON-RPC APIs

**Deliverables:**
- 4 services deployed
- API documentation
- Unit tests (80%+ coverage)

---

#### Team 2: ML/AI Services (2 Engineers)
**Responsibilities:**
- Multi-Candidate Generator Service
- Guards Engine Service
- Defect Classifier Service
- Brand Voice Service
- Learning Engine Service

**Tech Stack:**
- Python (ML logic)
- Claude Sonnet 4.5 API
- GPT-5 API
- Gemini 2.5 API
- Cloudflare Vectorize
- OpenCV, Pillow (image processing)

**Deliverables:**
- 5 services deployed
- Guard algorithms implemented
- Classification models trained
- Performance benchmarks

---

#### Team 3: Frontend/Integration (2 Engineers)
**Responsibilities:**
- Dify platform integration
- Conversational UI flows
- Publishing Service (social platforms)
- Multi-platform OAuth

**Tech Stack:**
- Dify (orchestration)
- Platform SDKs (LinkedIn, Twitter, Instagram, Facebook)
- OAuth 2.0 with PKCE
- React (if custom UI needed)

**Deliverables:**
- Dify chatflow configured
- 4 social platforms integrated
- OAuth flows tested
- Mobile-responsive interface

---

#### Team 4: DevOps (1 Engineer)
**Responsibilities:**
- Infrastructure as Code (Terraform)
- CI/CD pipelines
- Monitoring & observability
- Security & compliance

**Tech Stack:**
- Terraform (IaC)
- GitHub Actions (CI/CD)
- Cloudflare Analytics Engine
- Sentry (error tracking)
- Grafana (dashboards)

**Deliverables:**
- Automated deployments
- Monitoring dashboards
- Incident response runbooks
- Security audit reports

---

#### Team 5: Product Management (1 PM)
**Responsibilities:**
- Requirement clarification
- User acceptance testing
- Documentation
- Stakeholder communication

**Deliverables:**
- Product requirements docs
- User testing reports
- API documentation
- End-user guides

---

### 10.2 Implementation Roadmap (6 Months)

#### Month 1: Foundation
**Week 1-2:**
- ✅ Infrastructure setup (Cloudflare account, D1 database, Vectorize index)
- ✅ Database schema creation (9 tables)
- ✅ API Gateway skeleton
- ✅ Dify platform setup (Cloud or self-hosted)

**Week 3-4:**
- ✅ Intent Parser Service (basic NLP)
- ✅ Context Service (session management)
- ✅ Brand Voice Service (basic retrieval)
- ✅ First Dify integration test

**Deliverable:** Working Intent Parser that converts user text to IntentSpec JSON

---

#### Month 2: Core Generation
**Week 5-6:**
- ✅ Skill Retrieval Service (Vectorize integration)
- ✅ Multi-Candidate Generator (single model first)
- ✅ Basic guards (contrast, margins)

**Week 7-8:**
- ✅ Expand to 3 models (Claude, GPT-5, Gemini)
- ✅ Reranker Service (basic scoring)
- ✅ End-to-end test: Intent → Skills → Generate → Guards → Rerank

**Deliverable:** Working content generation pipeline (1 candidate)

---

#### Month 3: Multi-Candidate & Guards
**Week 9-10:**
- ✅ Multi-candidate generation (2-3 outputs)
- ✅ Implement all 7 guards
- ✅ Auto-fixers for guards

**Week 11-12:**
- ✅ Defect Classifier Service (heuristics + LLM)
- ✅ Revision workflow
- ✅ User testing (internal team)

**Deliverable:** Multi-candidate generation with quality guards

---

#### Month 4: Learning & Publishing
**Week 13-14:**
- ✅ Analytics Service (metrics logging)
- ✅ Learning Engine (skill lifecycle)
- ✅ Auto-promotion/retirement logic

**Week 15-16:**
- ✅ Publishing Service (LinkedIn first)
- ✅ OAuth 2.0 flows
- ✅ Expand to Twitter, Instagram, Facebook

**Deliverable:** Full learning loop + 4-platform publishing

---

#### Month 5: Multi-Tenant & Security
**Week 17-18:**
- ✅ Row-level security (RLS) implementation
- ✅ Per-tenant KMS encryption
- ✅ K-anonymity checks
- ✅ Audit logging

**Week 19-20:**
- ✅ Multi-tenant testing (3 test customers)
- ✅ Data residency (regional replicas)
- ✅ Penetration testing
- ✅ GDPR compliance audit

**Deliverable:** Production-ready multi-tenant system

---

#### Month 6: Polish & Launch
**Week 21-22:**
- ✅ Performance optimization
- ✅ Cost optimization (caching, batching)
- ✅ Documentation (API docs, user guides)
- ✅ User acceptance testing

**Week 23-24:**
- ✅ Beta launch (10 customers)
- ✅ Monitoring & incident response
- ✅ Feedback iteration
- ✅ Public launch

**Deliverable:** Production system with 10 paying customers

---

### 10.3 Acceptance Criteria

**Stage D is COMPLETE when:**

1. **Functional Requirements:**
   - ✅ User can request content via conversational text
   - ✅ System generates 2-3 diverse candidates
   - ✅ All 7 guards implemented with auto-fixers
   - ✅ User can select, approve, or revise candidates
   - ✅ Defect vs preference classification working
   - ✅ Content publishes to LinkedIn, Twitter, Instagram, Facebook
   - ✅ Multi-tenant isolation enforced (RLS, KMS)

2. **Performance Requirements:**
   - ✅ Intent parsing: < 500ms p95
   - ✅ Skill retrieval: < 200ms p95
   - ✅ Content generation (3 candidates): < 5s p95
   - ✅ Guards execution: < 1s p95
   - ✅ End-to-end workflow: < 8s p95

3. **Quality Requirements:**
   - ✅ Guard pass rate: > 85%
   - ✅ Auto-fix success rate: > 70%
   - ✅ User approval rate (first attempt): > 60%
   - ✅ Defect classification accuracy: > 90%

4. **Security Requirements:**
   - ✅ RLS enforced on all tenant-specific tables
   - ✅ Per-tenant KMS encryption active
   - ✅ K-anonymity suppression working (<5 tenants)
   - ✅ Audit logs capturing all sharing events
   - ✅ Penetration test passed (no critical vulnerabilities)

5. **Documentation Requirements:**
   - ✅ API documentation complete (11 endpoints)
   - ✅ Database schema documented
   - ✅ End-user guides published
   - ✅ Admin guides published
   - ✅ Incident response runbooks ready

---

## 11. Security & Compliance

### 11.1 Security Architecture

#### 11.1.1 Row-Level Security (RLS)

**Purpose:** Enforce tenant isolation at database level

**Implementation:**
```sql
-- Enable RLS on brand_profiles
ALTER TABLE brand_profiles ENABLE ROW LEVEL SECURITY;

-- Policy: Tenant can only access their own data
CREATE POLICY tenant_isolation ON brand_profiles
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant'));

-- Set tenant context per request
SET app.current_tenant = 'tenant-001';
```

**Testing:**
```sql
-- As tenant tenant-001
SET app.current_tenant = 'tenant-001';
SELECT * FROM brand_profiles;  -- Returns only tenant-001 data

-- Attempt cross-tenant access (should fail)
SET app.current_tenant = 'competitor-002';
SELECT * FROM brand_profiles WHERE tenant_id = 'tenant-001';  -- Returns 0 rows
```

---

#### 11.1.2 Per-Tenant KMS Encryption

**Purpose:** Ensure even DB admins cannot decrypt competitor brand data

**Implementation:**
```python
# Encrypt brand voice vectors with tenant-specific key
def store_brand_voice(tenant_id, brand_vectors):
    kms_key = get_tenant_kms_key(tenant_id)
    encrypted = encrypt_with_kms(brand_vectors, kms_key)

    db.execute("""
        INSERT INTO brand_profiles (tenant_id, embeddings, kms_key_id)
        VALUES (?, ?, ?)
    """, [tenant_id, encrypted, kms_key.id])

# Decrypt requires correct tenant key
def retrieve_brand_voice(tenant_id):
    row = db.query("""
        SELECT embeddings, kms_key_id FROM brand_profiles
        WHERE tenant_id = ?
    """, [tenant_id])

    kms_key = get_kms_key(row.kms_key_id)
    decrypted = decrypt_with_kms(row.embeddings, kms_key)
    return decrypted
```

**Key Management:**
- Keys stored in Cloudflare Workers KV
- Key rotation every 90 days
- Audit log on all key access

---

#### 11.1.3 K-Anonymity Suppression

**Purpose:** Prevent small-group re-identification

**Implementation:**
```python
def aggregate_skill_metrics(skill_id):
    # Count unique tenants using this skill
    unique_tenants = db.query("""
        SELECT COUNT(DISTINCT tenant_id)
        FROM revision_events
        WHERE skill_id = ?
    """, [skill_id]).value

    # Suppress if < 5 tenants (K-anonymity threshold)
    if unique_tenants < 5:
        log_audit("suppressed_metric", skill_id, reason="K < 5")
        return None  # Do NOT share to global learning

    # Safe to aggregate
    avg_pass_rate = db.query("""
        SELECT AVG(pass_rate) FROM skill_metrics_daily
        WHERE skill_id = ?
    """, [skill_id]).value

    return avg_pass_rate
```

---

#### 11.1.4 Audit Logging

**Purpose:** Track all data access and sharing events

**Implementation:**
```python
def promote_skill_to_global(skill_id, reason):
    # Perform promotion
    db.execute("""
        UPDATE skills SET visibility = 'global'
        WHERE skill_id = ?
    """, [skill_id])

    # Log audit event
    db.execute("""
        INSERT INTO sharing_events (action, skill_id, reason, actor, ts)
        VALUES ('promote_to_global', ?, ?, 'system_auto', NOW())
    """, [skill_id, reason])

    # Alert admin
    send_slack_notification(f"Skill {skill_id} promoted to global: {reason}")
```

**Audit Queries:**
```sql
-- View all promotions in last 30 days
SELECT * FROM sharing_events
WHERE action = 'promote_to_global' AND ts >= NOW() - INTERVAL '30 days'
ORDER BY ts DESC;

-- View all accesses to specific tenant's brand data
SELECT * FROM audit_logs
WHERE table_name = 'brand_profiles' AND tenant_id = 'tenant-001'
ORDER BY ts DESC;
```

---

#### 11.1.5 Data Residency

**Purpose:** Comply with GDPR, CCPA data locality requirements

**Implementation:**
```python
# Regional database replicas
regions = {
    "eu-west": "d1_eu_west_replica",
    "us-east": "d1_us_east_replica",
    "ap-southeast": "d1_apac_replica"
}

def store_brand_profile(tenant_id, brand_data):
    tenant = get_tenant(tenant_id)
    db_region = regions[tenant.region]

    # Store in tenant's region
    db_region.execute("""
        INSERT INTO brand_profiles (tenant_id, embeddings, ...)
        VALUES (?, ?, ...)
    """, [tenant_id, brand_data])
```

**Replication Rules:**
- EU data NEVER leaves EU (GDPR Article 44)
- US data can replicate globally (with consent)
- APAC follows local regulations

---

### 11.2 Compliance Certifications

#### 11.2.1 GDPR (EU General Data Protection Regulation)

**Requirements:**
- ✅ **Data Minimization:** Only structural patterns shared (not personal brand voice)
- ✅ **Right to Access:** Tenants can export all their data via API
- ✅ **Right to Deletion:** Tenants can delete account + all private data
- ✅ **Right to Portability:** Export in JSON format
- ✅ **Consent:** DPA (Data Processing Agreement) signed before skill sharing
- ✅ **Data Locality:** EU data stays in EU regions
- ✅ **Breach Notification:** 72-hour incident response process

**DPA Template:**
```
DATA PROCESSING AGREEMENT

By using the DIFY-AGENT service, you agree that:

1. Structural patterns (layouts, guards) may be shared across tenants to improve system quality
2. Brand voice (tone, lexicon, colors, logos) remains private and encrypted
3. Performance metrics are aggregated only if ≥5 tenants contributed (K-anonymity)
4. You can request deletion of all data at any time
5. Your data is stored in your selected region and not transferred without consent

[Tenant Signature]
```

---

#### 11.2.2 SOC 2 Type II

**Requirements:**
- ✅ **Security:** Encryption at rest (KMS), in transit (TLS 1.3)
- ✅ **Availability:** 99.9% uptime SLA, redundant infrastructure
- ✅ **Processing Integrity:** Guards ensure output quality
- ✅ **Confidentiality:** RLS, KMS encryption, audit logs
- ✅ **Privacy:** GDPR compliance, data minimization

**Annual Audit:**
- Third-party auditor reviews controls
- Penetration testing (quarterly)
- Vulnerability scanning (weekly)
- Incident response drills (quarterly)

---

#### 11.2.3 ISO 27001 (Information Security)

**Requirements:**
- ✅ **Risk Assessment:** Documented threat model
- ✅ **Access Control:** RBAC (Role-Based Access Control)
- ✅ **Cryptography:** KMS encryption, TLS 1.3
- ✅ **Incident Management:** 24/7 on-call rotation
- ✅ **Business Continuity:** Disaster recovery plan, backups

**Certification Process:**
- Year 1: Gap analysis, implement controls
- Year 2: Internal audit, fix gaps
- Year 3: External audit, certification

---

### 11.3 Privacy Protection Timeline

#### Temporal Delay (Anti-Correlation)

**Problem:** Competitor could correlate "new skill appeared" with "Tenant X ran campaign"

**Solution:** 30-60 day delay before promoting skills to global bank

**Implementation:**
```python
def auto_promote_skills():
    # Find skills eligible for promotion
    eligible = db.query("""
        SELECT skill_id, created_at, performance_metrics
        FROM skills
        WHERE visibility = 'tenant:*'
          AND age_days >= 30  -- Temporal delay
          AND uses >= 100
          AND defect_rate < 0.05
          AND unique_tenants >= 5  -- K-anonymity
    """)

    for skill in eligible:
        promote_to_global(skill.skill_id)
```

**Result:** Even if competitor monitors global bank, cannot determine source tenant.

---

### 11.4 Quarterly Privacy Audit

**Checklist:**
- [ ] Review all skill promotions (ensure K ≥ 5)
- [ ] Check for temporal correlation leaks
- [ ] Verify RLS policies active
- [ ] Audit KMS key access logs
- [ ] Test cross-tenant isolation (penetration test)
- [ ] Review data residency compliance
- [ ] Update DPA templates if needed
- [ ] Security awareness training for team

**Output:** Privacy Audit Report (quarterly)

---

## 12. Observability

### 12.1 SLOs (Service Level Objectives)

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| API Availability | 99.9% | < 99.5% |
| Intent Parser Latency (p95) | < 500ms | > 800ms |
| Skill Retrieval Latency (p95) | < 200ms | > 400ms |
| Content Generation Latency (p95) | < 5s | > 8s |
| Guards Execution Latency (p95) | < 1s | > 2s |
| Guard Pass Rate | > 85% | < 75% |
| User Approval Rate | > 60% | < 45% |
| Defect Classification Accuracy | > 90% | < 80% |

---

### 12.2 Monitoring Dashboards

#### Dashboard 1: System Health
**Metrics:**
- Requests per minute (RPM)
- Error rate (5xx errors)
- Latency (p50, p95, p99)
- Infrastructure (CPU, memory, DB connections)

**Tools:** Cloudflare Analytics Engine, Grafana

---

#### Dashboard 2: Content Generation
**Metrics:**
- Multi-candidate generation rate
- Guard pass/fail breakdown (per guard)
- Auto-fix success rate
- User selection distribution (which candidates chosen)
- Revision rate (% requiring regeneration)

**Tools:** Custom Grafana dashboard

---

#### Dashboard 3: Skill Performance
**Metrics:**
- Skill usage distribution (top 20 skills)
- Skill lifecycle (active/testing/champion/failing)
- Promotion/retirement events
- Defect vs preference classification breakdown

**Tools:** Custom Grafana dashboard

---

#### Dashboard 4: Multi-Tenant Insights
**Metrics:**
- Active tenants
- Per-tenant usage (anonymized)
- K-anonymity compliance (% metrics suppressed)
- Regional distribution

**Tools:** Admin-only dashboard

---

### 12.3 Distributed Tracing

**Tool:** Cloudflare Workers Trace

**Trace Structure:**
```
Request ID: req_abc123
├─ Intent Parser (120ms)
│  ├─ OpenAI GPT-5 call (80ms)
│  └─ Product lookup (40ms)
├─ Skill Retrieval (150ms)
│  ├─ Vectorize search (100ms)
│  └─ Metadata fetch (50ms)
├─ Multi-Candidate Generation (3.2s)
│  ├─ Candidate A (1.1s)
│  ├─ Candidate B (1.0s)
│  └─ Candidate C (1.1s)
├─ Guards Engine (800ms)
│  ├─ Contrast check (200ms)
│  ├─ Margin check (150ms)
│  ├─ Logo protection (250ms)
│  └─ Auto-fix (200ms)
└─ Reranker (50ms)

Total: 4.32s
```

**Benefits:**
- Identify slow services
- Debug latency spikes
- Optimize critical paths

---

### 12.4 Alerting

**Alert Channels:**
- PagerDuty (critical incidents)
- Slack (warnings)
- Email (weekly summaries)

**Alert Rules:**

| Alert | Condition | Severity | Action |
|-------|-----------|----------|--------|
| API Down | 5xx rate > 5% for 5min | Critical | Page on-call engineer |
| Latency Spike | p95 > 8s for 10min | Warning | Investigate in morning |
| Guard Failures | Pass rate < 75% for 1hr | Warning | Review guard logic |
| Low Approval Rate | < 45% for 6hr | Warning | Review content quality |
| DB Connection Pool | > 90% utilization | Critical | Scale DB connections |
| Cost Spike | Daily cost > 2× baseline | Warning | Check for runaway jobs |
| Security Incident | Failed RLS check | Critical | Immediate investigation |

---

### 12.5 Incident Response

**On-Call Rotation:**
- 24/7 coverage (5 engineers, weekly rotation)
- Primary + backup on-call
- Max response time: 15 minutes

**Incident Severity:**

| Severity | Definition | Response Time | Example |
|----------|------------|---------------|---------|
| P0 (Critical) | System down, data breach | 15 min | API returning 500s |
| P1 (High) | Degraded performance, partial outage | 1 hour | Latency > 10s |
| P2 (Medium) | Non-critical feature broken | 4 hours | Guard auto-fix failing |
| P3 (Low) | Minor issue, workaround exists | 1 day | UI typo |

**Incident Process:**
1. Alert triggers → Page on-call
2. Acknowledge within 15min
3. Investigate & mitigate
4. Post-incident review (PIR) within 48hr
5. Implement fixes & update runbooks

---

## 13. Integration

### 13.1 Integration with Stage A (Dify Setup)

**Stage A Deliverables:**
- ✅ Dify platform deployed (Cloud or self-hosted)
- ✅ PostgreSQL database configured
- ✅ Redis for session management
- ✅ S3/R2 for file storage
- ✅ OpenAI API connected

**Stage D Integration:**
- Use Dify's **Custom Tools** feature to call MCP Gateway REST endpoints
- Configure 11 custom tools (one per API endpoint)
- Map Dify conversation state to Context Service
- Leverage Dify's multi-turn conversation handling

**Example Dify Tool Configuration:**
```yaml
# Intent Parser Tool
name: parse_intent
description: Parse user request into structured IntentSpec
endpoint: https://api.dify-agent.com/v1/intent/parse
method: POST
parameters:
  - name: user_message
    type: string
    required: true
  - name: conversation_history
    type: array
    required: false
output_schema:
  intent_spec: object
```

---

### 13.2 Integration with Stage B (Skills System)

**Stage B Deliverables:**
- ✅ Meta-Skill Builder (generates skills from user requests)
- ✅ Skill storage & versioning
- ✅ Skill matching algorithm
- ✅ Skill library (initial 50 skills)

**Stage D Integration:**
- **Skill Retrieval Service** queries Stage B's skill library
- **Learning Engine** updates skill metrics (pass_rate, defect_rate)
- **Meta-Skill Builder** generates new skills when similarity < 0.65
- Shared database tables (skills, skill_versions, skill_embeddings)

**Data Flow:**
```
Stage D: Intent Parser → IntentSpec
    ↓
Stage B: Meta-Skill Builder checks if new skill needed
    ↓
Stage D: Skill Retrieval fetches top 5 skills
    ↓
Stage D: Multi-Candidate Generator applies skills
    ↓
Stage B: Skill metrics updated based on performance
```

---

### 13.3 Integration with Stage C (Knowledge Base)

**Stage C Deliverables:**
- ✅ Brand voice extraction (GPT-5)
- ✅ Product catalog scraping (Firecrawl)
- ✅ Vector embeddings (voyage-3-lite)
- ✅ RAG architecture (Vectorize)
- ✅ Supabase products database

**Stage D Integration:**
- **Brand Voice Service** retrieves brand profiles from Stage C's knowledge base
- **Intent Parser** queries product catalog for product verification
- **Content Generation** uses brand voice embeddings to ensure on-brand outputs
- Shared Vectorize index (brand_voice + product_catalog)

**Data Flow:**
```
Stage C: Brand voice extracted and stored as embeddings
    ↓
Stage D: Brand Voice Service retrieves for tenant
    ↓
Stage D: Multi-Candidate Generator applies brand voice to content
    ↓
Stage D: Guards check brand palette compliance
```

**Shared Resources:**
- Vectorize index: `tenant-brand-voice` (from Stage C)
- Supabase products table (from Stage C)
- Brand profiles table (created in Stage C, read by Stage D)

---

### 13.4 Integration with Existing MCP Gateway

**Current MCP Gateway (9 Tools):**
1. generate_content
2. verify_product_image
3. get_product_image
4. verify_product_context
5. get_product_price
6. check_stock
7. send_whatsapp
8. send_sms
9. publish_linkedin

**Stage D Extensions:**
- Add REST endpoints for 11 new services
- Maintain JSON-RPC 2.0 compatibility
- Extend authentication to support Dify tokens

**Gateway Architecture:**
```
Dify Platform
    ↓ HTTPS/REST
MCP Gateway (Cloudflare Workers)
    ├─ Existing Tools (9) → External Services
    └─ Stage D Services (11) → Internal Services
        ├─ Intent Parser
        ├─ Skill Retrieval
        ├─ Multi-Candidate Generator
        ├─ Guards Engine
        ├─ Reranker
        ├─ Defect Classifier
        ├─ Brand Voice
        ├─ Publishing
        ├─ Analytics
        ├─ Learning Engine
        └─ Context
```

---

### 13.5 End-to-End Workflow Example

**User Request:** "Create a LinkedIn post about our Product SKU-Product SKU-A142 with pricing"

**System Flow:**

```
1. Dify receives user message
   ↓
2. Dify calls MCP Gateway: parse_intent
   ↓
3. Intent Parser Service:
   - Calls OpenAI GPT-5 to extract intent
   - Queries Supabase for product "Product SKU-Product SKU-A142"
   - Returns IntentSpec JSON
   ↓
4. Dify calls MCP Gateway: retrieve_skills
   ↓
5. Skill Retrieval Service:
   - Embeds IntentSpec
   - Searches Vectorize index
   - Returns top 5 skills
   ↓
6. Dify calls MCP Gateway: generate_multi_candidate
   ↓
7. Multi-Candidate Generator:
   - Fetches brand voice from Stage C
   - Generates 3 candidates using skills
   - Calls Guards Engine for each
   - Auto-fixes failures
   ↓
8. Dify calls MCP Gateway: rerank
   ↓
9. Reranker Service:
   - Scores candidates
   - Returns ranked list
   ↓
10. Dify presents options to user (A/B/C)
    ↓
11. User selects "A"
    ↓
12. Dify calls MCP Gateway: publish_content
    ↓
13. Publishing Service:
    - Validates OAuth token
    - Uploads image to LinkedIn
    - Creates LinkedIn post
    - Returns post_id and URL
    ↓
14. Dify confirms to user: "Posted! View at: [link]"
    ↓
15. Analytics Service logs selection
    ↓
16. Learning Engine updates skill metrics (nightly batch)
```

**Total Latency:** 6-8 seconds (Intent → Published)

---

## Appendix A: Glossary

**IntentSpec:** Structured JSON representing user's content request (task, platform, modality, constraints)

**Skill:** Reusable content generation pattern (layout, prompt, constraints, guards)

**Multi-Candidate Generation:** Creating 2-3 diverse content variations for user selection

**Guards:** Automated quality checks (contrast, margins, overlap, brand compliance)

**Defect:** Skill failure (failed guard, factual error, technical issue)

**Preference:** User style choice (tone, layout, content additions) - NOT a skill failure

**K-Anonymity:** Privacy technique requiring ≥K users before aggregating data

**RLS (Row-Level Security):** Database-level tenant isolation

**KMS (Key Management Service):** Encryption key storage and rotation

**ε-greedy:** Exploration strategy (explore with probability ε, exploit otherwise)

**Reranking:** Scoring and sorting candidates by quality/cost/diversity

**Brand Voice:** Tenant-specific tone, lexicon, colors, logos (PRIVATE)

**Structural Pattern:** Generic layout/guard rules (SHARED across tenants)

---

## Appendix B: Decision Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2025-11-10 | Build full sophisticated system (NOT MVP) | User explicitly rejected phased approach, wants complete system from day 1 |
| 2025-11-10 | Shared structural skills + private brand voice | Balances learning efficiency with competitive protection |
| 2025-11-10 | Multi-candidate generation (2-3) | Gives users meaningful choice, enables exploration |
| 2025-11-10 | 7 guards with auto-fixers | Catches 90% of quality issues before user sees output |
| 2025-11-10 | Defect vs Preference classification | Prevents penalizing skills for user style choices |
| 2025-11-10 | K-anonymity threshold = 5 | Standard privacy threshold (EU GDPR recommendation) |
| 2025-11-10 | 30-60 day promotion delay | Prevents temporal correlation attacks |
| 2025-11-10 | Claude Sonnet 4.5 for content | Highest quality text generation (research-proven) |
| 2025-11-10 | Gemini 2.5 Flash for images | Lowest cost ($0.039/image), good quality |
| 2025-11-10 | 6-month build timeline | Realistic for research-grade system with 5-person team |

---

## Appendix C: References

**Research Documents:**
- STAGE-B-SKILLS-DECISIONS-ANALYSIS.md (Agent 1)
- PRODUCTION-TEMPLATE-STRATEGIES-2025.md (Agent 2)
- RAG-RETRIEVAL-VS-GENERATION-PATTERNS.md (Agent 3)
- COST-PERFORMANCE-TRADEOFFS-2025.md (Agent 4)
- SKILL-BANK-MANAGEMENT-PATTERNS.md (Agent 5)
- SKILL-EVOLUTION-STRATEGIES-BRIEF.md (Agent 6)

**Industry Standards:**
- WCAG 2.1 (Web Content Accessibility Guidelines)
- GDPR (EU General Data Protection Regulation)
- SOC 2 Type II (Security & Privacy)
- ISO 27001 (Information Security)

**Technology Documentation:**
- Cloudflare Workers: https://developers.cloudflare.com/workers
- Cloudflare Vectorize: https://developers.cloudflare.com/vectorize
- Dify: https://docs.dify.ai
- Claude API: https://docs.anthropic.com
- OpenAI API: https://platform.openai.com/docs

---

## Document Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2025-11-10 | Planning Agent | Initial production-ready specification |

---

**END OF STAGE D APPROVED PLAN**

This document is PRODUCTION-READY and self-contained. Development teams should NOT need to reference any planning or research documents to implement Stage D.
