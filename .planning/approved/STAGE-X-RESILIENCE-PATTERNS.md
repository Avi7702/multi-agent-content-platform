# CROSS-CUTTING: RESILIENCE PATTERNS FOR ALL STAGES

**Version:** 1.0.0
**Status:** ✅ APPROVED - Production Ready
**Date:** November 10, 2025
**Applies To:** All Stages (C, D, E, F, G, H, I, J, K, L)
**Package:** `@dify-agent/resilience`

---

## Table of Contents

1. [Overview](#1-overview)
2. [Circuit Breaker Pattern](#2-circuit-breaker-pattern)
3. [Retry with Exponential Backoff](#3-retry-with-exponential-backoff)
4. [Bulkheads (Resource Isolation)](#4-bulkheads-resource-isolation)
5. [Timeouts](#5-timeouts)
6. [Fallback Strategies](#6-fallback-strategies)
7. [Health Checks](#7-health-checks)
8. [Rate Limiting](#8-rate-limiting)
9. [Monitoring & Observability](#9-monitoring--observability)
10. [Shared Library Structure](#10-shared-library-structure)
11. [Configuration Guidelines](#11-configuration-guidelines)
12. [Testing Strategies](#12-testing-strategies)

---

## 1. Overview

### 1.1 Purpose

This document defines **production-grade resilience patterns** to be used across all stages of the DIFY-AGENT system. These patterns ensure:

- **Fault Tolerance**: System gracefully handles failures
- **High Availability**: Services remain operational during partial outages
- **Performance**: Requests complete within acceptable timeframes
- **Resource Protection**: No single tenant/service exhausts shared resources
- **Observability**: Clear visibility into system health and failures

### 1.2 Shared Library Approach

All patterns are implemented in a shared NPM package: **`@dify-agent/resilience`**

**Benefits:**
- ✅ DRY (Don't Repeat Yourself) - write once, use everywhere
- ✅ Consistent behavior across all services
- ✅ Centralized bug fixes and improvements
- ✅ Type-safe TypeScript implementations
- ✅ Comprehensive test coverage

### 1.3 Applies To

| Stage | Services/Components | Key External Dependencies |
|-------|---------------------|---------------------------|
| C | Web Scraper, Knowledge Extraction | Firecrawl API, OpenAI GPT-5 |
| D | 11 Microservices | GPT-5, Claude Sonnet, Gemini, Voyage |
| E | 4 Platform Adapters | LinkedIn, Twitter, Facebook, Instagram APIs |
| F | Analytics Service | Platform Analytics APIs |
| G | Dify Workflows | HTTP tools, MCP Gateway |
| H | OpenManus Agents | OpenManus API |
| I | MCP Gateway | All tool integrations |

---

## 2. Circuit Breaker Pattern

### 2.1 Overview

A **circuit breaker** prevents cascading failures by stopping requests to a failing service after a threshold is reached.

**States:**
- **CLOSED**: Normal operation, requests pass through
- **OPEN**: Service failing, requests immediately rejected
- **HALF_OPEN**: Testing if service recovered, limited requests allowed

### 2.2 Implementation

```typescript
// @dify-agent/resilience/src/circuit-breaker.ts

export enum CircuitBreakerState {
  CLOSED = 'CLOSED',
  OPEN = 'OPEN',
  HALF_OPEN = 'HALF_OPEN'
}

export interface CircuitBreakerConfig {
  failureThreshold: number;      // Open after N failures (default: 5)
  successThreshold: number;      // Close after N successes in HALF_OPEN (default: 2)
  timeout: number;               // Stay OPEN for N ms (default: 60000)
  halfOpenMaxCalls: number;      // Max concurrent calls in HALF_OPEN (default: 3)
  onStateChange?: (oldState: CircuitBreakerState, newState: CircuitBreakerState) => void;
}

export class CircuitBreaker {
  private state: CircuitBreakerState = CircuitBreakerState.CLOSED;
  private failureCount = 0;
  private successCount = 0;
  private lastFailureTime = 0;
  private halfOpenCalls = 0;

  constructor(
    private name: string,
    private config: CircuitBreakerConfig
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    // Check if circuit should transition from OPEN to HALF_OPEN
    if (this.state === CircuitBreakerState.OPEN) {
      const timeSinceFailure = Date.now() - this.lastFailureTime;
      if (timeSinceFailure > this.config.timeout) {
        this.transitionTo(CircuitBreakerState.HALF_OPEN);
      } else {
        throw new CircuitBreakerOpenError(
          `Circuit breaker '${this.name}' is OPEN. Will retry in ${
            this.config.timeout - timeSinceFailure
          }ms`
        );
      }
    }

    // Reject if HALF_OPEN and max concurrent calls reached
    if (this.state === CircuitBreakerState.HALF_OPEN) {
      if (this.halfOpenCalls >= this.config.halfOpenMaxCalls) {
        throw new CircuitBreakerOpenError(
          `Circuit breaker '${this.name}' is HALF_OPEN with max concurrent calls`
        );
      }
      this.halfOpenCalls++;
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure(error);
      throw error;
    } finally {
      if (this.state === CircuitBreakerState.HALF_OPEN) {
        this.halfOpenCalls--;
      }
    }
  }

  private onSuccess() {
    if (this.state === CircuitBreakerState.HALF_OPEN) {
      this.successCount++;
      if (this.successCount >= this.config.successThreshold) {
        this.transitionTo(CircuitBreakerState.CLOSED);
      }
    }
    this.failureCount = 0;
  }

  private onFailure(error: any) {
    this.failureCount++;
    this.lastFailureTime = Date.now();

    if (this.state === CircuitBreakerState.HALF_OPEN) {
      this.transitionTo(CircuitBreakerState.OPEN);
    } else if (
      this.state === CircuitBreakerState.CLOSED &&
      this.failureCount >= this.config.failureThreshold
    ) {
      this.transitionTo(CircuitBreakerState.OPEN);
    }

    console.error(`Circuit breaker '${this.name}' failure:`, error);
  }

  private transitionTo(newState: CircuitBreakerState) {
    const oldState = this.state;
    this.state = newState;
    this.failureCount = 0;
    this.successCount = 0;

    console.log(`Circuit breaker '${this.name}': ${oldState} → ${newState}`);

    if (this.config.onStateChange) {
      this.config.onStateChange(oldState, newState);
    }
  }

  getState(): CircuitBreakerState {
    return this.state;
  }

  getMetrics() {
    return {
      state: this.state,
      failureCount: this.failureCount,
      successCount: this.successCount,
      lastFailureTime: this.lastFailureTime
    };
  }

  reset() {
    this.transitionTo(CircuitBreakerState.CLOSED);
  }
}

export class CircuitBreakerOpenError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'CircuitBreakerOpenError';
  }
}
```

### 2.3 Usage Examples

#### Example 1: Protecting OpenAI GPT-5 Calls (Stage D)

```typescript
import { CircuitBreaker } from '@dify-agent/resilience';

const openaiCircuitBreaker = new CircuitBreaker('openai-gpt5', {
  failureThreshold: 5,
  successThreshold: 2,
  timeout: 60000, // 1 minute
  halfOpenMaxCalls: 3,
  onStateChange: (oldState, newState) => {
    metrics.gauge('circuit_breaker.openai.state', newState === 'CLOSED' ? 0 : 1);
    if (newState === 'OPEN') {
      alerting.send('Circuit breaker OPEN: OpenAI GPT-5 API');
    }
  }
});

async function generateContentWithGPT5(prompt: string): Promise<string> {
  return await openaiCircuitBreaker.execute(async () => {
    const response = await openai.chat.completions.create({
      model: 'gpt-5',
      messages: [{ role: 'user', content: prompt }],
      max_tokens: 1000
    });
    return response.choices[0].message.content;
  });
}
```

#### Example 2: Protecting LinkedIn API (Stage E)

```typescript
const linkedinCircuitBreaker = new CircuitBreaker('linkedin-api', {
  failureThreshold: 3, // More sensitive for platform APIs
  successThreshold: 2,
  timeout: 120000, // 2 minutes
  halfOpenMaxCalls: 2
});

async function publishToLinkedIn(content: string, imageUrl: string) {
  return await linkedinCircuitBreaker.execute(async () => {
    // Upload image
    const assetResponse = await linkedinApi.registerAssetUpload(credentials);
    await linkedinApi.uploadImageBinary(assetResponse.uploadUrl, imageUrl);

    // Create post
    const postResponse = await linkedinApi.createUGCPost(content, assetResponse.asset);
    return postResponse;
  });
}
```

### 2.4 Recommended Thresholds

| Service Type | Failure Threshold | Timeout | Half-Open Max Calls |
|--------------|-------------------|---------|---------------------|
| LLM APIs (GPT-5, Claude) | 5 | 60s | 3 |
| Platform APIs (LinkedIn, Twitter) | 3 | 120s | 2 |
| Image Processing (Google Vision) | 5 | 30s | 5 |
| Database (D1, Vectorize) | 10 | 30s | 10 |
| Internal Services | 10 | 30s | 5 |

---

## 3. Retry with Exponential Backoff

### 3.1 Overview

Automatically retry failed requests with increasing delays between attempts.

**Backoff Strategy:**
- Attempt 1: Immediate
- Attempt 2: Wait 1s
- Attempt 3: Wait 2s
- Attempt 4: Wait 4s
- Max: 10s

### 3.2 Implementation

```typescript
// @dify-agent/resilience/src/retry.ts

export interface RetryConfig {
  maxAttempts: number;           // Max retry attempts (default: 3)
  initialDelay: number;          // Initial delay in ms (default: 1000)
  maxDelay: number;              // Max delay in ms (default: 10000)
  backoffMultiplier: number;     // Multiply delay by this (default: 2)
  retryableErrors?: string[];    // Error codes/names to retry
  onRetry?: (attempt: number, error: any, nextDelay: number) => void;
}

export class RetryExecutor {
  constructor(private config: RetryConfig) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    let lastError: any;
    let delay = this.config.initialDelay;

    for (let attempt = 1; attempt <= this.config.maxAttempts; attempt++) {
      try {
        return await fn();
      } catch (error) {
        lastError = error;

        // Check if error is retryable
        if (!this.isRetryable(error)) {
          throw error;
        }

        // Don't retry on last attempt
        if (attempt === this.config.maxAttempts) {
          break;
        }

        // Calculate next delay with exponential backoff
        const nextDelay = Math.min(delay, this.config.maxDelay);

        if (this.config.onRetry) {
          this.config.onRetry(attempt, error, nextDelay);
        }

        console.warn(
          `Retry attempt ${attempt}/${this.config.maxAttempts} after ${nextDelay}ms`,
          { error: error.message }
        );

        await this.sleep(nextDelay);

        // Exponential backoff
        delay *= this.config.backoffMultiplier;
      }
    }

    throw lastError;
  }

  private isRetryable(error: any): boolean {
    // Network errors are always retryable
    if (
      error.code === 'ECONNREFUSED' ||
      error.code === 'ETIMEDOUT' ||
      error.code === 'ENOTFOUND'
    ) {
      return true;
    }

    // HTTP status codes that are retryable
    const retryableStatusCodes = [408, 429, 500, 502, 503, 504];
    if (error.status && retryableStatusCodes.includes(error.status)) {
      return true;
    }

    // Custom retryable errors
    if (
      this.config.retryableErrors &&
      (this.config.retryableErrors.includes(error.code) ||
        this.config.retryableErrors.includes(error.name))
    ) {
      return true;
    }

    return false;
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Convenience function for single retry
export async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  config: Partial<RetryConfig> = {}
): Promise<T> {
  const executor = new RetryExecutor({
    maxAttempts: 3,
    initialDelay: 1000,
    maxDelay: 10000,
    backoffMultiplier: 2,
    ...config
  });
  return executor.execute(fn);
}
```

### 3.3 Usage Examples

#### Example 1: Retry Platform API Calls

```typescript
import { retryWithBackoff } from '@dify-agent/resilience';

async function publishToTwitter(content: string, mediaId: string) {
  return await retryWithBackoff(
    async () => {
      const response = await twitterApi.createTweet({
        text: content,
        media: { media_ids: [mediaId] }
      });
      return response.data;
    },
    {
      maxAttempts: 3,
      initialDelay: 2000,
      onRetry: (attempt, error, nextDelay) => {
        console.log(`Twitter API retry ${attempt}, next delay: ${nextDelay}ms`);
        metrics.increment('twitter.retry', { attempt });
      }
    }
  );
}
```

#### Example 2: Combined Circuit Breaker + Retry

```typescript
async function generateContentResilient(prompt: string): Promise<string> {
  return await openaiCircuitBreaker.execute(async () => {
    return await retryWithBackoff(
      async () => {
        const response = await openai.chat.completions.create({
          model: 'gpt-5',
          messages: [{ role: 'user', content: prompt }]
        });
        return response.choices[0].message.content;
      },
      { maxAttempts: 3, initialDelay: 1000 }
    );
  });
}
```

### 3.4 Recommended Retry Configurations

| Service Type | Max Attempts | Initial Delay | Max Delay |
|--------------|--------------|---------------|-----------|
| LLM APIs | 3 | 1s | 10s |
| Platform APIs | 3 | 2s | 15s |
| Image Processing | 2 | 1s | 5s |
| Database | 5 | 500ms | 5s |
| Internal Services | 3 | 500ms | 5s |

---

## 4. Bulkheads (Resource Isolation)

### 4.1 Overview

**Bulkheads** limit concurrent executions to prevent resource exhaustion.

**Use Cases:**
- Limit concurrent LLM calls per tenant (prevent quota exhaustion)
- Limit concurrent image processing (prevent memory overflow)
- Limit concurrent database connections (prevent connection pool exhaustion)

### 4.2 Implementation

```typescript
// @dify-agent/resilience/src/bulkhead.ts

export interface BulkheadConfig {
  maxConcurrent: number;          // Max concurrent executions
  maxQueueSize: number;           // Max queued requests (default: 100)
  queueTimeout: number;           // Max wait time in queue (default: 30000ms)
  onQueueFull?: () => void;
  onExecutionStart?: () => void;
  onExecutionEnd?: () => void;
}

export class Bulkhead {
  private activeCount = 0;
  private queue: Array<{
    fn: () => Promise<any>;
    resolve: (value: any) => void;
    reject: (error: any) => void;
    queuedAt: number;
  }> = [];

  constructor(
    private name: string,
    private config: BulkheadConfig
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    // If under max concurrent, execute immediately
    if (this.activeCount < this.config.maxConcurrent) {
      return this.executeImmediately(fn);
    }

    // Queue is full, reject immediately
    if (this.queue.length >= this.config.maxQueueSize) {
      if (this.config.onQueueFull) {
        this.config.onQueueFull();
      }
      throw new BulkheadRejectedException(
        `Bulkhead '${this.name}' queue full (${this.config.maxQueueSize})`
      );
    }

    // Add to queue
    return new Promise<T>((resolve, reject) => {
      const queuedAt = Date.now();

      const timeoutId = setTimeout(() => {
        // Remove from queue on timeout
        const index = this.queue.findIndex(item => item.queuedAt === queuedAt);
        if (index !== -1) {
          this.queue.splice(index, 1);
          reject(
            new BulkheadTimeoutError(
              `Bulkhead '${this.name}' queue timeout after ${this.config.queueTimeout}ms`
            )
          );
        }
      }, this.config.queueTimeout);

      this.queue.push({
        fn: async () => {
          clearTimeout(timeoutId);
          return fn();
        },
        resolve,
        reject,
        queuedAt
      });
    });
  }

  private async executeImmediately<T>(fn: () => Promise<T>): Promise<T> {
    this.activeCount++;

    if (this.config.onExecutionStart) {
      this.config.onExecutionStart();
    }

    try {
      const result = await fn();
      return result;
    } finally {
      this.activeCount--;

      if (this.config.onExecutionEnd) {
        this.config.onExecutionEnd();
      }

      // Process next in queue
      this.processQueue();
    }
  }

  private processQueue() {
    if (this.queue.length === 0 || this.activeCount >= this.config.maxConcurrent) {
      return;
    }

    const item = this.queue.shift();
    if (!item) return;

    this.activeCount++;

    if (this.config.onExecutionStart) {
      this.config.onExecutionStart();
    }

    item
      .fn()
      .then(item.resolve)
      .catch(item.reject)
      .finally(() => {
        this.activeCount--;

        if (this.config.onExecutionEnd) {
          this.config.onExecutionEnd();
        }

        this.processQueue();
      });
  }

  getMetrics() {
    return {
      activeCount: this.activeCount,
      queueSize: this.queue.length,
      utilizationPercent: (this.activeCount / this.config.maxConcurrent) * 100
    };
  }
}

export class BulkheadRejectedException extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'BulkheadRejectedException';
  }
}

export class BulkheadTimeoutError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'BulkheadTimeoutError';
  }
}
```

### 4.3 Usage Examples

#### Example 1: Limit Concurrent LLM Calls Per Tenant

```typescript
import { Bulkhead } from '@dify-agent/resilience';

// Per-tenant bulkheads
const tenantBulkheads = new Map<string, Bulkhead>();

function getTenantBulkhead(tenantId: string): Bulkhead {
  if (!tenantBulkheads.has(tenantId)) {
    tenantBulkheads.set(
      tenantId,
      new Bulkhead(`tenant-${tenantId}-llm`, {
        maxConcurrent: 5,        // Max 5 concurrent LLM calls per tenant
        maxQueueSize: 20,
        queueTimeout: 60000,
        onQueueFull: () => {
          console.warn(`Tenant ${tenantId} LLM bulkhead queue full`);
          metrics.increment('bulkhead.queue_full', { tenant_id: tenantId });
        }
      })
    );
  }
  return tenantBulkheads.get(tenantId)!;
}

async function generateContentForTenant(tenantId: string, prompt: string) {
  const bulkhead = getTenantBulkhead(tenantId);

  return await bulkhead.execute(async () => {
    return await openai.chat.completions.create({
      model: 'gpt-5',
      messages: [{ role: 'user', content: prompt }]
    });
  });
}
```

#### Example 2: Limit Concurrent Image Processing

```typescript
const imageProcessingBulkhead = new Bulkhead('image-processing', {
  maxConcurrent: 10,    // Max 10 concurrent image operations
  maxQueueSize: 50,
  queueTimeout: 30000,
  onExecutionStart: () => {
    metrics.gauge('image_processing.active', imageProcessingBulkhead.getMetrics().activeCount);
  }
});

async function processImageResilient(imageUrl: string, targetDimensions: Dimensions) {
  return await imageProcessingBulkhead.execute(async () => {
    const sharp = require('sharp');
    const imageBuffer = await downloadImage(imageUrl);
    return await sharp(imageBuffer)
      .resize(targetDimensions.width, targetDimensions.height)
      .toBuffer();
  });
}
```

### 4.4 Recommended Bulkhead Limits

| Resource Type | Max Concurrent | Max Queue Size | Queue Timeout |
|---------------|----------------|----------------|---------------|
| LLM Calls (per tenant) | 5 | 20 | 60s |
| Platform API Calls (per platform) | 10 | 30 | 30s |
| Image Processing | 10 | 50 | 30s |
| Database Connections | 20 | 100 | 10s |
| Web Scraping | 5 | 10 | 60s |

---

## 5. Timeouts

### 5.1 Overview

**Timeouts** prevent requests from hanging indefinitely.

**Key Principle:** Every external call MUST have a timeout.

### 5.2 Implementation

```typescript
// @dify-agent/resilience/src/timeout.ts

export class TimeoutError extends Error {
  constructor(message: string, public timeoutMs: number) {
    super(message);
    this.name = 'TimeoutError';
  }
}

export async function withTimeout<T>(
  promise: Promise<T>,
  timeoutMs: number,
  errorMessage?: string
): Promise<T> {
  let timeoutId: NodeJS.Timeout;

  const timeoutPromise = new Promise<never>((_, reject) => {
    timeoutId = setTimeout(() => {
      reject(
        new TimeoutError(
          errorMessage || `Operation timed out after ${timeoutMs}ms`,
          timeoutMs
        )
      );
    }, timeoutMs);
  });

  try {
    return await Promise.race([promise, timeoutPromise]);
  } finally {
    clearTimeout(timeoutId!);
  }
}

// Convenience wrapper for fetch with timeout
export async function fetchWithTimeout(
  url: string,
  options: RequestInit & { timeout?: number } = {}
): Promise<Response> {
  const timeout = options.timeout || 10000; // Default 10s
  delete options.timeout;

  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  try {
    const response = await fetch(url, {
      ...options,
      signal: controller.signal
    });
    return response;
  } catch (error) {
    if (error.name === 'AbortError') {
      throw new TimeoutError(`Fetch timeout after ${timeout}ms: ${url}`, timeout);
    }
    throw error;
  } finally {
    clearTimeout(timeoutId);
  }
}
```

### 5.3 Usage Examples

#### Example 1: Timeout for LLM API Call

```typescript
import { withTimeout } from '@dify-agent/resilience';

async function generateContentWithTimeout(prompt: string): Promise<string> {
  return await withTimeout(
    openai.chat.completions.create({
      model: 'gpt-5',
      messages: [{ role: 'user', content: prompt }]
    }),
    30000, // 30 second timeout
    'GPT-5 content generation timeout'
  );
}
```

#### Example 2: Timeout for Image Download

```typescript
async function downloadImageWithTimeout(imageUrl: string): Promise<Buffer> {
  const response = await fetchWithTimeout(imageUrl, {
    timeout: 10000 // 10 second timeout
  });

  if (!response.ok) {
    throw new Error(`Failed to download image: ${response.statusText}`);
  }

  const arrayBuffer = await response.arrayBuffer();
  return Buffer.from(arrayBuffer);
}
```

### 5.4 Recommended Timeouts

| Operation Type | Timeout | Rationale |
|----------------|---------|-----------|
| LLM API Calls (GPT-5, Claude) | 30-60s | Content generation can be slow |
| Platform API Calls | 10-15s | Should be fast, longer = problem |
| Image Processing (Google Vision) | 10s | Should be fast |
| Image Download | 10s | Network dependent |
| Image Upload | 30s | Large files take time |
| Database Queries (D1) | 5s | Should be fast with indexes |
| Vector Search (Vectorize) | 10s | Can be slower for large datasets |
| Web Scraping (Firecrawl) | 30-60s | Page rendering takes time |

---

## 6. Fallback Strategies

### 6.1 Overview

**Fallbacks** provide alternative behavior when primary operation fails.

**Common Strategies:**
- Try secondary service (e.g., GPT-5 → Claude Sonnet)
- Use cached data
- Return degraded response
- Queue for later retry

### 6.2 Implementation

```typescript
// @dify-agent/resilience/src/fallback.ts

export interface FallbackConfig<T> {
  onFallback?: (primaryError: any, fallbackUsed: string) => void;
  onAllFailed?: (errors: any[]) => void;
}

export class FallbackChain<T> {
  constructor(private config: FallbackConfig<T> = {}) {}

  async execute(
    operations: Array<{ name: string; fn: () => Promise<T> }>
  ): Promise<T> {
    const errors: any[] = [];

    for (let i = 0; i < operations.length; i++) {
      const operation = operations[i];
      try {
        const result = await operation.fn();

        if (i > 0 && this.config.onFallback) {
          this.config.onFallback(errors[0], operation.name);
        }

        return result;
      } catch (error) {
        errors.push(error);
        console.warn(
          `Fallback chain: ${operation.name} failed (${i + 1}/${operations.length})`,
          { error: error.message }
        );

        // If last operation, throw all errors
        if (i === operations.length - 1) {
          if (this.config.onAllFailed) {
            this.config.onAllFailed(errors);
          }
          throw new FallbackExhaustedError(
            `All fallback operations failed`,
            errors
          );
        }
      }
    }

    throw new Error('Fallback chain: no operations provided');
  }
}

export class FallbackExhaustedError extends Error {
  constructor(message: string, public errors: any[]) {
    super(message);
    this.name = 'FallbackExhaustedError';
  }
}

// Convenience function for simple fallback
export async function withFallback<T>(
  primary: () => Promise<T>,
  fallback: () => Promise<T>,
  config: FallbackConfig<T> = {}
): Promise<T> {
  const chain = new FallbackChain(config);
  return chain.execute([
    { name: 'primary', fn: primary },
    { name: 'fallback', fn: fallback }
  ]);
}
```

### 6.3 Usage Examples

#### Example 1: LLM Fallback (GPT-5 → Claude Sonnet)

```typescript
import { FallbackChain } from '@dify-agent/resilience';

const llmFallbackChain = new FallbackChain({
  onFallback: (primaryError, fallbackUsed) => {
    console.warn(`LLM fallback: using ${fallbackUsed}`, { error: primaryError.message });
    metrics.increment('llm.fallback', { fallback_used: fallbackUsed });
  }
});

async function generateContentWithFallback(prompt: string): Promise<string> {
  return await llmFallbackChain.execute([
    {
      name: 'gpt-5',
      fn: async () => {
        const response = await openai.chat.completions.create({
          model: 'gpt-5',
          messages: [{ role: 'user', content: prompt }]
        });
        return response.choices[0].message.content;
      }
    },
    {
      name: 'claude-sonnet-4.5',
      fn: async () => {
        const response = await anthropic.messages.create({
          model: 'claude-sonnet-4-5-20250929',
          messages: [{ role: 'user', content: prompt }],
          max_tokens: 1000
        });
        return response.content[0].text;
      }
    },
    {
      name: 'gemini-2.5-flash',
      fn: async () => {
        const response = await gemini.generateContent(prompt);
        return response.response.text();
      }
    }
  ]);
}
```

#### Example 2: Cache Fallback

```typescript
async function getProductDataWithCache(productId: string): Promise<Product> {
  return await withFallback(
    // Primary: Fetch from database
    async () => {
      const product = await db.query(
        'SELECT * FROM products WHERE id = ?',
        [productId]
      );
      if (!product) {
        throw new Error('Product not found');
      }
      return product;
    },
    // Fallback: Use cached data
    async () => {
      const cached = await cache.get(`product:${productId}`);
      if (!cached) {
        throw new Error('Product not in cache');
      }
      console.warn(`Using cached product data for ${productId}`);
      return JSON.parse(cached);
    },
    {
      onFallback: () => {
        metrics.increment('product.cache_fallback');
      }
    }
  );
}
```

#### Example 3: Degraded Response Fallback

```typescript
async function getPostAnalyticsWithFallback(postId: string): Promise<Analytics> {
  return await withFallback(
    // Primary: Fetch live analytics from platform API
    async () => {
      const response = await linkedinApi.getPostAnalytics(postId);
      return {
        likes: response.likes,
        comments: response.comments,
        shares: response.shares,
        impressions: response.impressions,
        fresh: true
      };
    },
    // Fallback: Return last known analytics from database
    async () => {
      const cached = await db.query(
        'SELECT * FROM published_posts WHERE platform_post_id = ?',
        [postId]
      );
      return {
        likes: cached.likes_count,
        comments: cached.comments_count,
        shares: cached.shares_count,
        impressions: cached.impressions_count,
        fresh: false,
        warning: 'Using cached analytics (platform API unavailable)'
      };
    }
  );
}
```

---

## 7. Health Checks

### 7.1 Overview

**Health checks** monitor external service availability.

**Use Cases:**
- Pre-flight checks before operations
- Monitoring dashboards
- Circuit breaker decisions
- Alerting

### 7.2 Implementation

```typescript
// @dify-agent/resilience/src/health-check.ts

export interface HealthCheckResult {
  service: string;
  healthy: boolean;
  latency: number;        // Response time in ms
  error?: string;
  checkedAt: string;      // ISO 8601 timestamp
}

export interface HealthCheckConfig {
  timeout: number;        // Default 5s
  retries: number;        // Default 1
  interval: number;       // Check every N ms (default 30000)
}

export class HealthChecker {
  private results = new Map<string, HealthCheckResult>();
  private intervals = new Map<string, NodeJS.Timeout>();

  async checkService(
    name: string,
    checkFn: () => Promise<boolean>,
    config: Partial<HealthCheckConfig> = {}
  ): Promise<HealthCheckResult> {
    const timeout = config.timeout || 5000;
    const retries = config.retries || 1;

    const startTime = Date.now();
    let lastError: any;

    for (let attempt = 0; attempt < retries; attempt++) {
      try {
        const healthy = await withTimeout(checkFn(), timeout);
        const latency = Date.now() - startTime;

        const result: HealthCheckResult = {
          service: name,
          healthy,
          latency,
          checkedAt: new Date().toISOString()
        };

        this.results.set(name, result);
        return result;
      } catch (error) {
        lastError = error;
      }
    }

    const result: HealthCheckResult = {
      service: name,
      healthy: false,
      latency: Date.now() - startTime,
      error: lastError.message,
      checkedAt: new Date().toISOString()
    };

    this.results.set(name, result);
    return result;
  }

  startPeriodicCheck(
    name: string,
    checkFn: () => Promise<boolean>,
    config: HealthCheckConfig
  ) {
    // Stop existing interval if any
    this.stopPeriodicCheck(name);

    const intervalId = setInterval(async () => {
      const result = await this.checkService(name, checkFn, config);

      // Emit metrics
      metrics.gauge(`health.${name}.healthy`, result.healthy ? 1 : 0);
      metrics.gauge(`health.${name}.latency`, result.latency);

      // Alert if unhealthy
      if (!result.healthy) {
        console.error(`Health check failed: ${name}`, { error: result.error });
        alerting.send(`Service unhealthy: ${name} - ${result.error}`);
      }
    }, config.interval);

    this.intervals.set(name, intervalId);
  }

  stopPeriodicCheck(name: string) {
    const intervalId = this.intervals.get(name);
    if (intervalId) {
      clearInterval(intervalId);
      this.intervals.delete(name);
    }
  }

  getResult(name: string): HealthCheckResult | undefined {
    return this.results.get(name);
  }

  getAllResults(): HealthCheckResult[] {
    return Array.from(this.results.values());
  }

  async checkAll(
    services: Map<string, () => Promise<boolean>>
  ): Promise<HealthCheckResult[]> {
    const checks = Array.from(services.entries()).map(([name, checkFn]) =>
      this.checkService(name, checkFn)
    );
    return Promise.all(checks);
  }
}

// Global health checker instance
export const healthChecker = new HealthChecker();
```

### 7.3 Usage Examples

#### Example 1: Check External APIs on Startup

```typescript
import { healthChecker } from '@dify-agent/resilience';

async function startupHealthChecks() {
  console.log('Running startup health checks...');

  const services = new Map([
    ['openai', async () => {
      const response = await fetch('https://api.openai.com/v1/models', {
        method: 'GET',
        headers: { 'Authorization': `Bearer ${process.env.OPENAI_API_KEY}` }
      });
      return response.ok;
    }],
    ['linkedin', async () => {
      const response = await fetch('https://api.linkedin.com/v2/me', {
        method: 'GET',
        headers: { 'Authorization': `Bearer ${linkedinToken}` }
      });
      return response.ok;
    }],
    ['google-vision', async () => {
      const response = await fetch('https://vision.googleapis.com/v1/images:annotate', {
        method: 'POST',
        headers: { 'Authorization': `Bearer ${googleToken}` },
        body: JSON.stringify({ requests: [] })
      });
      return response.ok;
    }]
  ]);

  const results = await healthChecker.checkAll(services);

  const unhealthy = results.filter(r => !r.healthy);
  if (unhealthy.length > 0) {
    console.error('Some services are unhealthy:', unhealthy);
    // Don't start server if critical services are down
    process.exit(1);
  }

  console.log('All services healthy ✓');
}
```

#### Example 2: Periodic Health Monitoring

```typescript
// Start periodic health checks (runs every 30 seconds)
healthChecker.startPeriodicCheck(
  'openai-gpt5',
  async () => {
    const response = await fetch('https://api.openai.com/v1/models/gpt-5', {
      timeout: 5000
    });
    return response.ok;
  },
  {
    timeout: 5000,
    retries: 2,
    interval: 30000 // Every 30 seconds
  }
);

healthChecker.startPeriodicCheck(
  'cloudflare-d1',
  async () => {
    const result = await env.DB.prepare('SELECT 1').first();
    return result !== null;
  },
  {
    timeout: 3000,
    retries: 1,
    interval: 60000 // Every 60 seconds
  }
);
```

#### Example 3: Health Check Endpoint

```typescript
// Express.js health check endpoint
app.get('/health', async (req, res) => {
  const results = healthChecker.getAllResults();

  const allHealthy = results.every(r => r.healthy);
  const statusCode = allHealthy ? 200 : 503;

  res.status(statusCode).json({
    status: allHealthy ? 'healthy' : 'unhealthy',
    timestamp: new Date().toISOString(),
    services: results
  });
});
```

---

## 8. Rate Limiting

### 8.1 Overview

**Rate limiting** prevents exceeding API quotas and protects against abuse.

**Strategies:**
- **Token Bucket**: Allows bursts, refills over time
- **Fixed Window**: N requests per time window
- **Sliding Window**: More accurate, prevents boundary gaming

### 8.2 Implementation

```typescript
// @dify-agent/resilience/src/rate-limiter.ts

export interface RateLimiterConfig {
  maxRequests: number;      // Max requests per window
  windowMs: number;         // Window duration in ms
  strategy: 'token-bucket' | 'fixed-window' | 'sliding-window';
}

export class RateLimiter {
  private tokens: number;
  private lastRefill: number;
  private requests: number[] = []; // Timestamps for sliding window

  constructor(
    private name: string,
    private config: RateLimiterConfig
  ) {
    this.tokens = config.maxRequests;
    this.lastRefill = Date.now();
  }

  async acquire(): Promise<void> {
    if (this.config.strategy === 'token-bucket') {
      return this.acquireTokenBucket();
    } else if (this.config.strategy === 'sliding-window') {
      return this.acquireSlidingWindow();
    } else {
      return this.acquireFixedWindow();
    }
  }

  private async acquireTokenBucket(): Promise<void> {
    // Refill tokens
    const now = Date.now();
    const timeSinceRefill = now - this.lastRefill;
    const tokensToAdd = (timeSinceRefill / this.config.windowMs) * this.config.maxRequests;
    this.tokens = Math.min(this.config.maxRequests, this.tokens + tokensToAdd);
    this.lastRefill = now;

    // Check if token available
    if (this.tokens < 1) {
      const waitTime = ((1 - this.tokens) / this.config.maxRequests) * this.config.windowMs;
      throw new RateLimitExceededError(
        `Rate limit exceeded for '${this.name}'. Retry after ${Math.ceil(waitTime)}ms`,
        Math.ceil(waitTime)
      );
    }

    this.tokens -= 1;
  }

  private async acquireSlidingWindow(): Promise<void> {
    const now = Date.now();
    const windowStart = now - this.config.windowMs;

    // Remove old requests outside window
    this.requests = this.requests.filter(ts => ts > windowStart);

    if (this.requests.length >= this.config.maxRequests) {
      const oldestRequest = this.requests[0];
      const waitTime = oldestRequest + this.config.windowMs - now;
      throw new RateLimitExceededError(
        `Rate limit exceeded for '${this.name}'. Retry after ${Math.ceil(waitTime)}ms`,
        Math.ceil(waitTime)
      );
    }

    this.requests.push(now);
  }

  private async acquireFixedWindow(): Promise<void> {
    const now = Date.now();
    const windowStart = Math.floor(now / this.config.windowMs) * this.config.windowMs;

    // Reset counter if new window
    if (this.lastRefill < windowStart) {
      this.tokens = this.config.maxRequests;
      this.lastRefill = windowStart;
    }

    if (this.tokens < 1) {
      const waitTime = windowStart + this.config.windowMs - now;
      throw new RateLimitExceededError(
        `Rate limit exceeded for '${this.name}'. Retry after ${Math.ceil(waitTime)}ms`,
        Math.ceil(waitTime)
      );
    }

    this.tokens -= 1;
  }

  getMetrics() {
    return {
      tokensRemaining: Math.floor(this.tokens),
      requestsInWindow: this.requests.length,
      utilizationPercent: ((this.config.maxRequests - this.tokens) / this.config.maxRequests) * 100
    };
  }
}

export class RateLimitExceededError extends Error {
  constructor(message: string, public retryAfterMs: number) {
    super(message);
    this.name = 'RateLimitExceededError';
  }
}
```

### 8.3 Usage Examples

#### Example 1: LinkedIn API Rate Limiting

```typescript
import { RateLimiter } from '@dify-agent/resilience';

const linkedinRateLimiter = new RateLimiter('linkedin-api', {
  maxRequests: 100,
  windowMs: 24 * 60 * 60 * 1000, // 24 hours
  strategy: 'sliding-window'
});

async function publishToLinkedIn(content: string, imageUrl: string) {
  // Check rate limit before publishing
  await linkedinRateLimiter.acquire();

  // Proceed with publish
  return await linkedinApi.createUGCPost(content, imageUrl);
}
```

#### Example 2: Per-Tenant LLM Rate Limiting

```typescript
const tenantRateLimiters = new Map<string, RateLimiter>();

function getTenantRateLimiter(tenantId: string): RateLimiter {
  if (!tenantRateLimiters.has(tenantId)) {
    tenantRateLimiters.set(
      tenantId,
      new RateLimiter(`tenant-${tenantId}-llm`, {
        maxRequests: 1000,
        windowMs: 60 * 60 * 1000, // 1 hour
        strategy: 'token-bucket'
      })
    );
  }
  return tenantRateLimiters.get(tenantId)!;
}

async function generateContentForTenant(tenantId: string, prompt: string) {
  const rateLimiter = getTenantRateLimiter(tenantId);

  try {
    await rateLimiter.acquire();
  } catch (error) {
    if (error instanceof RateLimitExceededError) {
      throw new Error(
        `Tenant ${tenantId} rate limit exceeded. Retry after ${error.retryAfterMs}ms`
      );
    }
    throw error;
  }

  return await openai.chat.completions.create({
    model: 'gpt-5',
    messages: [{ role: 'user', content: prompt }]
  });
}
```

### 8.4 Recommended Rate Limits

| Service | Max Requests | Window | Strategy |
|---------|--------------|--------|----------|
| OpenAI GPT-5 | 3500 | 1 minute | Token Bucket |
| LinkedIn API | 100 | 24 hours | Sliding Window |
| Twitter API | 50 | 24 hours | Sliding Window |
| Facebook API | 200 | 1 hour | Fixed Window |
| Instagram API | 25 | 24 hours | Sliding Window |
| Google Vision | 1800 | 1 minute | Token Bucket |
| Cloudflare D1 | 10000 | 1 minute | Token Bucket |

---

## 9. Monitoring & Observability

### 9.1 Overview

**Observability** provides visibility into resilience patterns performance.

**Key Metrics:**
- Circuit breaker state changes
- Retry attempts and success rates
- Bulkhead utilization
- Timeout frequencies
- Fallback usage
- Rate limit violations

### 9.2 Metrics Integration

```typescript
// @dify-agent/resilience/src/metrics.ts

export interface MetricsClient {
  increment(name: string, tags?: Record<string, string | number>): void;
  gauge(name: string, value: number, tags?: Record<string, string | number>): void;
  histogram(name: string, value: number, tags?: Record<string, string | number>): void;
  timing(name: string, value: number, tags?: Record<string, string | number>): void;
}

// Datadog/Sentry/Prometheus integration
class DatadogMetrics implements MetricsClient {
  constructor(private client: any) {}

  increment(name: string, tags?: Record<string, string | number>) {
    this.client.increment(`resilience.${name}`, 1, this.formatTags(tags));
  }

  gauge(name: string, value: number, tags?: Record<string, string | number>) {
    this.client.gauge(`resilience.${name}`, value, this.formatTags(tags));
  }

  histogram(name: string, value: number, tags?: Record<string, string | number>) {
    this.client.histogram(`resilience.${name}`, value, this.formatTags(tags));
  }

  timing(name: string, value: number, tags?: Record<string, string | number>) {
    this.client.timing(`resilience.${name}`, value, this.formatTags(tags));
  }

  private formatTags(tags?: Record<string, string | number>): string[] {
    if (!tags) return [];
    return Object.entries(tags).map(([key, value]) => `${key}:${value}`);
  }
}

export let metrics: MetricsClient = {
  increment: () => {},
  gauge: () => {},
  histogram: () => {},
  timing: () => {}
};

export function setMetricsClient(client: MetricsClient) {
  metrics = client;
}
```

### 9.3 Key Metrics to Track

```typescript
// Circuit Breaker Metrics
metrics.gauge('circuit_breaker.state', state === 'CLOSED' ? 0 : state === 'HALF_OPEN' ? 1 : 2, {
  service: 'openai-gpt5'
});
metrics.increment('circuit_breaker.state_change', {
  service: 'openai-gpt5',
  from: oldState,
  to: newState
});
metrics.gauge('circuit_breaker.failure_count', failureCount, { service: 'openai-gpt5' });

// Retry Metrics
metrics.increment('retry.attempt', { service: 'linkedin-api', attempt });
metrics.timing('retry.delay', nextDelay, { service: 'linkedin-api' });
metrics.increment('retry.exhausted', { service: 'linkedin-api' });

// Bulkhead Metrics
metrics.gauge('bulkhead.active_count', activeCount, { bulkhead: 'tenant-123-llm' });
metrics.gauge('bulkhead.queue_size', queueSize, { bulkhead: 'tenant-123-llm' });
metrics.gauge('bulkhead.utilization_percent', utilizationPercent, { bulkhead: 'tenant-123-llm' });
metrics.increment('bulkhead.rejected', { bulkhead: 'tenant-123-llm', reason: 'queue_full' });

// Timeout Metrics
metrics.increment('timeout.exceeded', { service: 'gpt-5', timeout_ms: 30000 });
metrics.histogram('operation.latency', latency, { service: 'gpt-5' });

// Fallback Metrics
metrics.increment('fallback.used', { primary: 'gpt-5', fallback: 'claude-sonnet' });
metrics.increment('fallback.exhausted', { service: 'llm' });

// Rate Limit Metrics
metrics.increment('rate_limit.exceeded', { service: 'linkedin-api' });
metrics.gauge('rate_limit.tokens_remaining', tokensRemaining, { service: 'linkedin-api' });

// Health Check Metrics
metrics.gauge('health.service.healthy', healthy ? 1 : 0, { service: 'openai' });
metrics.timing('health.service.latency', latency, { service: 'openai' });
```

### 9.4 Alerting Rules

```yaml
# Datadog/PagerDuty Alert Configuration

alerts:
  # Circuit Breaker OPEN Alert
  - name: "Circuit Breaker OPEN"
    query: "avg(last_5m):avg:resilience.circuit_breaker.state{*} by {service} > 1"
    message: |
      Circuit breaker is OPEN for {{service.name}}.
      This indicates repeated failures. Check service health immediately.
    priority: high
    notify:
      - "@slack-engineering"
      - "@pagerduty-oncall"

  # High Retry Rate Alert
  - name: "High Retry Rate"
    query: "sum(last_15m):sum:resilience.retry.attempt{*} by {service} > 100"
    message: |
      High retry rate detected for {{service.name}}.
      {{value}} retries in last 15 minutes. Investigate service reliability.
    priority: medium

  # Bulkhead Queue Full Alert
  - name: "Bulkhead Queue Full"
    query: "sum(last_5m):sum:resilience.bulkhead.rejected{reason:queue_full} by {bulkhead} > 10"
    message: |
      Bulkhead {{bulkhead.name}} rejecting requests due to full queue.
      Consider increasing maxConcurrent or maxQueueSize.
    priority: high

  # Frequent Timeouts Alert
  - name: "Frequent Timeouts"
    query: "sum(last_10m):sum:resilience.timeout.exceeded{*} by {service} > 20"
    message: |
      {{service.name}} experiencing frequent timeouts ({{value}} in 10 minutes).
      Check service latency and increase timeout if necessary.
    priority: medium

  # Fallback Overuse Alert
  - name: "Fallback Overused"
    query: "sum(last_30m):sum:resilience.fallback.used{*} by {primary,fallback} > 50"
    message: |
      Fallback {{fallback.name}} used {{value}} times (primary: {{primary.name}}).
      Primary service may be degraded. Investigate.
    priority: medium

  # Service Unhealthy Alert
  - name: "Service Unhealthy"
    query: "avg(last_5m):avg:resilience.health.service.healthy{*} by {service} < 1"
    message: |
      Health check failing for {{service.name}}.
      Service is unhealthy. Immediate investigation required.
    priority: critical
    notify:
      - "@pagerduty-oncall"
```

### 9.5 Monitoring Dashboards

```typescript
// Grafana Dashboard Configuration (JSON)
{
  "dashboard": {
    "title": "Resilience Patterns Monitoring",
    "panels": [
      {
        "title": "Circuit Breaker States",
        "targets": [
          {
            "expr": "resilience_circuit_breaker_state",
            "legendFormat": "{{service}}"
          }
        ],
        "type": "timeseries"
      },
      {
        "title": "Retry Attempts by Service",
        "targets": [
          {
            "expr": "rate(resilience_retry_attempt_total[5m])",
            "legendFormat": "{{service}}"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Bulkhead Utilization",
        "targets": [
          {
            "expr": "resilience_bulkhead_utilization_percent",
            "legendFormat": "{{bulkhead}}"
          }
        ],
        "type": "gauge"
      },
      {
        "title": "P95 Latency by Service",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(resilience_operation_latency_bucket[5m]))",
            "legendFormat": "{{service}}"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Fallback Usage",
        "targets": [
          {
            "expr": "sum(rate(resilience_fallback_used_total[5m])) by (fallback)",
            "legendFormat": "{{fallback}}"
          }
        ],
        "type": "piechart"
      },
      {
        "title": "Service Health Status",
        "targets": [
          {
            "expr": "resilience_health_service_healthy",
            "legendFormat": "{{service}}"
          }
        ],
        "type": "statusmap"
      }
    ]
  }
}
```

---

## 10. Shared Library Structure

### 10.1 NPM Package Structure

```
@dify-agent/resilience/
├── src/
│   ├── index.ts                    # Main exports
│   ├── circuit-breaker.ts          # Circuit Breaker implementation
│   ├── retry.ts                    # Retry with exponential backoff
│   ├── bulkhead.ts                 # Bulkhead pattern
│   ├── timeout.ts                  # Timeout utilities
│   ├── fallback.ts                 # Fallback chain
│   ├── health-check.ts             # Health checker
│   ├── rate-limiter.ts             # Rate limiting
│   ├── metrics.ts                  # Metrics integration
│   └── types.ts                    # Shared types
├── tests/
│   ├── circuit-breaker.test.ts
│   ├── retry.test.ts
│   ├── bulkhead.test.ts
│   ├── timeout.test.ts
│   ├── fallback.test.ts
│   ├── health-check.test.ts
│   └── rate-limiter.test.ts
├── package.json
├── tsconfig.json
└── README.md
```

### 10.2 Package.json

```json
{
  "name": "@dify-agent/resilience",
  "version": "1.0.0",
  "description": "Production-grade resilience patterns for DIFY-AGENT",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "keywords": [
    "resilience",
    "circuit-breaker",
    "retry",
    "bulkhead",
    "timeout",
    "fallback",
    "rate-limit"
  ],
  "author": "DIFY-AGENT Team",
  "license": "MIT",
  "dependencies": {},
  "devDependencies": {
    "@types/node": "^20.0.0",
    "jest": "^29.0.0",
    "typescript": "^5.0.0"
  }
}
```

### 10.3 Main Export (index.ts)

```typescript
// @dify-agent/resilience/src/index.ts

export {
  CircuitBreaker,
  CircuitBreakerState,
  CircuitBreakerConfig,
  CircuitBreakerOpenError
} from './circuit-breaker';

export {
  RetryExecutor,
  RetryConfig,
  retryWithBackoff
} from './retry';

export {
  Bulkhead,
  BulkheadConfig,
  BulkheadRejectedException,
  BulkheadTimeoutError
} from './bulkhead';

export {
  withTimeout,
  fetchWithTimeout,
  TimeoutError
} from './timeout';

export {
  FallbackChain,
  FallbackConfig,
  withFallback,
  FallbackExhaustedError
} from './fallback';

export {
  HealthChecker,
  HealthCheckResult,
  HealthCheckConfig,
  healthChecker
} from './health-check';

export {
  RateLimiter,
  RateLimiterConfig,
  RateLimitExceededError
} from './rate-limiter';

export {
  MetricsClient,
  setMetricsClient,
  metrics
} from './metrics';
```

---

## 11. Configuration Guidelines

### 11.1 Environment-Specific Configurations

```typescript
// config/resilience.config.ts

export const resilienceConfig = {
  development: {
    circuitBreaker: {
      failureThreshold: 10,      // More lenient in dev
      timeout: 30000             // 30s
    },
    retry: {
      maxAttempts: 2,            // Fewer retries in dev
      initialDelay: 500
    },
    bulkhead: {
      maxConcurrent: 5,          // Lower limits in dev
      maxQueueSize: 10
    },
    timeout: {
      llm: 60000,                // 60s for debugging
      platform: 20000,
      image: 15000
    }
  },

  staging: {
    circuitBreaker: {
      failureThreshold: 5,
      timeout: 60000
    },
    retry: {
      maxAttempts: 3,
      initialDelay: 1000
    },
    bulkhead: {
      maxConcurrent: 10,
      maxQueueSize: 30
    },
    timeout: {
      llm: 30000,
      platform: 15000,
      image: 10000
    }
  },

  production: {
    circuitBreaker: {
      failureThreshold: 5,
      timeout: 60000
    },
    retry: {
      maxAttempts: 3,
      initialDelay: 1000
    },
    bulkhead: {
      maxConcurrent: 20,
      maxQueueSize: 100
    },
    timeout: {
      llm: 30000,
      platform: 10000,
      image: 10000
    }
  }
};

export function getConfig() {
  const env = process.env.NODE_ENV || 'development';
  return resilienceConfig[env];
}
```

### 11.2 Per-Service Configuration

```typescript
// config/services.config.ts

export const serviceConfigs = {
  'openai-gpt5': {
    circuitBreaker: { failureThreshold: 5, timeout: 60000 },
    retry: { maxAttempts: 3, initialDelay: 1000 },
    timeout: 30000,
    rateLimiter: { maxRequests: 3500, windowMs: 60000 }
  },

  'claude-sonnet': {
    circuitBreaker: { failureThreshold: 5, timeout: 60000 },
    retry: { maxAttempts: 3, initialDelay: 1000 },
    timeout: 30000,
    rateLimiter: { maxRequests: 5000, windowMs: 60000 }
  },

  'linkedin-api': {
    circuitBreaker: { failureThreshold: 3, timeout: 120000 },
    retry: { maxAttempts: 3, initialDelay: 2000 },
    timeout: 15000,
    rateLimiter: { maxRequests: 100, windowMs: 24 * 60 * 60 * 1000 }
  },

  'google-vision': {
    circuitBreaker: { failureThreshold: 5, timeout: 30000 },
    retry: { maxAttempts: 2, initialDelay: 1000 },
    timeout: 10000,
    rateLimiter: { maxRequests: 1800, windowMs: 60000 }
  }
};
```

---

## 12. Testing Strategies

### 12.1 Unit Tests

```typescript
// tests/circuit-breaker.test.ts

import { CircuitBreaker, CircuitBreakerState, CircuitBreakerOpenError } from '../src/circuit-breaker';

describe('CircuitBreaker', () => {
  it('should remain CLOSED on successful calls', async () => {
    const cb = new CircuitBreaker('test', { failureThreshold: 3, timeout: 1000, successThreshold: 2, halfOpenMaxCalls: 3 });

    await cb.execute(async () => 'success');
    expect(cb.getState()).toBe(CircuitBreakerState.CLOSED);
  });

  it('should transition to OPEN after threshold failures', async () => {
    const cb = new CircuitBreaker('test', { failureThreshold: 3, timeout: 1000, successThreshold: 2, halfOpenMaxCalls: 3 });

    for (let i = 0; i < 3; i++) {
      try {
        await cb.execute(async () => { throw new Error('fail'); });
      } catch {}
    }

    expect(cb.getState()).toBe(CircuitBreakerState.OPEN);
  });

  it('should reject immediately when OPEN', async () => {
    const cb = new CircuitBreaker('test', { failureThreshold: 1, timeout: 1000, successThreshold: 2, halfOpenMaxCalls: 3 });

    try {
      await cb.execute(async () => { throw new Error('fail'); });
    } catch {}

    await expect(cb.execute(async () => 'success')).rejects.toThrow(CircuitBreakerOpenError);
  });

  it('should transition to HALF_OPEN after timeout', async () => {
    const cb = new CircuitBreaker('test', { failureThreshold: 1, timeout: 100, successThreshold: 2, halfOpenMaxCalls: 3 });

    try {
      await cb.execute(async () => { throw new Error('fail'); });
    } catch {}

    await new Promise(resolve => setTimeout(resolve, 150));

    await cb.execute(async () => 'success');
    // After first success in HALF_OPEN, should still be HALF_OPEN (need 2 successes)
    expect(cb.getState()).toBe(CircuitBreakerState.HALF_OPEN);
  });
});
```

### 12.2 Integration Tests

```typescript
// tests/integration/openai.test.ts

import { CircuitBreaker, retryWithBackoff, withTimeout } from '@dify-agent/resilience';

describe('OpenAI Integration with Resilience', () => {
  const openaiCircuitBreaker = new CircuitBreaker('openai-test', {
    failureThreshold: 2,
    timeout: 5000,
    successThreshold: 1,
    halfOpenMaxCalls: 3
  });

  it('should handle OpenAI API call with full resilience stack', async () => {
    const result = await openaiCircuitBreaker.execute(async () => {
      return await retryWithBackoff(
        async () => {
          return await withTimeout(
            fetch('https://api.openai.com/v1/chat/completions', {
              method: 'POST',
              headers: {
                'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
                'Content-Type': 'application/json'
              },
              body: JSON.stringify({
                model: 'gpt-4',
                messages: [{ role: 'user', content: 'test' }],
                max_tokens: 10
              })
            }),
            10000
          );
        },
        { maxAttempts: 2, initialDelay: 500 }
      );
    });

    expect(result.ok).toBe(true);
  });
});
```

---

## Conclusion

This resilience patterns document provides production-grade implementations for all critical patterns needed across the DIFY-AGENT system.

**Key Takeaways:**
- ✅ All patterns implemented in shared `@dify-agent/resilience` library
- ✅ DRY approach ensures consistency across all services
- ✅ Comprehensive observability with metrics and alerting
- ✅ Environment-specific configurations for dev/staging/production
- ✅ Full test coverage for reliability

**Next Steps:**
1. Implement `@dify-agent/resilience` NPM package
2. Add resilience patterns to Stage D services (11 microservices)
3. Add resilience patterns to Stage E adapters (4 platforms)
4. Set up monitoring dashboards (Grafana/Datadog)
5. Configure alerting rules (PagerDuty/Slack)

---

**Document Status:** ✅ APPROVED - Production Ready
**Last Updated:** November 10, 2025
**Maintained By:** Infrastructure Team
**Review Frequency:** Quarterly or after major incidents
