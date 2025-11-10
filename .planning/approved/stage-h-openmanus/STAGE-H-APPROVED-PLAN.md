# Stage H: OpenManus Integration - APPROVED PLAN

**Version**: 1.0.0
**Status**: ✅ APPROVED
**Date**: November 10, 2025
**Stage**: H of 12 (Autonomous Intelligence Layer)
**Dependencies**: Stage B (Skill System), Stage C (Knowledge Base), Stage D (Content Generation), Stage G (Dify Frontend)

---

## Executive Summary

### Purpose
Integrate OpenManus as the autonomous intelligence layer for the DIFY-AGENT system, enabling multi-agent coordination for complex tasks: meta-skill generation, brand voice analysis, content refinement, and competitive intelligence.

### Critical Requirement: Autonomous Multi-Agent Workflows
The system MUST support autonomous task execution where:
- **Planning Agent** decomposes complex requests into subtasks
- **Research Agent** gathers data from web, knowledge bases, competitors
- **Coder Agent** generates skills, code, templates
- **Reporter Agent** synthesizes results into actionable insights

**Why OpenManus?**
- Multi-agent coordination out-of-the-box
- Integration with Dify via HTTP endpoints
- Open-source (full control, IP ownership)
- Production-ready deployment patterns

### Key Deliverables
1. **OpenManus Deployment** - Fly.io machines with 4 agent types
2. **4 Custom API Endpoints** - REST APIs for Dify to call
3. **Agent Coordination System** - Inter-agent communication, shared memory
4. **Dify HTTP Tool Integration** - Sync + async patterns for long tasks
5. **Task Queue System** - Durable Objects for async task management
6. **Monitoring & Observability** - Datadog integration, performance metrics

### Timeline
**12 weeks** (3 months)

### Budget
- **Development**: $105,000 - $180,000 (280-480 hours @ $375/hour)
- **Monthly Operational**: $120-$170/month ($1.20-$1.70 per 100 customers)

### Success Criteria
- ✅ All 4 agents deployed and operational
- ✅ 4 API endpoints functional with <10s response time (sync) or async polling
- ✅ Dify workflows successfully call OpenManus endpoints
- ✅ Meta-skill generation produces valid Stage B format skills
- ✅ Brand voice analysis accuracy >85% vs human evaluation
- ✅ Content refinement improves engagement by ≥10%
- ✅ 99% uptime SLA

---

## Table of Contents

1. [System Architecture](#1-system-architecture)
2. [OpenManus Framework Overview](#2-openmanus-framework-overview)
3. [Agent System Design](#3-agent-system-design)
4. [Custom API Endpoints](#4-custom-api-endpoints)
5. [Agent Coordination Patterns](#5-agent-coordination-patterns)
6. [Dify Integration](#6-dify-integration)
7. [Async Task Handling](#7-async-task-handling)
8. [Deployment Architecture](#8-deployment-architecture)
9. [Security & Authentication](#9-security--authentication)
10. [Monitoring & Observability](#10-monitoring--observability)
11. [Cost Analysis](#11-cost-analysis)
12. [Implementation Timeline](#12-implementation-timeline)
13. [Testing Strategy](#13-testing-strategy)
14. [Deployment Plan](#14-deployment-plan)
15. [Risk Assessment](#15-risk-assessment)
16. [Appendices](#16-appendices)

---

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   USER INTERFACE LAYER                           │
│                                                                  │
│  Next.js 15 Frontend (from Stage G)                            │
│  - User types request: "Generate skill from these 5 posts"     │
│  - Dify shows progress: "Planning analysis... 20% complete"    │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                  ORCHESTRATION LAYER                             │
│                                                                  │
│  Dify Cloud (from Stage G)                                      │
│  - Main Chatflow workflow                                       │
│  - HTTP Tool Node → calls OpenManus API                         │
│  - Polling loop for async tasks                                 │
│  - Progress updates to user                                     │
└────────────────────────┬────────────────────────────────────────┘
                         │ HTTPS (REST API)
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              AUTONOMOUS INTELLIGENCE LAYER ⭐ NEW               │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │     OpenManus API Gateway (Cloudflare Workers)          │  │
│  │                                                           │  │
│  │  Endpoints:                                              │  │
│  │  - POST /api/generate-skill         (async 5-10 min)    │  │
│  │  - POST /api/analyze-brand-voice    (async 3-5 min)     │  │
│  │  - POST /api/refine-content         (sync 15-45s)       │  │
│  │  - POST /api/competitive-analysis   (async 10-30 min)   │  │
│  │                                                           │  │
│  │  Task Queue: Durable Objects                             │  │
│  │  - Store task state                                      │  │
│  │  - Progress tracking                                     │  │
│  │  - Result retrieval                                      │  │
│  │                                                           │  │
│  └────────────────────┬─────────────────────────────────────┘  │
│                       │                                          │
│                       ▼                                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         OpenManus Multi-Agent System                     │  │
│  │         (Fly.io Machines)                                │  │
│  │                                                           │  │
│  │   ┌─────────────┐    ┌──────────────┐                  │  │
│  │   │  Planning   │───>│   Research   │                  │  │
│  │   │   Agent     │    │    Agent     │                  │  │
│  │   └──────┬──────┘    └──────┬───────┘                  │  │
│  │          │                   │                           │  │
│  │          │    ┌──────────────┴───────┐                  │  │
│  │          └───>│  Shared Memory       │                  │  │
│  │               │  (Redis)             │                  │  │
│  │          ┌───>│  - Task state        │                  │  │
│  │          │    │  - Agent messages    │                  │  │
│  │          │    │  - Artifacts         │                  │  │
│  │          │    └──────────────┬───────┘                  │  │
│  │          │                   │                           │  │
│  │   ┌──────┴──────┐    ┌──────┴───────┐                  │  │
│  │   │    Coder    │    │   Reporter   │                  │  │
│  │   │    Agent    │<───│    Agent     │                  │  │
│  │   └─────────────┘    └──────────────┘                  │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                     SERVICES LAYER                               │
│                                                                  │
│  MCP Gateway + Microservices (from Stages C, D, E, F)          │
│  - Stage C: Knowledge Base (RAG, Vectorize)                    │
│  - Stage D: Content Generation (11 microservices, 7 guards)    │
│  - Stage E: Multi-Platform Publishing (4 adapters)             │
│  - Stage F: Analytics & Performance                             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 1.2 Data Flow Example: Generate Skill from Examples

```
1. User (via Dify): "Create a skill from these 5 Product Line A posts"
   ↓
2. Dify Workflow: Calls OpenManus API
   POST /api/generate-skill/start
   Body: {tenant_id, examples: [5 posts with images, metrics]}
   Response: {task_id: "task_abc123", status: "queued"}
   ↓
3. Dify Polling Loop (every 10s):
   GET /api/generate-skill/status/task_abc123
   Response: {status: "running", progress: 35, eta: 180, current_step: "Planning Agent analyzing patterns"}
   ↓
4. OpenManus Multi-Agent Execution:

   Step 1: Planning Agent (15s)
   - Analyze 5 example posts
   - Identify patterns: layout, tone, CTAs, hashtags
   - Create subtasks: "Extract layout", "Analyze tone", "Generate YAML"
   - Assign: Research → Coder → Reporter

   Step 2: Research Agent (2 min)
   - Query knowledge base (Stage C) for similar high-performing posts
   - Analyze competitor posts with similar themes
   - Extract successful patterns: "Posts with product images get 2.3x engagement"

   Step 3: Coder Agent (3 min)
   - Generate YAML skill file (Stage B format)
   - Apply patterns: layout_type: "product_showcase", tone: "professional"
   - Run validation: Check Stage D guards compatibility
   - Output: Complete skill YAML + validation results

   Step 4: Reporter Agent (30s)
   - Synthesize results
   - Calculate confidence score (85%)
   - Predict performance (+15% engagement vs baseline)
   - Create user-facing report
   ↓
5. Dify Gets Completion:
   GET /api/generate-skill/status/task_abc123
   Response: {status: "completed", result_available: true}
   ↓
6. Dify Fetches Result:
   GET /api/generate-skill/result/task_abc123
   Response: {
     skill_id: "skill_xyz",
     skill_yaml: "...",
     confidence_score: 85,
     patterns_found: ["Product images required", "Professional tone", "Benefit-focused"],
     validation: {passes_guards: true},
     estimated_performance: {engagement_rate: +15%}
   }
   ↓
7. Dify presents to user:
   "✅ Skill generated! Confidence: 85%
   Found 3 key patterns. Passes all validation guards.
   Estimated +15% engagement vs your baseline.
   Would you like to: A) Use this skill B) Refine it C) Generate another?"
```

---

## 2. OpenManus Framework Overview

### 2.1 What is OpenManus?

OpenManus is an open-source multi-agent framework developed by the MetaGPT team in 2025 for autonomous AI task execution.

**Key Features:**
- **Modular Architecture**: BaseAgent, PlanningAgent, SWEAgent, ToolCallAgent
- **Multi-Agent Coordination**: Built-in communication, shared memory
- **LLM-Agnostic**: Works with GPT-4, Claude, Gemini
- **Reinforcement Learning**: OpenManus-RL for agent improvement
- **Production-Ready**: Docker, Kubernetes, MCP integration

**Why OpenManus for Stage H:**
- ✅ Open source (full control, no vendor lock-in)
- ✅ Designed for complex autonomous tasks
- ✅ Active development (2025 launch)
- ✅ Multi-agent coordination primitives
- ✅ Integration with Dify via HTTP APIs

### 2.2 Alternative Frameworks Considered

| Framework | Pros | Cons | Cost | Decision |
|-----------|------|------|------|----------|
| **OpenManus** | Open source, multi-agent, active dev | Newer (2025), fewer docs | Free (self-host) | ✅ **SELECTED** |
| CrewAI | Mature, easy setup, managed option | $25/month managed | $25/mo or free | Alternative if issues |
| Microsoft AutoGen | Enterprise-grade, Azure integration | Requires Azure, complex | Varies | Too enterprise-heavy |
| LangGraph | Most flexible, graph-based | Steep learning curve | Free | Too complex for needs |

**Recommendation**: Use OpenManus (primary), CrewAI (fallback if needed)

### 2.3 OpenManus Components for DIFY-AGENT

**Core Components Used:**
1. **PlanningAgent** - Task decomposition, strategy
2. **ReActAgent** (Research) - Web search, data gathering
3. **ToolCallAgent** (Coder) - Skill generation, code writing
4. **BaseAgent** (Reporter) - Result synthesis, reporting

**Tools Required:**
- `GoogleSearch` - Competitor analysis
- `PythonExecute` - Code generation, validation
- `BrowserUseTool` - Web scraping
- Custom tools: `VectorizeSearch` (Stage C), `GuardValidator` (Stage D)

---

## 3. Agent System Design

### 3.1 Four Core Agents

#### Agent 1: Planning Agent
**Role**: Task decomposition, strategy optimization, agent coordination

**Responsibilities:**
- Parse user request and identify intent
- Break complex tasks into subtasks
- Assign subtasks to appropriate agents
- Define success criteria
- Estimate resource requirements

**Tools:**
- Task decomposition algorithm
- Dependency graph builder
- Context analyzer (brand voice, past performance)

**Example Task Breakdown** (Generate Skill):
```python
{
  "task_id": "task_abc123",
  "subtasks": [
    {
      "id": "sub_1",
      "type": "analyze_examples",
      "assigned_to": "research_agent",
      "inputs": ["5 example posts"],
      "outputs": ["patterns", "similar_posts"],
      "estimated_time": "2 min"
    },
    {
      "id": "sub_2",
      "type": "generate_yaml",
      "assigned_to": "coder_agent",
      "inputs": ["patterns from sub_1"],
      "outputs": ["skill_yaml", "validation_results"],
      "estimated_time": "3 min",
      "depends_on": ["sub_1"]
    },
    {
      "id": "sub_3",
      "type": "create_report",
      "assigned_to": "reporter_agent",
      "inputs": ["skill_yaml from sub_2"],
      "outputs": ["final_report"],
      "estimated_time": "30 sec",
      "depends_on": ["sub_2"]
    }
  ],
  "total_estimated_time": "5.5 min"
}
```

**LLM Used:** GPT-4 Turbo (planning requires reasoning)

---

#### Agent 2: Research Agent
**Role**: Information gathering, web search, data analysis

**Responsibilities:**
- Search knowledge base (Stage C Vectorize)
- Scrape competitor social media
- Analyze trends in social media data
- Extract brand voice from existing posts
- Find similar high-performing posts

**Tools:**
- `VectorizeSearch` - Semantic search in knowledge base
- `GoogleSearch` - Web search for competitors
- `BrowserUseTool` - Web scraping (Playwright)
- `Firecrawl` - Structured web scraping (Stage C)

**Example Research Tasks:**
1. **Brand Voice Analysis**: Analyze 50 past posts → extract tone, lexicon, style
2. **Competitor Analysis**: Scrape top 3 competitors' LinkedIn → identify content gaps
3. **Pattern Detection**: Find 10 similar high-performing posts → extract common elements

**LLM Used:**
- GPT-4o-mini (analysis, summarization) - cost-effective
- Claude Sonnet 4.5 (complex reasoning) - when needed

---

#### Agent 3: Coder Agent
**Role**: Code generation, skill creation, validation

**Responsibilities:**
- Generate YAML skill files (Stage B format)
- Create brand voice embeddings
- Write validation code
- Run Stage D guards
- Generate performance prediction code

**Tools:**
- `PythonExecute` - Run Python code for generation
- `YAMLValidator` - Validate skill format
- `GuardRunner` - Run Stage D validation guards
- `EmbeddingGenerator` - Create brand voice vectors

**Example Generated Skill** (abbreviated):
```yaml
skill_id: skill_product_showcase_v1
name: "Product Showcase Post"
version: 1.0.0
platforms: [linkedin, facebook]

layout:
  type: "image_with_text"
  components:
    - type: "hero_image"
      requirements: "Product photo, 1200x627px"
    - type: "headline"
      max_length: 150
      tone: "professional, benefit-focused"
    - type: "body"
      max_length: 1000
      structure: "Problem → Solution → CTA"
    - type: "cta"
      required: true
      examples: ["Learn more", "Shop now", "Get quote"]

brand_voice:
  tone: "professional yet approachable"
  formality: 70  # 0-100 scale
  technical_level: 60
  keywords: ["[product-name]", "quality", "delivery"]

guards:
  - name: "brand_voice_score"
    min_score: 75
  - name: "readability"
    target: "grade_8"
  - name: "image_quality"
    min_resolution: "1200x627"
```

**LLM Used:** GPT-4 (skill generation requires structured output)

---

#### Agent 4: Reporter Agent
**Role**: Results synthesis, documentation, user-facing reports

**Responsibilities:**
- Synthesize outputs from other agents
- Calculate confidence scores
- Generate natural language explanations
- Create visualizations (charts, metrics)
- Provide actionable recommendations

**Tools:**
- `ReportTemplateEngine` - Structured reports
- `ChartGenerator` - Metrics visualization
- `ConfidenceCalculator` - Score validation results

**Example Report Output:**
```markdown
# Skill Generation Report

## Summary
Successfully generated new skill: **Product Showcase Post**
- **Confidence:** 85% (based on 5 examples + 23 similar posts analyzed)
- **Estimated Performance:** +15% engagement vs baseline
- **Validation:** ✅ Passes all 7 Stage D guards

## Key Patterns Identified
1. **Visual**: Product images drive 2.3x more engagement
2. **Tone**: Professional yet approachable (70/100 formality)
3. **Structure**: Problem → Solution → CTA format performs best
4. **Length**: 200-400 words optimal for LinkedIn
5. **CTAs**: "Learn more" outperforms "Buy now" by 40%

## Recommendations
- Use this skill for Product Line A promotions
- Test on LinkedIn first (highest predicted performance: +18%)
- A/B test CTA variations after 10 posts
- Monitor engagement rate vs 1.2% baseline

## Next Steps
Would you like to:
A) Deploy this skill to production
B) Refine based on additional examples
C) Generate a variation for Instagram
```

**LLM Used:** GPT-4o-mini (report generation is straightforward)

---

### 3.2 Agent Lifecycle & States

```typescript
enum AgentState {
  IDLE = "idle",               // Waiting for tasks
  PLANNING = "planning",       // Planning Agent active
  RESEARCHING = "researching", // Research Agent active
  CODING = "coding",           // Coder Agent active
  REPORTING = "reporting",     // Reporter Agent active
  ERROR = "error",             // Task failed
  COMPLETED = "completed"      // Task complete
}

interface AgentTask {
  task_id: string;
  agent_type: "planning" | "research" | "coder" | "reporter";
  state: AgentState;
  inputs: any;
  outputs?: any;
  error?: Error;
  started_at: string;
  completed_at?: string;
  execution_time_ms?: number;
}
```

---

## 4. Custom API Endpoints

### 4.1 Endpoint Overview

| Endpoint | Method | Type | Avg Time | Use Case |
|----------|--------|------|----------|----------|
| `/api/generate-skill` | POST | Async | 5-10 min | Meta-skill generation from examples |
| `/api/analyze-brand-voice` | POST | Async | 3-5 min | Extract brand voice profile |
| `/api/refine-content` | POST | Sync | 15-45s | Iterative content improvement |
| `/api/competitive-analysis` | POST | Async | 10-30 min | Competitor social media analysis |

**Authentication:** Bearer token (API key per tenant)
**Rate Limiting:** 100 req/hour (async), 1000 req/hour (sync)

---

### 4.2 Endpoint 1: Generate Skill

**Purpose**: Meta-skill generation - Create new social post skills from example posts

**URL**: `POST /api/generate-skill/start`

**Input Schema:**
```typescript
interface GenerateSkillRequest {
  tenant_id: string;
  examples: Array<{
    platform: 'linkedin' | 'twitter' | 'facebook' | 'instagram';
    content: string;
    image_url?: string;
    performance_metrics: {
      likes: number;
      comments: number;
      shares: number;
      impressions: number;
    };
  }>;
  skill_name?: string;
  target_platforms: string[];
}
```

**Agent Workflow:**
1. **Planning Agent** (15s): Analyze examples, identify patterns, create extraction plan
2. **Research Agent** (2 min): Find similar high-performing posts from Stage C knowledge base
3. **Coder Agent** (3 min): Generate YAML skill file (Stage B format), run guards
4. **Reporter Agent** (30s): Validate skill, calculate confidence, create report

**Output Schema:**
```typescript
interface GenerateSkillResponse {
  skill_id: string;
  skill_name: string;
  skill_yaml: string;
  confidence_score: number;
  patterns_found: string[];
  validation_results: {
    passes_guards: boolean;
    guard_results: Array<{guard: string; passed: boolean; score?: number}>;
  };
  estimated_performance: {
    engagement_rate_increase: number;
    confidence: number;
  };
  recommendations: string[];
}
```

**Example Request:**
```json
{
  "tenant_id": "tenant_123",
  "examples": [
    {
      "platform": "linkedin",
      "content": "Introducing our Product Line A - Built for professionals who demand quality...",
      "image_url": "https://r2.cloudflare.com/product-a.jpg",
      "performance_metrics": {
        "likes": 45,
        "comments": 12,
        "shares": 8,
        "impressions": 2300
      }
    }
    // ... 4 more examples
  ],
  "skill_name": "Product Launch Announcement",
  "target_platforms": ["linkedin", "facebook"]
}
```

**Example Response:**
```json
{
  "skill_id": "skill_xyz789",
  "skill_name": "Product Launch Announcement",
  "skill_yaml": "skill_id: skill_xyz789\nname: Product Launch Announcement\n...",
  "confidence_score": 85,
  "patterns_found": [
    "All examples use product images (100%)",
    "Average tone formality: 72/100 (professional)",
    "CTAs present in 80% of examples",
    "Benefit-focused headlines (100%)"
  ],
  "validation_results": {
    "passes_guards": true,
    "guard_results": [
      {"guard": "brand_voice_score", "passed": true, "score": 82},
      {"guard": "readability", "passed": true, "score": 8.2},
      {"guard": "length_check", "passed": true}
    ]
  },
  "estimated_performance": {
    "engagement_rate_increase": 15,
    "confidence": 78
  },
  "recommendations": [
    "Deploy for Product Line A campaigns",
    "Test on LinkedIn first (highest predicted performance)",
    "Monitor engagement vs 1.2% baseline for 10 posts"
  ]
}
```

**Async Pattern:**
- Start: `POST /api/generate-skill/start` → Returns `{task_id, status: "queued"}`
- Poll: `GET /api/generate-skill/status/:task_id` → Returns `{status, progress, eta}`
- Result: `GET /api/generate-skill/result/:task_id` → Returns full response above

**Execution Time:** 5-10 minutes (depends on number of examples)

---

### 4.3 Endpoint 2: Analyze Brand Voice

**Purpose**: Extract brand voice profile from customer's existing social media posts

**URL**: `POST /api/analyze-brand-voice/start`

**Input Schema:**
```typescript
interface AnalyzeBrandVoiceRequest {
  tenant_id: string;
  posts: Array<{
    platform: string;
    content: string;
    created_at: string;
    engagement: {
      likes: number;
      comments: number;
      shares: number;
    };
  }>;
  include_competitors?: boolean;
}
```

**Agent Workflow:**
1. **Planning Agent** (10s): Plan analysis strategy (tone, lexicon, style dimensions)
2. **Research Agent** (2-3 min):
   - Analyze all posts for patterns
   - If `include_competitors=true`, scrape competitor posts
3. **Coder Agent** (1-2 min):
   - Generate brand voice embeddings
   - Calculate voice dimensions (formality, enthusiasm, technicality)
   - Extract lexicon (common phrases, keywords, avoid-words)
4. **Reporter Agent** (30s): Create human-readable brand voice profile

**Output Schema:**
```typescript
interface AnalyzeBrandVoiceResponse {
  brand_voice_profile: {
    tone: {
      primary: string;
      secondary: string[];
      confidence: number;
    };
    lexicon: {
      common_phrases: string[];
      keywords: string[];
      avoid_words: string[];
    };
    style: {
      sentence_length: "short" | "medium" | "long";
      paragraph_structure: string;
      emoji_usage: "none" | "rare" | "moderate" | "heavy";
      hashtag_strategy: string;
    };
    voice_dimensions: {
      formality: number;
      enthusiasm: number;
      technicality: number;
    };
  };
  competitor_differentiation?: {
    unique_to_customer: string[];
    opportunities: string[];
  };
  sample_posts: {
    high_scoring: string[];
    low_scoring: string[];
  };
  embedding_vector: number[];
}
```

**Example Response:**
```json
{
  "brand_voice_profile": {
    "tone": {
      "primary": "professional yet approachable",
      "secondary": ["informative", "customer-focused"],
      "confidence": 87
    },
    "lexicon": {
      "common_phrases": [
        "industry-leading",
        "delivered on time",
        "quality you can trust"
      ],
      "keywords": ["Product Line A", "Product Line B", "construction", "delivery"],
      "avoid_words": ["cheap", "basic", "generic"]
    },
    "style": {
      "sentence_length": "medium",
      "paragraph_structure": "2-3 paragraphs, 3-5 sentences each",
      "emoji_usage": "rare",
      "hashtag_strategy": "2-3 relevant hashtags, #Industry focused"
    },
    "voice_dimensions": {
      "formality": 72,
      "enthusiasm": 65,
      "technicality": 58
    }
  },
  "competitor_differentiation": {
    "unique_to_customer": [
      "Emphasizes delivery speed (competitors focus on price)",
      "Uses customer testimonials (competitors use product specs)"
    ],
    "opportunities": [
      "Competitors never use humor - opportunity to stand out with light humor",
      "Video content rare in competitors - high engagement potential"
    ]
  },
  "sample_posts": {
    "high_scoring": [
      "Just delivered 20 tons of Product Line B to Company XYZ site - on schedule, every time. #Industry"
    ],
    "low_scoring": [
      "Check out our products!" // Too generic, low formality
    ]
  },
  "embedding_vector": [0.234, -0.123, 0.456, ...]
}
```

**Storage:** Brand voice profile stored in Stage C knowledge base (Dify KB)

**Execution Time:** 3-5 minutes (50 posts), 10-15 minutes (with competitors)

---

### 4.4 Endpoint 3: Refine Content

**Purpose**: Iterative content improvement based on user feedback

**URL**: `POST /api/refine-content` (Synchronous)

**Input Schema:**
```typescript
interface RefineContentRequest {
  tenant_id: string;
  original_content: {
    text: string;
    image_url?: string;
    platform: string;
  };
  feedback: string;
  iteration: number;
  brand_voice_profile_id?: string;
}
```

**Agent Workflow:**
1. **Planning Agent** (5s): Parse feedback, create refinement strategy
2. **Research Agent** (5-10s): Find similar successful posts with requested attributes
3. **Coder Agent** (5-20s): Generate 2-3 refined options
4. **Reporter Agent** (5s): Explain changes, predict performance impact

**Output Schema:**
```typescript
interface RefineContentResponse {
  refined_options: Array<{
    content: {
      text: string;
      image_suggestions?: string[];
    };
    changes_made: string[];
    brand_voice_score: number;
    predicted_performance: {
      engagement_rate_change: number;
      confidence: number;
    };
  }>;
  feedback_interpretation: string;
}
```

**Example Request:**
```json
{
  "tenant_id": "tenant_123",
  "original_content": {
    "text": "We sell Product Line A. Contact us for pricing.",
    "platform": "linkedin"
  },
  "feedback": "Make it more professional and add a call to action",
  "iteration": 1
}
```

**Example Response:**
```json
{
  "refined_options": [
    {
      "content": {
        "text": "Transform your projects with industry-leading Product Line A. Our solutions deliver exceptional quality and on-time performance that companies trust. Ready to elevate your next project? Get your custom quote today.",
        "image_suggestions": ["Product showcase image", "Customer testimonial"]
      },
      "changes_made": [
        "Increased formality from 40 to 75",
        "Added benefit-focused opening",
        "Included specific CTA: 'Get your custom quote'",
        "Emphasized trust and reliability (brand keywords)"
      ],
      "brand_voice_score": 82,
      "predicted_performance": {
        "engagement_rate_change": 12,
        "confidence": 75
      }
    },
    {
      "content": {
        "text": "Looking for reliable Product Line A? We've helped over 500 companies complete projects on time and on budget. Connect with our team to discuss your requirements.",
        "image_suggestions": ["Company stats infographic", "Product in use"]
      },
      "changes_made": [
        "Increased formality from 40 to 70",
        "Added social proof (500 companies)",
        "Softer CTA: 'Connect with our team'",
        "Question-based opening (engagement tactic)"
      ],
      "brand_voice_score": 79,
      "predicted_performance": {
        "engagement_rate_change": 10,
        "confidence": 72
      }
    }
  ],
  "feedback_interpretation": "User wants: (1) Higher formality/professionalism, (2) Explicit call-to-action. Generated 2 options with different CTA styles."
}
```

**Execution Time:** 15-45 seconds (synchronous, no polling needed)

**Max Iterations:** 5 (prevent infinite loops)

---

### 4.5 Endpoint 4: Competitive Analysis

**Purpose**: Analyze competitor social media presence and identify opportunities

**URL**: `POST /api/competitive-analysis/start`

**Input Schema:**
```typescript
interface CompetitiveAnalysisRequest {
  tenant_id: string;
  competitors: Array<{
    name: string;
    platforms: {
      linkedin?: string;
      twitter?: string;
      facebook?: string;
      instagram?: string;
    };
  }>;
  analysis_period_days: number;
  focus_areas?: string[];
}
```

**Agent Workflow:**
1. **Planning Agent** (30s): Create scraping strategy, define analysis dimensions
2. **Research Agent** (5-20 min):
   - Scrape competitor posts (ethical, respect robots.txt)
   - Parallel execution: 1 agent per competitor
   - Extract content, engagement metrics
3. **Coder Agent** (2-5 min):
   - Analyze patterns
   - Calculate metrics (posting frequency, engagement rates)
   - Identify gaps
4. **Reporter Agent** (1-2 min): Create actionable insights report

**Output Schema:**
```typescript
interface CompetitiveAnalysisResponse {
  competitors_analyzed: number;
  total_posts_analyzed: number;
  analysis_period: {from: string; to: string};
  insights: {
    content_themes: Array<{
      theme: string;
      frequency: number;
      avg_engagement_rate: number;
      opportunity: string;
    }>;
    posting_frequency: {
      by_competitor: Array<{
        name: string;
        posts_per_week: number;
        best_days: string[];
        best_times: string[];
      }>;
      recommendations: string[];
    };
    engagement_tactics: {
      questions_vs_statements: {competitor_avg: number; recommendation: string};
      call_to_action_usage: {competitor_avg: number; recommendation: string};
      visual_content_ratio: {competitor_avg: number; recommendation: string};
    };
  };
  opportunities: Array<{
    category: string;
    description: string;
    priority: "high" | "medium" | "low";
    estimated_impact: string;
  }>;
  content_gaps: string[];
}
```

**Example Response:**
```json
{
  "competitors_analyzed": 3,
  "total_posts_analyzed": 147,
  "analysis_period": {"from": "2025-10-10", "to": "2025-11-10"},
  "insights": {
    "content_themes": [
      {
        "theme": "Customer testimonials",
        "frequency": 12,
        "avg_engagement_rate": 3.4,
        "opportunity": "Competitor A gets 3.2x your engagement on testimonials - consider increasing frequency"
      },
      {
        "theme": "Product launches",
        "frequency": 8,
        "avg_engagement_rate": 2.8,
        "opportunity": "Your launch posts perform 15% better than competitors - maintain this strength"
      }
    ],
    "posting_frequency": {
      "by_competitor": [
        {
          "name": "Competitor A",
          "posts_per_week": 4.2,
          "best_days": ["Tuesday", "Thursday"],
          "best_times": ["9:00 AM", "1:00 PM"]
        }
      ],
      "recommendations": [
        "Competitor A posts 4x per week vs your 2x - consider increasing to 3-4x",
        "Tuesday 9 AM is optimal across all competitors (avg 2.1x baseline engagement)"
      ]
    },
    "engagement_tactics": {
      "questions_vs_statements": {
        "competitor_avg": 35,
        "recommendation": "Competitors use questions in 35% of posts vs your 10% - increase to 25-30%"
      },
      "call_to_action_usage": {
        "competitor_avg": 78,
        "recommendation": "You match competitor CTA usage (80% vs 78%) - maintain current approach"
      },
      "visual_content_ratio": {
        "competitor_avg": 92,
        "recommendation": "You lead with 98% visual content vs 92% competitor avg - maintain this advantage"
      }
    }
  },
  "opportunities": [
    {
      "category": "Video content",
      "description": "Only 12% of competitor posts use video vs 35% industry average. Early mover advantage.",
      "priority": "high",
      "estimated_impact": "Video posts average 4.2x engagement - potential +180% overall engagement"
    },
    {
      "category": "User-generated content",
      "description": "None of the analyzed competitors feature customer photos/videos",
      "priority": "medium",
      "estimated_impact": "UGC posts get 2.8x more trust signals (comments mentioning 'real', 'authentic')"
    }
  ],
  "content_gaps": [
    "Behind-the-scenes content (competitors: 0%, industry avg: 18%)",
    "Employee spotlights (competitors: 2%, industry avg: 12%)",
    "Industry news commentary (competitors: 8%, industry avg: 22%)"
  ]
}
```

**Execution Time:** 10-30 minutes (depends on number of competitors and posts)

**Caching:** Results cached for 24 hours (expensive operation)

**Ethical Considerations:**
- Respect robots.txt
- Rate limit scraping (max 1 request per 2 seconds)
- Public data only (no authentication bypass)
- Attribution when using insights

---

## 5. Agent Coordination Patterns

### 5.1 Communication Patterns

#### Pattern 1: Sequential (Pipeline)
```
Planning → Research → Coder → Reporter
```

**Use Case:** Skill generation (linear dependency)

**Message Flow:**
```typescript
// Step 1: Planning Agent outputs
{
  "from_agent": "planning",
  "to_agent": "research",
  "message_type": "task",
  "payload": {
    "action": "find_similar_posts",
    "patterns_to_match": ["product_image", "benefit_focused", "professional_tone"],
    "min_engagement_rate": 2.0
  }
}

// Step 2: Research Agent outputs
{
  "from_agent": "research",
  "to_agent": "coder",
  "message_type": "result",
  "payload": {
    "similar_posts": [...],
    "patterns_found": {...},
    "success_factors": [...]
  }
}

// Step 3: Coder Agent outputs
{
  "from_agent": "coder",
  "to_agent": "reporter",
  "message_type": "result",
  "payload": {
    "skill_yaml": "...",
    "validation_results": {...}
  }
}
```

---

#### Pattern 2: Parallel + Merge
```
       ┌─→ Research Agent A (competitor 1) ─┐
Planning → Research Agent B (competitor 2) ─┼→ Coder → Reporter
       └─→ Research Agent C (competitor 3) ─┘
```

**Use Case:** Competitive analysis (parallel scraping)

**Coordination:**
```typescript
// Planning Agent spawns 3 parallel research tasks
const tasks = [
  {agent: "research_A", competitor: "competitor_1"},
  {agent: "research_B", competitor: "competitor_2"},
  {agent: "research_C", competitor: "competitor_3"}
];

// Execute in parallel
const results = await Promise.all(
  tasks.map(task => executeAgent(task.agent, task))
);

// Merge results
const merged = {
  competitors_analyzed: 3,
  total_posts: results.reduce((sum, r) => sum + r.posts.length, 0),
  insights: mergeInsights(results)
};
```

---

#### Pattern 3: Iterative (Feedback Loop)
```
Planning → Coder → Validator → (fail) → Coder (retry with feedback)
                              → (pass) → Reporter
```

**Use Case:** Content refinement (multiple iterations)

**Feedback Loop:**
```typescript
let iteration = 0;
let content = original_content;
const max_iterations = 5;

while (iteration < max_iterations) {
  // Coder generates refined content
  content = await coderAgent.refine(content, feedback);

  // Validate against brand voice
  const validation = await validate(content, brand_voice_profile);

  if (validation.score >= target_score) {
    // Success - send to Reporter
    return await reporterAgent.summarize(content, validation);
  }

  // Failure - create feedback for next iteration
  feedback = createFeedback(validation);
  iteration++;
}

throw new Error("Max iterations reached without success");
```

---

### 5.2 Shared Memory Architecture

**Option Selected:** Redis (Recommended for OpenManus)

**Advantages:**
- ✅ Fast (in-memory, <1ms latency)
- ✅ Atomic operations (INCR, MULTI/EXEC)
- ✅ Pub/Sub for real-time updates
- ✅ TTL for automatic cleanup (tasks auto-expire after 7 days)
- ✅ Multiple data structures (strings, hashes, lists, sets, sorted sets)

**Deployment:** Redis cluster (3 nodes, replication factor 2)

**Data Structures:**

```typescript
// Task state
task:{task_id} = {
  status: "queued" | "running" | "completed" | "failed",
  progress: 0-100,
  current_agent: "planning" | "research" | "coder" | "reporter",
  started_at: "2025-11-10T10:30:00Z",
  eta_seconds: 300,
  tenant_id: "tenant_123"
}

// Agent messages queue
agent:messages:{agent_name} = [
  {from: "planning", to: "research", task_id: "...", payload: {...}},
  {from: "research", to: "coder", task_id: "...", payload: {...}}
]

// Artifacts storage
task:{task_id}:artifacts = {
  planning_output: {...},
  research_output: {...},
  coder_output: {...},
  reporter_output: {...}
}

// Progress tracking
task:{task_id}:progress = {
  completed_steps: ["planning", "research"],
  current_step: "coder",
  total_steps: 4,
  step_details: {
    planning: {started: "...", completed: "...", duration_ms: 15234},
    research: {started: "...", completed: "...", duration_ms: 123456}
  }
}
```

**Operations:**

```typescript
// Update task status
await redis.hset(`task:${task_id}`, {
  status: "running",
  progress: 35,
  current_agent: "research",
  eta_seconds: 180
});

// Publish progress update (Pub/Sub)
await redis.publish(`task:${task_id}:progress`, JSON.stringify({
  progress: 35,
  message: "Research Agent analyzing competitor posts..."
}));

// Store artifact
await redis.hset(`task:${task_id}:artifacts`, "research_output", JSON.stringify(result));

// Set expiration (7 days)
await redis.expire(`task:${task_id}`, 7 * 24 * 60 * 60);
```

---

### 5.3 Inter-Agent Communication

**Method Selected:** Message Queue (Redis List + Pub/Sub)

```typescript
class AgentMessageBus {
  constructor(private redis: Redis) {}

  async sendMessage(message: AgentMessage) {
    // Push to target agent's queue
    await this.redis.rpush(
      `agent:messages:${message.to_agent}`,
      JSON.stringify(message)
    );

    // Notify agent (Pub/Sub)
    await this.redis.publish(
      `agent:notify:${message.to_agent}`,
      "new_message"
    );
  }

  async receiveMessages(agent_name: string): Promise<AgentMessage[]> {
    // Pop all messages from queue
    const messages = await this.redis.lrange(`agent:messages:${agent_name}`, 0, -1);
    await this.redis.del(`agent:messages:${agent_name}`);

    return messages.map(m => JSON.parse(m));
  }

  async subscribe(agent_name: string, handler: (msg: AgentMessage) => void) {
    // Subscribe to notifications
    await this.redis.subscribe(`agent:notify:${agent_name}`, () => {
      const messages = await this.receiveMessages(agent_name);
      messages.forEach(handler);
    });
  }
}
```

**Usage:**

```typescript
// Planning Agent sends task to Research Agent
await messageBus.sendMessage({
  from_agent: "planning",
  to_agent: "research",
  task_id: "task_abc123",
  message_type: "task",
  payload: {
    action: "find_similar_posts",
    query: {patterns: [...], min_engagement: 2.0}
  },
  timestamp: new Date().toISOString()
});

// Research Agent listens for messages
messageBus.subscribe("research", async (message) => {
  if (message.message_type === "task") {
    const result = await performResearch(message.payload);

    // Send result to next agent
    await messageBus.sendMessage({
      from_agent: "research",
      to_agent: "coder",
      task_id: message.task_id,
      message_type: "result",
      payload: result,
      timestamp: new Date().toISOString()
    });
  }
});
```

---

### 5.4 Error Handling & Recovery

**Retry Logic:**

```typescript
async function executeAgentWithRetry(
  agent: Agent,
  task: AgentTask,
  maxRetries: number = 3
): Promise<AgentResult> {
  let lastError: Error;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await agent.execute(task);
    } catch (error) {
      lastError = error;

      // Log error
      logger.error(`Agent ${agent.name} failed attempt ${attempt}/${maxRetries}`, {
        task_id: task.task_id,
        error: error.message
      });

      // Exponential backoff
      const delay = Math.min(Math.pow(2, attempt) * 1000, 30000); // Max 30s
      await sleep(delay);

      // Adaptive retry: Modify task based on error type
      if (error.message.includes("rate limit")) {
        task.batch_size = Math.floor(task.batch_size / 2);
        logger.info(`Reduced batch size to ${task.batch_size} due to rate limit`);
      }

      if (error.message.includes("timeout")) {
        task.timeout_ms *= 1.5;
        logger.info(`Increased timeout to ${task.timeout_ms}ms`);
      }
    }
  }

  // All retries failed
  throw new TaskExecutionError(
    `Agent ${agent.name} failed after ${maxRetries} attempts`,
    {task_id: task.task_id, last_error: lastError}
  );
}
```

**Partial Results:**

```typescript
// If Research Agent fails to scrape 1 of 3 competitors, continue with 2
interface PartialResult {
  success: boolean;
  completed_items: number;
  failed_items: number;
  partial_data: any[];
  errors: Array<{item: string; error: Error}>;
}

async function scrapeCompetitors(competitors: string[]): Promise<PartialResult> {
  const results = await Promise.allSettled(
    competitors.map(c => scrapeCompetitor(c))
  );

  const successful = results.filter(r => r.status === "fulfilled");
  const failed = results.filter(r => r.status === "rejected");

  if (successful.length === 0) {
    throw new Error("All competitors failed to scrape");
  }

  return {
    success: true,
    completed_items: successful.length,
    failed_items: failed.length,
    partial_data: successful.map(r => r.value),
    errors: failed.map((r, i) => ({
      item: competitors[i],
      error: r.reason
    }))
  };
}
```

---

## 6. Dify Integration

### 6.1 HTTP Tool Configuration in Dify

**Dify HTTP Tool Node Setup:**

```json
{
  "node_type": "http_request",
  "name": "Call OpenManus Generate Skill",
  "config": {
    "method": "POST",
    "url": "https://openmanus-api.workers.dev/api/generate-skill/start",
    "headers": {
      "Authorization": "Bearer {{env.OPENMANUS_API_KEY}}",
      "Content-Type": "application/json"
    },
    "body": {
      "tenant_id": "{{tenant_id}}",
      "examples": "{{skill_examples}}",
      "skill_name": "{{skill_name}}",
      "target_platforms": ["linkedin", "facebook"]
    },
    "timeout": 10000,
    "variables": {
      "task_id": "$.task_id",
      "status": "$.status"
    }
  }
}
```

**Environment Variables in Dify:**
```
OPENMANUS_API_KEY=sk_openmanus_abc123xyz...
OPENMANUS_BASE_URL=https://openmanus-api.workers.dev
```

---

### 6.2 Async Polling Workflow in Dify

**Dify Workflow Example: Generate Skill with Progress Updates**

```
┌─────────────────────────────────────────────┐
│ 1. Start Node (Trigger)                    │
│    Input: user_message, skill_examples     │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ 2. HTTP POST - Start Task                  │
│    URL: /api/generate-skill/start          │
│    Output: task_id, status                 │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ 3. Answer Node - Notify User               │
│    "✅ Started skill generation (Task ID:  │
│     {{task_id}}). This will take 5-10 min."│
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ 4. Loop Node (Polling)                     │
│    Max iterations: 60 (10 min with 10s)   │
│    │                                        │
│    ├─> 5. Wait 10 seconds                 │
│    │                                        │
│    ├─> 6. HTTP GET - Poll Status          │
│    │    URL: /api/generate-skill/status/  │
│    │         {{task_id}}                   │
│    │    Output: status, progress, eta,    │
│    │            current_step              │
│    │                                        │
│    ├─> 7. Answer Node - Progress Update   │
│    │    "⏳ {{progress}}% complete...     │
│    │     {{current_step}}                 │
│    │     ETA: {{eta}} seconds"            │
│    │                                        │
│    └─> 8. Condition Node                  │
│         IF status == "completed":          │
│            Break loop                      │
│         ELSE IF status == "failed":        │
│            Break loop                      │
│         ELSE:                              │
│            Continue loop                   │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ 9. Condition Node - Check Final Status     │
│    IF status == "completed":               │
│       Go to step 10                        │
│    ELSE:                                   │
│       Go to step 12 (Error handler)       │
└──────────────────┬──────────────────────────┘
                   │ (completed)
                   ▼
┌─────────────────────────────────────────────┐
│ 10. HTTP GET - Fetch Result                │
│     URL: /api/generate-skill/result/       │
│          {{task_id}}                       │
│     Output: skill_yaml, confidence_score,  │
│             patterns, validation           │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ 11. Answer Node - Present Results          │
│     "✅ Skill generated successfully!      │
│                                             │
│     **Confidence:** {{confidence_score}}%  │
│     **Patterns Found:**                    │
│     {{#each patterns}}                     │
│       - {{this}}                           │
│     {{/each}}                              │
│                                             │
│     Would you like to:                     │
│     A) Deploy this skill                   │
│     B) Refine it further                   │
│     C) Generate another variation"         │
└─────────────────────────────────────────────┘

(Error path from step 9)
                   │ (failed/timeout)
                   ▼
┌─────────────────────────────────────────────┐
│ 12. Answer Node - Error Handler            │
│     "❌ Skill generation failed or timed   │
│     out after 10 minutes. Please try:      │
│     - Reducing number of examples          │
│     - Checking example quality             │
│     - Contacting support if issue persists"│
└─────────────────────────────────────────────┘
```

**Key Configuration:**
- **Timeout per request:** 10s (HTTP tool timeout)
- **Polling interval:** 10s (Wait node)
- **Max iterations:** 60 (10 minutes total)
- **Progress updates:** Every poll (Answer node shows progress%)

---

### 6.3 Synchronous Endpoint (Refine Content)

**Simple HTTP Request (No Polling):**

```
┌────────────────────────────────────────────┐
│ 1. HTTP POST - Refine Content             │
│    URL: /api/refine-content                │
│    Body: {                                 │
│      tenant_id: "...",                     │
│      original_content: {...},             │
│      feedback: "{{user_feedback}}"        │
│    }                                       │
│    Timeout: 60000 (60s)                   │
│    Output: refined_options                │
└──────────────────┬───────────────────────────┘
                   │
                   ▼
┌────────────────────────────────────────────┐
│ 2. Answer Node - Present Options          │
│    "Here are 2 refined versions:          │
│                                            │
│     **Option A:**                         │
│     {{refined_options[0].content.text}}   │
│     Changes: {{refined_options[0]         │
│                .changes_made}}            │
│     Brand voice: {{refined_options[0]     │
│                   .brand_voice_score}}/100│
│                                            │
│     **Option B:**                         │
│     {{refined_options[1].content.text}}   │
│     Changes: {{refined_options[1]         │
│                .changes_made}}            │
│     Brand voice: {{refined_options[1]     │
│                   .brand_voice_score}}/100│
│                                            │
│     Which do you prefer? (Type A or B)"   │
└────────────────────────────────────────────┘
```

**Timeout Handling:**
- If request takes >60s: Return partial results if available
- Show error message if complete failure
- Allow user to retry with adjusted parameters

---

### 6.4 Error Handling in Dify Workflows

**HTTP Request Error Handling:**

```json
{
  "node_type": "http_request",
  "error_handling": {
    "on_timeout": {
      "action": "continue",
      "variables": {
        "error_message": "Request timed out after 60 seconds"
      }
    },
    "on_http_error": {
      "4xx": {
        "action": "continue",
        "variables": {
          "error_message": "Client error: {{response.status}} - {{response.body.error}}"
        }
      },
      "5xx": {
        "action": "retry",
        "max_retries": 2,
        "retry_delay": 5000,
        "on_final_failure": {
          "action": "continue",
          "variables": {
            "error_message": "Server error after 2 retries"
          }
        }
      }
    }
  }
}
```

**Error Response Pattern:**

```typescript
// OpenManus API error response format
{
  "success": false,
  "error": {
    "code": "TASK_FAILED",
    "message": "Agent execution failed: Rate limit exceeded",
    "details": {
      "agent": "research",
      "reason": "OpenAI rate limit: 10,000 tokens/min exceeded",
      "retry_after": 60
    }
  }
}
```

**Dify Condition Node for Error Handling:**

```
IF {{http_response.success}} == false:
  Answer Node: "❌ Error: {{http_response.error.message}}"
ELSE:
  Continue to success path
```

---

## 7. Async Task Handling

### 7.1 Task Queue Architecture (Cloudflare Durable Objects)

**Why Durable Objects:**
- ✅ Strongly consistent (single-writer guarantee)
- ✅ No external dependencies (Redis alternative)
- ✅ Built-in persistence (automatic state management)
- ✅ Low latency (<10ms typical)
- ✅ Cost-effective ($0.02 per million requests)

**Durable Object Implementation:**

```typescript
export class TaskQueueDO {
  state: DurableObjectState;
  tasks: Map<string, TaskState>;

  constructor(state: DurableObjectState) {
    this.state = state;
    this.tasks = new Map();
  }

  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);
    const path = url.pathname;

    // Create task
    if (path.endsWith("/start") && request.method === "POST") {
      const body = await request.json();
      const task_id = crypto.randomUUID();

      const taskState: TaskState = {
        task_id,
        tenant_id: body.tenant_id,
        status: "queued",
        progress: 0,
        created_at: new Date().toISOString(),
        endpoint: body.endpoint,
        inputs: body.inputs
      };

      this.tasks.set(task_id, taskState);
      await this.state.storage.put(`task:${task_id}`, taskState);

      // Queue task for execution
      await this.queueExecution(task_id);

      return new Response(JSON.stringify({task_id, status: "queued"}), {
        headers: {"Content-Type": "application/json"}
      });
    }

    // Get status
    if (path.includes("/status/") && request.method === "GET") {
      const task_id = path.split("/status/")[1];
      const taskState = this.tasks.get(task_id) || await this.state.storage.get(`task:${task_id}`);

      if (!taskState) {
        return new Response(JSON.stringify({error: "Task not found"}), {status: 404});
      }

      return new Response(JSON.stringify(taskState), {
        headers: {"Content-Type": "application/json"}
      });
    }

    // Get result
    if (path.includes("/result/") && request.method === "GET") {
      const task_id = path.split("/result/")[1];
      const result = await this.state.storage.get(`result:${task_id}`);

      if (!result) {
        return new Response(JSON.stringify({error: "Result not found or task not complete"}), {status: 404});
      }

      return new Response(JSON.stringify(result), {
        headers: {"Content-Type": "application/json"}
      });
    }

    return new Response("Not found", {status: 404});
  }

  async queueExecution(task_id: string) {
    // Trigger OpenManus agent execution (separate Worker)
    await fetch("https://openmanus-agents.workers.dev/execute", {
      method: "POST",
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({task_id})
    });
  }

  async updateProgress(task_id: string, progress: number, current_step: string, eta: number) {
    const taskState = this.tasks.get(task_id);
    if (taskState) {
      taskState.progress = progress;
      taskState.current_step = current_step;
      taskState.eta_seconds = eta;
      taskState.updated_at = new Date().toISOString();

      this.tasks.set(task_id, taskState);
      await this.state.storage.put(`task:${task_id}`, taskState);
    }
  }

  async storeResult(task_id: string, result: any) {
    const taskState = this.tasks.get(task_id);
    if (taskState) {
      taskState.status = "completed";
      taskState.progress = 100;
      taskState.completed_at = new Date().toISOString();

      this.tasks.set(task_id, taskState);
      await this.state.storage.put(`task:${task_id}`, taskState);
      await this.state.storage.put(`result:${task_id}`, result);

      // Set TTL (7 days)
      await this.state.storage.setAlarm(Date.now() + 7 * 24 * 60 * 60 * 1000);
    }
  }

  async alarm() {
    // Cleanup expired tasks
    const now = Date.now();
    const expiredKeys: string[] = [];

    for (const [key, value] of await this.state.storage.list<TaskState>()) {
      if (value.completed_at) {
        const completed = new Date(value.completed_at).getTime();
        if (now - completed > 7 * 24 * 60 * 60 * 1000) {
          expiredKeys.push(key);
        }
      }
    }

    await this.state.storage.delete(expiredKeys);
  }
}
```

**Worker Binding:**

```toml
# wrangler.toml
[[durable_objects.bindings]]
name = "TASK_QUEUE"
class_name = "TaskQueueDO"
script_name = "openmanus-api"
```

---

### 7.2 Progress Tracking with ETA

**ETA Calculation:**

```typescript
interface ProgressUpdate {
  task_id: string;
  completed_steps: string[];
  current_step: string;
  total_steps: number;
  step_durations: Record<string, number>; // milliseconds
}

function calculateETA(progress: ProgressUpdate): number {
  const completed_time = progress.completed_steps.reduce(
    (sum, step) => sum + (progress.step_durations[step] || 0),
    0
  );

  const avg_step_time = completed_time / progress.completed_steps.length;
  const remaining_steps = progress.total_steps - progress.completed_steps.length;

  // ETA = avg time per step * remaining steps
  const eta_ms = avg_step_time * remaining_steps;

  // Add current step estimated remaining time
  const current_step_elapsed = Date.now() - progress.step_durations[progress.current_step + "_start"];
  const current_step_eta = Math.max(0, avg_step_time - current_step_elapsed);

  return Math.ceil((eta_ms + current_step_eta) / 1000); // seconds
}
```

**Progress Update Example:**

```json
{
  "task_id": "task_abc123",
  "status": "running",
  "progress": 65,
  "current_step": "Coder Agent generating YAML skill file",
  "eta_seconds": 120,
  "completed_steps": [
    {"step": "Planning Agent", "duration_ms": 15234},
    {"step": "Research Agent", "duration_ms": 123456}
  ],
  "total_steps": 4,
  "updated_at": "2025-11-10T10:35:42Z"
}
```

---

### 7.3 Webhook Alternative (Optional)

**Webhook Pattern** (if Dify adds webhook support in future):

```
1. Dify starts task:
   POST /api/generate-skill/start
   Body: {
     tenant_id: "...",
     examples: [...],
     callback_url: "https://dify-webhook.example.com/task-complete"
   }
   Response: {task_id: "..."}

2. OpenManus processes task (5-10 min)

3. OpenManus calls webhook when complete:
   POST https://dify-webhook.example.com/task-complete
   Body: {
     task_id: "...",
     status: "completed",
     result: {...}
   }

4. Dify receives webhook, resumes workflow
```

**Current Status:** Polling pattern used (Dify doesn't support webhooks as of 2025)

---

## 8. Deployment Architecture

### 8.1 Fly.io Deployment Design

**Platform:** Fly.io (container orchestration platform)

**Machine Specifications:**
- **Planning Agent:** 2 machines, shared-cpu-2x (2 vCPU, 4GB RAM each)
- **Research Agent:** 2 machines, shared-cpu-2x (2 vCPU, 4GB RAM each)
- **Coder Agent:** 2 machines, shared-cpu-2x (2 vCPU, 4GB RAM each)
- **Reporter Agent:** 1 machine, shared-cpu-1x (1 vCPU, 2GB RAM)
- **API Gateway:** 2 machines, shared-cpu-1x (1 vCPU, 2GB RAM each)
- **Total Resources:** 13 vCPU, 30 GB RAM across 9 machines
- **Auto-scaling:** Manual replica scaling via `fly scale count`
- **Regions:** Primary (iad - US East), Secondary (fra - EU)

**Configuration (fly.toml):**
```toml
app = "openmanus-prod"
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  AGENT_ENV = "production"
  LOG_LEVEL = "info"
```

---

### 8.2 Fly.io App Configuration

**Planning Agent (fly.toml):**

```toml
app = "openmanus-planning-agent"
primary_region = "iad"

[build]
  dockerfile = "Dockerfile.planning"

[env]
  AGENT_TYPE = "planning"
  AGENT_ENV = "production"
  PORT = "8080"

[[services]]
  internal_port = 8080
  protocol = "tcp"

  [[services.ports]]
    port = 80
    handlers = ["http"]

  [[services.ports]]
    port = 443
    handlers = ["tls", "http"]

  [[services.http_checks]]
    interval = "10s"
    timeout = "2s"
    grace_period = "5s"
    method = "GET"
    path = "/health"

[[vm]]
  size = "shared-cpu-2x"
  memory = "4gb"

[processes]
  app = "python -m openmanus.agents.planning"

[[services.concurrency]]
  type = "connections"
  hard_limit = 100
  soft_limit = 80
```

**Similar configurations for Research, Coder, Reporter agents** (each with their own app name and AGENT_TYPE)

---

**Redis (Upstash Managed Service):**

Fly.io best practice: Use Upstash Redis (managed) instead of self-hosted.

**Configuration:**
```bash
# Create Upstash Redis cluster
fly redis create openmanus-redis --plan pro-2k

# Connection URL will be auto-injected as secret
# REDIS_URL=redis://:password@fly-openmanus-redis.upstash.io:6379
```

**Upstash Redis Specifications:**
- **Plan:** Pro 2K ($30/month)
- **Memory:** 2GB RAM
- **Connections:** 2,000 concurrent
- **TLS:** Enabled by default
- **Persistence:** Automatic snapshots
- **Regions:** Global edge replication
- **High Availability:** Built-in (no manual cluster setup needed)

---

**API Gateway (fly.toml):**

```toml
app = "openmanus-api-gateway"
primary_region = "iad"

[build]
  dockerfile = "Dockerfile.gateway"

[env]
  PORT = "8080"

[[services]]
  internal_port = 8080
  protocol = "tcp"
  auto_stop_machines = false
  auto_start_machines = true

  [[services.ports]]
    port = 443
    handlers = ["tls", "http"]

  [services.concurrency]
    type = "requests"
    hard_limit = 250
    soft_limit = 200

[[vm]]
  size = "shared-cpu-1x"
  memory = "2gb"
```

**DNS & SSL:**
- Fly.io provides automatic SSL certificates (Let's Encrypt)
- Default URL: `openmanus-api-gateway.fly.dev`
- Custom domain: Configure via `fly certs add openmanus-api.yourdomain.com`

---

### 8.3 Secrets Management

**Fly.io Secrets (encrypted at rest, injected as environment variables):**

```bash
# Set secrets for each agent app
fly secrets set -a openmanus-planning-agent \
  REDIS_URL="redis://:password@fly-openmanus-redis.upstash.io:6379" \
  OPENAI_API_KEY="sk-..." \
  CLAUDE_API_KEY="sk-ant-..." \
  STAGE_C_VECTORIZE_API_KEY="..." \
  STAGE_D_GATEWAY_API_KEY="..."

# Repeat for: research-agent, coder-agent, reporter-agent, api-gateway
```

**Secrets Format:**
- Stored encrypted at rest by Fly.io
- Auto-injected as environment variables at runtime
- Accessible via `os.getenv("OPENAI_API_KEY")`
- No plaintext storage in code or config files

**Encryption:** Fly.io vault (AES-256 encryption)

**Rotation:** API keys rotated every 90 days (automated via CI/CD)
- Use `fly secrets set` to update
- Zero-downtime rotation (new secrets take effect on next deploy)

---

### 8.4 Monitoring & Logging

**Fly.io Built-in Monitoring:**
- Dashboard: `fly dashboard -a openmanus-planning-agent`
- Logs: `fly logs -a openmanus-planning-agent`
- Metrics: CPU, memory, network, HTTP requests/errors
- Alerts: Configure via Fly.io dashboard

**Datadog Integration (Python apps):**

```python
# Install in Dockerfile
RUN pip install ddtrace

# Run with Datadog tracing
CMD ["ddtrace-run", "python", "-m", "openmanus.agents.planning"]
```

**Set Datadog API key:**
```bash
fly secrets set -a openmanus-planning-agent DD_API_KEY="..."
fly secrets set -a openmanus-planning-agent DD_ENV="production"
fly secrets set -a openmanus-planning-agent DD_SERVICE="openmanus-planning"
```

**Metrics Tracked:**
- Task completion rate (%)
- Average task execution time (seconds)
- Agent utilization (% of time busy)
- LLM API call latency (ms)
- Error rate by agent type (%)
- Cost per task ($)
- Fly.io machine health (CPU, memory, restarts)

---

## 9. Security & Authentication

### 9.1 API Key Management

**API Key Format:**
```
sk_openmanus_<environment>_<tenant_id>_<random_32_chars>

Example:
sk_openmanus_prod_tenant123_a7f8d9e2b3c4f5a6e7d8c9b0a1f2e3d4
```

**Storage:**
- Dify environment variables (encrypted)
- Cloudflare Workers secrets
- Fly.io secrets (AES-256 encrypted vault)

**Rotation Schedule:**
- Production keys: Every 90 days
- Development keys: Every 180 days
- Automatic rotation via CI/CD pipeline

---

### 9.2 Request Authentication

**Bearer Token Validation:**

```typescript
async function authenticateRequest(request: Request, env: Env): Promise<{tenant_id: string} | null> {
  const auth_header = request.headers.get("Authorization");

  if (!auth_header || !auth_header.startsWith("Bearer ")) {
    return null;
  }

  const api_key = auth_header.substring(7);

  // Validate format
  if (!api_key.startsWith("sk_openmanus_")) {
    return null;
  }

  // Extract tenant ID from key
  const parts = api_key.split("_");
  if (parts.length < 4) {
    return null;
  }

  const tenant_id = parts[3];

  // Verify key in database
  const valid = await env.D1.prepare(
    "SELECT tenant_id FROM api_keys WHERE key_hash = ? AND revoked_at IS NULL"
  ).bind(await hashApiKey(api_key)).first();

  if (!valid) {
    return null;
  }

  return {tenant_id};
}

async function hashApiKey(key: string): Promise<string> {
  const encoder = new TextEncoder();
  const data = encoder.encode(key);
  const hash = await crypto.subtle.digest("SHA-256", data);
  return Array.from(new Uint8Array(hash))
    .map(b => b.toString(16).padStart(2, "0"))
    .join("");
}
```

**Usage in Worker:**

```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const auth = await authenticateRequest(request, env);

    if (!auth) {
      return new Response(JSON.stringify({
        error: "Unauthorized",
        message: "Invalid or missing API key"
      }), {
        status: 401,
        headers: {"Content-Type": "application/json"}
      });
    }

    // Process request with auth.tenant_id
    const result = await handleRequest(request, auth.tenant_id, env);
    return new Response(JSON.stringify(result));
  }
};
```

---

### 9.3 Rate Limiting

**Rate Limit Rules:**

| Endpoint Type | Limit | Window | Burst |
|---------------|-------|--------|-------|
| Async (generate-skill, analyze-brand-voice, competitive-analysis) | 100 requests | 1 hour | 10 |
| Sync (refine-content) | 1000 requests | 1 hour | 50 |
| Status polling | 10000 requests | 1 hour | 100 |

**Implementation (Token Bucket Algorithm):**

```typescript
interface RateLimitBucket {
  tokens: number;
  last_refill: number;
  max_tokens: number;
  refill_rate: number; // tokens per second
}

async function checkRateLimit(
  tenant_id: string,
  endpoint_type: "async" | "sync" | "status",
  env: Env
): Promise<{allowed: boolean; retry_after?: number}> {
  const key = `ratelimit:${tenant_id}:${endpoint_type}`;

  const limits = {
    async: {max: 100, refill: 100 / 3600}, // 100 per hour = 0.028/sec
    sync: {max: 1000, refill: 1000 / 3600},
    status: {max: 10000, refill: 10000 / 3600}
  };

  const config = limits[endpoint_type];

  // Get current bucket state
  let bucket: RateLimitBucket = await env.KV.get(key, "json") || {
    tokens: config.max,
    last_refill: Date.now(),
    max_tokens: config.max,
    refill_rate: config.refill
  };

  // Refill tokens based on time elapsed
  const now = Date.now();
  const elapsed_sec = (now - bucket.last_refill) / 1000;
  const new_tokens = Math.min(
    bucket.max_tokens,
    bucket.tokens + elapsed_sec * bucket.refill_rate
  );

  bucket.tokens = new_tokens;
  bucket.last_refill = now;

  // Check if request allowed
  if (bucket.tokens >= 1) {
    bucket.tokens -= 1;
    await env.KV.put(key, JSON.stringify(bucket), {expirationTtl: 3600});
    return {allowed: true};
  } else {
    // Calculate retry_after
    const tokens_needed = 1 - bucket.tokens;
    const retry_after = Math.ceil(tokens_needed / bucket.refill_rate);
    return {allowed: false, retry_after};
  }
}
```

**429 Response:**

```json
{
  "error": "Rate limit exceeded",
  "message": "You have exceeded the rate limit for async endpoints (100 req/hour)",
  "retry_after": 120,
  "limit": 100,
  "window": 3600,
  "remaining": 0
}
```

---

### 9.4 Input Validation & Sanitization

**Request Validation:**

```typescript
import { z } from "zod";

const GenerateSkillRequestSchema = z.object({
  tenant_id: z.string().regex(/^tenant_[a-z0-9]{8}$/),
  examples: z.array(z.object({
    platform: z.enum(["linkedin", "twitter", "facebook", "instagram"]),
    content: z.string().min(10).max(5000),
    image_url: z.string().url().optional(),
    performance_metrics: z.object({
      likes: z.number().int().nonnegative(),
      comments: z.number().int().nonnegative(),
      shares: z.number().int().nonnegative(),
      impressions: z.number().int().positive()
    })
  })).min(1).max(10),
  skill_name: z.string().min(3).max(100).optional(),
  target_platforms: z.array(z.string()).min(1).max(4)
});

async function validateRequest(body: any): Promise<{valid: boolean; errors?: string[]}> {
  try {
    GenerateSkillRequestSchema.parse(body);
    return {valid: true};
  } catch (error) {
    if (error instanceof z.ZodError) {
      return {
        valid: false,
        errors: error.errors.map(e => `${e.path.join(".")}: ${e.message}`)
      };
    }
    return {valid: false, errors: ["Unknown validation error"]};
  }
}
```

**XSS Prevention:**
- All user-provided content sanitized before storage
- HTML entities escaped in responses
- No direct HTML rendering (JSON responses only)

**SQL Injection Prevention:**
- Prepared statements only (no string concatenation)
- D1 bindings used for all queries

---

## 10. Monitoring & Observability

### 10.1 Metrics

**Key Performance Indicators (KPIs):**

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| Task Success Rate | >95% | <90% |
| Avg Task Execution Time | <6 min (async) | >10 min |
| Avg Refine Content Time | <30s (sync) | >45s |
| P95 API Latency | <100ms (status poll) | >200ms |
| Agent Utilization | 40-70% | >85% (overload) or <20% (underutilized) |
| Error Rate | <2% | >5% |
| LLM API Latency (GPT-4) | <3s | >10s |
| Cost Per Task | <$0.50 | >$1.00 |

**Datadog Dashboard Example:**

```typescript
// Custom metrics sent to Datadog
import { metrics } from "@datadog/browser-rum";

// Task completion
metrics.increment("openmanus.task.completed", 1, {
  tags: ["endpoint:generate_skill", "tenant:tenant_123", "status:success"]
});

// Task execution time
metrics.distribution("openmanus.task.duration", task_duration_ms, {
  tags: ["endpoint:generate_skill", "agent:planning"]
});

// LLM API call
metrics.distribution("openmanus.llm.latency", llm_latency_ms, {
  tags: ["provider:openai", "model:gpt-4"]
});

// Cost tracking
metrics.increment("openmanus.cost.tokens", tokens_used, {
  tags: ["provider:openai", "model:gpt-4", "cost_usd": (tokens_used * 0.00003).toString()]
});
```

---

### 10.2 Logging

**Structured Logging:**

```typescript
interface LogEntry {
  timestamp: string;
  level: "debug" | "info" | "warn" | "error";
  message: string;
  task_id?: string;
  tenant_id?: string;
  agent?: string;
  duration_ms?: number;
  error?: {
    message: string;
    stack?: string;
    code?: string;
  };
  metadata?: Record<string, any>;
}

function log(entry: Omit<LogEntry, "timestamp">): void {
  const logEntry: LogEntry = {
    timestamp: new Date().toISOString(),
    ...entry
  };

  console.log(JSON.stringify(logEntry));

  // Send to Datadog
  if (entry.level === "error" || entry.level === "warn") {
    datadogLogs.logger.log(logEntry.message, logEntry.level, {
      ...logEntry.metadata,
      task_id: logEntry.task_id,
      tenant_id: logEntry.tenant_id
    });
  }
}

// Usage
log({
  level: "info",
  message: "Planning Agent completed task decomposition",
  task_id: "task_abc123",
  tenant_id: "tenant_123",
  agent: "planning",
  duration_ms: 15234,
  metadata: {
    subtasks_created: 3,
    estimated_time_sec: 330
  }
});
```

**Log Retention:**
- Info logs: 7 days
- Warn/Error logs: 30 days
- Audit logs (API calls): 90 days

---

### 10.3 Alerting

**PagerDuty Integration:**

```yaml
# Alert Rules
alerts:
  - name: "High Error Rate"
    condition: "error_rate > 5%"
    window: "5 minutes"
    severity: "critical"
    notification: "pagerduty"

  - name: "Task Queue Backlog"
    condition: "queued_tasks > 50"
    window: "10 minutes"
    severity: "warning"
    notification: "slack"

  - name: "LLM API Timeout"
    condition: "llm_timeout_rate > 10%"
    window: "5 minutes"
    severity: "high"
    notification: "pagerduty"

  - name: "Agent Down"
    condition: "agent_health_check_fails >= 3"
    window: "1 minute"
    severity: "critical"
    notification: "pagerduty + slack"
```

**Slack Notifications:**

```typescript
async function sendSlackAlert(alert: Alert): Promise<void> {
  await fetch(SLACK_WEBHOOK_URL, {
    method: "POST",
    headers: {"Content-Type": "application/json"},
    body: JSON.stringify({
      text: `:rotating_light: *${alert.name}* (${alert.severity})`,
      blocks: [
        {
          type: "section",
          text: {
            type: "mrkdwn",
            text: `*Alert:* ${alert.name}\n*Severity:* ${alert.severity}\n*Condition:* ${alert.condition}\n*Time:* ${alert.timestamp}`
          }
        },
        {
          type: "actions",
          elements: [
            {
              type: "button",
              text: {type: "plain_text", text: "View Dashboard"},
              url: `https://app.datadoghq.com/dashboard/${DASHBOARD_ID}`
            },
            {
              type: "button",
              text: {type: "plain_text", text: "Acknowledge"},
              action_id: "ack_alert"
            }
          ]
        }
      ]
    })
  });
}
```

---

### 10.4 Tracing (Distributed Tracing)

**OpenTelemetry Integration:**

```typescript
import { trace } from "@opentelemetry/api";

const tracer = trace.getTracer("openmanus-api");

async function executeTask(task_id: string) {
  const span = tracer.startSpan("execute_task", {
    attributes: {
      "task.id": task_id,
      "task.endpoint": "generate_skill"
    }
  });

  try {
    // Planning Agent
    const planSpan = tracer.startSpan("planning_agent", {parent: span});
    const plan = await planningAgent.execute(task_id);
    planSpan.end();

    // Research Agent
    const researchSpan = tracer.startSpan("research_agent", {parent: span});
    const research = await researchAgent.execute(task_id, plan);
    researchSpan.end();

    // Coder Agent
    const coderSpan = tracer.startSpan("coder_agent", {parent: span});
    const code = await coderAgent.execute(task_id, research);
    coderSpan.end();

    // Reporter Agent
    const reportSpan = tracer.startSpan("reporter_agent", {parent: span});
    const report = await reporterAgent.execute(task_id, code);
    reportSpan.end();

    span.setStatus({code: SpanStatusCode.OK});
    return report;
  } catch (error) {
    span.recordException(error);
    span.setStatus({code: SpanStatusCode.ERROR, message: error.message});
    throw error;
  } finally {
    span.end();
  }
}
```

**Trace Visualization in Datadog:**
- See complete request flow from Dify → OpenManus → Agents → LLM APIs
- Identify bottlenecks (which agent/step takes longest)
- Debug errors (see exact failure point)

---

## 11. Cost Analysis

### 11.1 Development Costs

**Total: $105,000 - $180,000**

See [stage-h-cost-timeline-analysis.md](./stage-h-cost-timeline-analysis.md) for detailed breakdown.

**Summary:**
- Phase 1: OpenManus Deployment ($15K-$25.5K)
- Phase 2: Agent Development ($30K-$48K)
- Phase 3: API Endpoints ($30K-$45K)
- Phase 4: Dify Integration ($15K-$30K)
- Phase 5: Testing & QA ($15K-$30K)

**Labor:** 280-480 hours @ $375/hour

---

### 11.2 Operational Costs (Monthly)

**Infrastructure:**
| Component | Specification | Monthly Cost |
|-----------|--------------|--------------|
| **Fly.io Machines** | 9 machines (7× shared-cpu-2x, 2× shared-cpu-1x) | $150 |
| **Upstash Redis** | Pro 2K plan (2GB, 2K connections) | $30 |
| **Cloudflare Workers** (API Gateway) | Unlimited plan | $5 |
| **Durable Objects** (Task Queue) | ~100K requests/month | $2 |
| **Fly.io SSL** | Automatic Let's Encrypt | $0 |
| **Fly.io Volumes** | 10GB persistent storage | $2 |
| **Subtotal Infrastructure** | | **$189** |

**LLM API Costs** (based on 1000 tasks/month):
| Model | Usage | Tokens/Month | Cost |
|-------|-------|--------------|------|
| **GPT-4 Turbo** | Planning, Coder, Reporter | 500K input + 200K output | $7.50 |
| **GPT-4o-mini** | Research, Refine | 1M input + 300K output | $0.18 |
| **Claude Sonnet 4.5** | Complex reasoning (10% of tasks) | 50K input + 20K output | $0.27 |
| **Subtotal LLM** | | | **$7.95** |

**Supporting Services:**
| Service | Cost |
|---------|------|
| Datadog (monitoring) | $15 |
| PagerDuty (alerting) | $0 (free tier) |
| **Subtotal Supporting** | **$15** |

**Grand Total: $212/month** ($189 infra + $8 LLM + $15 monitoring)

**Per Customer Cost** (100 customers): **$2.12/customer/month**

---

### 11.3 Cost Optimization Strategies

1. **Multi-Model Routing** (60-80% savings on LLM costs):
   - Use GPT-4o-mini for simple tasks (10x cheaper)
   - Reserve GPT-4 for complex reasoning
   - Current: $7.95/month → Optimized: $2.50/month

2. **Auto-stop Machines** (20-30% savings on compute):
   - Enable auto-stop for idle machines
   - Fly.io stops machines after 5 minutes of no requests
   - Current: $150/month → Optimized: $110/month

3. **Caching** (reduce duplicate work):
   - Cache competitor analysis (24h TTL)
   - Cache brand voice analysis (7 day TTL)
   - Estimated: 20% reduction in LLM calls

**Optimized Total: ~$160/month ($1.60/customer/month)**
**Savings: $52/month (25% reduction)**

---

## 12. Implementation Timeline

**Total: 12 weeks (3 months)**

See [stage-h-cost-timeline-analysis.md](./stage-h-cost-timeline-analysis.md) for detailed Gantt chart.

**Summary:**

### Phase 1: Infrastructure Setup (Weeks 1-2)
- Fly.io apps creation (5 agent apps + API gateway)
- Upstash Redis provisioning (`fly redis create`)
- Cloudflare Workers setup (task queue)
- Secrets management (`fly secrets set`)
- Monitoring & logging (Datadog, Fly.io dashboard)

**Deliverable:** Deployed infrastructure, ready for agents

---

### Phase 2: Agent Development (Weeks 3-5)
- Planning Agent implementation
- Research Agent implementation
- Coder Agent implementation
- Reporter Agent implementation
- Inter-agent communication (Redis message bus)
- Shared memory system

**Deliverable:** 4 functional agents with coordination

---

### Phase 3: API Endpoints (Weeks 6-8)
- `/api/generate-skill` (async, 5-10 min)
- `/api/analyze-brand-voice` (async, 3-5 min)
- `/api/refine-content` (sync, 15-45s)
- `/api/competitive-analysis` (async, 10-30 min)
- Task queue (Durable Objects)
- Auth & rate limiting

**Deliverable:** 4 REST endpoints, fully functional

---

### Phase 4: Dify Integration (Weeks 9-10)
- Dify HTTP tool configurations
- Async polling workflows
- Progress update handling
- Error handling workflows
- Testing with Dify frontend

**Deliverable:** Dify workflows calling OpenManus successfully

---

### Phase 5: Testing & Refinement (Weeks 11-12)
- Unit tests (80%+ coverage)
- Integration tests (E2E scenarios)
- Performance tests (load, stress)
- Security audit
- Documentation
- Production deployment

**Deliverable:** Production-ready system, 99% uptime SLA

---

## 13. Testing Strategy

### 13.1 Unit Tests (Vitest)

**Agent Tests:**

```typescript
import { describe, it, expect, vi } from "vitest";
import { PlanningAgent } from "./planning-agent";

describe("PlanningAgent", () => {
  it("should decompose skill generation task into subtasks", async () => {
    const agent = new PlanningAgent();
    const task = {
      endpoint: "generate_skill",
      inputs: {
        examples: [/* 5 examples */],
        skill_name: "Product Showcase"
      }
    };

    const plan = await agent.createPlan(task);

    expect(plan.subtasks).toHaveLength(4);
    expect(plan.subtasks[0].agent).toBe("planning");
    expect(plan.subtasks[1].agent).toBe("research");
    expect(plan.subtasks[2].agent).toBe("coder");
    expect(plan.subtasks[3].agent).toBe("reporter");
    expect(plan.total_estimated_time_sec).toBeGreaterThan(0);
  });

  it("should handle invalid inputs gracefully", async () => {
    const agent = new PlanningAgent();
    const task = {
      endpoint: "generate_skill",
      inputs: {
        examples: [], // Invalid: no examples
        skill_name: "Test"
      }
    };

    await expect(agent.createPlan(task)).rejects.toThrow("At least 1 example required");
  });
});
```

**Target:** 80%+ code coverage

---

### 13.2 Integration Tests (Playwright)

**End-to-End Scenario:**

```typescript
import { test, expect } from "@playwright/test";

test("generate skill from examples - full flow", async ({ request }) => {
  // Step 1: Start task
  const startResponse = await request.post("/api/generate-skill/start", {
    headers: {
      "Authorization": "Bearer sk_openmanus_test_..."
    },
    data: {
      tenant_id: "tenant_test",
      examples: [
        {
          platform: "linkedin",
          content: "Sample post content...",
          performance_metrics: {
            likes: 50,
            comments: 10,
            shares: 5,
            impressions: 1000
          }
        }
        // ... 4 more examples
      ],
      skill_name: "Product Launch",
      target_platforms: ["linkedin", "facebook"]
    }
  });

  expect(startResponse.status()).toBe(200);
  const {task_id, status} = await startResponse.json();
  expect(status).toBe("queued");

  // Step 2: Poll status until completion
  let taskStatus = "queued";
  let attempts = 0;
  const maxAttempts = 60; // 10 minutes

  while (taskStatus !== "completed" && taskStatus !== "failed" && attempts < maxAttempts) {
    await new Promise(resolve => setTimeout(resolve, 10000)); // Wait 10s

    const statusResponse = await request.get(`/api/generate-skill/status/${task_id}`, {
      headers: {"Authorization": "Bearer sk_openmanus_test_..."}
    });

    const statusData = await statusResponse.json();
    taskStatus = statusData.status;

    expect(statusData).toHaveProperty("progress");
    expect(statusData.progress).toBeGreaterThanOrEqual(0);
    expect(statusData.progress).toBeLessThanOrEqual(100);

    attempts++;
  }

  expect(taskStatus).toBe("completed");

  // Step 3: Fetch result
  const resultResponse = await request.get(`/api/generate-skill/result/${task_id}`, {
    headers: {"Authorization": "Bearer sk_openmanus_test_..."}
  });

  expect(resultResponse.status()).toBe(200);
  const result = await resultResponse.json();

  expect(result).toHaveProperty("skill_id");
  expect(result).toHaveProperty("skill_yaml");
  expect(result.confidence_score).toBeGreaterThan(0);
  expect(result.validation_results.passes_guards).toBe(true);
});
```

**Test Scenarios:**
1. ✅ Generate skill (happy path)
2. ✅ Analyze brand voice
3. ✅ Refine content (synchronous)
4. ✅ Competitive analysis with partial failures
5. ✅ Rate limiting behavior
6. ✅ Authentication failures
7. ✅ Task timeout handling
8. ✅ Invalid input validation

---

### 13.3 Performance Tests (k6)

**Load Test:**

```javascript
import http from "k6/http";
import { check, sleep } from "k6";

export let options = {
  stages: [
    { duration: "2m", target: 10 },  // Ramp up to 10 users
    { duration: "5m", target: 10 },  // Stay at 10 users
    { duration: "2m", target: 20 },  // Ramp to 20 users
    { duration: "5m", target: 20 },  // Stay at 20 users
    { duration: "2m", target: 0 },   // Ramp down
  ],
  thresholds: {
    http_req_duration: ["p(95)<2000"], // 95% of requests < 2s (status poll)
    http_req_failed: ["rate<0.05"],    // <5% failure rate
  },
};

export default function () {
  // Start task
  const startRes = http.post(
    "https://openmanus-api.workers.dev/api/refine-content",
    JSON.stringify({
      tenant_id: "tenant_load_test",
      original_content: {
        text: "Simple post content",
        platform: "linkedin"
      },
      feedback: "Make it more professional",
      iteration: 1
    }),
    {
      headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer sk_openmanus_test_..."
      }
    }
  );

  check(startRes, {
    "status is 200": (r) => r.status === 200,
    "response time < 45s": (r) => r.timings.duration < 45000,
  });

  sleep(1);
}
```

**Performance Targets:**
- Sync endpoints (refine-content): P95 < 45s, P99 < 60s
- Status polling: P95 < 100ms, P99 < 200ms
- Task start: P95 < 500ms, P99 < 1s

---

## 14. Deployment Plan

### 14.1 Pre-Deployment Checklist

**Code:**
- [ ] All unit tests passing (80%+ coverage)
- [ ] All integration tests passing
- [ ] Performance tests meet targets
- [ ] Security audit completed (no critical vulnerabilities)
- [ ] Code reviewed by 2+ engineers
- [ ] Documentation complete

**Infrastructure:**
- [ ] Fly.io apps deployed (5 agents + gateway)
- [ ] Upstash Redis provisioned and tested
- [ ] Secrets configured in Fly.io vault
- [ ] Monitoring/alerting configured (Datadog + Fly.io)
- [ ] DNS configured (openmanus-api.workers.dev)
- [ ] SSL/TLS certificates issued

**Dify Integration:**
- [ ] HTTP tools configured
- [ ] API keys generated and stored
- [ ] Workflows tested in Dify staging
- [ ] Error handling tested

---

### 14.2 Deployment Steps

**Phase 1: Staging Deployment (Week 11)**

```bash
# 1. Deploy to staging cluster
kubectl apply -f k8s/staging/ --namespace openmanus-staging

# 2. Run smoke tests
npm run test:smoke -- --env=staging

# 3. Test Dify integration
# (Manual: Create test workflow in Dify staging)

# 4. Load test
k6 run tests/load-test.js --env staging
```

**Phase 2: Production Deployment (Week 12)**

```bash
# 1. Deploy to production (blue-green deployment)
kubectl apply -f k8s/production/ --namespace openmanus-prod

# 2. Run health checks
kubectl get pods -n openmanus-prod
kubectl logs -f deployment/planning-agent -n openmanus-prod

# 3. Switch traffic (gradual rollout)
kubectl set image deployment/openmanus-api \
  openmanus-api=gcr.io/dify-agent/openmanus-api:v1.0.0 \
  -n openmanus-prod

# 4. Monitor for 1 hour
# (Watch Datadog dashboard for errors, latency)

# 5. If issues: Rollback
kubectl rollout undo deployment/openmanus-api -n openmanus-prod

# 6. If successful: Complete
kubectl rollout status deployment/openmanus-api -n openmanus-prod
```

---

### 14.3 Rollback Plan

**Trigger Rollback If:**
- Error rate >10% for 5+ minutes
- P95 latency >2x baseline for 10+ minutes
- Critical bug discovered (data loss, security)

**Rollback Steps:**

```bash
# 1. Immediate rollback
kubectl rollout undo deployment/openmanus-api -n openmanus-prod

# 2. Verify old version running
kubectl get pods -n openmanus-prod -o wide

# 3. Clear task queue (prevent processing with old code)
redis-cli FLUSHDB

# 4. Notify users (via Dify)
# "⚠️ Maintenance in progress. Service restored to previous version."

# 5. Investigate issue
kubectl logs deployment/openmanus-api -n openmanus-prod --previous

# 6. Fix bug, re-test, re-deploy
```

---

### 14.4 Post-Deployment Validation

**24-Hour Monitoring Period:**

1. **Hour 1-4:** Intensive monitoring
   - Check all dashboards every 30 minutes
   - Monitor error logs in real-time
   - Test all 4 endpoints manually
   - Verify Dify integration working

2. **Hour 5-24:** Standard monitoring
   - Check dashboards every 2 hours
   - Review error logs 3x per day
   - Respond to alerts within 15 minutes

**Success Criteria:**
- [ ] All endpoints responding (200 OK)
- [ ] Error rate <2%
- [ ] P95 latency within targets
- [ ] No critical bugs reported
- [ ] Dify workflows executing successfully
- [ ] All 4 agents operational

---

## 15. Risk Assessment

### 15.1 Technical Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **OpenManus framework immature** | Medium | High | Have CrewAI as fallback framework (2-week migration path) |
| **LLM API rate limits** | Medium | Medium | Multi-provider strategy (OpenAI + Anthropic + Google), queue buffering |
| **Platform complexity** | Low | Low | Fly.io simplicity (fly.toml config), expert contractor for initial setup |
| **Dify HTTP tool timeout (120s)** | High | Low | Async polling pattern handles long tasks, no issue |
| **Redis cluster failure** | Low | High | 3-node cluster with auto-failover, daily backups |
| **Agent coordination bugs** | Medium | Medium | Extensive testing, retry logic, partial results support |
| **Cost overruns (LLM)** | Medium | Low | Multi-model routing, caching, budget alerts at 80% |

---

### 15.2 Business Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **Timeline slippage** | Medium | Medium | 20% buffer built into estimates, weekly status reviews |
| **Budget overrun** | Low | Medium | Fixed-price contractor quotes, 15% contingency fund |
| **Key developer leaves** | Low | High | Knowledge transfer sessions, comprehensive docs, pair programming |
| **Dify changes HTTP tool API** | Low | Medium | Abstract HTTP client, monitor Dify changelog, test on updates |
| **Competitor releases similar** | Low | Low | Our IP in agent coordination + skill system, hard to replicate |

---

### 15.3 Security Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **API key leakage** | Low | High | Secrets in KMS, auto-rotation every 90 days, audit logs |
| **Prompt injection attack** | Medium | Medium | Input sanitization, prompt validation, output filtering |
| **DDoS attack** | Low | Medium | Cloudflare DDoS protection, rate limiting, IP blocking |
| **Unauthorized access** | Low | High | Bearer token auth, API key per tenant, HTTPS only |
| **Data exfiltration** | Low | High | Tenant isolation (task_id scoping), audit logs, encryption at rest |

---

## 16. Appendices

### Appendix A: Glossary

**Terms:**
- **Agent**: Autonomous software entity that executes tasks (Planning, Research, Coder, Reporter)
- **Durable Object**: Cloudflare's strongly consistent storage primitive
- **Meta-Skill**: A skill that generates other skills (skill builder)
- **Async Pattern**: Three-phase request pattern (start → poll → result)
- **Task Queue**: System for managing long-running background tasks
- **Shared Memory**: Redis-based storage for inter-agent communication
- **Bearer Token**: Authentication token passed in HTTP Authorization header

---

### Appendix B: API Response Examples

See [Section 4](#4-custom-api-endpoints) for complete examples of all 4 endpoints.

---

### Appendix C: Deployment Commands

All deployment commands documented in [Section 14](#14-deployment-plan).

---

### Appendix D: Troubleshooting Guide

**Common Issues:**

1. **Task stuck in "queued" status**
   - Check: Are agents running? (`kubectl get pods`)
   - Check: Is Redis accessible? (`redis-cli PING`)
   - Check: Task queue logs (`kubectl logs deployment/openmanus-api`)

2. **High error rate from LLM API**
   - Check: Rate limits exceeded? (OpenAI dashboard)
   - Check: API key valid? (test with `curl`)
   - Mitigation: Retry with exponential backoff, fallback to Claude

3. **Dify polling timeout**
   - Check: Task actually completing? (Check Redis: `GET task:task_id`)
   - Check: ETA accurate? (May need recalibration)
   - Mitigation: Increase max poll iterations in Dify workflow

---

### Appendix E: References

**Documentation:**
- OpenManus GitHub: https://github.com/FoundationAgents/OpenManus
- Dify Documentation: https://docs.dify.ai
- Fly.io Documentation: https://fly.io/docs/
- Fly.io Machines: https://fly.io/docs/machines/
- Cloudflare Workers: https://developers.cloudflare.com/workers/
- Upstash Redis: https://docs.upstash.com/redis

**Research Documents:**
- [openmanus-architecture-research.md](C:\Users\avibm\NDS SOCIAL SYSTEM\NEW-SYSTEM\openmanus-architecture-research.md)
- [dify-openmanus-integration-design.md](C:\Users\avibm\NDS SOCIAL SYSTEM\NEW-SYSTEM\dify-openmanus-integration-design.md)
- [stage-h-cost-timeline-analysis.md](./stage-h-cost-timeline-analysis.md)

---

## ✅ Stage H Approved

**This plan is approved for implementation.**

**Prerequisites:**
- Stage D (Content Generation) - 11 microservices implemented
- Stage G (Dify + Custom Frontend) - Deployed and operational
- Budget approved: $105K-$180K development + $160/month operational

**Next Steps:**
1. Approve budget and timeline (12 weeks)
2. Assign team (DevOps, Backend engineers, AI/ML specialist, Dify specialist, QA)
3. Begin Phase 1: OpenManus Deployment Setup

**Questions? Contact:** planning-agent@dify-agent.local

---

**Document Version:** 1.0.0
**Last Updated:** November 10, 2025
**Status:** ✅ APPROVED
**Ready for:** Implementation (after Stage D & G complete)
