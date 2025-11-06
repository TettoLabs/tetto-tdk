# AI2 Research Brief: Effort 1 - TDK Backend Foundation

**Role:** You are MEGA AI 2 (Implementation Guide Creator)
**Goal:** Deep research to create bulletproof implementation guides for Effort 1
**Context:** AI1 has completed initial research. You have full repo access. Validate everything.

---

## Executive Summary

**Effort 1:** TDK Backend Foundation - Core infrastructure for API key-based access to Tetto agents

**Critical Architecture Decision:** WALLET-PER-CUSTOMER (not wallet pool!)

**Why this matters:**
- Agents like WarmMemory use `caller_wallet` as the primary app identifier
- Using shared wallet pool would mix data between TDK customers
- Each TDK customer needs a dedicated Solana wallet for data isolation

**Your mission:** Validate this architecture and create implementation guides that prevent errors.

---

## What AI1 Validated ✅

### 1. How Agents Identify Callers

**Source:** `/home/user/tetto-sdk/src/types.ts:17-35`

```typescript
export interface TettoContext {
  caller_wallet: string;        // ← PRIMARY IDENTIFIER
  caller_agent_id: string | null;
  intent_id: string;
  timestamp: number;
  version: string;
}
```

**Validation:** Agents receive `tetto_context.caller_wallet` in EVERY request (handler.ts:98)

**Impact:** If we use shared wallet pool, all TDK customers have same `caller_wallet` → data mixing!

### 2. WarmMemory Pattern

**Source:** `/home/user/warmmemory-plugin/src/index.ts:176`

```typescript
if (options.targetWallet) input.wallet = options.targetWallet;
```

**Pattern identified:**
- `caller_wallet` = App identity (who owns the data)
- `input.wallet` (targetWallet) = Namespace override (optional)
- Storage key pattern: `{caller_wallet}:{namespace}:{key}`

**User's concern (VALIDATED):** "agents use the wallet to identify the user (app). they also have mechanisms to store namespaces so that namespace + wallet = calling app data for a given app customer."

### 3. SDK Wallet Creation Pattern

**Source:** `/home/user/tetto-sdk/src/wallet-helpers.ts:20-29`

```typescript
export function createWalletFromKeypair(keypair: Keypair): TettoWallet {
  return {
    publicKey: keypair.publicKey,
    signTransaction: async (tx: Transaction) => {
      tx.sign(keypair);
      return tx;
    },
  };
}
```

**Validation:** Wallet creation is simple (just keypair generation). Cost: FREE (0 SOL to create)

### 4. Heterogeneous Pricing Confirmed

**Source:** TDK research docs + marketplace observation

**Agent costs range:**
- WarmMemory: $0.001 per call
- WarmAnswers: $0.04 per call
- Other agents: $0.005 - $0.02

**Impact:** Cannot use flat-rate billing. Must use Stripe metered billing with variable event values.

### 5. Stripe Metered Billing Validation

**Research:** Stripe API supports metered billing with variable event values

```typescript
await stripe.subscriptionItems.createUsageRecord(subscriptionItemId, {
  quantity: 1,
  timestamp: Math.floor(Date.now() / 1000),
  action: 'increment'
});
```

**Confirmed:** Can report usage per operation with variable costs

---

## CRITICAL Research Gaps (AI2 Must Fill!)

### GAP 1: Tetto-Portal Database Schema 🚨

**What we DON'T know:**
- Current `users` table structure
- Current `api_keys` table (for agent registration)
- Existing auth pattern (Supabase? JWT? Magic links?)
- How to ensure TDK tables don't conflict

**Why critical:**
- Cannot design TDK schema without knowing existing structure
- Risk of foreign key conflicts
- Risk of auth pattern mismatches

**Research tasks:**
1. Clone tetto-portal repo (you have access)
2. Read `/tetto-portal/supabase/migrations/*.sql` (or equivalent)
3. Identify existing tables: `users`, `api_keys`, `agents`, `receipts`
4. Document current auth.users structure
5. Check if `uuid_generate_v4()` extension exists
6. Validate RLS (Row Level Security) patterns

**Deliverable:** Complete existing schema documentation (300-500 lines)

---

### GAP 2: Existing Encryption Patterns 🚨

**What we DON'T know:**
- Does tetto-portal already encrypt anything?
- What encryption libraries are in use? (`crypto`, `bcrypt`, `argon2`?)
- How are agent registration API keys hashed?
- What env var patterns exist for secrets?

**Why critical:**
- Need to follow existing security patterns
- Cannot duplicate encryption key env vars
- Risk of using wrong hashing algorithm

**Research tasks:**
1. Search tetto-portal for: `bcrypt`, `crypto.createCipheriv`, `hash`, `encrypt`
2. Check `/tetto-portal/lib/crypto/*` or `/utils/crypto/*`
3. Identify how current API keys are hashed (bcrypt? sha256?)
4. Document existing env var patterns (`.env.example`)
5. Check if `ENCRYPTION_KEY` or similar already exists

**Deliverable:** Encryption patterns report + recommendation (align with existing or propose new)

---

### GAP 3: Stripe Integration Details 🚨

**What we DON'T know:**
- Is Stripe already integrated in tetto-portal?
- How are customers created? (signup flow)
- What Stripe mode? (test vs. production)
- Existing webhook handlers?
- Payment method collection pattern?

**Why critical:**
- Cannot design billing flow without knowing existing integration
- Risk of duplicate Stripe customers
- Risk of conflicting webhook handlers

**Research tasks:**
1. Search tetto-portal for: `stripe`, `@stripe/stripe-js`, `STRIPE_SECRET_KEY`
2. Check `/tetto-portal/app/api/webhooks/stripe/route.ts` (or equivalent)
3. Identify customer creation flow (if exists)
4. Document existing Stripe products/prices (if any)
5. Check `.env.example` for Stripe keys

**Deliverable:** Stripe integration report (existing setup + TDK additions needed)

---

### GAP 4: Deployment Architecture 🚨

**What we DON'T know:**
- Where should TDK API live? (tetto-portal repo? separate tdk-api repo?)
- What domain/subdomain? (api.tetto.io? tdk.tetto.io? tetto.io/api/v1/tdk?)
- Existing API route structure in tetto-portal?
- Vercel deployment? Railway? Self-hosted?
- CORS configuration needed?

**Why critical:**
- Cannot design routes without knowing deployment strategy
- Risk of CORS issues if wrong domain
- Risk of Next.js route conflicts

**Research tasks:**
1. Check tetto-portal deployment setup (Vercel config? Railway?)
2. Review existing API routes: `/tetto-portal/app/api/*`
3. Identify domain strategy (check DNS records or deployment URLs)
4. Document existing middleware pattern (auth, CORS, rate limiting)
5. Check if API versioning exists (`/api/v1/`, `/api/v2/`)

**Deliverable:** Deployment architecture recommendation (where to build TDK API)

---

### GAP 5: Agent Cost Calculation Validation 🚨

**What we DON'T know:**
- Where is agent `price_usdc` stored in database?
- How are agent costs currently calculated for receipts?
- Existing cost calculation functions?
- How to get agent cost before making call? (needed for wallet funding check)

**Why critical:**
- TDK needs to calculate cost BEFORE call (to check wallet balance)
- Need to apply TDK markup (50% default)
- Cannot hardcode agent costs (they change)

**Research tasks:**
1. Check tetto-portal database schema for `agents` table
2. Find `price_usdc` or `price_display` field
3. Search for existing cost calculation: `calculateCost`, `agentCost`, `pricing`
4. Identify how to query agent cost: `GET /api/agents/:id`
5. Document any existing markup/fee logic

**Deliverable:** Cost calculation pattern + validation that we can query agent costs

---

### GAP 6: Wallet Funding Strategy 🚨

**What we DON'T know:**
- Optimal initial funding amount (0.1 SOL + $50 USDC? More? Less?)
- Current Solana transaction costs on mainnet
- Top-up threshold (when to refund wallet?)
- Treasury wallet management pattern
- How to transfer USDC on Solana? (SPL token transfers)

**Why critical:**
- Under-fund = calls fail (bad UX)
- Over-fund = locked capital (expensive at scale)
- Wrong top-up logic = too many transactions (expensive)

**Research tasks:**
1. Research current Solana transaction costs (check recent tx on explorer)
2. Estimate: How many agent calls can 0.1 SOL cover?
3. Estimate: How many $0.001 calls = $50 USDC?
4. Validate USDC transfer code (SPL token transfer pattern)
5. Research: Should we pre-fund or fund on-demand?
6. Check existing tetto-portal for treasury wallet pattern

**Deliverable:** Funding strategy recommendation with calculations

---

### GAP 7: Rate Limiting Implementation 🚨

**What we DON'T know:**
- Existing rate limiting in tetto-portal?
- Redis available? (or in-memory?)
- Rate limit storage pattern (database? cache?)
- How to handle distributed rate limiting (multiple instances?)
- Different tiers = different limits (how to configure?)

**Why critical:**
- Free tier needs rate limiting (prevent abuse)
- Cannot build robust rate limiting without understanding infrastructure
- Risk of bypassing limits if implemented wrong

**Research tasks:**
1. Search tetto-portal for: `ratelimit`, `rate-limit`, `upstash`, `redis`
2. Check existing middleware: `/middleware.ts` or `/lib/ratelimit.ts`
3. Identify available infrastructure (Redis? Upstash? Database?)
4. Document existing rate limit patterns (per user? per IP? per key?)
5. Check if `@upstash/ratelimit` or similar already in package.json

**Deliverable:** Rate limiting implementation pattern (align with existing)

---

### GAP 8: Authentication Flow Validation 🚨

**What we DON'T know:**
- How do users currently sign up? (email/password? wallet? OAuth?)
- Supabase Auth configuration?
- Magic links? Email verification required?
- Password requirements?
- How are sessions managed? (JWT? cookies?)

**Why critical:**
- TDK signup flow must align with existing auth
- Cannot create duplicate user records
- Risk of auth conflicts

**Research tasks:**
1. Check tetto-portal auth flow: `/app/auth/*` or `/app/login/*`
2. Identify Supabase Auth setup (check `/lib/supabase.ts`)
3. Document signup requirements (email verification? password strength?)
4. Check existing user creation flow (any Stripe customer creation?)
5. Validate: Can we extend existing auth or need separate TDK auth?

**Deliverable:** Auth flow documentation + recommendation (extend vs. separate)

---

## Architecture Decisions Needing Validation

### DECISION 1: Wallet-Per-Customer vs. Wallet Pool

**AI1's new recommendation:** Wallet-per-customer (1:1 mapping)

**Must validate:**
- [ ] Confirm agents use `caller_wallet` as primary identifier (check WarmMemory agent code if accessible)
- [ ] Estimate total cost: 10,000 customers × (0.1 SOL + $50 USDC) = ?
- [ ] Validate: Is this capital efficient? (funds are customer's money, just locked)
- [ ] Research: Can we reduce initial funding? (0.01 SOL + $10 USDC for testing?)
- [ ] Alternative: Lazy wallet creation (create on first call, not at signup)

**If validation fails:** Fallback to synthetic namespace approach (document why it failed)

### DECISION 2: Subdomain Strategy

**AI1's recommendation:** `api.tetto.io/v1/...` (separate from marketplace)

**Must validate:**
- [ ] Check existing domain setup (DNS records)
- [ ] Validate: Is api.tetto.io available?
- [ ] Alternative: `tetto.io/api/tdk/v1/...` (same domain, different path)
- [ ] Check CORS implications (if different domain)
- [ ] Validate SSL certificate setup

### DECISION 3: Same Repo vs. Separate Repo

**Options:**
- A) Build TDK in tetto-portal repo (`/app/api/tdk/v1/...`)
- B) Separate tdk-api repo (api.tetto.io)

**Must validate:**
- [ ] Check tetto-portal size/complexity (is it bloated?)
- [ ] Validate: Can we deploy subdomain from same repo? (Vercel path routing)
- [ ] Consider: Separate repo = separate deployment = more overhead
- [ ] Recommendation: Same repo unless portal is too complex

### DECISION 4: Treasury Wallet Management

**Options:**
- A) Single treasury wallet (simple, single point of failure)
- B) Multi-sig treasury (secure, complex)
- C) Per-environment treasury (dev, staging, prod)

**Must validate:**
- [ ] Check existing tetto-portal wallet management
- [ ] Research: How is protocol wallet currently managed?
- [ ] Validate: What's the funding flow? (bank → USDC → treasury → customer wallets)
- [ ] Security: How to protect treasury wallet secret?

---

## Expected Deliverables from AI2

After completing research, you will create:

### 1. **TDK_BACKEND_FOUNDATION_START_HERE.md** (2,000-3,000 lines)

**Structure:**
```markdown
# Effort 1: TDK Backend Foundation

## Overview
- What this effort builds
- Success criteria
- Time estimate: 30-40 hours

## Architecture Decisions
- [DECISION] Wallet-per-customer (with validation)
- [DECISION] Subdomain: api.tetto.io
- [DECISION] Deploy in tetto-portal repo
- [DECISION] Treasury wallet: single wallet with rotation

## Database Schema
- Full SQL with indexes, constraints, RLS
- Migration strategy (how to deploy safely)

## Checkpoints (8-10 CPs)

### CP1: Database Schema & Migrations
**Files:**
- /supabase/migrations/20250101_tdk_foundation.sql
**Validation:**
- Run migration on staging
- Verify RLS policies
- Test foreign keys

### CP2: Wallet Encryption Module
**Files:**
- /lib/crypto/wallet-encryption.ts (200 lines)
- /lib/crypto/wallet-encryption.test.ts (150 lines)
**Validation:**
- All tests pass
- Encryption roundtrip works
- Tampering detected

[... 8-10 total checkpoints ...]

## Testing Strategy
- Unit tests (Jest)
- Integration tests (Supabase client)
- E2E test: Create key → Fund wallet → Call agent

## Security Checklist
- [ ] Wallet secrets encrypted at rest
- [ ] Master key in env var (not code)
- [ ] API keys hashed (bcrypt)
- [ ] RLS policies tested
- [ ] Rate limiting enabled

## Deployment Steps
1. Set WALLET_ENCRYPTION_KEY env var
2. Run database migration
3. Generate treasury wallet
4. Fund treasury wallet
5. Deploy API routes
6. Test with curl
```

### 2. **RESEARCH_FINDINGS_EFFORT1.md** (1,000-1,500 lines)

**Document all your research:**
- Existing schema analysis
- Encryption pattern decision (with rationale)
- Stripe integration plan
- Cost calculations (wallet funding)
- Rate limiting approach
- All validation results

**Format:**
```markdown
# Research Findings: Effort 1

## Existing Schema Analysis
[Complete documentation of current database]

## Encryption Pattern Decision
**Existing pattern:** [What you found]
**Recommendation:** [What to use for TDK]
**Rationale:** [Why this is best]

[... for each research gap ...]
```

### 3. **IMPLEMENTATION_RISKS_EFFORT1.md** (500-800 lines)

**Document every risk you identify:**

```markdown
# Implementation Risks: Effort 1

## Risk 1: Wallet Funding Insufficient
**Probability:** Medium
**Impact:** High (calls fail)
**Mitigation:** Monitor wallet balances, alert at 20% threshold

## Risk 2: Database Migration Conflicts
**Probability:** Low
**Impact:** High (production downtime)
**Mitigation:** Test on staging first, have rollback plan

[... all risks ...]
```

---

## AI2 Research Protocol

### Phase 1: Repository Access (2 hours)
1. Clone tetto-portal: `git clone https://github.com/TettoLabs/tetto-portal`
2. Install dependencies: `npm install`
3. Read README.md completely
4. Understand deployment setup (Vercel? Railway?)
5. Map out existing API routes

### Phase 2: Database Deep Dive (3 hours)
1. Find all migrations: `/supabase/migrations/*.sql`
2. Document every table (structure, indexes, RLS)
3. Identify auth pattern (Supabase Auth configuration)
4. Map foreign key relationships
5. Validate: Can we extend without conflicts?

### Phase 3: Security Patterns (2 hours)
1. Search for encryption code (all repos)
2. Identify hashing patterns (API keys, passwords)
3. Document env var conventions
4. Check for existing `ENCRYPTION_KEY` or similar
5. Validate: Follow existing or propose new?

### Phase 4: Stripe Integration (2 hours)
1. Search for Stripe code
2. Map customer creation flow
3. Identify webhook handlers
4. Check for existing metered billing
5. Plan: Extend existing or add new?

### Phase 5: Cost Calculations (2 hours)
1. Query agent costs (all agents)
2. Calculate: 0.1 SOL = how many calls?
3. Calculate: $50 USDC = how many calls?
4. Research: SOL transaction costs
5. Optimize: Reduce funding if possible

### Phase 6: Architecture Validation (3 hours)
1. Validate wallet-per-customer approach
2. Test: Create 10 wallets, fund them, make calls
3. Measure: Cost per wallet creation
4. Verify: Data isolation works
5. Decision: Confirm or adjust architecture

### Phase 7: Rate Limiting Research (1 hour)
1. Search for existing rate limiting
2. Identify available infrastructure (Redis?)
3. Document patterns
4. Plan TDK rate limiting

### Phase 8: Create Deliverables (10 hours)
1. Write START_HERE (3,000 lines)
2. Write RESEARCH_FINDINGS (1,500 lines)
3. Write IMPLEMENTATION_RISKS (800 lines)
4. Review everything (validation)
5. Hand off to AI3

**Total Time:** ~25 hours of deep research

---

## Success Criteria for AI2

Your research is complete when:

✅ All 8 research gaps filled with concrete findings
✅ All 4 architecture decisions validated (or adjusted with rationale)
✅ START_HERE document is detailed enough for AI3 to implement without questions
✅ Every code file path specified (AI3 knows exactly what to create)
✅ Every validation step documented (AI3 knows how to test)
✅ All risks identified with mitigations
✅ Time estimates realistic (based on similar work in codebase)
✅ No assumptions left unvalidated

---

## Critical Reminder

**DO NOT SKIP VALIDATION!**

AI1 made reasonable assumptions based on SDK research and docs. But assumptions ≠ facts.

You have access to:
- Full tetto-portal codebase
- Full tetto-sdk codebase
- Production database schema
- Deployed agents (can call them to test)
- All env vars (check .env.example)

**Use this access to VALIDATE everything before creating guides.**

If you find that wallet-per-customer won't work → DOCUMENT WHY and propose alternative.
If you find better encryption pattern → DOCUMENT and recommend.
If you find deployment blocker → DOCUMENT and provide workaround.

Your job is to prevent AI3 from hitting surprises during implementation.

**Good luck! 🚀**
