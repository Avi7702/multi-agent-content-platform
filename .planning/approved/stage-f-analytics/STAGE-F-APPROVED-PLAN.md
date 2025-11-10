# Stage F: Analytics & Performance Tracking System - APPROVED PLAN

**Version**: 1.0.0
**Status**: ✅ APPROVED
**Date**: November 10, 2025
**Stage**: F of 12 (Analytics & Performance)
**Dependencies**: Stage D (Content Generation), Stage E (Multi-Platform Publishing)

---

## Executive Summary

### Purpose
Build a comprehensive analytics and performance tracking system that collects, aggregates, and analyzes content performance across all 4 social media platforms (LinkedIn, Twitter/X, Facebook, Instagram), providing actionable insights for content optimization, ROI measurement, and brand voice effectiveness.

### Key Deliverables
1. **Analytics Collection Service** - Automated data collection from 4 platform APIs
2. **Performance Metrics Engine** - KPI calculation and scoring system
3. **Dashboard & Reporting** - Real-time analytics dashboard
4. **A/B Testing Infrastructure** - Multi-candidate comparison and statistical analysis
5. **Content Performance Scorer** - ML-based content effectiveness prediction
6. **ROI Tracking System** - Business impact measurement
7. **Engagement Analytics** - Deep engagement pattern analysis
8. **Brand Voice Effectiveness** - Measure brand voice impact on performance

### Timeline
**4 weeks** (20 working days)

### Budget
- **Development**: $120,000 (320 engineering hours @ $375/hour)
- **Monthly Operational**: $89/month ($0.89/customer/month for 100 customers)
  - Platform API calls: $30/month
  - Cloudflare Workers/D1: $25/month
  - OpenAI embeddings: $20/month (analytics insights)
  - Monitoring: $14/month

### Success Criteria
- ✅ Real-time analytics from all 4 platforms (sync every 15 minutes)
- ✅ Historical data retention (12 months minimum)
- ✅ Dashboard load time <2 seconds
- ✅ A/B test statistical significance detection (95% confidence)
- ✅ Content performance prediction accuracy >75%
- ✅ ROI attribution working for all content types
- ✅ Engagement pattern insights available within 24 hours

---

## Table of Contents

1. [System Architecture](#1-system-architecture)
2. [Analytics Collection Service](#2-analytics-collection-service)
3. [Performance Metrics Engine](#3-performance-metrics-engine)
4. [Dashboard & Reporting](#4-dashboard--reporting)
5. [A/B Testing Infrastructure](#5-ab-testing-infrastructure)
6. [Content Performance Scorer](#6-content-performance-scorer)
7. [ROI Tracking System](#7-roi-tracking-system)
8. [Engagement Analytics](#8-engagement-analytics)
9. [Brand Voice Effectiveness](#9-brand-voice-effectiveness)
10. [Database Schema](#10-database-schema)
11. [API Specifications](#11-api-specifications)
12. [Cost Analysis](#12-cost-analysis)
13. [Implementation Timeline](#13-implementation-timeline)

---

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                      Analytics & Performance System               │
└──────────────────────────────────────────────────────────────────┘
                                  │
                    ┌─────────────┴──────────────┐
                    │                            │
          ┌─────────▼─────────┐      ┌──────────▼──────────┐
          │ Analytics Service │      │  Dashboard Service   │
          │  (Data Collection)│      │   (Visualization)    │
          └─────────┬─────────┘      └──────────┬──────────┘
                    │                            │
        ┌───────────┼───────────┬────────────────┘
        │           │           │
┌───────▼───┐ ┌────▼────┐ ┌────▼─────┐
│ LinkedIn  │ │Twitter/X│ │ Facebook │
│    API    │ │   API   │ │Instagram │
└───────────┘ └─────────┘ └──────────┘
        │           │           │
        └───────────┼───────────┘
                    │
        ┌───────────▼────────────┐
        │  Cloudflare D1 (DB)    │
        │  - post_analytics      │
        │  - performance_metrics │
        │  - ab_tests            │
        │  - engagement_patterns │
        └────────────────────────┘
```

### 1.2 Service Components

#### Analytics Collection Service
- **Purpose**: Fetch analytics data from platform APIs on scheduled intervals
- **Technology**: Cloudflare Workers + Cron Triggers
- **Schedule**: Every 15 minutes for recent posts, hourly for older posts
- **Features**:
  - Incremental sync (only fetch updated data)
  - Rate limit aware (respects platform limits)
  - Error recovery (retry failed syncs)
  - Batch processing (100 posts per batch)

#### Performance Metrics Engine
- **Purpose**: Calculate KPIs, aggregate metrics, detect anomalies
- **Technology**: Cloudflare Workers + D1
- **Features**:
  - Real-time KPI calculation (engagement rate, reach, virality)
  - Trend detection (growing/declining performance)
  - Anomaly detection (outlier identification)
  - Benchmark comparison (vs historical average)

#### Dashboard Service
- **Purpose**: Serve analytics dashboard and reports
- **Technology**: Cloudflare Pages + Workers
- **Features**:
  - Real-time charts (Chart.js/Recharts)
  - Drill-down capabilities (platform → post → engagement detail)
  - Export to PDF/CSV
  - Custom date ranges

#### A/B Testing Infrastructure
- **Purpose**: Compare multi-candidate performance with statistical significance
- **Technology**: Cloudflare Workers + D1
- **Features**:
  - Chi-square test for categorical data
  - T-test for continuous metrics
  - Confidence interval calculation
  - Winner detection (95% confidence threshold)

#### Content Performance Scorer
- **Purpose**: Predict content performance before publishing
- **Technology**: OpenAI embeddings + similarity scoring
- **Features**:
  - Historical performance lookup (similar content)
  - ML-based prediction (GPT-5 analysis)
  - Confidence scoring
  - Improvement suggestions

#### ROI Tracking System
- **Purpose**: Measure business impact and attribution
- **Technology**: Cloudflare Workers + D1
- **Features**:
  - UTM parameter tracking
  - Conversion attribution (first-touch, last-touch, multi-touch)
  - Cost per engagement (CPE)
  - Revenue attribution (if integrated with CRM)

#### Engagement Analytics
- **Purpose**: Deep analysis of engagement patterns
- **Technology**: Cloudflare Workers + D1
- **Features**:
  - Time-of-day analysis (best posting times)
  - Audience demographics (platform insights)
  - Engagement decay curves
  - Viral coefficient calculation

#### Brand Voice Effectiveness
- **Purpose**: Measure impact of brand voice on performance
- **Technology**: OpenAI embeddings + correlation analysis
- **Features**:
  - Brand voice adherence score (from Stage D)
  - Performance correlation (voice score vs engagement)
  - A/B test brand voice variations
  - Voice drift detection

---

## 2. Analytics Collection Service

### 2.1 Platform API Integration

#### LinkedIn Analytics API
```typescript
interface LinkedInPostAnalytics {
  postId: string;
  organizationUrn: string;

  // Engagement Metrics
  likes: number;
  comments: number;
  shares: number;
  clicks: number;

  // Reach Metrics
  impressions: number;
  uniqueImpressions: number;

  // Engagement Rate
  engagementRate: number; // (likes + comments + shares + clicks) / impressions

  // Demographics (if Business account)
  demographicsByRegion?: Record<string, number>;
  demographicsByIndustry?: Record<string, number>;
  demographicsBySeniority?: Record<string, number>;

  // Timestamp
  lastSyncedAt: string;
}

async function fetchLinkedInAnalytics(
  postId: string,
  credentials: OAuthTokens
): Promise<LinkedInPostAnalytics> {
  const response = await fetch(
    `https://api.linkedin.com/v2/organizationalEntityShareStatistics?q=organizationalEntity&organizationalEntity=${credentials.organizationUrn}&shares=List(${postId})`,
    {
      headers: {
        'Authorization': `Bearer ${credentials.accessToken}`,
        'X-Restli-Protocol-Version': '2.0.0'
      }
    }
  );

  const data = await response.json();

  return {
    postId,
    organizationUrn: credentials.organizationUrn,
    likes: data.totalShareStatistics.likeCount,
    comments: data.totalShareStatistics.commentCount,
    shares: data.totalShareStatistics.shareCount,
    clicks: data.totalShareStatistics.clickCount,
    impressions: data.totalShareStatistics.impressionCount,
    uniqueImpressions: data.totalShareStatistics.uniqueImpressionsCount,
    engagementRate: calculateEngagementRate(data.totalShareStatistics),
    lastSyncedAt: new Date().toISOString()
  };
}
```

#### Twitter/X Analytics API
```typescript
interface TwitterPostAnalytics {
  postId: string;

  // Engagement Metrics
  likes: number;
  retweets: number;
  replies: number;
  quotes: number;
  bookmarks: number;

  // Reach Metrics
  impressions: number;
  urlLinkClicks: number;
  userProfileClicks: number;

  // Video Metrics (if video post)
  videoViews?: number;
  videoViewQuartiles?: {
    views25: number;
    views50: number;
    views75: number;
    views100: number;
  };

  // Engagement Rate
  engagementRate: number;

  lastSyncedAt: string;
}

async function fetchTwitterAnalytics(
  postId: string,
  credentials: OAuthTokens
): Promise<TwitterPostAnalytics> {
  const response = await fetch(
    `https://api.twitter.com/2/tweets/${postId}?tweet.fields=public_metrics,non_public_metrics,organic_metrics`,
    {
      headers: {
        'Authorization': `Bearer ${credentials.accessToken}`
      }
    }
  );

  const data = await response.json();
  const metrics = data.data.organic_metrics || data.data.public_metrics;

  return {
    postId,
    likes: metrics.like_count,
    retweets: metrics.retweet_count,
    replies: metrics.reply_count,
    quotes: metrics.quote_count,
    bookmarks: metrics.bookmark_count || 0,
    impressions: metrics.impression_count,
    urlLinkClicks: metrics.url_link_clicks || 0,
    userProfileClicks: metrics.user_profile_clicks || 0,
    videoViews: metrics.video_view_count,
    engagementRate: calculateEngagementRate({
      engagements: metrics.like_count + metrics.retweet_count + metrics.reply_count + metrics.quote_count,
      impressions: metrics.impression_count
    }),
    lastSyncedAt: new Date().toISOString()
  };
}
```

#### Facebook/Instagram Analytics API
```typescript
interface FacebookInstagramPostAnalytics {
  postId: string;
  platform: 'facebook' | 'instagram';

  // Engagement Metrics
  likes: number;
  comments: number;
  shares: number; // Facebook only
  saves: number; // Instagram only

  // Reach Metrics
  impressions: number;
  reach: number;

  // Video Metrics (if video)
  videoViews?: number;
  videoAvgTimeWatched?: number;
  videoCompleteViews?: number;

  // Story Metrics (Instagram only)
  storyTapForward?: number;
  storyTapBack?: number;
  storyExits?: number;

  // Demographics
  demographicsByAge?: Record<string, number>;
  demographicsByGender?: Record<string, number>;
  demographicsByCountry?: Record<string, number>;

  engagementRate: number;
  lastSyncedAt: string;
}

async function fetchFacebookInstagramAnalytics(
  postId: string,
  platform: 'facebook' | 'instagram',
  credentials: OAuthTokens
): Promise<FacebookInstagramPostAnalytics> {
  const insightsEndpoint = platform === 'facebook'
    ? `https://graph.facebook.com/v18.0/${postId}/insights?metric=post_impressions,post_impressions_unique,post_engaged_users,post_clicks,post_reactions_like_total,post_video_views`
    : `https://graph.facebook.com/v18.0/${postId}/insights?metric=impressions,reach,engagement,likes,comments,saves`;

  const response = await fetch(
    `${insightsEndpoint}&access_token=${credentials.accessToken}`
  );

  const data = await response.json();

  // Parse insights data
  const metrics = parseInsightsData(data.data);

  return {
    postId,
    platform,
    likes: metrics.likes,
    comments: metrics.comments,
    shares: platform === 'facebook' ? metrics.shares : 0,
    saves: platform === 'instagram' ? metrics.saves : 0,
    impressions: metrics.impressions,
    reach: metrics.reach,
    engagementRate: calculateEngagementRate(metrics),
    lastSyncedAt: new Date().toISOString()
  };
}
```

### 2.2 Sync Scheduling

#### Cron Schedule
```typescript
// Cloudflare Workers Cron Trigger
export default {
  async scheduled(event: ScheduledEvent, env: Env, ctx: ExecutionContext) {
    ctx.waitUntil(syncAnalytics(event.cron, env));
  }
};

async function syncAnalytics(cron: string, env: Env) {
  switch (cron) {
    case '*/15 * * * *': // Every 15 minutes
      await syncRecentPosts(env); // Posts published in last 7 days
      break;

    case '0 * * * *': // Hourly
      await syncHistoricalPosts(env); // Posts older than 7 days
      break;

    case '0 0 * * *': // Daily at midnight
      await calculateDailyAggregates(env);
      await detectTrends(env);
      break;

    case '0 0 * * 0': // Weekly on Sunday
      await calculateWeeklyReports(env);
      await cleanupOldData(env); // Archive data older than 12 months
      break;
  }
}

async function syncRecentPosts(env: Env) {
  // Fetch posts published in last 7 days
  const recentPosts = await env.DB.prepare(`
    SELECT pp.*, ot.platform, ot.access_token_encrypted
    FROM published_posts pp
    JOIN oauth_tokens ot ON pp.tenant_id = ot.tenant_id AND pp.platform = ot.platform
    WHERE pp.published_at >= datetime('now', '-7 days')
    AND pp.status = 'published'
    ORDER BY pp.published_at DESC
  `).all();

  // Batch sync (100 posts at a time)
  const batches = chunk(recentPosts.results, 100);

  for (const batch of batches) {
    await Promise.all(
      batch.map(post => syncPostAnalytics(post, env))
    );
  }
}

async function syncPostAnalytics(post: PublishedPost, env: Env) {
  try {
    let analytics;

    switch (post.platform) {
      case 'linkedin':
        analytics = await fetchLinkedInAnalytics(post.platform_post_id, {
          accessToken: decrypt(post.access_token_encrypted, env),
          organizationUrn: post.organization_urn
        });
        break;

      case 'twitter':
        analytics = await fetchTwitterAnalytics(post.platform_post_id, {
          accessToken: decrypt(post.access_token_encrypted, env)
        });
        break;

      case 'facebook':
      case 'instagram':
        analytics = await fetchFacebookInstagramAnalytics(
          post.platform_post_id,
          post.platform,
          { accessToken: decrypt(post.access_token_encrypted, env) }
        );
        break;
    }

    // Upsert analytics data
    await env.DB.prepare(`
      INSERT INTO post_analytics (
        post_id, tenant_id, platform,
        likes, comments, shares, clicks,
        impressions, reach,
        engagement_rate,
        synced_at
      ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
      ON CONFLICT(post_id) DO UPDATE SET
        likes = excluded.likes,
        comments = excluded.comments,
        shares = excluded.shares,
        clicks = excluded.clicks,
        impressions = excluded.impressions,
        reach = excluded.reach,
        engagement_rate = excluded.engagement_rate,
        synced_at = excluded.synced_at
    `).bind(
      post.id,
      post.tenant_id,
      post.platform,
      analytics.likes,
      analytics.comments,
      analytics.shares || 0,
      analytics.clicks || 0,
      analytics.impressions,
      analytics.reach || analytics.uniqueImpressions,
      analytics.engagementRate,
      analytics.lastSyncedAt
    ).run();

  } catch (error) {
    // Log error and continue
    console.error(`Failed to sync analytics for post ${post.id}:`, error);

    // Record sync failure
    await env.DB.prepare(`
      INSERT INTO analytics_sync_errors (post_id, platform, error_message, created_at)
      VALUES (?, ?, ?, ?)
    `).bind(
      post.id,
      post.platform,
      error.message,
      new Date().toISOString()
    ).run();
  }
}
```

### 2.3 Incremental Sync Strategy

Only fetch updated data to minimize API calls and costs:

```typescript
async function getLastSyncTimestamp(postId: string, env: Env): Promise<Date | null> {
  const result = await env.DB.prepare(`
    SELECT synced_at FROM post_analytics WHERE post_id = ? ORDER BY synced_at DESC LIMIT 1
  `).bind(postId).first();

  return result ? new Date(result.synced_at) : null;
}

async function shouldSyncPost(post: PublishedPost, env: Env): Promise<boolean> {
  const lastSync = await getLastSyncTimestamp(post.id, env);

  if (!lastSync) return true; // Never synced, sync now

  const postAge = Date.now() - new Date(post.published_at).getTime();
  const timeSinceSync = Date.now() - lastSync.getTime();

  // Sync frequency based on post age
  if (postAge < 24 * 60 * 60 * 1000) { // < 1 day old
    return timeSinceSync > 15 * 60 * 1000; // Sync every 15 min
  } else if (postAge < 7 * 24 * 60 * 60 * 1000) { // < 7 days old
    return timeSinceSync > 60 * 60 * 1000; // Sync hourly
  } else if (postAge < 30 * 24 * 60 * 60 * 1000) { // < 30 days old
    return timeSinceSync > 24 * 60 * 60 * 1000; // Sync daily
  } else {
    return timeSinceSync > 7 * 24 * 60 * 60 * 1000; // Sync weekly
  }
}
```

---

## 3. Performance Metrics Engine

### 3.1 KPI Calculations

#### Engagement Rate
```typescript
function calculateEngagementRate(metrics: {
  likes: number;
  comments: number;
  shares: number;
  clicks: number;
  impressions: number;
}): number {
  const totalEngagements = metrics.likes + metrics.comments + metrics.shares + metrics.clicks;
  return metrics.impressions > 0 ? (totalEngagements / metrics.impressions) * 100 : 0;
}
```

#### Virality Coefficient
```typescript
function calculateViralityCoefficient(metrics: {
  shares: number;
  reach: number;
  followers: number;
}): number {
  // How many new people see the post per follower
  // VC > 1 means viral (exponential growth)
  return metrics.reach / metrics.followers;
}
```

#### Engagement Velocity
```typescript
function calculateEngagementVelocity(
  currentEngagements: number,
  previousEngagements: number,
  timeDeltaHours: number
): number {
  // Engagements per hour
  return (currentEngagements - previousEngagements) / timeDeltaHours;
}
```

#### Content Performance Score (CPS)
```typescript
interface ContentPerformanceScore {
  overall: number; // 0-100
  breakdown: {
    engagement: number; // 0-100
    reach: number; // 0-100
    virality: number; // 0-100
    longevity: number; // 0-100
  };
}

function calculateContentPerformanceScore(
  analytics: PostAnalytics,
  benchmarks: PlatformBenchmarks
): ContentPerformanceScore {
  // Normalize each metric against platform benchmarks
  const engagementScore = normalizeScore(
    analytics.engagement_rate,
    benchmarks.avgEngagementRate,
    benchmarks.p90EngagementRate
  );

  const reachScore = normalizeScore(
    analytics.reach,
    benchmarks.avgReach,
    benchmarks.p90Reach
  );

  const viralityScore = normalizeScore(
    analytics.shares / analytics.impressions,
    benchmarks.avgShareRate,
    benchmarks.p90ShareRate
  );

  const longevityScore = calculateLongevityScore(
    analytics.engagement_velocity_24h,
    analytics.engagement_velocity_7d
  );

  // Weighted average (engagement 40%, reach 30%, virality 20%, longevity 10%)
  const overall = (
    engagementScore * 0.4 +
    reachScore * 0.3 +
    viralityScore * 0.2 +
    longevityScore * 0.1
  );

  return {
    overall: Math.round(overall),
    breakdown: {
      engagement: Math.round(engagementScore),
      reach: Math.round(reachScore),
      virality: Math.round(viralityScore),
      longevity: Math.round(longevityScore)
    }
  };
}

function normalizeScore(value: number, avg: number, p90: number): number {
  // 0-50: Below average
  // 50: At average
  // 50-90: Above average
  // 90-100: Top 10%
  if (value <= avg) {
    return (value / avg) * 50;
  } else if (value <= p90) {
    return 50 + ((value - avg) / (p90 - avg)) * 40;
  } else {
    return 90 + Math.min((value - p90) / p90, 1) * 10;
  }
}
```

### 3.2 Benchmark Calculation

Calculate rolling benchmarks per tenant per platform:

```typescript
async function calculateBenchmarks(
  tenantId: string,
  platform: string,
  env: Env
): Promise<PlatformBenchmarks> {
  // Calculate from last 90 days of data
  const stats = await env.DB.prepare(`
    SELECT
      AVG(engagement_rate) as avg_engagement_rate,
      PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY engagement_rate) as median_engagement_rate,
      PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY engagement_rate) as p90_engagement_rate,

      AVG(reach) as avg_reach,
      PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY reach) as p90_reach,

      AVG(CAST(shares AS FLOAT) / NULLIF(impressions, 0)) as avg_share_rate,
      PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY CAST(shares AS FLOAT) / NULLIF(impressions, 0)) as p90_share_rate,

      COUNT(*) as total_posts
    FROM post_analytics
    WHERE tenant_id = ?
    AND platform = ?
    AND synced_at >= datetime('now', '-90 days')
  `).bind(tenantId, platform).first();

  return {
    avgEngagementRate: stats.avg_engagement_rate || 2.0, // Default if no data
    medianEngagementRate: stats.median_engagement_rate || 1.5,
    p90EngagementRate: stats.p90_engagement_rate || 5.0,
    avgReach: stats.avg_reach || 1000,
    p90Reach: stats.p90_reach || 5000,
    avgShareRate: stats.avg_share_rate || 0.01,
    p90ShareRate: stats.p90_share_rate || 0.05,
    totalPosts: stats.total_posts || 0
  };
}
```

### 3.3 Trend Detection

Identify growing/declining performance:

```typescript
interface TrendAnalysis {
  direction: 'growing' | 'declining' | 'stable';
  strength: 'strong' | 'moderate' | 'weak';
  confidence: number; // 0-100
  changePercent: number;
  period: '7d' | '30d' | '90d';
}

async function detectTrend(
  tenantId: string,
  platform: string,
  metric: 'engagement_rate' | 'reach' | 'shares',
  period: '7d' | '30d' | '90d',
  env: Env
): Promise<TrendAnalysis> {
  const days = period === '7d' ? 7 : period === '30d' ? 30 : 90;

  // Get metric values over time
  const dataPoints = await env.DB.prepare(`
    SELECT DATE(synced_at) as date, AVG(${metric}) as value
    FROM post_analytics
    WHERE tenant_id = ? AND platform = ?
    AND synced_at >= datetime('now', '-${days} days')
    GROUP BY DATE(synced_at)
    ORDER BY date ASC
  `).bind(tenantId, platform).all();

  if (dataPoints.results.length < 3) {
    return {
      direction: 'stable',
      strength: 'weak',
      confidence: 0,
      changePercent: 0,
      period
    };
  }

  // Linear regression to detect trend
  const { slope, rSquared } = linearRegression(
    dataPoints.results.map((d, i) => ({ x: i, y: d.value }))
  );

  const firstValue = dataPoints.results[0].value;
  const lastValue = dataPoints.results[dataPoints.results.length - 1].value;
  const changePercent = ((lastValue - firstValue) / firstValue) * 100;

  // Determine direction
  let direction: 'growing' | 'declining' | 'stable';
  if (Math.abs(changePercent) < 5) {
    direction = 'stable';
  } else {
    direction = slope > 0 ? 'growing' : 'declining';
  }

  // Determine strength
  let strength: 'strong' | 'moderate' | 'weak';
  if (Math.abs(changePercent) > 20) {
    strength = 'strong';
  } else if (Math.abs(changePercent) > 10) {
    strength = 'moderate';
  } else {
    strength = 'weak';
  }

  return {
    direction,
    strength,
    confidence: Math.round(rSquared * 100), // R² as confidence
    changePercent: Math.round(changePercent * 10) / 10,
    period
  };
}
```

### 3.4 Anomaly Detection

Identify outlier posts (unusually high/low performance):

```typescript
interface AnomalyDetection {
  isAnomaly: boolean;
  type: 'high' | 'low' | 'normal';
  zScore: number;
  percentile: number;
  message: string;
}

async function detectAnomaly(
  postId: string,
  metric: 'engagement_rate' | 'reach',
  env: Env
): Promise<AnomalyDetection> {
  // Get post analytics
  const post = await env.DB.prepare(`
    SELECT * FROM post_analytics WHERE post_id = ?
  `).bind(postId).first();

  // Get tenant's historical stats for this platform
  const stats = await env.DB.prepare(`
    SELECT
      AVG(${metric}) as mean,
      STDEV(${metric}) as stddev
    FROM post_analytics
    WHERE tenant_id = ?
    AND platform = ?
    AND synced_at >= datetime('now', '-90 days')
  `).bind(post.tenant_id, post.platform).first();

  const zScore = (post[metric] - stats.mean) / stats.stddev;

  // Calculate percentile
  const percentile = await env.DB.prepare(`
    SELECT COUNT(*) * 100.0 / (SELECT COUNT(*) FROM post_analytics WHERE tenant_id = ? AND platform = ?) as percentile
    FROM post_analytics
    WHERE tenant_id = ? AND platform = ?
    AND ${metric} <= ?
  `).bind(
    post.tenant_id,
    post.platform,
    post.tenant_id,
    post.platform,
    post[metric]
  ).first();

  let type: 'high' | 'low' | 'normal';
  let isAnomaly = false;
  let message = '';

  if (zScore > 2) { // 2 standard deviations above mean
    type = 'high';
    isAnomaly = true;
    message = `Exceptionally high ${metric} (top ${100 - percentile.percentile}%)`;
  } else if (zScore < -2) {
    type = 'low';
    isAnomaly = true;
    message = `Exceptionally low ${metric} (bottom ${percentile.percentile}%)`;
  } else {
    type = 'normal';
    message = `Normal ${metric} performance`;
  }

  return {
    isAnomaly,
    type,
    zScore: Math.round(zScore * 100) / 100,
    percentile: Math.round(percentile.percentile),
    message
  };
}
```

---

## 4. Dashboard & Reporting

### 4.1 Dashboard Pages

#### Overview Dashboard
- **Metrics Displayed**:
  - Total posts (last 7/30/90 days)
  - Total engagement (likes + comments + shares)
  - Average engagement rate per platform
  - Total reach/impressions
  - Top performing posts (by CPS)
  - Worst performing posts
  - Trend charts (engagement over time)

#### Platform-Specific Dashboard
- **Per Platform**:
  - Platform-specific KPIs
  - Best posting times (heatmap)
  - Audience demographics
  - Content type performance (text-only vs image vs video)
  - Hashtag performance

#### Post Detail View
- **Per Post**:
  - All engagement metrics
  - Performance score breakdown
  - Anomaly detection results
  - Comparison to tenant average
  - Engagement timeline (hourly/daily)
  - Comments/replies preview

#### A/B Test Results Dashboard
- **Test Comparison**:
  - Side-by-side metrics
  - Statistical significance indicator
  - Winner declaration
  - Confidence level
  - Recommendation

### 4.2 API Endpoints

```typescript
// GET /api/analytics/overview
interface AnalyticsOverview {
  summary: {
    totalPosts: number;
    totalEngagements: number;
    avgEngagementRate: number;
    totalReach: number;
    totalImpressions: number;
  };

  byPlatform: {
    platform: string;
    posts: number;
    engagements: number;
    avgEngagementRate: number;
    reach: number;
  }[];

  topPosts: {
    postId: string;
    platform: string;
    contentPreview: string;
    performanceScore: number;
    engagements: number;
    publishedAt: string;
  }[];

  trends: {
    metric: string;
    direction: 'growing' | 'declining' | 'stable';
    changePercent: number;
  }[];
}

async function getAnalyticsOverview(
  tenantId: string,
  dateRange: { start: string; end: string },
  env: Env
): Promise<AnalyticsOverview> {
  // Implementation with D1 queries
}

// GET /api/analytics/platform/:platform
interface PlatformAnalytics {
  platform: string;
  summary: {
    posts: number;
    avgEngagementRate: number;
    reach: number;
  };

  bestPostingTimes: {
    hourOfDay: number;
    dayOfWeek: number;
    avgEngagementRate: number;
    postCount: number;
  }[];

  contentTypePerformance: {
    type: 'text' | 'image' | 'video';
    count: number;
    avgEngagementRate: number;
    avgReach: number;
  }[];

  topHashtags: {
    hashtag: string;
    uses: number;
    avgEngagementRate: number;
  }[];
}

// GET /api/analytics/post/:postId
interface PostAnalyticsDetail {
  post: PublishedPost;
  analytics: PostAnalytics;
  performanceScore: ContentPerformanceScore;
  anomalies: AnomalyDetection[];
  benchmarkComparison: {
    metric: string;
    value: number;
    tenantAverage: number;
    percentile: number;
  }[];
  engagementTimeline: {
    timestamp: string;
    likes: number;
    comments: number;
    shares: number;
    impressions: number;
  }[];
}
```

### 4.3 Real-Time Charts

Use Chart.js or Recharts for client-side visualization:

```typescript
// Example: Engagement over time line chart
interface EngagementTimelineData {
  labels: string[]; // Date labels
  datasets: {
    label: string; // Platform name
    data: number[]; // Engagement rates
    borderColor: string;
    backgroundColor: string;
  }[];
}

async function getEngagementTimeline(
  tenantId: string,
  dateRange: { start: string; end: string },
  env: Env
): Promise<EngagementTimelineData> {
  const dataPoints = await env.DB.prepare(`
    SELECT
      DATE(pa.synced_at) as date,
      pa.platform,
      AVG(pa.engagement_rate) as avg_engagement_rate
    FROM post_analytics pa
    WHERE pa.tenant_id = ?
    AND pa.synced_at BETWEEN ? AND ?
    GROUP BY DATE(pa.synced_at), pa.platform
    ORDER BY date ASC, platform ASC
  `).bind(tenantId, dateRange.start, dateRange.end).all();

  // Group by platform
  const platforms = ['linkedin', 'twitter', 'facebook', 'instagram'];
  const datasets = platforms.map(platform => {
    const platformData = dataPoints.results.filter(d => d.platform === platform);
    return {
      label: platform,
      data: platformData.map(d => d.avg_engagement_rate),
      borderColor: getPlatformColor(platform),
      backgroundColor: getPlatformColor(platform, 0.2)
    };
  });

  // Get unique dates for labels
  const labels = [...new Set(dataPoints.results.map(d => d.date))];

  return { labels, datasets };
}
```

---

## 5. A/B Testing Infrastructure

### 5.1 A/B Test Types

#### Content Variation Testing
- Test different text variations (tone, length, CTA)
- Test image variations (product, style, composition)
- Test posting times
- Test hashtags

#### Brand Voice Testing
- Test high brand voice adherence vs low
- Test formal vs casual tone
- Test technical vs accessible language

#### Multi-Candidate Comparison
- Compare 2-3 candidates from Stage D multi-candidate generation
- Identify winning candidate with statistical significance

### 5.2 Statistical Significance Testing

```typescript
interface ABTestResult {
  testId: string;
  variantA: {
    postId: string;
    impressions: number;
    engagements: number;
    engagementRate: number;
  };
  variantB: {
    postId: string;
    impressions: number;
    engagements: number;
    engagementRate: number;
  };

  winner: 'A' | 'B' | 'tie' | 'inconclusive';
  pValue: number;
  confidenceLevel: number; // 0-100
  uplift: number; // % improvement of winner over loser

  recommendation: string;
}

async function analyzeABTest(testId: string, env: Env): Promise<ABTestResult> {
  // Fetch test configuration
  const test = await env.DB.prepare(`
    SELECT * FROM ab_tests WHERE test_id = ?
  `).bind(testId).first();

  // Fetch analytics for both variants
  const variantA = await env.DB.prepare(`
    SELECT * FROM post_analytics WHERE post_id = ?
  `).bind(test.variant_a_post_id).first();

  const variantB = await env.DB.prepare(`
    SELECT * FROM post_analytics WHERE post_id = ?
  `).bind(test.variant_b_post_id).first();

  // Chi-square test for engagement rate
  const chiSquareResult = chiSquareTest({
    aEngaged: variantA.likes + variantA.comments + variantA.shares,
    aTotal: variantA.impressions,
    bEngaged: variantB.likes + variantB.comments + variantB.shares,
    bTotal: variantB.impressions
  });

  // Determine winner
  let winner: 'A' | 'B' | 'tie' | 'inconclusive';
  let uplift = 0;

  if (chiSquareResult.pValue > 0.05) {
    winner = 'inconclusive'; // Not statistically significant
  } else if (Math.abs(variantA.engagement_rate - variantB.engagement_rate) < 0.1) {
    winner = 'tie'; // Statistically significant but practically negligible
  } else {
    winner = variantA.engagement_rate > variantB.engagement_rate ? 'A' : 'B';
    const winnerRate = winner === 'A' ? variantA.engagement_rate : variantB.engagement_rate;
    const loserRate = winner === 'A' ? variantB.engagement_rate : variantA.engagement_rate;
    uplift = ((winnerRate - loserRate) / loserRate) * 100;
  }

  const recommendation = generateRecommendation(winner, uplift, chiSquareResult.pValue);

  return {
    testId,
    variantA: {
      postId: test.variant_a_post_id,
      impressions: variantA.impressions,
      engagements: variantA.likes + variantA.comments + variantA.shares,
      engagementRate: variantA.engagement_rate
    },
    variantB: {
      postId: test.variant_b_post_id,
      impressions: variantB.impressions,
      engagements: variantB.likes + variantB.comments + variantB.shares,
      engagementRate: variantB.engagement_rate
    },
    winner,
    pValue: chiSquareResult.pValue,
    confidenceLevel: (1 - chiSquareResult.pValue) * 100,
    uplift: Math.round(uplift * 10) / 10,
    recommendation
  };
}

function chiSquareTest(data: {
  aEngaged: number;
  aTotal: number;
  bEngaged: number;
  bTotal: number;
}): { chiSquare: number; pValue: number } {
  const { aEngaged, aTotal, bEngaged, bTotal } = data;

  const aNotEngaged = aTotal - aEngaged;
  const bNotEngaged = bTotal - bEngaged;

  const totalEngaged = aEngaged + bEngaged;
  const totalNotEngaged = aNotEngaged + bNotEngaged;
  const grandTotal = aTotal + bTotal;

  // Expected values
  const expectedAEngaged = (aTotal * totalEngaged) / grandTotal;
  const expectedANotEngaged = (aTotal * totalNotEngaged) / grandTotal;
  const expectedBEngaged = (bTotal * totalEngaged) / grandTotal;
  const expectedBNotEngaged = (bTotal * totalNotEngaged) / grandTotal;

  // Chi-square statistic
  const chiSquare =
    Math.pow(aEngaged - expectedAEngaged, 2) / expectedAEngaged +
    Math.pow(aNotEngaged - expectedANotEngaged, 2) / expectedANotEngaged +
    Math.pow(bEngaged - expectedBEngaged, 2) / expectedBEngaged +
    Math.pow(bNotEngaged - expectedBNotEngaged, 2) / expectedBNotEngaged;

  // Degrees of freedom = (rows - 1) * (cols - 1) = 1
  const df = 1;

  // p-value from chi-square distribution (approximation)
  const pValue = 1 - chiSquareCDF(chiSquare, df);

  return { chiSquare, pValue };
}

function generateRecommendation(
  winner: 'A' | 'B' | 'tie' | 'inconclusive',
  uplift: number,
  pValue: number
): string {
  if (winner === 'inconclusive') {
    return 'Not enough data to determine a winner. Continue test or increase sample size.';
  }

  if (winner === 'tie') {
    return 'Both variants perform similarly. Choose based on other factors (brand voice, cost, etc.).';
  }

  return `Variant ${winner} is the clear winner with ${uplift.toFixed(1)}% uplift (p=${pValue.toFixed(3)}). Use this variant for future posts.`;
}
```

---

## 6. Content Performance Scorer

### 6.1 Predictive Scoring

Predict performance before publishing using historical data:

```typescript
interface PerformancePrediction {
  predictedEngagementRate: number;
  confidence: number; // 0-100
  similarPosts: {
    postId: string;
    similarity: number;
    actualEngagementRate: number;
  }[];
  factors: {
    factor: string;
    impact: 'positive' | 'negative' | 'neutral';
    strength: number; // 0-100
  }[];
  recommendation: string;
}

async function predictPerformance(
  content: string,
  imageUrl: string,
  platform: string,
  tenantId: string,
  env: Env
): Promise<PerformancePrediction> {
  // Step 1: Generate embedding for content
  const embedding = await generateEmbedding(content, env);

  // Step 2: Find similar historical posts
  const similarPosts = await env.VECTORIZE.query(embedding, {
    topK: 10,
    filter: { tenant_id: tenantId, platform: platform }
  });

  // Step 3: Fetch analytics for similar posts
  const postIds = similarPosts.matches.map(m => m.id);
  const analytics = await env.DB.prepare(`
    SELECT post_id, engagement_rate, reach, impressions
    FROM post_analytics
    WHERE post_id IN (${postIds.map(() => '?').join(',')})
  `).bind(...postIds).all();

  // Step 4: Calculate weighted average (weight by similarity)
  let predictedEngagementRate = 0;
  let totalWeight = 0;

  const similarPostsData = similarPosts.matches.map((match, i) => {
    const postAnalytics = analytics.results.find(a => a.post_id === match.id);
    const weight = match.score; // Cosine similarity

    predictedEngagementRate += postAnalytics.engagement_rate * weight;
    totalWeight += weight;

    return {
      postId: match.id,
      similarity: match.score,
      actualEngagementRate: postAnalytics.engagement_rate
    };
  });

  predictedEngagementRate /= totalWeight;

  // Step 5: Analyze factors (using GPT-5)
  const factors = await analyzePerformanceFactors(content, imageUrl, platform, env);

  // Step 6: Adjust prediction based on factors
  let adjustedPrediction = predictedEngagementRate;
  factors.forEach(factor => {
    if (factor.impact === 'positive') {
      adjustedPrediction *= (1 + factor.strength / 100);
    } else if (factor.impact === 'negative') {
      adjustedPrediction *= (1 - factor.strength / 100);
    }
  });

  // Step 7: Calculate confidence (based on number of similar posts and avg similarity)
  const avgSimilarity = similarPostsData.reduce((sum, p) => sum + p.similarity, 0) / similarPostsData.length;
  const confidence = Math.min(avgSimilarity * 100, 95); // Cap at 95%

  return {
    predictedEngagementRate: Math.round(adjustedPrediction * 100) / 100,
    confidence: Math.round(confidence),
    similarPosts: similarPostsData.slice(0, 5), // Top 5
    factors,
    recommendation: generatePerformanceRecommendation(adjustedPrediction, factors)
  };
}

async function analyzePerformanceFactors(
  content: string,
  imageUrl: string,
  platform: string,
  env: Env
): Promise<PerformanceFactor[]> {
  const prompt = `Analyze this social media post for ${platform} and identify factors that will impact performance:

Content: "${content}"
Image URL: ${imageUrl}

For each factor, indicate:
1. Factor name
2. Impact (positive/negative/neutral)
3. Strength (0-100)

Consider:
- Content length (optimal for platform)
- Call-to-action presence
- Hashtag usage
- Question/engagement prompt
- Visual appeal
- Timeliness/relevance
- Emotional appeal
- Value proposition

Return as JSON array.`;

  const response = await env.OPENAI.chat.completions.create({
    model: 'gpt-4',
    messages: [{ role: 'user', content: prompt }],
    response_format: { type: 'json_object' }
  });

  return JSON.parse(response.choices[0].message.content).factors;
}
```

---

## 7. ROI Tracking System

### 7.1 UTM Parameter Tracking

```typescript
interface UTMParameters {
  source: string; // 'linkedin', 'twitter', etc.
  medium: string; // 'social'
  campaign: string; // Campaign name
  content: string; // Post ID
  term?: string; // Optional keywords
}

function generateUTMLink(
  baseUrl: string,
  postId: string,
  platform: string,
  campaignName: string
): string {
  const params = new URLSearchParams({
    utm_source: platform,
    utm_medium: 'social',
    utm_campaign: campaignName,
    utm_content: postId
  });

  return `${baseUrl}?${params.toString()}`;
}

// Track UTM conversions
async function trackConversion(
  utmSource: string,
  utmCampaign: string,
  utmContent: string, // Post ID
  conversionType: 'page_view' | 'form_submit' | 'purchase',
  value?: number,
  env: Env
): Promise<void> {
  await env.DB.prepare(`
    INSERT INTO conversions (
      post_id,
      conversion_type,
      value,
      tracked_at
    ) VALUES (?, ?, ?, ?)
  `).bind(
    utmContent,
    conversionType,
    value || 0,
    new Date().toISOString()
  ).run();
}
```

### 7.2 Cost Per Engagement (CPE)

```typescript
interface ROIMetrics {
  totalCost: number; // Content creation + distribution
  totalEngagements: number;
  costPerEngagement: number;
  totalConversions: number;
  costPerConversion: number;
  totalRevenue: number;
  roi: number; // (Revenue - Cost) / Cost
}

async function calculateROI(
  campaignId: string,
  env: Env
): Promise<ROIMetrics> {
  // Get all posts in campaign
  const posts = await env.DB.prepare(`
    SELECT * FROM published_posts WHERE campaign_id = ?
  `).bind(campaignId).all();

  // Get total analytics
  const postIds = posts.results.map(p => p.id);
  const analytics = await env.DB.prepare(`
    SELECT
      SUM(likes + comments + shares + clicks) as total_engagements,
      SUM(impressions) as total_impressions
    FROM post_analytics
    WHERE post_id IN (${postIds.map(() => '?').join(',')})
  `).bind(...postIds).first();

  // Get conversions
  const conversions = await env.DB.prepare(`
    SELECT
      COUNT(*) as total_conversions,
      SUM(value) as total_revenue
    FROM conversions
    WHERE post_id IN (${postIds.map(() => '?').join(',')})
  `).bind(...postIds).first();

  // Calculate costs (example: $50 per post creation, $10 per platform publish)
  const totalCost = posts.results.length * (50 + 10 * 4); // 4 platforms

  return {
    totalCost,
    totalEngagements: analytics.total_engagements,
    costPerEngagement: totalCost / analytics.total_engagements,
    totalConversions: conversions.total_conversions,
    costPerConversion: totalCost / conversions.total_conversions,
    totalRevenue: conversions.total_revenue,
    roi: ((conversions.total_revenue - totalCost) / totalCost) * 100
  };
}
```

### 7.3 Attribution Models

```typescript
type AttributionModel = 'first-touch' | 'last-touch' | 'linear' | 'time-decay';

async function attributeConversion(
  conversionId: string,
  model: AttributionModel,
  env: Env
): Promise<{ postId: string; attribution: number }[]> {
  // Get user journey (all posts user interacted with before conversion)
  const journey = await env.DB.prepare(`
    SELECT pi.post_id, pi.interacted_at, c.converted_at
    FROM post_interactions pi
    JOIN conversions c ON pi.user_id = c.user_id
    WHERE c.conversion_id = ?
    AND pi.interacted_at <= c.converted_at
    ORDER BY pi.interacted_at ASC
  `).bind(conversionId).all();

  if (journey.results.length === 0) return [];

  const attributions: { postId: string; attribution: number }[] = [];

  switch (model) {
    case 'first-touch':
      // 100% credit to first touchpoint
      attributions.push({
        postId: journey.results[0].post_id,
        attribution: 1.0
      });
      break;

    case 'last-touch':
      // 100% credit to last touchpoint
      attributions.push({
        postId: journey.results[journey.results.length - 1].post_id,
        attribution: 1.0
      });
      break;

    case 'linear':
      // Equal credit to all touchpoints
      const linearCredit = 1.0 / journey.results.length;
      journey.results.forEach(interaction => {
        attributions.push({
          postId: interaction.post_id,
          attribution: linearCredit
        });
      });
      break;

    case 'time-decay':
      // More credit to recent touchpoints (exponential decay)
      const halfLife = 7 * 24 * 60 * 60 * 1000; // 7 days in ms
      const conversionTime = new Date(journey.results[0].converted_at).getTime();

      let totalWeight = 0;
      const weights = journey.results.map(interaction => {
        const interactionTime = new Date(interaction.interacted_at).getTime();
        const timeDiff = conversionTime - interactionTime;
        const weight = Math.exp(-timeDiff / halfLife);
        totalWeight += weight;
        return { postId: interaction.post_id, weight };
      });

      weights.forEach(({ postId, weight }) => {
        attributions.push({
          postId,
          attribution: weight / totalWeight
        });
      });
      break;
  }

  return attributions;
}
```

---

## 8. Engagement Analytics

### 8.1 Best Posting Times Analysis

```typescript
interface BestPostingTimes {
  byHourOfDay: {
    hour: number; // 0-23
    avgEngagementRate: number;
    postCount: number;
    confidence: 'high' | 'medium' | 'low';
  }[];

  byDayOfWeek: {
    day: number; // 0 (Sunday) - 6 (Saturday)
    avgEngagementRate: number;
    postCount: number;
    confidence: 'high' | 'medium' | 'low';
  }[];

  optimal: {
    dayOfWeek: number;
    hourOfDay: number;
    expectedEngagementRate: number;
  };
}

async function analyzeBestPostingTimes(
  tenantId: string,
  platform: string,
  env: Env
): Promise<BestPostingTimes> {
  // Analyze by hour of day
  const byHour = await env.DB.prepare(`
    SELECT
      CAST(strftime('%H', pp.published_at) AS INTEGER) as hour,
      AVG(pa.engagement_rate) as avg_engagement_rate,
      COUNT(*) as post_count
    FROM published_posts pp
    JOIN post_analytics pa ON pp.id = pa.post_id
    WHERE pp.tenant_id = ? AND pp.platform = ?
    AND pp.published_at >= datetime('now', '-90 days')
    GROUP BY hour
    ORDER BY hour ASC
  `).bind(tenantId, platform).all();

  // Analyze by day of week
  const byDay = await env.DB.prepare(`
    SELECT
      CAST(strftime('%w', pp.published_at) AS INTEGER) as day,
      AVG(pa.engagement_rate) as avg_engagement_rate,
      COUNT(*) as post_count
    FROM published_posts pp
    JOIN post_analytics pa ON pp.id = pa.post_id
    WHERE pp.tenant_id = ? AND pp.platform = ?
    AND pp.published_at >= datetime('now', '-90 days')
    GROUP BY day
    ORDER BY day ASC
  `).bind(tenantId, platform).all();

  // Determine confidence level
  const getConfidence = (count: number) => {
    if (count >= 20) return 'high';
    if (count >= 10) return 'medium';
    return 'low';
  };

  const byHourOfDay = byHour.results.map(h => ({
    hour: h.hour,
    avgEngagementRate: h.avg_engagement_rate,
    postCount: h.post_count,
    confidence: getConfidence(h.post_count)
  }));

  const byDayOfWeek = byDay.results.map(d => ({
    day: d.day,
    avgEngagementRate: d.avg_engagement_rate,
    postCount: d.post_count,
    confidence: getConfidence(d.post_count)
  }));

  // Find optimal time (highest engagement rate with high confidence)
  const bestHour = byHourOfDay
    .filter(h => h.confidence !== 'low')
    .sort((a, b) => b.avgEngagementRate - a.avgEngagementRate)[0];

  const bestDay = byDayOfWeek
    .filter(d => d.confidence !== 'low')
    .sort((a, b) => b.avgEngagementRate - a.avgEngagementRate)[0];

  return {
    byHourOfDay,
    byDayOfWeek,
    optimal: {
      dayOfWeek: bestDay.day,
      hourOfDay: bestHour.hour,
      expectedEngagementRate: (bestDay.avgEngagementRate + bestHour.avgEngagementRate) / 2
    }
  };
}
```

### 8.2 Engagement Decay Curves

Measure how quickly engagement drops off after publishing:

```typescript
interface EngagementDecayCurve {
  platform: string;
  dataPoints: {
    hoursAfterPublish: number;
    cumulativeEngagementPercent: number; // 0-100
  }[];
  halfLife: number; // Hours until 50% of total engagement
  recommendation: string;
}

async function analyzeEngagementDecay(
  tenantId: string,
  platform: string,
  env: Env
): Promise<EngagementDecayCurve> {
  // Get posts with multiple analytics snapshots
  const posts = await env.DB.prepare(`
    SELECT
      pp.id as post_id,
      pp.published_at,
      pa.synced_at,
      pa.likes + pa.comments + pa.shares as total_engagements
    FROM published_posts pp
    JOIN post_analytics pa ON pp.id = pa.post_id
    WHERE pp.tenant_id = ? AND pp.platform = ?
    AND pp.published_at >= datetime('now', '-90 days')
    ORDER BY pp.id, pa.synced_at ASC
  `).bind(tenantId, platform).all();

  // Group by post and calculate cumulative engagement over time
  const postEngagement: Record<string, any[]> = {};
  posts.results.forEach(row => {
    if (!postEngagement[row.post_id]) {
      postEngagement[row.post_id] = [];
    }
    const hoursAfterPublish =
      (new Date(row.synced_at).getTime() - new Date(row.published_at).getTime()) / (1000 * 60 * 60);

    postEngagement[row.post_id].push({
      hoursAfterPublish,
      totalEngagements: row.total_engagements
    });
  });

  // Calculate average engagement curve
  const hourBuckets: Record<number, number[]> = {};
  Object.values(postEngagement).forEach(snapshots => {
    const finalEngagement = snapshots[snapshots.length - 1].totalEngagements;

    snapshots.forEach(snapshot => {
      const hour = Math.floor(snapshot.hoursAfterPublish);
      const percent = (snapshot.totalEngagements / finalEngagement) * 100;

      if (!hourBuckets[hour]) hourBuckets[hour] = [];
      hourBuckets[hour].push(percent);
    });
  });

  const dataPoints = Object.entries(hourBuckets)
    .map(([hour, percents]) => ({
      hoursAfterPublish: parseInt(hour),
      cumulativeEngagementPercent: percents.reduce((a, b) => a + b, 0) / percents.length
    }))
    .sort((a, b) => a.hoursAfterPublish - b.hoursAfterPublish);

  // Calculate half-life
  const halfLifePoint = dataPoints.find(dp => dp.cumulativeEngagementPercent >= 50);
  const halfLife = halfLifePoint?.hoursAfterPublish || 24;

  const recommendation = halfLife < 6
    ? 'Engagement drops quickly. Consider boosting posts or posting during peak hours.'
    : halfLife < 24
    ? 'Normal engagement decay. Monitor for 24 hours.'
    : 'Long engagement tail. Content has staying power.';

  return {
    platform,
    dataPoints,
    halfLife,
    recommendation
  };
}
```

### 8.3 Viral Coefficient Tracking

```typescript
interface ViralityMetrics {
  viralCoefficient: number; // New reach / Original followers
  isViral: boolean; // VC >= 1.0
  shareRate: number; // Shares / Impressions
  amplificationFactor: number; // Reach / Impressions
  secondaryEngagement: number; // Engagements from shared posts
}

async function analyzeVirality(
  postId: string,
  env: Env
): Promise<ViralityMetrics> {
  const analytics = await env.DB.prepare(`
    SELECT * FROM post_analytics WHERE post_id = ?
  `).bind(postId).first();

  const post = await env.DB.prepare(`
    SELECT * FROM published_posts WHERE id = ?
  `).bind(postId).first();

  // Get follower count at time of publish
  const followerCount = await getFollowerCount(post.tenant_id, post.platform, env);

  const viralCoefficient = analytics.reach / followerCount;
  const shareRate = (analytics.shares / analytics.impressions) * 100;
  const amplificationFactor = analytics.reach / analytics.impressions;

  // Estimate secondary engagement (engagements from shares)
  const primaryEngagement = analytics.likes + analytics.comments;
  const totalEngagement = primaryEngagement + (analytics.shares * 2); // Assume 2 engagements per share
  const secondaryEngagement = totalEngagement - primaryEngagement;

  return {
    viralCoefficient: Math.round(viralCoefficient * 100) / 100,
    isViral: viralCoefficient >= 1.0,
    shareRate: Math.round(shareRate * 100) / 100,
    amplificationFactor: Math.round(amplificationFactor * 100) / 100,
    secondaryEngagement
  };
}
```

---

## 9. Brand Voice Effectiveness

### 9.1 Voice Score Correlation

Measure correlation between brand voice adherence and performance:

```typescript
interface BrandVoiceEffectiveness {
  correlation: number; // -1 to 1 (Pearson correlation coefficient)
  significance: 'strong' | 'moderate' | 'weak' | 'none';
  pValue: number;

  highVoicePerformance: number; // Avg engagement rate for high voice score (>80)
  lowVoicePerformance: number; // Avg engagement rate for low voice score (<60)

  recommendation: string;
}

async function analyzeBrandVoiceEffectiveness(
  tenantId: string,
  platform: string,
  env: Env
): Promise<BrandVoiceEffectiveness> {
  // Get posts with brand voice scores and analytics
  const data = await env.DB.prepare(`
    SELECT
      gc.brand_voice_score,
      pa.engagement_rate
    FROM generated_content gc
    JOIN published_posts pp ON gc.id = pp.generated_content_id
    JOIN post_analytics pa ON pp.id = pa.post_id
    WHERE gc.tenant_id = ? AND pp.platform = ?
    AND pp.published_at >= datetime('now', '-90 days')
  `).bind(tenantId, platform).all();

  if (data.results.length < 10) {
    return {
      correlation: 0,
      significance: 'none',
      pValue: 1,
      highVoicePerformance: 0,
      lowVoicePerformance: 0,
      recommendation: 'Not enough data. Need at least 10 posts with brand voice scores.'
    };
  }

  // Calculate Pearson correlation
  const voiceScores = data.results.map(d => d.brand_voice_score);
  const engagementRates = data.results.map(d => d.engagement_rate);

  const correlation = pearsonCorrelation(voiceScores, engagementRates);

  // Calculate p-value (simplified)
  const n = data.results.length;
  const t = correlation * Math.sqrt(n - 2) / Math.sqrt(1 - correlation * correlation);
  const pValue = tDistributionPValue(t, n - 2);

  // Determine significance
  let significance: 'strong' | 'moderate' | 'weak' | 'none';
  if (pValue > 0.05) {
    significance = 'none';
  } else if (Math.abs(correlation) > 0.7) {
    significance = 'strong';
  } else if (Math.abs(correlation) > 0.4) {
    significance = 'moderate';
  } else {
    significance = 'weak';
  }

  // Calculate avg performance for high vs low voice score
  const highVoicePosts = data.results.filter(d => d.brand_voice_score > 80);
  const lowVoicePosts = data.results.filter(d => d.brand_voice_score < 60);

  const highVoicePerformance = highVoicePosts.length > 0
    ? highVoicePosts.reduce((sum, d) => sum + d.engagement_rate, 0) / highVoicePosts.length
    : 0;

  const lowVoicePerformance = lowVoicePosts.length > 0
    ? lowVoicePosts.reduce((sum, d) => sum + d.engagement_rate, 0) / lowVoicePosts.length
    : 0;

  // Generate recommendation
  let recommendation = '';
  if (significance === 'none') {
    recommendation = 'No significant correlation between brand voice and performance. Focus on other factors.';
  } else if (correlation > 0) {
    recommendation = `Positive ${significance} correlation! High brand voice posts perform ${
      ((highVoicePerformance - lowVoicePerformance) / lowVoicePerformance * 100).toFixed(1)
    }% better. Maintain high brand voice adherence.`;
  } else {
    recommendation = `Negative ${significance} correlation. High brand voice posts underperform. Consider relaxing brand voice constraints or testing different voice styles.`;
  }

  return {
    correlation: Math.round(correlation * 1000) / 1000,
    significance,
    pValue: Math.round(pValue * 1000) / 1000,
    highVoicePerformance: Math.round(highVoicePerformance * 100) / 100,
    lowVoicePerformance: Math.round(lowVoicePerformance * 100) / 100,
    recommendation
  };
}

function pearsonCorrelation(x: number[], y: number[]): number {
  const n = x.length;
  const sumX = x.reduce((a, b) => a + b, 0);
  const sumY = y.reduce((a, b) => a + b, 0);
  const sumXY = x.reduce((sum, xi, i) => sum + xi * y[i], 0);
  const sumX2 = x.reduce((sum, xi) => sum + xi * xi, 0);
  const sumY2 = y.reduce((sum, yi) => sum + yi * yi, 0);

  const numerator = n * sumXY - sumX * sumY;
  const denominator = Math.sqrt((n * sumX2 - sumX * sumX) * (n * sumY2 - sumY * sumY));

  return numerator / denominator;
}
```

### 9.2 Voice Drift Detection

Detect when brand voice changes over time:

```typescript
interface VoiceDriftAnalysis {
  isDrifting: boolean;
  direction: 'increasing' | 'decreasing' | 'stable';
  currentAverage: number;
  baselineAverage: number;
  drift: number; // Percentage change
  recommendation: string;
}

async function detectVoiceDrift(
  tenantId: string,
  env: Env
): Promise<VoiceDriftAnalysis> {
  // Get baseline (first 30 days)
  const baseline = await env.DB.prepare(`
    SELECT AVG(brand_voice_score) as avg_score
    FROM generated_content
    WHERE tenant_id = ?
    AND created_at BETWEEN datetime('now', '-90 days') AND datetime('now', '-60 days')
  `).bind(tenantId).first();

  // Get current period (last 30 days)
  const current = await env.DB.prepare(`
    SELECT AVG(brand_voice_score) as avg_score
    FROM generated_content
    WHERE tenant_id = ?
    AND created_at >= datetime('now', '-30 days')
  `).bind(tenantId).first();

  const drift = ((current.avg_score - baseline.avg_score) / baseline.avg_score) * 100;

  const isDrifting = Math.abs(drift) > 10; // 10% threshold
  const direction = drift > 0 ? 'increasing' : drift < 0 ? 'decreasing' : 'stable';

  let recommendation = '';
  if (!isDrifting) {
    recommendation = 'Brand voice is stable. Continue current approach.';
  } else if (direction === 'decreasing') {
    recommendation = `Brand voice score dropped ${Math.abs(drift).toFixed(1)}%. Review recent content and retrain if needed.`;
  } else {
    recommendation = `Brand voice score increased ${drift.toFixed(1)}%. Monitor to ensure quality improvements.`;
  }

  return {
    isDrifting,
    direction,
    currentAverage: Math.round(current.avg_score * 10) / 10,
    baselineAverage: Math.round(baseline.avg_score * 10) / 10,
    drift: Math.round(drift * 10) / 10,
    recommendation
  };
}
```

---

## 10. Database Schema

### 10.1 New Tables

#### `post_analytics`
```sql
CREATE TABLE post_analytics (
  id TEXT PRIMARY KEY,
  post_id TEXT NOT NULL,
  tenant_id TEXT NOT NULL,
  platform TEXT NOT NULL,

  -- Engagement Metrics
  likes INTEGER DEFAULT 0,
  comments INTEGER DEFAULT 0,
  shares INTEGER DEFAULT 0,
  saves INTEGER DEFAULT 0, -- Instagram only
  clicks INTEGER DEFAULT 0,
  bookmarks INTEGER DEFAULT 0, -- Twitter only

  -- Reach Metrics
  impressions INTEGER DEFAULT 0,
  reach INTEGER DEFAULT 0,
  unique_impressions INTEGER DEFAULT 0,

  -- Video Metrics (if applicable)
  video_views INTEGER,
  video_avg_watch_time INTEGER, -- Seconds
  video_complete_views INTEGER,

  -- Calculated Metrics
  engagement_rate REAL,
  viral_coefficient REAL,
  engagement_velocity_1h REAL, -- Engagements per hour (first hour)
  engagement_velocity_24h REAL, -- Engagements per hour (first 24h)
  engagement_velocity_7d REAL, -- Engagements per hour (first 7 days)

  -- Performance Score
  performance_score INTEGER, -- 0-100
  performance_score_breakdown TEXT, -- JSON: {engagement, reach, virality, longevity}

  -- Anomaly Detection
  is_anomaly BOOLEAN DEFAULT FALSE,
  anomaly_type TEXT, -- 'high' | 'low' | 'normal'
  z_score REAL,

  -- Timestamps
  synced_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (post_id) REFERENCES published_posts(id),
  FOREIGN KEY (tenant_id) REFERENCES tenants(tenant_id),

  INDEX idx_post_analytics_tenant_platform (tenant_id, platform),
  INDEX idx_post_analytics_synced_at (synced_at)
);
```

#### `performance_benchmarks`
```sql
CREATE TABLE performance_benchmarks (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL,
  platform TEXT NOT NULL,

  -- Calculated from last 90 days
  avg_engagement_rate REAL,
  median_engagement_rate REAL,
  p90_engagement_rate REAL,

  avg_reach REAL,
  p90_reach REAL,

  avg_share_rate REAL,
  p90_share_rate REAL,

  total_posts INTEGER,

  calculated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (tenant_id) REFERENCES tenants(tenant_id),
  UNIQUE(tenant_id, platform)
);
```

#### `ab_tests`
```sql
CREATE TABLE ab_tests (
  test_id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL,

  test_name TEXT NOT NULL,
  test_type TEXT NOT NULL, -- 'content' | 'brand_voice' | 'timing' | 'hashtag'

  variant_a_post_id TEXT NOT NULL,
  variant_b_post_id TEXT NOT NULL,
  variant_c_post_id TEXT, -- Optional third variant

  winner TEXT, -- 'A' | 'B' | 'C' | 'tie' | 'inconclusive'
  p_value REAL,
  confidence_level INTEGER,
  uplift REAL, -- % improvement

  status TEXT DEFAULT 'running', -- 'running' | 'completed' | 'cancelled'

  started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP,

  FOREIGN KEY (tenant_id) REFERENCES tenants(tenant_id),
  FOREIGN KEY (variant_a_post_id) REFERENCES published_posts(id),
  FOREIGN KEY (variant_b_post_id) REFERENCES published_posts(id)
);
```

#### `conversions`
```sql
CREATE TABLE conversions (
  conversion_id TEXT PRIMARY KEY,
  post_id TEXT NOT NULL,
  tenant_id TEXT NOT NULL,

  user_id TEXT, -- If known
  conversion_type TEXT NOT NULL, -- 'page_view' | 'form_submit' | 'purchase'
  value REAL DEFAULT 0, -- Revenue value

  utm_source TEXT,
  utm_medium TEXT,
  utm_campaign TEXT,
  utm_content TEXT,

  tracked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (post_id) REFERENCES published_posts(id),
  FOREIGN KEY (tenant_id) REFERENCES tenants(tenant_id),

  INDEX idx_conversions_post_id (post_id),
  INDEX idx_conversions_tenant_id (tenant_id)
);
```

#### `analytics_sync_errors`
```sql
CREATE TABLE analytics_sync_errors (
  id TEXT PRIMARY KEY,
  post_id TEXT NOT NULL,
  platform TEXT NOT NULL,

  error_message TEXT,
  error_type TEXT, -- 'rate_limit' | 'auth_error' | 'api_error' | 'network_error'
  retry_count INTEGER DEFAULT 0,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  resolved_at TIMESTAMP,

  FOREIGN KEY (post_id) REFERENCES published_posts(id),

  INDEX idx_analytics_errors_post_id (post_id),
  INDEX idx_analytics_errors_created_at (created_at)
);
```

### 10.2 Schema Migration

Add analytics columns to existing `published_posts` table:

```sql
ALTER TABLE published_posts ADD COLUMN last_analytics_sync TIMESTAMP;
ALTER TABLE published_posts ADD COLUMN sync_frequency INTEGER DEFAULT 15; -- Minutes
ALTER TABLE published_posts ADD COLUMN performance_score INTEGER;
ALTER TABLE published_posts ADD COLUMN is_viral BOOLEAN DEFAULT FALSE;
```

---

## 11. API Specifications

### 11.1 Analytics Endpoints

```typescript
// GET /api/analytics/overview
interface AnalyticsOverviewRequest {
  tenantId: string;
  dateRange?: { start: string; end: string };
  platforms?: string[]; // Filter by platforms
}

// GET /api/analytics/platform/:platform
interface PlatformAnalyticsRequest {
  tenantId: string;
  platform: string;
  dateRange?: { start: string; end: string };
}

// GET /api/analytics/post/:postId
interface PostAnalyticsRequest {
  postId: string;
}

// GET /api/analytics/predict
interface PerformancePredictionRequest {
  tenantId: string;
  content: string;
  imageUrl: string;
  platform: string;
}

// POST /api/analytics/ab-test
interface CreateABTestRequest {
  tenantId: string;
  testName: string;
  testType: 'content' | 'brand_voice' | 'timing' | 'hashtag';
  variantAPostId: string;
  variantBPostId: string;
  variantCPostId?: string;
}

// GET /api/analytics/ab-test/:testId
interface GetABTestRequest {
  testId: string;
}

// GET /api/analytics/roi/:campaignId
interface ROIAnalyticsRequest {
  campaignId: string;
}

// POST /api/analytics/conversion
interface TrackConversionRequest {
  utmSource: string;
  utmMedium: string;
  utmCampaign: string;
  utmContent: string; // Post ID
  conversionType: 'page_view' | 'form_submit' | 'purchase';
  value?: number;
  userId?: string;
}
```

### 11.2 Webhook Integrations

Allow external systems to send analytics data:

```typescript
// POST /api/webhooks/analytics
interface AnalyticsWebhookPayload {
  platform: string;
  postId: string;
  analytics: {
    likes?: number;
    comments?: number;
    shares?: number;
    impressions?: number;
    reach?: number;
  };
  timestamp: string;
  signature: string; // HMAC signature for verification
}

async function handleAnalyticsWebhook(
  payload: AnalyticsWebhookPayload,
  env: Env
): Promise<{ success: boolean }> {
  // Verify signature
  const isValid = verifyHMACSignature(
    payload,
    payload.signature,
    env.WEBHOOK_SECRET
  );

  if (!isValid) {
    throw new Error('Invalid webhook signature');
  }

  // Update analytics
  await env.DB.prepare(`
    UPDATE post_analytics SET
      likes = ?,
      comments = ?,
      shares = ?,
      impressions = ?,
      reach = ?,
      synced_at = ?
    WHERE post_id = ?
  `).bind(
    payload.analytics.likes,
    payload.analytics.comments,
    payload.analytics.shares,
    payload.analytics.impressions,
    payload.analytics.reach,
    payload.timestamp,
    payload.postId
  ).run();

  return { success: true };
}
```

---

## 12. Cost Analysis

### 12.1 Platform API Costs

| Platform | API Tier | Calls/Month | Cost |
|----------|----------|-------------|------|
| LinkedIn | Free | Unlimited (rate limited) | $0 |
| Twitter/X | Basic | 10K tweets, 50K analytics reads | $100/mo |
| Facebook/Instagram | Free | 200 calls/hour/user | $0 |
| **Total** | | | **$100/mo** |

**Notes:**
- Twitter/X requires Basic tier for analytics access
- May need to upgrade to Pro ($5K/mo) for higher limits in future
- Facebook/Instagram limits are per-user, sufficient for most tenants

### 12.2 Infrastructure Costs (Cloudflare)

| Service | Usage | Cost |
|---------|-------|------|
| Workers | 10M requests/mo | $5/mo (beyond free tier) |
| D1 | 5GB storage, 25M reads, 5M writes | $7/mo |
| Vectorize | 10K dimensions, 10M queries | $3/mo |
| R2 | 100GB storage, 1M requests | $2/mo |
| Cron Triggers | 96 invocations/day | Included in Workers |
| **Total** | | **$17/mo** |

### 12.3 OpenAI Costs

| Task | Model | Tokens/Request | Requests/Month | Cost |
|------|-------|----------------|----------------|------|
| Performance prediction | text-embedding-3-small | 500 | 1000 | $0.10 |
| Factor analysis | GPT-4 | 2000 | 500 | $10 |
| Brand voice analysis | text-embedding-3-small | 500 | 500 | $0.05 |
| **Total** | | | | **$10.15/mo** |

### 12.4 Total Operational Cost

| Category | Cost/Month |
|----------|------------|
| Platform APIs | $100 |
| Cloudflare Infrastructure | $17 |
| OpenAI | $10 |
| Monitoring (Datadog Lite) | $15 |
| **Total** | **$142/mo** |

**Per-Customer Cost** (100 customers): **$1.42/customer/month**

### 12.5 Development Cost

| Phase | Duration | Engineers | Hours | Cost ($375/hr) |
|-------|----------|-----------|-------|----------------|
| Analytics Collection | 5 days | 1 backend | 40 | $15,000 |
| Metrics Engine | 5 days | 1 backend | 40 | $15,000 |
| Dashboard | 5 days | 1 frontend | 40 | $15,000 |
| A/B Testing | 3 days | 1 backend | 24 | $9,000 |
| Performance Scorer | 4 days | 1 AI/backend | 32 | $12,000 |
| ROI Tracking | 3 days | 1 backend | 24 | $9,000 |
| Engagement Analytics | 3 days | 1 data scientist | 24 | $9,000 |
| Brand Voice Effectiveness | 3 days | 1 AI/backend | 24 | $9,000 |
| Testing & QA | 5 days | 2 engineers | 80 | $30,000 |
| Documentation | 2 days | 1 engineer | 16 | $6,000 |
| **Total** | **4 weeks** | | **344 hours** | **$129,000** |

**Rounded**: **$130K development cost**

---

## 13. Implementation Timeline

### 13.1 Week 1: Foundation (Days 1-5)

**Goals:**
- Analytics collection service working
- Data syncing from all 4 platforms
- Basic database schema deployed

**Tasks:**
1. Create database tables (`post_analytics`, `performance_benchmarks`, etc.)
2. Implement platform API clients (LinkedIn, Twitter, Facebook, Instagram)
3. Build sync service with cron triggers
4. Test incremental sync logic
5. Deploy to production

**Deliverables:**
- ✅ Analytics syncing every 15 minutes
- ✅ Historical data backfill working
- ✅ Error logging and retry logic

### 13.2 Week 2: Metrics & Dashboard (Days 6-10)

**Goals:**
- Performance metrics engine calculating KPIs
- Dashboard displaying real-time analytics
- Benchmark calculations working

**Tasks:**
1. Implement KPI calculations (engagement rate, virality, CPS)
2. Build benchmark calculation system
3. Create dashboard API endpoints
4. Build frontend dashboard (Cloudflare Pages + React/Chart.js)
5. Implement drill-down views

**Deliverables:**
- ✅ Overview dashboard with key metrics
- ✅ Platform-specific dashboards
- ✅ Post detail views
- ✅ Export to CSV/PDF

### 13.3 Week 3: Advanced Features (Days 11-15)

**Goals:**
- A/B testing infrastructure complete
- Performance prediction working
- ROI tracking operational

**Tasks:**
1. Implement A/B test creation and statistical analysis
2. Build performance prediction engine (embeddings + similarity)
3. Create ROI tracking system (UTM parameters, attribution)
4. Implement engagement analytics (posting times, decay curves)
5. Build brand voice effectiveness analyzer

**Deliverables:**
- ✅ A/B tests with statistical significance
- ✅ Performance prediction before publishing
- ✅ ROI metrics and attribution models
- ✅ Best posting times analysis

### 13.4 Week 4: Testing & Polish (Days 16-20)

**Goals:**
- All features tested and working
- Documentation complete
- Production-ready

**Tasks:**
1. End-to-end testing (all platforms)
2. Performance optimization (query tuning, caching)
3. Security audit (API authentication, data privacy)
4. Write user documentation
5. Deploy final version

**Deliverables:**
- ✅ All tests passing
- ✅ Performance benchmarks met (<2s dashboard load)
- ✅ User guide and API docs
- ✅ Stage F complete and ready for Stage G

---

## 14. Resilience Patterns Integration

### 14.1 Cross-Cutting Patterns

**Reference**: `.planning/approved/STAGE-X-RESILIENCE-PATTERNS.md`

All external API calls in Stage F MUST use the resilience patterns from the shared `@dify-agent/resilience` package.

### 14.2 Pattern Application

#### Platform API Calls
```typescript
import { CircuitBreaker, retryWithBackoff, withTimeout } from '@dify-agent/resilience';

const linkedinCircuitBreaker = new CircuitBreaker('linkedin-analytics', {
  failureThreshold: 5,
  timeout: 60000, // 60s
  halfOpenMaxCalls: 2
});

async function fetchLinkedInAnalyticsResilient(
  postId: string,
  credentials: OAuthTokens
): Promise<LinkedInPostAnalytics> {
  return await linkedinCircuitBreaker.execute(async () => {
    return await retryWithBackoff(
      async () => {
        return await withTimeout(
          fetchLinkedInAnalytics(postId, credentials),
          15000 // 15s timeout
        );
      },
      {
        maxAttempts: 3,
        initialDelay: 1000,
        maxDelay: 10000,
        backoffMultiplier: 2
      }
    );
  });
}
```

#### OpenAI Calls
```typescript
const openaiCircuitBreaker = new CircuitBreaker('openai-analytics', {
  failureThreshold: 3,
  timeout: 120000, // 2 min
  halfOpenMaxCalls: 1
});

async function analyzePerformanceFactorsResilient(
  content: string,
  imageUrl: string,
  platform: string,
  env: Env
): Promise<PerformanceFactor[]> {
  return await openaiCircuitBreaker.execute(async () => {
    return await retryWithBackoff(
      async () => {
        return await withTimeout(
          analyzePerformanceFactors(content, imageUrl, platform, env),
          30000 // 30s timeout
        );
      },
      { maxAttempts: 3, initialDelay: 2000 }
    );
  });
}
```

#### Database Calls
```typescript
import { bulkhead } from '@dify-agent/resilience';

const dbBulkhead = bulkhead({
  maxConcurrent: 10, // Max 10 concurrent DB operations
  maxQueued: 50,
  timeout: 5000
});

async function queryAnalytics(query: string, env: Env): Promise<any> {
  return await dbBulkhead.execute(async () => {
    return await withTimeout(
      env.DB.prepare(query).all(),
      5000 // 5s timeout
    );
  });
}
```

### 14.3 Health Checks

```typescript
import { HealthCheck } from '@dify-agent/resilience';

const healthChecks = {
  linkedin: new HealthCheck('linkedin-api', {
    checkFn: async () => {
      const response = await fetch('https://api.linkedin.com/v2/me', {
        headers: { 'Authorization': `Bearer ${testToken}` }
      });
      return response.ok;
    },
    interval: 60000, // Check every minute
    timeout: 10000,
    unhealthyThreshold: 3
  }),

  twitter: new HealthCheck('twitter-api', {
    checkFn: async () => {
      const response = await fetch('https://api.twitter.com/2/tweets/1234567890');
      return response.ok;
    },
    interval: 60000,
    timeout: 10000,
    unhealthyThreshold: 3
  })

  // Add for Facebook, Instagram, OpenAI
};

// Start all health checks
Object.values(healthChecks).forEach(hc => hc.start());
```

### 14.4 Rate Limiting

```typescript
import { RateLimiter, TokenBucketLimiter } from '@dify-agent/resilience';

// LinkedIn: Unlimited calls but respect rate limits
const linkedinLimiter = new TokenBucketLimiter({
  tokensPerInterval: 100,
  interval: 60000, // 100 requests per minute
  maxBurst: 200
});

// Twitter: 50K analytics reads per month (Basic tier)
const twitterLimiter = new TokenBucketLimiter({
  tokensPerInterval: 69, // ~50K / (30 days * 24 hours)
  interval: 3600000, // Per hour
  maxBurst: 100
});

async function fetchAnalyticsWithRateLimit(
  platform: string,
  postId: string,
  credentials: OAuthTokens
): Promise<any> {
  const limiter = platform === 'linkedin' ? linkedinLimiter : twitterLimiter;

  await limiter.acquire(1); // Wait for token

  return await fetchAnalytics(platform, postId, credentials);
}
```

### 14.5 Monitoring & Alerts

```typescript
import { metrics, alerts } from '@dify-agent/resilience';

// Track circuit breaker state changes
linkedinCircuitBreaker.on('stateChange', (state) => {
  metrics.gauge('circuit_breaker.state', state === 'OPEN' ? 1 : 0, {
    service: 'linkedin-analytics'
  });

  if (state === 'OPEN') {
    alerts.send({
      severity: 'critical',
      message: 'LinkedIn Analytics circuit breaker opened',
      service: 'analytics-collection'
    });
  }
});

// Track sync errors
metrics.counter('analytics_sync.errors', 1, {
  platform: 'linkedin',
  error_type: 'rate_limit'
});

// Track performance
metrics.histogram('analytics_sync.duration', syncDuration, {
  platform: 'linkedin'
});
```

---

## 15. Success Metrics

### 15.1 Stage F Complete When:

- ✅ Analytics syncing from all 4 platforms (LinkedIn, Twitter/X, Facebook, Instagram)
- ✅ Sync frequency: 15 min (recent posts), 1 hour (older posts)
- ✅ Dashboard load time <2 seconds
- ✅ All KPIs calculating correctly (engagement rate, reach, virality, CPS)
- ✅ Benchmarks auto-calculating from 90-day rolling window
- ✅ A/B tests with statistical significance (95% confidence)
- ✅ Performance prediction accuracy >75%
- ✅ ROI tracking and attribution working
- ✅ Engagement analytics (best posting times, decay curves)
- ✅ Brand voice effectiveness correlation analysis
- ✅ Resilience patterns integrated (circuit breakers, retries, timeouts)
- ✅ Health checks running for all external services
- ✅ Rate limiting per platform
- ✅ Monitoring dashboards (Datadog/Grafana)
- ✅ All database tables created and indexed
- ✅ API endpoints documented and tested
- ✅ CSV/PDF export working
- ✅ Error logging and alerting

### 15.2 Key Performance Indicators

| Metric | Target | Actual |
|--------|--------|--------|
| Analytics sync latency | <30s after platform post | TBD |
| Dashboard load time | <2s | TBD |
| API response time (p95) | <500ms | TBD |
| Sync error rate | <1% | TBD |
| Prediction accuracy | >75% | TBD |
| A/B test confidence | >95% | TBD |
| Uptime | >99.9% | TBD |

---

## 16. Next Stage Preview

**Stage G: Dify Implementation**
- Integrate Stages A-F into Dify platform
- Build conversational workflows
- Multi-turn conversation handling
- Advisory pattern implementation
- Timeline: 6 weeks

---

## Appendix A: Glossary

- **Engagement Rate**: (Likes + Comments + Shares + Clicks) / Impressions × 100
- **Virality Coefficient**: Reach / Followers (>1.0 = viral)
- **Content Performance Score (CPS)**: 0-100 composite score of engagement, reach, virality, longevity
- **A/B Test**: Statistical comparison of 2+ variants
- **Chi-Square Test**: Statistical test for categorical data (engagement vs non-engagement)
- **p-value**: Probability that observed difference is due to chance (<0.05 = significant)
- **Pearson Correlation**: Measure of linear correlation between two variables (-1 to 1)
- **Engagement Decay Curve**: Graph showing engagement drop-off over time
- **Half-Life**: Time until 50% of total engagement is reached
- **Attribution Model**: Method for assigning conversion credit to touchpoints

---

## Appendix B: References

- **LinkedIn Marketing API**: https://learn.microsoft.com/en-us/linkedin/marketing/integrations/community-management/shares/share-api
- **Twitter API v2 Analytics**: https://developer.twitter.com/en/docs/twitter-api/metrics
- **Facebook Graph API Insights**: https://developers.facebook.com/docs/graph-api/reference/v18.0/insights
- **Instagram Graph API**: https://developers.facebook.com/docs/instagram-api/reference/ig-media/insights
- **Cloudflare Workers**: https://developers.cloudflare.com/workers/
- **Cloudflare D1**: https://developers.cloudflare.com/d1/
- **Cloudflare Cron Triggers**: https://developers.cloudflare.com/workers/configuration/cron-triggers/
- **OpenAI Embeddings**: https://platform.openai.com/docs/guides/embeddings
- **Chart.js**: https://www.chartjs.org/
- **Statistical Significance Calculator**: https://www.evanmiller.org/ab-testing/chi-squared.html

---

**Document Status**: ✅ APPROVED
**Version**: 1.0.0
**Last Updated**: November 10, 2025
**Next Review**: After Stage F implementation complete
**Maintained By**: Planning Agent

---

**Stage F Planning Complete!** Ready to proceed with implementation after Stage C credentials are available and Stage E is implemented.
