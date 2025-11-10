# Skill Matching Algorithm

**Status:** 🟡 DRAFT - Planning Stage
**Location:** .planning/
**Approved:** NO
**Last Updated:** 2025-11-06

---

## Table of Contents

1. [Overview](#overview)
2. [Complete Algorithm](#complete-algorithm)
3. [Step 1: Intent Classification](#step-1-intent-classification)
4. [Step 2: Skill Search](#step-2-skill-search)
5. [Step 3: Ranking](#step-3-ranking)
6. [Step 4: Confidence Check](#step-4-confidence-check)
7. [Step 5: Execute](#step-5-execute)
8. [Complete Examples](#complete-examples)

---

## Overview

### What is Skill Matching?

When a user makes a natural language request like "Create educational post about Product T12," the system must:
1. Understand what they want (intent classification)
2. Find relevant skills (search)
3. Rank by best fit (ranking)
4. Decide confidence level (threshold check)
5. Execute or ask for clarification

**Skill matching** is the algorithm that selects the right skill for the user's request.

### Why Skill Matching Matters

**Without skill matching:**
- User must manually browse and select skills
- Requires understanding of skill taxonomy
- Slow and error-prone

**With skill matching:**
- Natural language requests ("Make a LinkedIn post about Product Line B")
- Automatic skill selection
- Fast and intuitive

### Success Criteria

A good skill matching algorithm:
- ✅ Accurately classifies user intent (>90% accuracy)
- ✅ Finds relevant skills (high recall)
- ✅ Ranks best match first (high precision)
- ✅ Knows when to ask for clarification (appropriate confidence thresholds)
- ✅ Handles ambiguous requests gracefully
- ✅ Fast (<500ms for matching)

---

## Complete Algorithm

### 5-Step Process

```
User Request
    ↓
STEP 1: Intent Classification
    Extract: post_type, product, platform, style, audience
    ↓
STEP 2: Skill Search
    Query database with filters
    ↓
STEP 3: Ranking
    Score each skill: confidence × usage × recency × match_quality
    ↓
STEP 4: Confidence Check
    score < 0.5: Ask user
    score ≥ 0.5: Auto-execute
    ↓
STEP 5: Execute
    Load skill, proceed with content generation
```

### Flow Diagram

```
┌─────────────────────────────────────┐
│ User: "Create educational post      │
│       about Product T12"            │
└──────────────┬──────────────────────┘
               ↓
┌──────────────────────────────────────┐
│ STEP 1: Intent Classification        │
│                                      │
│ Extracted entities:                  │
│ - post_type: educational             │
│ - product: Product T12               │
│ - platform: linkedin (inferred)      │
│ - style: professional (inferred)     │
│ - audience: null                     │
└──────────────┬───────────────────────┘
               ↓
┌──────────────────────────────────────┐
│ STEP 2: Skill Search                 │
│                                      │
│ SQL: SELECT * FROM skills            │
│ WHERE post_type = 'educational'      │
│   AND (platform = 'linkedin'         │
│        OR platform IS NULL)          │
│   AND status = 'active'              │
│                                      │
│ Results: 5 skills found              │
└──────────────┬───────────────────────┘
               ↓
┌──────────────────────────────────────┐
│ STEP 3: Ranking                      │
│                                      │
│ Skill 1: score = 0.87                │
│ Skill 2: score = 0.76                │
│ Skill 3: score = 0.65                │
│ Skill 4: score = 0.52                │
│ Skill 5: score = 0.41                │
│                                      │
│ Top match: Skill 1 (0.87)            │
└──────────────┬───────────────────────┘
               ↓
┌──────────────────────────────────────┐
│ STEP 4: Confidence Check             │
│                                      │
│ score = 0.87 ≥ 0.5                   │
│ → AUTO-EXECUTE                       │
└──────────────┬───────────────────────┘
               ↓
┌──────────────────────────────────────┐
│ STEP 5: Execute                      │
│                                      │
│ Load Skill 1                         │
│ Generate content using skill         │
└──────────────────────────────────────┘
```

---

## Step 1: Intent Classification

### Purpose

Extract structured information from natural language user request.

### Entity Extraction

**Entities to extract:**
- `post_type`: educational | promotional | announcement | story | engagement
- `product`: Product name or category
- `platform`: linkedin | twitter | instagram | facebook
- `style`: professional | casual | urgent | inspiring | humorous
- `audience`: Target audience (engineers, industry professionals, etc.)
- `urgency`: Immediate request or draft for review

### Claude Sonnet 4.5 Prompt for Intent Classification

```javascript
const INTENT_CLASSIFICATION_PROMPT = `You are an AI assistant that extracts structured information from user requests.

USER REQUEST:
"{user_request}"

TASK:
Extract the following entities from the request.

OUTPUT FORMAT (JSON):
{
  "post_type": "educational | promotional | announcement | story | engagement | unknown",
  "post_type_confidence": 0.0-1.0,
  "post_type_reasoning": "Why you classified it this way",

  "product": "Product name or null",
  "product_category": "Product category or null",

  "platform": "linkedin | twitter | instagram | facebook | null",
  "platform_confidence": 0.0-1.0,
  "platform_source": "explicit (user stated) | inferred (guessed from context)",

  "style": "professional | casual | urgent | inspiring | humorous | null",
  "style_confidence": 0.0-1.0,

  "audience": "Target audience description or null",

  "urgency": "immediate | draft | scheduled | null",

  "additional_requirements": ["requirement1", "requirement2"],

  "ambiguities": ["ambiguity1", "ambiguity2"]
}

CLASSIFICATION GUIDELINES:

POST TYPES:
- "educational": User wants to teach/explain (keywords: explain, teach, educate, how to, understand, learn, guide)
- "promotional": User wants to sell/promote (keywords: promote, sale, offer, discount, buy, limited time)
- "announcement": User wants to announce news (keywords: announce, new, launch, introducing, update)
- "story": User wants to tell a story (keywords: story, journey, experience, case study)
- "engagement": User wants to engage audience (keywords: poll, question, discussion, thoughts, opinions)

PLATFORM INFERENCE:
- If not explicitly stated, infer from context:
  - B2B content, professional tone → linkedin
  - Short, concise, trending topics → twitter
  - Visual-first, lifestyle → instagram
  - Community, personal → facebook

STYLE INFERENCE:
- Professional: Industry terms, formal language, B2B context
- Casual: Conversational, friendly, accessible
- Urgent: Time-sensitive, action words, deadlines
- Inspiring: Motivational, aspirational, uplifting
- Humorous: Jokes, wit, playfulness

PRODUCT EXTRACTION:
- Extract exact product name if mentioned (e.g., "Product T12", "Product SKU-A142")
- Extract category if no specific product (e.g., "Product Line B", "Product Line A", "steel")

AMBIGUITIES:
- Note anything unclear that might require user clarification
- Examples: "Not sure which platform", "Could be educational or promotional"

Be precise. If uncertain, note the ambiguity.`;

async function classifyIntent(userRequest) {
  const prompt = INTENT_CLASSIFICATION_PROMPT.replace('{user_request}', userRequest);

  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    messages: [
      { role: 'user', content: prompt }
    ],
    max_tokens: 1024,
    temperature: 0.1 // Very low for consistent classification
  });

  return JSON.parse(response.choices[0].message.content);
}
```

### Example Classifications

#### Example 1: Clear Request

**User request:**
> "Create an educational LinkedIn post about Product T12 specifications"

**Classification:**
```json
{
  "post_type": "educational",
  "post_type_confidence": 0.98,
  "post_type_reasoning": "User explicitly said 'educational' and wants to share specifications (teaching)",

  "product": "Product T12",
  "product_category": "Product Line B",

  "platform": "linkedin",
  "platform_confidence": 1.0,
  "platform_source": "explicit",

  "style": "professional",
  "style_confidence": 0.85,

  "audience": null,

  "urgency": "draft",

  "additional_requirements": ["Include specifications"],

  "ambiguities": []
}
```

#### Example 2: Ambiguous Request

**User request:**
> "Make a post about our new Product Line B sale"

**Classification:**
```json
{
  "post_type": "promotional",
  "post_type_confidence": 0.92,
  "post_type_reasoning": "Keyword 'sale' indicates promotional intent",

  "product": "Product Line B",
  "product_category": "Product Line B",

  "platform": null,
  "platform_confidence": 0.0,
  "platform_source": "not_specified",

  "style": "urgent",
  "style_confidence": 0.65,

  "audience": null,

  "urgency": "draft",

  "additional_requirements": [],

  "ambiguities": [
    "Platform not specified - need to ask user (LinkedIn? Twitter? Instagram?)",
    "Which Product Line B items are on sale? Need specific products."
  ]
}
```

#### Example 3: Implicit Platform

**User request:**
> "Write a professional post explaining the difference between Grade 60 and Grade 75 Product Line B"

**Classification:**
```json
{
  "post_type": "educational",
  "post_type_confidence": 0.95,
  "post_type_reasoning": "User wants to 'explain' (educational keyword)",

  "product": "Product Line B",
  "product_category": "Product Line B",

  "platform": "linkedin",
  "platform_confidence": 0.75,
  "platform_source": "inferred",
  "platform_reasoning": "Professional tone + educational content suggests LinkedIn",

  "style": "professional",
  "style_confidence": 0.98,

  "audience": "engineers or industry professionals interested in specifications",

  "urgency": "draft",

  "additional_requirements": [
    "Compare Grade 60 and Grade 75",
    "Explain differences"
  ],

  "ambiguities": []
}
```

### Intent Classification Implementation

```javascript
// intent-classifier.js

export async function classifyUserIntent(userRequest, userContext = {}) {
  // Step 1: Run Claude Sonnet 4.5 classification
  const claudeClassification = await classifyIntent(userRequest);

  // Step 2: Enhance with user context
  const enhancedClassification = enhanceWithContext(claudeClassification, userContext);

  // Step 3: Validate and normalize
  const normalized = normalizeClassification(enhancedClassification);

  return normalized;
}

function enhanceWithContext(classification, userContext) {
  // User's default platform preference
  if (!classification.platform && userContext.default_platform) {
    classification.platform = userContext.default_platform;
    classification.platform_source = 'user_default';
    classification.platform_confidence = 0.60;
  }

  // User's industry/audience
  if (!classification.audience && userContext.default_audience) {
    classification.audience = userContext.default_audience;
  }

  // User's typical style
  if (!classification.style && userContext.typical_style) {
    classification.style = userContext.typical_style;
    classification.style_confidence = 0.50;
  }

  return classification;
}

function normalizeClassification(classification) {
  // Ensure all fields exist
  return {
    post_type: classification.post_type || 'unknown',
    post_type_confidence: classification.post_type_confidence || 0,

    product: classification.product || null,
    product_category: classification.product_category || null,

    platform: classification.platform || null,
    platform_confidence: classification.platform_confidence || 0,

    style: classification.style || null,
    style_confidence: classification.style_confidence || 0,

    audience: classification.audience || null,

    urgency: classification.urgency || 'draft',

    additional_requirements: classification.additional_requirements || [],

    ambiguities: classification.ambiguities || []
  };
}
```

---

## Step 2: Skill Search

### Purpose

Query the skills database to find all potentially relevant skills.

### Database Schema (Reference)

```sql
CREATE TABLE skills (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  skill_id TEXT UNIQUE NOT NULL,
  skill_name TEXT NOT NULL,
  version TEXT NOT NULL,
  yaml_content TEXT NOT NULL,

  -- Searchable fields
  post_type TEXT NOT NULL,
  platform TEXT NOT NULL,
  style TEXT NOT NULL,
  target_audience TEXT,

  -- Metadata
  confidence_score REAL NOT NULL,
  usage_count INTEGER DEFAULT 0,
  success_rate REAL DEFAULT 0,
  avg_execution_time REAL,

  -- Status
  status TEXT DEFAULT 'active',

  -- Timestamps
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,

  -- Indexes for fast search
  INDEX idx_post_type (post_type),
  INDEX idx_platform (platform),
  INDEX idx_style (style),
  INDEX idx_status (status)
);
```

### Search Strategy

#### Strategy 1: Exact Match (Highest Priority)

```sql
SELECT *
FROM skills
WHERE post_type = ?
  AND platform = ?
  AND style = ?
  AND status = 'active'
ORDER BY confidence_score DESC, usage_count DESC
LIMIT 10;
```

**When to use:** All key attributes are known with high confidence.

#### Strategy 2: Partial Match (Medium Priority)

```sql
SELECT *
FROM skills
WHERE post_type = ?
  AND platform = ?
  AND status = 'active'
ORDER BY confidence_score DESC, usage_count DESC
LIMIT 20;
```

**When to use:** post_type and platform known, style unknown or low confidence.

#### Strategy 3: Broad Match (Low Priority)

```sql
SELECT *
FROM skills
WHERE post_type = ?
  AND status = 'active'
ORDER BY confidence_score DESC, usage_count DESC
LIMIT 30;
```

**When to use:** Only post_type known, platform and style unknown.

#### Strategy 4: Semantic Search (Fallback)

```javascript
// Use embeddings to find semantically similar skills
async function semanticSkillSearch(userRequest, topK = 10) {
  // Generate embedding for user request
  const requestEmbedding = await generateEmbedding(userRequest);

  // Query Vectorize index
  const results = await env.SKILL_VECTORS.query(requestEmbedding, {
    topK: topK,
    returnMetadata: true
  });

  // Get full skill data
  const skillIds = results.matches.map(m => m.metadata.skill_id);
  const skills = await db.query(`
    SELECT * FROM skills
    WHERE skill_id IN (${skillIds.map(() => '?').join(',')})
      AND status = 'active'
  `, skillIds);

  return skills.map((skill, i) => ({
    ...skill,
    semantic_score: results.matches[i].score
  }));
}
```

**When to use:** Classification is very ambiguous or no SQL matches found.

### Search Implementation

```javascript
// skill-search.js

export async function searchSkills(classification, env) {
  const results = {
    exact_matches: [],
    partial_matches: [],
    broad_matches: [],
    semantic_matches: []
  };

  // Determine search strategy based on classification confidence
  const useExact = classification.post_type_confidence >= 0.8 &&
                   classification.platform_confidence >= 0.8 &&
                   classification.style_confidence >= 0.7;

  const usePartial = classification.post_type_confidence >= 0.7 &&
                     classification.platform_confidence >= 0.7;

  const useBroad = classification.post_type_confidence >= 0.6;

  // Execute searches in parallel
  const searches = [];

  if (useExact && classification.style) {
    searches.push(
      exactMatchSearch(
        classification.post_type,
        classification.platform,
        classification.style,
        env
      ).then(matches => results.exact_matches = matches)
    );
  }

  if (usePartial) {
    searches.push(
      partialMatchSearch(
        classification.post_type,
        classification.platform,
        env
      ).then(matches => results.partial_matches = matches)
    );
  }

  if (useBroad) {
    searches.push(
      broadMatchSearch(
        classification.post_type,
        env
      ).then(matches => results.broad_matches = matches)
    );
  }

  // Always do semantic search as backup
  searches.push(
    semanticSkillSearch(
      classification.original_request,
      env
    ).then(matches => results.semantic_matches = matches)
  );

  await Promise.all(searches);

  return results;
}

async function exactMatchSearch(postType, platform, style, env) {
  const result = await env.DB.prepare(`
    SELECT *
    FROM skills
    WHERE post_type = ?
      AND platform = ?
      AND style = ?
      AND status = 'active'
    ORDER BY confidence_score DESC, usage_count DESC
    LIMIT 10
  `).bind(postType, platform, style).all();

  return result.results || [];
}

async function partialMatchSearch(postType, platform, env) {
  const result = await env.DB.prepare(`
    SELECT *
    FROM skills
    WHERE post_type = ?
      AND platform = ?
      AND status = 'active'
    ORDER BY confidence_score DESC, usage_count DESC
    LIMIT 20
  `).bind(postType, platform).all();

  return result.results || [];
}

async function broadMatchSearch(postType, env) {
  const result = await env.DB.prepare(`
    SELECT *
    FROM skills
    WHERE post_type = ?
      AND status = 'active'
    ORDER BY confidence_score DESC, usage_count DESC
    LIMIT 30
  `).bind(postType).all();

  return result.results || [];
}
```

### Combining Search Results

```javascript
function combineSearchResults(results) {
  const combined = [];
  const seen = new Set();

  // Priority order: exact → partial → broad → semantic
  const priority = [
    { matches: results.exact_matches, weight: 1.0 },
    { matches: results.partial_matches, weight: 0.8 },
    { matches: results.broad_matches, weight: 0.6 },
    { matches: results.semantic_matches, weight: 0.5 }
  ];

  for (const { matches, weight } of priority) {
    for (const skill of matches) {
      if (!seen.has(skill.skill_id)) {
        combined.push({
          ...skill,
          search_weight: weight
        });
        seen.add(skill.skill_id);
      }
    }
  }

  return combined;
}
```

---

## Step 3: Ranking

### Purpose

Score each candidate skill to determine the best match.

### Ranking Formula

```
final_score = (
  match_quality_score × 0.40 +
  confidence_score × 0.25 +
  usage_score × 0.20 +
  recency_score × 0.10 +
  success_rate_score × 0.05
)
```

### Component Calculations

#### 1. Match Quality Score (40% weight)

How well does the skill match the classified intent?

```javascript
function calculateMatchQuality(skill, classification, searchWeight) {
  let score = 0;
  let maxScore = 0;

  // post_type match (critical)
  maxScore += 40;
  if (skill.post_type === classification.post_type) {
    score += 40 * classification.post_type_confidence;
  }

  // platform match (important)
  maxScore += 30;
  if (skill.platform === classification.platform) {
    score += 30 * classification.platform_confidence;
  } else if (!classification.platform) {
    score += 15; // Partial credit if platform not specified
  }

  // style match (moderate)
  maxScore += 20;
  if (skill.style === classification.style) {
    score += 20 * classification.style_confidence;
  } else if (!classification.style) {
    score += 10; // Partial credit if style not specified
  }

  // audience match (optional)
  maxScore += 10;
  if (classification.audience) {
    const audienceMatch = checkAudienceMatch(skill.target_audience, classification.audience);
    score += 10 * audienceMatch;
  } else {
    score += 5; // Partial credit if no audience specified
  }

  // Apply search weight (exact match gets full weight, semantic gets reduced)
  const normalizedScore = (score / maxScore) * searchWeight;

  return normalizedScore;
}

function checkAudienceMatch(skillAudience, classifiedAudience) {
  if (!skillAudience || !classifiedAudience) return 0.5;

  // Simple keyword matching (could be enhanced with embeddings)
  const skillKeywords = skillAudience.toLowerCase().split(/\s+/);
  const classKeywords = classifiedAudience.toLowerCase().split(/\s+/);

  const intersection = skillKeywords.filter(k => classKeywords.includes(k));
  const union = new Set([...skillKeywords, ...classKeywords]);

  return intersection.length / union.size; // Jaccard similarity
}
```

#### 2. Confidence Score (25% weight)

How confident are we in this skill's quality?

```javascript
function calculateConfidenceScore(skill) {
  // Skill's intrinsic confidence score (from meta-skill builder)
  return skill.confidence_score; // Already 0-1
}
```

#### 3. Usage Score (20% weight)

How frequently has this skill been used successfully?

```javascript
function calculateUsageScore(skill, maxUsage) {
  if (maxUsage === 0) return 0.5; // No usage data yet

  // Normalize usage count (log scale to prevent heavily used skills from dominating)
  const normalizedUsage = Math.log(skill.usage_count + 1) / Math.log(maxUsage + 1);

  return normalizedUsage;
}
```

#### 4. Recency Score (10% weight)

How recently was this skill created or updated?

```javascript
function calculateRecencyScore(skill) {
  const now = Date.now();
  const updated = new Date(skill.updated_at).getTime();
  const age_days = (now - updated) / (1000 * 60 * 60 * 24);

  // Decay over 180 days
  // New skills: 1.0, 90-day-old: 0.5, 180-day-old: 0.0
  const recency = Math.max(0, 1 - (age_days / 180));

  return recency;
}
```

#### 5. Success Rate Score (5% weight)

How often do users approve content generated by this skill?

```javascript
function calculateSuccessRateScore(skill) {
  if (skill.usage_count === 0) return 0.5; // No data, neutral score

  // success_rate already 0-1 (percentage of approvals)
  return skill.success_rate || 0.5;
}
```

### Complete Ranking Implementation

```javascript
// skill-ranker.js

export function rankSkills(skills, classification) {
  // Find max usage for normalization
  const maxUsage = Math.max(...skills.map(s => s.usage_count), 1);

  // Calculate scores for each skill
  const scored = skills.map(skill => {
    const matchQuality = calculateMatchQuality(skill, classification, skill.search_weight || 1.0);
    const confidence = calculateConfidenceScore(skill);
    const usage = calculateUsageScore(skill, maxUsage);
    const recency = calculateRecencyScore(skill);
    const successRate = calculateSuccessRateScore(skill);

    const finalScore = (
      matchQuality * 0.40 +
      confidence * 0.25 +
      usage * 0.20 +
      recency * 0.10 +
      successRate * 0.05
    );

    return {
      skill: skill,
      scores: {
        final: finalScore,
        match_quality: matchQuality,
        confidence: confidence,
        usage: usage,
        recency: recency,
        success_rate: successRate
      }
    };
  });

  // Sort by final score (descending)
  scored.sort((a, b) => b.scores.final - a.scores.final);

  return scored;
}
```

---

## Step 4: Confidence Check

### Purpose

Decide whether to auto-execute the top skill or ask the user for clarification.

### Confidence Thresholds

| Score Range | Action | Reasoning |
|-------------|--------|-----------|
| 0.80 - 1.00 | Auto-execute | Very confident, proceed immediately |
| 0.50 - 0.79 | Auto-execute with preview | Confident enough, show preview first |
| 0.30 - 0.49 | Ask user (show top 3) | Uncertain, let user choose |
| 0.00 - 0.29 | Create new skill | No good match, offer to create new |

### Decision Logic

```javascript
// confidence-checker.js

export function checkConfidence(rankedSkills, classification) {
  if (rankedSkills.length === 0) {
    return {
      action: 'create_new',
      reasoning: 'No matching skills found',
      recommendation: 'Use meta-skill builder to create new skill'
    };
  }

  const topSkill = rankedSkills[0];
  const topScore = topSkill.scores.final;

  // Check for ambiguities from classification
  const hasAmbiguities = classification.ambiguities.length > 0;

  // Very high confidence
  if (topScore >= 0.80 && !hasAmbiguities) {
    return {
      action: 'auto_execute',
      skill: topSkill.skill,
      score: topScore,
      reasoning: 'High confidence match',
      preview: false
    };
  }

  // Good confidence
  if (topScore >= 0.50 && !hasAmbiguities) {
    return {
      action: 'auto_execute',
      skill: topSkill.skill,
      score: topScore,
      reasoning: 'Good confidence match',
      preview: true // Show preview before posting
    };
  }

  // Low confidence or ambiguities present
  if (topScore >= 0.30 || hasAmbiguities) {
    return {
      action: 'ask_user',
      options: rankedSkills.slice(0, 3).map(s => ({
        skill: s.skill,
        score: s.scores.final,
        summary: generateSkillSummary(s.skill)
      })),
      reasoning: hasAmbiguities
        ? `Ambiguities detected: ${classification.ambiguities.join(', ')}`
        : 'Multiple similar matches, need user input',
      ambiguities: classification.ambiguities
    };
  }

  // Very low confidence
  return {
    action: 'create_new',
    reasoning: `Best match score (${topScore.toFixed(2)}) is too low`,
    recommendation: 'Create new skill using meta-skill builder',
    closest_match: topSkill.skill
  };
}

function generateSkillSummary(skill) {
  const parsed = YAML.parse(skill.yaml_content);

  return {
    skill_id: skill.skill_id,
    name: skill.skill_name,
    description: `${parsed.skill.identity.style} ${parsed.skill.identity.post_type} post for ${parsed.skill.identity.platform}`,
    target_audience: parsed.skill.identity.target_audience.primary,
    usage_count: skill.usage_count,
    success_rate: skill.success_rate
  };
}
```

### User Interaction for Ambiguous Cases

```javascript
async function askUserForClarification(decision, classification) {
  if (decision.action === 'ask_user') {
    // Present options to user
    const message = `I found ${decision.options.length} skills that might match your request:\n\n`;

    decision.options.forEach((option, i) => {
      message += `${i + 1}. ${option.summary.name}\n`;
      message += `   - ${option.summary.description}\n`;
      message += `   - Used ${option.summary.usage_count} times (${(option.summary.success_rate * 100).toFixed(0)}% success)\n`;
      message += `   - Match score: ${(option.score * 100).toFixed(0)}%\n\n`;
    });

    if (decision.ambiguities.length > 0) {
      message += `⚠️ Note: ${decision.ambiguities.join(', ')}\n\n`;
    }

    message += `Which skill would you like to use? (1-${decision.options.length}) Or type 'new' to create a new skill.`;

    // Send to user, wait for response
    const userChoice = await getUserResponse(message);

    if (userChoice === 'new') {
      return { action: 'create_new' };
    }

    const choiceIndex = parseInt(userChoice) - 1;
    if (choiceIndex >= 0 && choiceIndex < decision.options.length) {
      return {
        action: 'auto_execute',
        skill: decision.options[choiceIndex].skill,
        user_selected: true
      };
    }

    return { action: 'error', message: 'Invalid choice' };
  }

  return decision;
}
```

---

## Step 5: Execute

### Purpose

Load the selected skill and proceed with content generation.

### Skill Loading

```javascript
// skill-executor.js

export async function executeSkill(skillId, userRequest, classification, env) {
  // Step 1: Load skill (check cache first)
  let skillYAML = await env.SKILL_CACHE.get(`skill:${skillId}`);

  if (!skillYAML) {
    // Cache miss, load from database
    const result = await env.DB.prepare(`
      SELECT yaml_content FROM skills WHERE skill_id = ?
    `).bind(skillId).first();

    if (!result) {
      throw new Error(`Skill ${skillId} not found`);
    }

    skillYAML = result.yaml_content;

    // Cache for next time
    await env.SKILL_CACHE.put(`skill:${skillId}`, skillYAML, {
      expirationTtl: 86400 // 24 hours
    });
  }

  // Step 2: Parse skill
  const skill = YAML.parse(skillYAML);

  // Step 3: Prepare execution context
  const executionContext = {
    skill: skill,
    user_request: userRequest,
    classification: classification,
    product: classification.product,
    timestamp: new Date().toISOString()
  };

  // Step 4: Proceed to skill execution engine
  // (See skill-execution-engine.md for details)
  const result = await runSkillExecutionEngine(executionContext, env);

  // Step 5: Record usage
  await recordSkillUsage(skillId, result.success, env);

  return result;
}

async function recordSkillUsage(skillId, success, env) {
  // Increment usage count
  await env.DB.prepare(`
    UPDATE skills
    SET usage_count = usage_count + 1,
        updated_at = CURRENT_TIMESTAMP
    WHERE skill_id = ?
  `).bind(skillId).run();

  // Record in usage log
  await env.DB.prepare(`
    INSERT INTO skill_usage (skill_id, success, created_at)
    VALUES (?, ?, CURRENT_TIMESTAMP)
  `).bind(skillId, success ? 1 : 0).run();
}
```

---

## Complete Examples

### Example 1: Clear Educational Request

**User Request:**
> "Create educational post about Product T12 for LinkedIn"

#### Step 1: Intent Classification
```json
{
  "post_type": "educational",
  "post_type_confidence": 0.98,
  "product": "Product T12",
  "platform": "linkedin",
  "platform_confidence": 1.0,
  "style": "professional",
  "style_confidence": 0.85,
  "ambiguities": []
}
```

#### Step 2: Skill Search
```
Exact match search:
  post_type = 'educational' AND platform = 'linkedin' AND style = 'professional'
  → Found 3 skills
```

#### Step 3: Ranking
```
Skill 1: "Educational Product Post - LinkedIn Professional"
  - match_quality: 0.95 (exact match)
  - confidence: 0.87
  - usage: 0.82 (42 uses)
  - recency: 0.95 (created 10 days ago)
  - success_rate: 0.91
  → FINAL SCORE: 0.89

Skill 2: "Educational Technical Post - LinkedIn"
  - match_quality: 0.88 (platform + type match, style slightly different)
  - confidence: 0.79
  - usage: 0.45 (8 uses)
  - recency: 0.60 (created 80 days ago)
  - success_rate: 0.85
  → FINAL SCORE: 0.76

Skill 3: "Educational Explainer - LinkedIn Professional"
  - match_quality: 0.90
  - confidence: 0.65
  - usage: 0.30 (3 uses)
  - recency: 0.99 (created yesterday)
  - success_rate: 1.00
  → FINAL SCORE: 0.72
```

#### Step 4: Confidence Check
```
Top score: 0.89 ≥ 0.80
No ambiguities
→ ACTION: Auto-execute
```

#### Step 5: Execute
```
Load Skill 1 ("Educational Product Post - LinkedIn Professional")
Proceed to content generation
```

**Result:** ✅ Content generated automatically, user sees preview

---

### Example 2: Ambiguous Promotional Request

**User Request:**
> "Make a post about our Product Line B sale"

#### Step 1: Intent Classification
```json
{
  "post_type": "promotional",
  "post_type_confidence": 0.92,
  "product": "Product Line B",
  "platform": null,
  "platform_confidence": 0.0,
  "style": "urgent",
  "style_confidence": 0.65,
  "ambiguities": [
    "Platform not specified",
    "Which Product Line B items are on sale?"
  ]
}
```

#### Step 2: Skill Search
```
Partial match search (platform unknown):
  post_type = 'promotional'
  → Found 8 skills across platforms
```

#### Step 3: Ranking
```
Skill 1: "Promotional Flash Sale - LinkedIn"
  → FINAL SCORE: 0.68

Skill 2: "Promotional Urgent - Twitter"
  → FINAL SCORE: 0.62

Skill 3: "Promotional Sale - Instagram"
  → FINAL SCORE: 0.58
```

#### Step 4: Confidence Check
```
Top score: 0.68 (between 0.50-0.79)
HAS ambiguities: ["Platform not specified", "Which products?"]
→ ACTION: Ask user
```

#### User Interaction
```
Agent: "I found 3 promotional skills that might match:

1. Promotional Flash Sale - LinkedIn
   - urgent promotional post for linkedin
   - Used 15 times (87% success)
   - Match score: 68%

2. Promotional Urgent - Twitter
   - urgent promotional post for twitter
   - Used 8 times (75% success)
   - Match score: 62%

3. Promotional Sale - Instagram
   - urgent promotional post for instagram
   - Used 12 times (83% success)
   - Match score: 58%

⚠️ Note: Platform not specified, Which Product Line B items are on sale?

Which platform would you like to use? (1-3)"

User: "1"

Agent: "Great! Which Product Line B items are on sale? (e.g., Product T12, Product SKU-A142, Grade 60)"

User: "Grade 60 Product Line B, all sizes"
```

#### Step 5: Execute
```
Load Skill 1 ("Promotional Flash Sale - LinkedIn")
Add product context: "Grade 60 Product Line B, all sizes"
Proceed to content generation
```

**Result:** ✅ Content generated after user clarification

---

### Example 3: No Match - Create New Skill

**User Request:**
> "Write a humorous story about how our delivery driver saved a project"

#### Step 1: Intent Classification
```json
{
  "post_type": "story",
  "post_type_confidence": 0.95,
  "product": null,
  "platform": null,
  "platform_confidence": 0.0,
  "style": "humorous",
  "style_confidence": 0.92,
  "ambiguities": ["Platform not specified"]
}
```

#### Step 2: Skill Search
```
Broad match search:
  post_type = 'story'
  → Found 1 skill

Skill: "Customer Success Story - LinkedIn Professional"
  - style: "professional" (not humorous)
```

#### Step 3: Ranking
```
Skill 1: "Customer Success Story - LinkedIn Professional"
  - match_quality: 0.40 (post_type match, but style mismatch)
  - confidence: 0.82
  - usage: 0.55
  - recency: 0.70
  - success_rate: 0.88
  → FINAL SCORE: 0.52
```

#### Step 4: Confidence Check
```
Top score: 0.52 (just above threshold)
BUT: Large style mismatch (professional vs humorous)
HAS ambiguities: ["Platform not specified"]
→ ACTION: Create new skill (style is critical, mismatch is significant)
```

#### User Interaction
```
Agent: "I found a 'Customer Success Story' skill, but it's professional tone, not humorous.

Would you like to:
A) Create a new 'Humorous Story' skill (recommended)
B) Use the existing professional story skill
C) Provide example humorous posts to build skill from

Which option?"

User: "C"

Agent: "Great! Please upload 3-5 example posts with humorous tone, and I'll build a custom skill for you."
```

**Result:** User provides examples → Meta-skill builder creates new skill

---

## Summary

### Skill Matching Flow

```
Natural Language Request
    ↓
Intent Classification (Claude Sonnet 4.5)
    ↓
Database Search (SQL + Vectorize)
    ↓
Ranking Algorithm (weighted scoring)
    ↓
Confidence Check (thresholds)
    ↓
Execute / Ask / Create
```

### Key Performance Metrics

- **Intent classification accuracy**: Target >90%
- **Search recall**: Find relevant skills >95% of the time
- **Ranking precision**: Top match is correct >85% of the time
- **Total matching time**: <500ms (excluding Claude Sonnet 4.5 calls)

### Optimization Opportunities

1. **Cache frequent intent classifications** (similar requests → same classification)
2. **Pre-compute skill embeddings** for faster semantic search
3. **Learn from user corrections** (when user picks option 2 instead of option 1, adjust ranking)
4. **A/B test ranking weights** to optimize for approval rate

### Next Steps

1. Implement intent classification with Claude Sonnet 4.5
2. Build SQL search queries with proper indexes
3. Develop ranking algorithm with all 5 components
4. Create confidence threshold logic
5. Build user interaction flows for ambiguous cases
6. Test with real user requests and iterate

---

**Document prepared by**: Skill Matching Algorithm Team
**For project**: DIFY-AGENT
**Status**: DRAFT for review and approval
**Related documents**: `skill-file-format.md`, `meta-skill-builder.md`, `skill-execution-engine.md`
