# Stage B Skills Decisions Analysis

**Status**: ✅ Complete
**Date**: 2025-11-10
**Purpose**: Extract existing decisions from Stage B about skill generation vs retrieval strategy
**Context**: Stage D (content generation workflow) planning

---

## Executive Summary

Stage B planning established a **comprehensive skill system** with clear decisions about skill generation, retrieval, storage, and execution. The system uses a **generation-first approach** with the Meta-Skill Builder, then switches to **retrieval-first for execution**.

### Key Decisions Made:

1. ✅ **Generation Strategy**: Meta-Skill Builder generates skills from 3-15 example posts
2. ✅ **Retrieval Strategy**: Skill Matching Algorithm retrieves skills using intent classification + vector search
3. ✅ **Storage**: Cloudflare D1 (SQL) + Vectorize (vector search) + KV (cache)
4. ✅ **Confidence Scoring**: 4-factor weighted algorithm (0-1 scale)
5. ✅ **Versioning**: Semantic versioning (MAJOR.MINOR.PATCH)
6. ✅ **Initial Skills**: Generated from examples, not hardcoded
7. ✅ **Skill Bank Size**: No hard limit, auto-archive low performers
8. ✅ **Creation Threshold**: Confidence score ≥0.50 to save

---

## 1. Generation vs Retrieval Strategy

### Decision: **Generation-First, Then Retrieval-First**

**From**: `decisions/002-skill-builder-meta-skill.md`

#### Generation (First-Time Creation)

**When**: User needs a skill that doesn't exist yet
**Method**: Meta-Skill Builder generates from examples

**Quote**:
> "User provides 3-5 examples of what it should do... Bot: 'Analyzing... I'll create a skill that does X'"

**Process**:
1. User uploads 3-15 example posts
2. Vision Analysis (Google Vision API) - 2-5s per image
3. Pattern Recognition (Claude Sonnet 4.5) - 5-10s
4. Rule Extraction (Claude Sonnet 4.5) - 10-20s
5. Skill Generation (Claude Sonnet 4.5) - 10-15s
6. Validation (5 validators) - 3-5s
7. User Approval

**Total Time**: 30-55 seconds + user review

**Files**: `meta-skill-builder.md`

#### Retrieval (Ongoing Use)

**When**: User requests content generation
**Method**: Skill Matching Algorithm

**Quote** from `skill-matching-algorithm.md`:
> "When a user makes a natural language request like 'Create educational post about T12 rebar,' the system must: 1. Understand what they want (intent classification), 2. Find relevant skills (search), 3. Rank by best fit (ranking), 4. Decide confidence level (threshold check)"

**Process**:
1. Intent Classification (Claude Sonnet 4.5) - identifies post_type, platform, style
2. Skill Search (SQL + Vectorize) - queries database with filters
3. Ranking (weighted scoring) - scores each candidate skill
4. Confidence Check (thresholds) - decides auto-execute vs ask
5. Execute or Ask User

**Total Time**: <500ms (target: 150-350ms)

**Files**: `skill-matching-algorithm.md`

### Why This Approach?

**From `002-skill-builder-meta-skill.md`**:

**Advantages**:
- ✅ Infinite extensibility without developer intervention
- ✅ Customer-specific customization
- ✅ Fast iteration (minutes vs weeks)
- ✅ Continuous improvement from usage
- ✅ Competitive moat (hard to replicate)

**Disadvantages**:
- ❌ Generated skills may be unpredictable
- ❌ Requires validation framework
- ❌ Quality may vary based on examples
- ❌ Complex debugging when skills fail

---

## 2. Skill Matching Algorithm

### Decision: **4-Step Matching Process**

**From**: `skill-matching-algorithm.md`

#### Step 1: Intent Classification

**Technology**: Claude Sonnet 4.5
**Input**: User's natural language request
**Output**: Structured classification with confidence scores

**Entities Extracted**:
- `post_type` (educational | promotional | announcement | story | engagement)
- `platform` (linkedin | twitter | instagram | facebook)
- `style` (professional | casual | urgent | inspiring | humorous)
- `audience` (target audience description)
- `urgency` (immediate | draft | scheduled)

**Confidence Scores**: Each entity gets 0.0-1.0 confidence

**Example**:
```json
{
  "post_type": "educational",
  "post_type_confidence": 0.98,
  "platform": "linkedin",
  "platform_confidence": 1.0,
  "style": "professional",
  "style_confidence": 0.89
}
```

#### Step 2: Skill Search

**Four Search Strategies** (in priority order):

1. **Exact Match** (highest priority): post_type + platform + style match
2. **Partial Match** (medium): post_type + platform match
3. **Broad Match** (low): post_type only
4. **Semantic Search** (fallback): Vector similarity using Vectorize

**Quote**:
> "Strategy 1: Exact Match (Highest Priority) - When to use: All key attributes are known with high confidence."

**Database**: Cloudflare D1 (SQL) + Vectorize (vectors)

#### Step 3: Ranking

**Formula**:
```
final_score = (
  match_quality_score × 0.40 +
  confidence_score × 0.25 +
  usage_score × 0.20 +
  recency_score × 0.10 +
  success_rate_score × 0.05
)
```

**Component Breakdown**:
- **Match Quality (40%)**: How well skill matches classified intent
- **Confidence Score (25%)**: Meta-skill builder's confidence (0-1)
- **Usage Score (20%)**: Frequency of use (log-scaled)
- **Recency Score (10%)**: How recently created/updated
- **Success Rate (5%)**: User approval rate

#### Step 4: Confidence Check

**Thresholds**:

| Score Range | Action | Notes |
|-------------|--------|-------|
| 0.80-1.00 | Auto-execute | Very confident, proceed immediately |
| 0.50-0.79 | Auto-execute with preview | Show preview first |
| 0.30-0.49 | Ask user (show top 3) | Let user choose |
| 0.00-0.29 | Create new skill | No good match, use meta-skill builder |

**Quote**:
> "score < 0.5: Ask user, score ≥ 0.5: Auto-execute"

### Performance Target

**Target**: <500ms total matching time
**Actual**: 150-350ms (exceeds target)

---

## 3. Skill Storage Strategy

### Decision: **Cloudflare Stack (D1 + Vectorize + KV)**

**From**: `STAGE-B-APPROVED-PLAN.md` and `skill-library-management.md`

#### Primary Storage: Cloudflare D1 (SQL)

**5 Tables**:

1. **`skills`**: Main skill storage (30 fields)
   - YAML content, metadata, performance metrics
   - Indexed by: skill_id, post_type, platform, style, usage_count, success_rate

2. **`skill_versions`**: Version history (9 fields)
   - Every update creates new version
   - Supports rollback

3. **`skill_usage`**: Execution tracking (23 fields)
   - Detailed logs of every execution
   - Performance metrics, user feedback

4. **`skill_examples`**: Training examples (9 fields)
   - Example posts used to create skills

5. **`user_skill_customizations`**: User edits (6 fields)
   - User-specific customizations

#### Vector Storage: Cloudflare Vectorize

**Purpose**: Semantic similarity search

**Index**: `brand-voice-patterns`
- **Dimensions**: 384 (Workers AI BGE-small-en-v1.5)
- **Metric**: Cosine similarity
- **Data**: Embeddings of skill examples and brand voice

**Usage**:
- Semantic skill search (fallback when SQL fails)
- Brand voice similarity checking

**Quote** from `STAGE-B-APPROVED-PLAN.md`:
> "Cloudflare Vectorize - Vector database for semantic search"

#### Cache: Cloudflare KV

**Purpose**: Hot skill caching

**Cache Criteria**:
- Used in last 24 hours, OR
- Usage count > 10, OR
- Featured skills

**Performance**:
- Cache hit: ~10ms
- Cache miss: ~50ms (D1 query)
- Target hit rate: >80%

**TTL**: 24 hours

**Invalidation**: On skill update

---

## 4. Skill Versioning Strategy

### Decision: **Semantic Versioning (MAJOR.MINOR.PATCH)**

**From**: `skill-library-management.md`

**Format**: `X.Y.Z` (e.g., `1.2.3`)

#### Version Increment Rules:

**MAJOR** (X.0.0):
- Changing `post_type` (educational → promotional)
- Changing `platform` (linkedin → twitter)
- Changing `style` significantly
- Complete rewrite

**MINOR** (x.Y.0):
- Adding new custom rule
- Adding new guardrail
- Adding forbidden word
- Refining tone characteristics

**PATCH** (x.y.Z):
- Fixing typos
- Correcting minor formatting
- Updating metadata only

**Quote**:
> "Skills use semantic versioning: MAJOR.MINOR.PATCH - MAJOR (1.0.0 → 2.0.0): Breaking changes (post_type, platform, core structure changed)"

#### Version History

**All versions stored** in `skill_versions` table

**Rollback support**:
- Can revert to any previous version
- Creates new patch version with old content
- Records rollback in version history

**Example**:
```
v1.5.3 → (rollback to v1.3.0) → v1.3.1 (with v1.3.0 content)
```

---

## 5. Confidence Scoring

### Decision: **4-Factor Weighted Algorithm**

**From**: `meta-skill-builder.md`

**Formula**:
```
confidence_score = (
  example_count_factor × 0.30 +
  pattern_consistency_factor × 0.35 +
  validation_score × 0.25 +
  brand_voice_match × 0.10
)
```

#### Factor 1: Example Count (30% weight)

**Calculation**:
- 3 examples: 0.40
- 5 examples: 0.50
- 8 examples: 0.65
- 10 examples: 0.75
- 15+ examples: 1.00

**Quote**:
> "More examples = higher confidence. Diminishing returns after 10 examples"

#### Factor 2: Pattern Consistency (35% weight)

**Checks**:
- Length consistency across examples
- Tone consistency (5 voice characteristics)
- Structure consistency
- Hashtag consistency
- Format consistency

**Calculation**: Average of 5 consistency checks

#### Factor 3: Validation Score (25% weight)

**5 Validators** (run in parallel):
1. Completeness (all required sections present)
2. Conflicts (no contradicting rules)
3. Safety (no dangerous patterns)
4. Brand Voice (≥0.75 similarity)
5. Platform Requirements (meets limits)

**Score**: Average of validator scores

#### Factor 4: Brand Voice Match (10% weight)

**Calculation**:
- Compare skill examples to brand voice index (Vectorize)
- Cosine similarity score
- Target: ≥0.75

### Confidence Thresholds

| Score | Level | Recommendation |
|-------|-------|----------------|
| 0.90-1.00 | Excellent | Use with confidence |
| 0.75-0.89 | Good | Safe to use |
| 0.60-0.74 | Fair | Use with caution |
| 0.50-0.59 | Low | Review carefully |
| <0.50 | Poor | Not recommended |

**Minimum to save**: ≥0.50

---

## 6. Initial Skills Strategy

### Decision: **Generate from Examples, No Hardcoded Skills**

**From**: `002-skill-builder-meta-skill.md` and `STAGE-B-APPROVED-PLAN.md`

#### No Pre-Built Library

**Quote** from `002-skill-builder-meta-skill.md`:
> "Traditional approach requires developers to code each new feature/skill. We need a way to add new capabilities without developer intervention."

**Approach**:
- Start with **zero hardcoded skills**
- Users provide example posts
- Meta-Skill Builder generates skills
- Skills improve through usage

#### First-Time User Flow

**Scenario**: New user, no skills yet

**Process**:
1. User: "Create a LinkedIn post about rebar"
2. Bot: "I don't have a skill for this yet. Can you provide 3-5 example posts?"
3. User uploads examples
4. Meta-Skill Builder generates skill (30-55 seconds)
5. Bot: "Skill created! Want to try it?"
6. User: "Yes"
7. Bot generates content using new skill

**Subsequent uses**: Skill now available for retrieval

#### Skill Growth Over Time

**Month 1**: 5-10 skills (user-generated)
**Month 3**: 20-30 skills (heavily used ones)
**Month 6**: 50+ skills (diverse use cases)

**Auto-improvement**:
- Usage data improves confidence scores
- Low performers archived
- Popular skills cached

---

## 7. Skill Bank Size & Limits

### Decision: **No Hard Limit, Auto-Archive Low Performers**

**From**: `skill-library-management.md`

#### No Maximum Skill Count

**Quote**:
> "Archive skills that: 1. Low performance (Success rate < 50% after 10+ uses), 2. Unused (Not used in 90+ days), 3. Deprecated (Replaced by better version)"

**Approach**: Grow as needed, automatically prune

#### Auto-Archive Criteria

**Trigger for archival**:

1. **Low Performance**:
   - Success rate < 50%
   - After ≥10 uses
   - Gives statistical significance

2. **Unused**:
   - Not used in 90+ days
   - Prevents bloat

3. **Deprecated**:
   - Manually marked by user
   - Replaced by better version

**Process**:
- Status set to `archived` (soft delete)
- Not returned in searches
- Can be unarchived if needed
- Version history preserved

#### Active Skill Management

**Caching Strategy**:
- Hot skills (used in 24h): Cached
- Usage count >10: Cached
- Featured skills: Cached
- **Cache limit**: ~50 skills

**Database**: No practical limit (Cloudflare D1 scales)

**Search Performance**:
- Indexed searches: <50ms
- Semantic searches: 20-50ms
- Not degraded by skill count (indexes)

---

## 8. Skill Creation Threshold

### Decision: **Generate When Confidence ≥0.50, Ask User Below**

**From**: `skill-matching-algorithm.md` and `meta-skill-builder.md`

#### Generation Threshold

**Minimum confidence to save**: 0.50

**If confidence <0.50**:
- Meta-Skill Builder rejects skill
- Asks for more examples
- Provides quality feedback

**Quote** from confidence thresholds:
> "< 0.50: Poor - Unreliable patterns - Not recommended, gather more examples"

#### Matching Threshold

**When no skill matches request**:

**If top match score <0.30**:
- Don't use existing skill
- Trigger Meta-Skill Builder
- Create new skill

**Quote**:
> "0.00-0.29: Create new skill - No good match, use meta-skill builder"

#### Quality Assurance

**5 Validators Must Pass**:

1. ✅ Completeness: All required sections
2. ✅ No Conflicts: Rules don't contradict
3. ✅ Safety: No dangerous patterns
4. ✅ Brand Voice: ≥0.75 similarity
5. ✅ Platform: Meets requirements

**If validators fail**: Skill not saved, user shown issues

---

## Gaps & Open Questions

### What's Already Decided:

1. ✅ Generation-first approach (Meta-Skill Builder)
2. ✅ Retrieval algorithm (4-step matching)
3. ✅ Storage architecture (D1 + Vectorize + KV)
4. ✅ Confidence scoring (4 factors, weighted)
5. ✅ Versioning (semantic versioning)
6. ✅ No initial hardcoded skills
7. ✅ Auto-archive strategy
8. ✅ Creation thresholds

### What's Still Open (Stage D to Decide):

1. ⚠️ **Dify Integration**: How does Dify call Meta-Skill Builder?
2. ⚠️ **Conversational Flow**: How to present "provide examples" in chat?
3. ⚠️ **Skill Preview**: How to show generated skill to user in Dify?
4. ⚠️ **Error Handling**: What if generation fails mid-conversation?
5. ⚠️ **Skill Selection UI**: Text-based A/B/C or something else?
6. ⚠️ **Feedback Loop**: How to collect user feedback in Dify?
7. ⚠️ **Multi-Turn State**: How to preserve context across skill generation?

**Note**: These are **Dify-specific** questions, not skill system questions.

---

## Relevant Quotes from Documents

### On Generation Strategy

**From `002-skill-builder-meta-skill.md`**:

> "We choose Option 3: Skill-Builder Meta-Skill. Rationale: 1. True Extensibility - Not limited to predefined capabilities, 2. No Developer Required - Business users can create skills, 3. Leverages AI Strengths - GPT-4/5 excellent at pattern recognition"

### On Retrieval Strategy

**From `skill-matching-algorithm.md`**:

> "Skill matching is the algorithm that selects the right skill for the user's request. Success Criteria: Accurately classifies user intent (>90% accuracy), Finds relevant skills (high recall), Ranks best match first (high precision), Fast (<500ms for matching)"

### On Storage

**From `STAGE-B-APPROVED-PLAN.md`**:

> "Storage: Cloudflare Vectorize - Vector database for semantic search, Cloudflare D1 - SQL database for skills, Cloudflare KV - Cache for hot skills"

### On Confidence

**From `meta-skill-builder.md`**:

> "The confidence score (0.0 to 1.0) indicates how reliable this skill is expected to be. Formula: confidence_score = (example_count_factor × 0.30 + pattern_consistency_factor × 0.35 + validation_score × 0.25 + brand_voice_match × 0.10)"

### On Auto-Archive

**From `skill-library-management.md`**:

> "Archive skills that: 1. Low performance: Success rate < 50% after 10+ uses, 2. Unused: Not used in 90+ days, 3. Deprecated: Replaced by better version, 4. User request: Manual archival"

---

## Conclusions

### What's Decided vs What's Open

#### ✅ DECIDED (Stage B):

**Generation & Retrieval**:
- Meta-Skill Builder generates from examples
- Skill Matching Algorithm retrieves via intent classification
- Generation: 30-55s, Retrieval: <500ms

**Storage**:
- Cloudflare D1 (SQL) for skills
- Cloudflare Vectorize for semantic search
- Cloudflare KV for caching
- 5 database tables defined

**Confidence & Quality**:
- 4-factor weighted confidence scoring
- 5 validators for quality assurance
- Thresholds: ≥0.50 to save, ≥0.30 to use

**Versioning & Management**:
- Semantic versioning (MAJOR.MINOR.PATCH)
- Full version history with rollback
- Auto-archive low performers

**Initial Skills**:
- Start with zero hardcoded skills
- Generate from user examples
- Grow organically through usage

**Performance**:
- Skill matching: <500ms
- Cache hit: ~10ms
- Cache miss: ~50ms

#### ⚠️ OPEN (Stage D to Decide):

**Dify Integration**:
- How Dify chatflow calls Meta-Skill Builder
- Conversational UX for example collection
- Skill preview in Dify UI
- Error handling in multi-turn conversation
- Feedback collection mechanism

**Stage D Focus**: Integration with Dify, not changing skill system architecture

---

## Recommendations for Stage D

### Use Stage B Decisions As-Is

**Don't Re-Decide**:
- ❌ Storage architecture (already decided: D1 + Vectorize + KV)
- ❌ Matching algorithm (already decided: 4-step process)
- ❌ Confidence scoring (already decided: 4-factor formula)
- ❌ Versioning strategy (already decided: semantic versioning)

**Stage D Should Focus On**:
- ✅ Dify chatflow design
- ✅ Conversational UX patterns
- ✅ REST API design for Dify ↔ MCP Gateway
- ✅ Multi-turn conversation state management
- ✅ Error handling in Dify workflows

### Integration Points

**What Stage D Needs from Stage B**:

1. **MCP Tools to Build**:
   - `generate_skill` (calls Meta-Skill Builder)
   - `search_skills` (calls Skill Matching)
   - `execute_skill` (runs skill execution engine)
   - `record_feedback` (updates performance metrics)

2. **REST Endpoints** (MCP Gateway):
   - `POST /tools/generate_skill`
   - `POST /tools/search_skills`
   - `POST /tools/execute_skill`
   - `POST /tools/record_feedback`

3. **Dify Custom Tools**: Map to MCP Gateway endpoints

**Stage B provides the engine, Stage D integrates it with Dify.**

---

## Document Metadata

**Files Analyzed**:
- ✅ `STAGE-B-APPROVED-PLAN.md`
- ✅ `decisions/002-skill-builder-meta-skill.md`
- ✅ `decisions/003-conversational-advisory-pattern.md`
- ✅ `decisions/skill-matching-algorithm.md`
- ✅ `research/skill-file-format.md`
- ✅ `research/meta-skill-builder.md`
- ✅ `research/skill-execution-engine.md`
- ✅ `research/skill-library-management.md`

**Total Pages Analyzed**: ~8,000 lines of documentation

**Extraction Date**: 2025-11-10

**Next Steps**: Use this analysis for Stage D (Content Generation Workflow) planning

---

**Status**: ✅ Analysis Complete
**Stage B Decisions**: Fully Documented
**Stage D Can Proceed**: Yes, with clarity on what's decided vs open
