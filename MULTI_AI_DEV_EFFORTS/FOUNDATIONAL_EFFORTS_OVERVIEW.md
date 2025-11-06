# TDK Foundational Efforts - High Level Overview

**Created by:** MEGA AI 1 (Researcher)
**Date:** 2025-11-06
**Status:** Awaiting user approval before AI2 research phase

---

## Executive Summary

After deep research of the TDK requirements, SDK architecture, and plugin patterns, I've identified **3 foundational efforts** that will establish solid footing for the Tetto Development Kit (TDK).

**Critical Discovery:** Wallet-per-customer architecture is REQUIRED (not wallet pool) due to how agents identify callers.

**Total Time Estimate:** 80-100 hours across 3 efforts

**Repos Involved:**
- `tetto-portal` - Backend API and database
- `tetto-tdk` (new) - TDK client library
- `tetto-plugin-memory` (new) - WarmMemory plugin for TDK
- `tetto-plugin-answers` (new) - WarmAnswers plugin for TDK

---

## Effort 1: TDK Backend Foundation

**Goal:** Build the core infrastructure for API key-based access to Tetto agents

**What it builds:**
- Authentication system (email/password, separate from web3 marketplace)
- API key generation with dedicated Solana wallet per customer
- Database schema (TDK tables in tetto-portal)
- Wallet encryption (AES-256-GCM for private keys)
- Treasury wallet management (funds customer wallets)
- Core API endpoint: `POST /api/tdk/v1/call`
- Usage tracking and billing integration (Stripe metered billing)
- Rate limiting (tier-based)

**Critical Architecture Decision: WALLET-PER-CUSTOMER**

**Why:**
- Agents receive `tetto_context.caller_wallet` as the primary caller identifier
- Agents like WarmMemory use wallet address to separate app data
- Using shared wallet pool would mix data between TDK customers
- **Each TDK customer needs a dedicated wallet for data isolation**

**Database Schema:**
```sql
-- Each API key has its own dedicated wallet (1:1 mapping)
CREATE TABLE tdk_api_keys (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id),
  key_hash TEXT UNIQUE NOT NULL,

  -- Dedicated wallet for this customer
  dedicated_wallet_public_key TEXT NOT NULL UNIQUE,
  dedicated_wallet_secret_encrypted TEXT NOT NULL,

  -- Cached balances
  wallet_balance_sol NUMERIC DEFAULT 0,
  wallet_balance_usdc NUMERIC DEFAULT 0,

  -- Usage & billing
  tier TEXT DEFAULT 'free',
  stripe_customer_id TEXT,
  ops_this_month INTEGER DEFAULT 0,
  rate_limit_per_minute INTEGER,

  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Treasury wallet (funds customer wallets)
CREATE TABLE tdk_treasury (
  id SERIAL PRIMARY KEY,
  public_key TEXT NOT NULL,
  secret_encrypted TEXT NOT NULL,
  balance_sol NUMERIC,
  balance_usdc NUMERIC,
  total_funded_out NUMERIC DEFAULT 0
);

-- Usage tracking (for metered billing)
CREATE TABLE tdk_usage (
  id UUID PRIMARY KEY,
  api_key_id UUID REFERENCES tdk_api_keys(id),
  agent_id TEXT NOT NULL,
  agent_cost_usd NUMERIC NOT NULL,
  tdk_markup_pct NUMERIC DEFAULT 0.5,
  developer_cost_usd NUMERIC NOT NULL,
  stripe_reported BOOLEAN DEFAULT FALSE,
  timestamp TIMESTAMPTZ DEFAULT NOW()
);
```

**Key Endpoints:**
- `POST /api/tdk/auth/register` - Create TDK account (email/password)
- `POST /api/tdk/auth/login` - Login to TDK dashboard
- `POST /api/tdk/keys/generate` - Generate API key + dedicated wallet
- `POST /api/tdk/v1/call` - Call any agent (API key auth)
- `GET /api/tdk/usage/current` - Get current month usage
- `GET /api/tdk/keys` - List API keys

**Wallet Funding Flow:**
```typescript
// When customer makes API call:
1. Check dedicated wallet balance
2. If balance < threshold (e.g., $5 USDC remaining)
   → Fund from treasury wallet (async, non-blocking)
3. Use dedicated wallet to pay for agent call
4. Track cost → report to Stripe
```

**Checkpoints (8-10 CPs):**
1. Database schema & migrations
2. Wallet encryption module (AES-256-GCM)
3. Treasury wallet setup & funding
4. API key generation endpoint (creates wallet)
5. Authentication middleware (API key validation)
6. Core `/v1/call` endpoint
7. Usage tracking & Stripe integration
8. Rate limiting middleware
9. Wallet balance monitoring
10. Testing & validation

**Time Estimate:** 35-45 hours

**Success Criteria:**
- ✅ Developer can create TDK account (email/password)
- ✅ Developer can generate API key → dedicated wallet created
- ✅ Developer can call any agent via API key
- ✅ Wallet automatically funded when low
- ✅ Usage tracked and billed to Stripe
- ✅ Rate limiting works per tier
- ✅ Data isolated per customer (different wallets)

---

## Effort 2: TDK Client Library

**Goal:** Build the TypeScript client library that developers use to call agents

**What it builds:**
- NPM package: `@tetto/tdk`
- Simple API for calling agents (no crypto knowledge needed)
- Built-in shortcuts: `memory.*`, `answers.*`
- Generic method: `callAgent(agentId, input)`
- Type safety (TypeScript definitions)
- Error handling and retries
- Usage tracking client-side

**Key Design: Hybrid Approach**
- **Shortcuts** for popular agents (elegant, discoverable)
- **Generic callAgent()** for any agent (extensible)

**Example Usage:**
```typescript
import { TettoDK } from '@tetto/tdk';

const tdk = new TettoDK({ apiKey: process.env.TETTO_API_KEY });

// Shortcut methods (elegant)
await tdk.memory.set('user:prefs', { theme: 'dark' });
const prefs = await tdk.memory.get('user:prefs');

await tdk.answers.teach('The capital of France is Paris');
const answer = await tdk.answers.ask('What is the capital of France?');

// Generic method (extensible)
const result = await tdk.callAgent('sentiment-analyzer', {
  text: 'I love this product!'
});
```

**Architecture:**
```typescript
class TettoDK {
  private apiKey: string;
  private apiUrl: string;

  // Built-in shortcuts
  memory: MemoryShortcut;
  answers: AnswersShortcut;

  // Generic method
  async callAgent(agentId: string, input: object): Promise<any> {
    // POST to /api/tdk/v1/call
    // Auth: Bearer {apiKey}
    // Body: { agent_id, input }
  }

  // Usage tracking
  async getUsage(): Promise<Usage> { /* ... */ }
}
```

**Checkpoints (6-8 CPs):**
1. Project setup (TypeScript, tsup, testing)
2. Core TettoDK class (callAgent method)
3. Memory shortcut implementation
4. Answers shortcut implementation
5. Error handling & retries
6. Type definitions
7. Documentation & examples
8. NPM publishing

**Time Estimate:** 25-30 hours

**Success Criteria:**
- ✅ Published to NPM as `@tetto/tdk`
- ✅ Works in Node.js and browser
- ✅ Shortcuts work: `tdk.memory.*`, `tdk.answers.*`
- ✅ Generic `callAgent()` works for any agent
- ✅ TypeScript types complete
- ✅ Examples in /examples folder
- ✅ Tests pass (100% coverage on core)

---

## Effort 3: Plugin System & First Plugins

**Goal:** Enable community plugins and create first two official plugins

**What it builds:**
- Plugin interface for TDK client
- Official plugin: `@tetto/plugin-memory` (WarmMemory)
- Official plugin: `@tetto/plugin-answers` (WarmAnswers)
- Plugin documentation
- Plugin template repo

**Why plugins:**
- Extensibility (community can add agents)
- Separation of concerns (core vs. plugins)
- Standardization (consistent API patterns)
- Discoverability (npm search, marketplace)

**Plugin Interface:**
```typescript
// @tetto/tdk/types
export interface TDKPlugin {
  name: string;
  version: string;

  // Initialization
  init(tdk: TettoDK): void;

  // Cleanup (optional)
  destroy?(): void;
}
```

**Example Plugin Usage:**
```typescript
import { TettoDK } from '@tetto/tdk';
import { MemoryPlugin } from '@tetto/plugin-memory';
import { AnswersPlugin } from '@tetto/plugin-answers';

const tdk = new TettoDK({ apiKey: process.env.TETTO_API_KEY });

// Register plugins
tdk.use(MemoryPlugin);
tdk.use(AnswersPlugin);

// Use plugins (same API as shortcuts)
await tdk.memory.set('key', 'value');
const answer = await tdk.answers.ask('question');
```

**Memory Plugin Implementation:**
```typescript
// @tetto/plugin-memory
import type { TDKPlugin, TettoDK } from '@tetto/tdk';

export const MemoryPlugin: TDKPlugin = {
  name: 'memory',
  version: '1.0.0',

  init(tdk: TettoDK) {
    tdk.memory = {
      async set(key: string, value: any) {
        return await tdk.callAgent('warmmemory', {
          action: 'store',
          key,
          value: JSON.stringify(value)
        });
      },

      async get(key: string) {
        const result = await tdk.callAgent('warmmemory', {
          action: 'retrieve',
          key
        });
        return result.exists ? JSON.parse(result.value) : null;
      },

      async list(prefix: string = '') {
        const result = await tdk.callAgent('warmmemory', {
          action: 'list',
          prefix
        });
        return result.items;
      },

      async delete(key: string) {
        return await tdk.callAgent('warmmemory', {
          action: 'delete',
          key
        });
      }
    };
  }
};
```

**Checkpoints (5-6 CPs):**
1. Plugin interface design
2. Plugin registration system in TDK core
3. Memory plugin implementation
4. Answers plugin implementation
5. Plugin documentation & template
6. Testing (plugin isolation, lifecycle)

**Time Estimate:** 20-25 hours

**Success Criteria:**
- ✅ Plugin interface defined and documented
- ✅ TDK core supports `tdk.use(plugin)`
- ✅ Memory plugin published: `@tetto/plugin-memory`
- ✅ Answers plugin published: `@tetto/plugin-answers`
- ✅ Plugin template repo available
- ✅ Plugin isolation tested (no conflicts)
- ✅ Documentation shows how to create plugins

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     DEVELOPER'S APP                          │
│                                                              │
│  import { TettoDK } from '@tetto/tdk';                      │
│  const tdk = new TettoDK({ apiKey: 'tk_...' });            │
│                                                              │
│  await tdk.memory.set('key', 'value');  ← Shortcut         │
│  await tdk.callAgent('agent-id', {...}); ← Generic         │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       │ HTTP: POST /api/tdk/v1/call
                       │ Auth: Bearer tk_abc123...
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    TDK API (tetto-portal)                   │
│                                                              │
│  1. Validate API key (check tdk_api_keys table)            │
│  2. Get dedicated wallet for this customer                  │
│  3. Check wallet balance (fund if low)                      │
│  4. Call agent using dedicated wallet                       │
│  5. Track usage → Stripe metered billing                    │
│                                                              │
│  Database:                                                   │
│  ┌─────────────────────────────────────────────┐            │
│  │ tdk_api_keys                                │            │
│  │  - id: uuid-abc123                          │            │
│  │  - dedicated_wallet_public_key: Sol123...   │            │
│  │  - dedicated_wallet_secret_encrypted: {...} │            │
│  │  - stripe_customer_id: cus_xyz              │            │
│  └─────────────────────────────────────────────┘            │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       │ Uses dedicated wallet (Sol123...)
                       │ caller_wallet = Sol123... (unique!)
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    TETTO AGENT (e.g., WarmMemory)           │
│                                                              │
│  Receives:                                                   │
│  {                                                           │
│    input: { action: 'store', key: 'test', value: '...' },  │
│    tetto_context: {                                         │
│      caller_wallet: "Sol123...",  ← Unique per customer!   │
│      intent_id: "...",                                      │
│      timestamp: 1234567890                                  │
│    }                                                         │
│  }                                                           │
│                                                              │
│  Storage: wallet = Sol123... (isolated per TDK customer)   │
└─────────────────────────────────────────────────────────────┘
```

**Key Insight:** Each TDK customer has unique `caller_wallet` → perfect data isolation!

---

## Critical Dependencies Between Efforts

**Effort 1 → Effort 2:**
- Effort 2 needs API endpoint from Effort 1 (`/api/tdk/v1/call`)
- Effort 2 needs auth specification (how to pass API key)
- Can work in parallel if API contract defined first

**Effort 1 → Effort 3:**
- Effort 3 needs Effort 2 (plugins extend TDK client)
- Cannot start Effort 3 until Effort 2 has plugin interface

**Recommended Order:**
1. **Effort 1** (Backend Foundation) - Start first, blocks others
2. **Effort 2** (Client Library) - Start after Effort 1 CP5 (API endpoint done)
3. **Effort 3** (Plugins) - Start after Effort 2 CP2 (plugin interface defined)

**Parallelization Opportunity:**
- Effort 1 CP1-CP4 (database, encryption, wallet) → Solo
- Effort 1 CP5+ (API endpoint) + Effort 2 CP1-CP2 (setup, core) → Parallel
- Effort 2 CP3+ (shortcuts) + Effort 3 → Parallel

---

## Risk Analysis

### Risk 1: Wallet Funding Costs
**Probability:** Medium
**Impact:** High (expensive at scale)

**Scenario:** 10,000 customers × $50 USDC = $500k locked capital

**Mitigation:**
- Start with lower funding ($10 USDC initial)
- Lazy wallet creation (create on first call, not signup)
- Top-up on-demand (not pre-funding)
- Monitor actual usage patterns (may need less than estimated)

### Risk 2: Agent Data Isolation Not Guaranteed
**Probability:** Low
**Impact:** Critical (data leakage)

**Scenario:** Some agents might NOT use `caller_wallet` for isolation

**Mitigation:**
- Validate with WarmMemory agent code (AI2 research)
- Test data isolation with multiple API keys
- Document agent requirements (must respect `caller_wallet`)
- Worst case: Fallback to synthetic namespace wallets

### Risk 3: Stripe Metered Billing Complexity
**Probability:** Medium
**Impact:** Medium (billing errors)

**Scenario:** Variable pricing per agent complicates billing

**Mitigation:**
- Use Stripe usage records with metadata (agent_id, cost)
- Aggregate costs before reporting (batch reporting)
- Test thoroughly on Stripe test mode
- Monitor invoice accuracy

### Risk 4: Database Migration Conflicts
**Probability:** Low
**Impact:** High (production downtime)

**Scenario:** TDK tables conflict with existing schema

**Mitigation:**
- AI2 must validate existing schema first
- Test migration on staging environment
- Use unique table prefixes (`tdk_*`)
- Have rollback plan ready

---

## Next Steps

**For User (You):**
1. Review this high-level overview
2. Approve or request changes to the 3 efforts
3. Confirm wallet-per-customer architecture is acceptable
4. Approve AI2 to begin deep research

**For AI2 (Next Agent):**
1. Read `AI2_RESEARCH_BRIEF_EFFORT1.md`
2. Clone tetto-portal repo
3. Validate all 8 research gaps
4. Create detailed START_HERE for Effort 1
5. Document all findings and risks

**For AI3 (Implementation Agent):**
1. Receive START_HERE from AI2
2. Implement Effort 1 checkpoint by checkpoint
3. Validate each CP before moving to next
4. Hand off completed Effort 1

---

## Questions for User

Before proceeding to AI2 research phase:

1. **Wallet-per-customer approach:** Acceptable? Any concerns about locked capital?

2. **Initial wallet funding:** Start with $50 USDC or lower (e.g., $10)?

3. **Subdomain preference:** `api.tetto.io` vs `tetto.io/api/tdk`?

4. **Deployment:** Build in tetto-portal repo or separate tdk-api repo?

5. **Timeline:** Is 80-100 hours across 3 efforts acceptable for foundational work?

6. **Effort priority:** Should we focus on Effort 1 first, or parallelize with Effort 2?

**Awaiting your approval to proceed! 🚀**
