# Effort 1: TDK Backend Foundation - High Level Overview

**Status:** 🔬 Research Phase (AI2 not started)
**Implementation Time:** 35-45 hours
**Complexity:** High (database, encryption, billing integration)

---

## 🎯 What It Does (One Sentence)

Builds the complete backend infrastructure that lets developers call Tetto agents using simple API keys instead of managing Solana wallets and crypto.

---

## 🏗️ What Gets Built

### **Core System**
A REST API that abstracts all crypto complexity from developers.

**Before TDK:**
```typescript
// Developer needs to:
1. Create Solana wallet
2. Fund wallet with SOL + USDC
3. Import tetto-sdk
4. Manage wallet keypairs
5. Sign transactions manually
6. Handle crypto errors

const wallet = createWalletFromKeypair(keypair);  // Complex!
await tetto.callAgent('warmmemory', input, wallet);
```

**After TDK (Effort 1 complete):**
```typescript
// Developer only needs:
const apiKey = 'tk_abc123...';  // Simple!

fetch('https://api.tetto.io/v1/call', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${apiKey}` },
  body: JSON.stringify({
    agent_id: 'warmmemory',
    input: { action: 'store', key: 'test', value: 'data' }
  })
});
```

---

## 🔑 Key Components

### 1. **Authentication System**
- Email/password signup (no web3 wallet needed)
- Separate from marketplace (marketplace = web3, TDK = traditional auth)
- Session management with JWT

**Endpoints:**
- `POST /api/tdk/auth/register` - Create account
- `POST /api/tdk/auth/login` - Get session token

### 2. **API Key Management**
- Generate API keys for developer apps
- Each API key gets a **dedicated Solana wallet** (auto-created)
- Wallet funded automatically from treasury

**Endpoint:**
- `POST /api/tdk/keys/generate` → Returns API key + wallet info

### 3. **Dedicated Wallet System** ⭐ CRITICAL
Every API key has its own Solana wallet (1:1 mapping).

**Why?** Agents identify callers by wallet address. Shared wallets = data mixing!

**How it works:**
```
Developer signs up
    ↓
Generates API key (tk_abc123)
    ↓
Backend creates Solana wallet (Sol1a2b3c...)
    ↓
Encrypts private key (AES-256-GCM)
    ↓
Funds wallet from treasury (0.01 SOL + $10 USDC)
    ↓
Stores in database (dedicated_wallet_public_key, dedicated_wallet_secret_encrypted)
    ↓
Developer makes API calls
    ↓
Backend uses dedicated wallet to pay agents
    ↓
Agents see unique caller_wallet → Perfect data isolation!
```

### 4. **Core API Endpoint**
The main endpoint that calls any agent.

**Endpoint:**
```
POST /api/tdk/v1/call
Authorization: Bearer tk_abc123...

Body:
{
  "agent_id": "warmmemory",
  "input": {
    "action": "store",
    "key": "user:prefs",
    "value": "{\"theme\": \"dark\"}"
  }
}
```

**What happens internally:**
1. Validate API key (check database)
2. Get dedicated wallet for this key
3. Decrypt wallet private key
4. Check wallet balance (USDC for agent cost, SOL for gas)
5. Top up if low (from treasury wallet)
6. Call agent using dedicated wallet
7. Track cost for billing
8. Return agent response

### 5. **Treasury Wallet**
A master wallet that funds all customer wallets.

**Setup:**
- Single Solana wallet with large balance (e.g., 100 SOL + $10,000 USDC)
- Private key encrypted (AES-256-GCM)
- Monitored for low balance (alerts at 20%)

**Function:**
- Automatically refills customer wallets when low
- Transfers: Treasury → Customer Wallet → Agent

### 6. **Wallet Encryption**
All private keys encrypted at rest.

**Algorithm:** AES-256-GCM (authenticated encryption)
**Master Key:** Stored in env var `WALLET_ENCRYPTION_KEY`
**Security:** Auth tag prevents tampering

### 7. **Usage Tracking & Billing**
Tracks every agent call for billing.

**Database:**
```sql
tdk_usage:
  - agent_id: 'warmmemory'
  - agent_cost_usd: 0.001
  - tdk_markup_pct: 0.5 (50% markup)
  - developer_cost_usd: 0.0015
  - timestamp: now()
```

**Stripe Integration:**
- Metered billing (pay-as-you-go)
- Variable event values (different agents cost different amounts)
- Monthly invoices
- Webhook handlers for payment events

### 8. **Rate Limiting**
Prevents abuse on free tier.

**Tiers:**
- Free: 100 requests/minute
- Paid: 1,000 requests/minute
- Pro: 10,000 requests/minute

**Implementation:** Redis or Upstash for distributed rate limiting

---

## 📊 Database Schema (4 Tables)

### `tdk_api_keys`
Stores API keys with dedicated wallets.
```sql
- id (uuid)
- user_id (references auth.users)
- key_hash (bcrypt hash of API key)
- dedicated_wallet_public_key (Solana address)
- dedicated_wallet_secret_encrypted (AES-256-GCM)
- wallet_balance_sol (cached)
- wallet_balance_usdc (cached)
- tier ('free', 'paid', 'pro')
- stripe_customer_id
- ops_this_month (usage counter)
- rate_limit_per_minute
```

### `tdk_treasury`
Treasury wallet that funds customers.
```sql
- id (serial)
- public_key (treasury Solana address)
- secret_encrypted (AES-256-GCM)
- balance_sol (cached)
- balance_usdc (cached)
- total_funded_out (tracking)
```

### `tdk_usage`
Tracks every agent call for billing.
```sql
- id (uuid)
- api_key_id (references tdk_api_keys)
- agent_id ('warmmemory', 'sentiment', etc.)
- agent_cost_usd (what agent charged)
- tdk_markup_pct (our markup, e.g., 0.5 = 50%)
- developer_cost_usd (what we bill)
- stripe_reported (boolean)
- timestamp
```

### `tdk_billing_periods`
Monthly billing records.
```sql
- id (uuid)
- user_id (references auth.users)
- api_key_id (references tdk_api_keys)
- period_start
- period_end
- total_operations (count)
- total_cost_usd (sum)
- stripe_invoice_id
- status ('pending', 'paid', 'failed')
```

---

## 🔄 End-to-End Flow

**Developer Journey:**

```
1. SIGNUP
   POST /api/tdk/auth/register
   { email: "dev@example.com", password: "..." }
   ↓
   ✅ User account created in auth.users

2. GENERATE API KEY
   POST /api/tdk/keys/generate
   ↓
   Backend:
   - Generates Solana keypair (FREE)
   - Encrypts private key
   - Funds wallet from treasury (0.01 SOL + $10 USDC)
   - Saves to tdk_api_keys table
   ↓
   ✅ Returns API key: "tk_abc123..."

3. MAKE API CALL
   POST /api/tdk/v1/call
   Authorization: Bearer tk_abc123...
   { agent_id: "warmmemory", input: {...} }
   ↓
   Backend:
   - Validates API key
   - Gets dedicated wallet (Sol1a2b3c...)
   - Checks balance ($9.50 USDC left)
   - Calls agent using dedicated wallet
   - Tracks cost ($0.001 to tdk_usage table)
   ↓
   ✅ Returns agent response

4. BILLING (End of Month)
   Stripe webhook triggered
   ↓
   Backend:
   - Queries tdk_usage for this API key
   - Sums total costs ($15.43 this month)
   - Reports to Stripe as usage record
   - Stripe charges customer's credit card
   ↓
   ✅ Developer billed automatically
```

---

## 🎯 Success Metrics

**After Effort 1 is complete:**

✅ Developer can sign up in 30 seconds (email + password)
✅ Developer can generate API key in 5 seconds
✅ Developer can call any agent with 3 lines of code
✅ Zero crypto knowledge required
✅ Automatic billing (no manual invoices)
✅ Data isolated per developer (unique wallet = unique identity)
✅ Scalable to 10,000+ developers

---

## 🚧 What It DOESN'T Include

**Not in Effort 1:**
- ❌ TDK Client Library (that's Effort 2)
- ❌ Plugin system (that's Effort 3)
- ❌ Dashboard UI (future effort)
- ❌ Webhooks for developers (future effort)
- ❌ Custom rate limits (future enhancement)
- ❌ Team accounts (future enhancement)

**Effort 1 = Backend only.** Developers use raw HTTP requests.

---

## 📋 Implementation Checkpoints (10 CPs)

AI3 will implement in this order:

1. **CP1: Database Schema** - Create tables, migrations
2. **CP2: Wallet Encryption** - AES-256-GCM module
3. **CP3: Treasury Wallet** - Setup & fund master wallet
4. **CP4: API Key Generation** - Endpoint + wallet creation
5. **CP5: Authentication** - Register/login endpoints
6. **CP6: Core Call Endpoint** - Main `/v1/call` logic
7. **CP7: Usage Tracking** - Track costs, Stripe integration
8. **CP8: Rate Limiting** - Tier-based limits
9. **CP9: Testing** - End-to-end validation
10. **CP10: Monitoring** - Alerts, logging, observability

**Estimated time:** 3-5 hours per checkpoint = 35-45 hours total

---

## ⚠️ Top 5 Risks

1. **Wallet funding insufficient** → Calls fail (bad UX)
   - Mitigation: Conservative initial funding, aggressive top-up

2. **Encryption key leak** → All wallets compromised
   - Mitigation: Master key in env var only, rotation procedure

3. **Treasury wallet depleted** → Cannot fund customers
   - Mitigation: Monitor balance, alert at 20%, auto-refill

4. **Stripe billing errors** → Wrong charges
   - Mitigation: Thorough testing on Stripe test mode first

5. **Database migration fails** → Production downtime
   - Mitigation: Test on staging, have rollback plan ready

---

## 🔗 Dependencies

**Before starting Effort 1:**
- ✅ Access to tetto-portal repo (contains existing database)
- ✅ Supabase access (database & auth)
- ✅ Stripe account (test + production)
- ✅ Solana RPC access (for wallet operations)
- ✅ Treasury wallet funded (10 SOL + $10k USDC)

**After Effort 1:**
- ✅ Can start Effort 2 (TDK Client Library)
- ✅ Can start Effort 3 (Plugin System)

---

## 💰 Cost Breakdown

**Initial setup:**
- Treasury wallet: $10,000 USDC + 10 SOL (~$2,000) = $12,000 locked

**Per developer:**
- Wallet creation: FREE
- Initial funding: $10 USDC + 0.01 SOL (~$2) = ~$12

**At scale (1,000 developers):**
- Locked capital: $12,000 (treasury) + $12,000 (customers) = $24,000
- Note: Customer funds are THEIR money (we bill them monthly)
- ROI: 50% markup on agent calls = revenue stream

---

## 📊 Before vs. After

### Before Effort 1
```
Developer wants to use WarmMemory agent:
1. Learn about Solana wallets ❌ (hard)
2. Install crypto libraries ❌ (complex)
3. Create wallet, save secret ❌ (scary)
4. Buy SOL from exchange ❌ (friction)
5. Swap SOL for USDC ❌ (confusing)
6. Import tetto-sdk ❌ (more code)
7. Sign transactions manually ❌ (error-prone)
8. Handle blockchain errors ❌ (unpredictable)

Time to first call: 2-3 hours (or give up)
```

### After Effort 1
```
Developer wants to use WarmMemory agent:
1. Sign up at api.tetto.io ✅ (30 seconds)
2. Generate API key ✅ (5 seconds)
3. Make HTTP POST request ✅ (3 lines of code)

Time to first call: 5 minutes
```

---

## 🎯 Why This Matters

**Problem:** Tetto agents are powerful, but crypto barriers prevent adoption.

**Solution:** Effort 1 removes ALL crypto complexity.

**Impact:**
- 10x more developers can use Tetto agents
- Revenue model (50% markup = sustainable business)
- Data isolation (each developer gets unique wallet)
- Scalable (no architectural limits)

**This is the foundation for everything else. Get this right, and Efforts 2-3 are easy.** 🚀

---

## 📖 Where to Learn More

- **Detailed mission:** `START_HERE.md` (938 lines for AI2)
- **Architecture rationale:** `../MULTI_AI_DEV_EFFORTS/WALLET_PER_CUSTOMER_DECISION.md`
- **All 3 efforts:** `../MULTI_AI_DEV_EFFORTS/FOUNDATIONAL_EFFORTS_OVERVIEW.md`
- **Original research:** `../research/TDK_CORE_START_HERE.md`

---

**Effort 1 is the hardest (database, crypto, billing). Efforts 2-3 are easier (client libraries, plugins).**

**Once Effort 1 is done, TDK is REAL.** ✨
