# Effort 1: TDK Backend Foundation

**Status:** 🔬 Research Phase - Awaiting AI2
**Created:** 2025-11-06
**Estimated Implementation Time:** 35-45 hours (after research)

---

## 📁 Folder Structure

```
EFFORT_1_TDK_BACKEND_FOUNDATION/
├── START_HERE.md              ← READ THIS FIRST (AI2's mission brief)
├── README.md                  ← You are here
├── research/                  ← AI2 fills this (research findings)
│   ├── existing_schema.md
│   ├── encryption_patterns.md
│   ├── stripe_integration.md
│   ├── deployment_architecture.md
│   ├── agent_costs.md
│   ├── wallet_funding.md
│   ├── rate_limiting.md
│   ├── authentication.md
│   └── risks.md
├── guides/                    ← AI2 creates these (implementation guides)
│   ├── 1_database_schema.md
│   ├── 2_wallet_encryption.md
│   ├── 3_treasury_wallet.md
│   ├── 4_api_key_generation.md
│   ├── 5_core_call_endpoint.md
│   ├── 6_usage_tracking_billing.md
│   ├── 7_rate_limiting.md
│   └── 8_authentication.md
└── checkpoints/               ← AI2 outlines these (for AI3 implementation)
    ├── CP1_database.md
    ├── CP2_encryption.md
    ├── CP3_treasury.md
    ├── CP4_api_keys.md
    ├── CP5_auth.md
    ├── CP6_call_endpoint.md
    ├── CP7_billing.md
    ├── CP8_rate_limiting.md
    ├── CP9_testing.md
    └── CP10_monitoring.md
```

---

## 🎯 What This Effort Builds

**TDK Backend Foundation** - Core infrastructure for API key-based access to Tetto agents.

### Key Features
- ✅ Email/password authentication (separate from marketplace)
- ✅ API key generation with **dedicated Solana wallet per customer**
- ✅ Wallet encryption (AES-256-GCM)
- ✅ Treasury wallet management
- ✅ Core API: `POST /api/tdk/v1/call`
- ✅ Usage tracking & Stripe metered billing
- ✅ Rate limiting (tier-based)

### Critical Architecture: Wallet-Per-Customer

Each TDK customer gets a dedicated Solana wallet (1:1 mapping with API key).

**Why?** Agents use `caller_wallet` as the primary identifier. Shared wallet pool would cause data mixing.

**Read full rationale:** `../MULTI_AI_DEV_EFFORTS/WALLET_PER_CUSTOMER_DECISION.md`

---

## 📋 Current Status

**Phase:** Research (AI2 not started yet)

**What AI1 completed:**
- ✅ Deep research on SDK patterns
- ✅ Validated wallet-per-customer architecture
- ✅ Identified 8 critical research gaps
- ✅ Created comprehensive START_HERE for AI2

**What AI2 must do:**
1. Fill 8 research gaps (20-25 hours)
2. Create 8 implementation guides (15-20 hours)
3. Outline 10 checkpoints for AI3 (5 hours)
4. Document all risks (included in research)

**What AI3 will do:**
- Implement 10 checkpoints
- Deploy to production
- Validate each step

---

## 🚀 How to Use This Folder

### If you're AI2 (Guide Creator):
1. **Start here:** Read `START_HERE.md` completely
2. **Clone tetto-portal:** Get access to existing codebase
3. **Fill research gaps:** Create files in `/research` folder
4. **Create guides:** Detailed implementation guides in `/guides` folder
5. **Outline checkpoints:** For AI3 in `/checkpoints` folder
6. **Hand off:** Create `IMPLEMENTATION_READY.md` summary

### If you're AI3 (Implementer):
1. **Wait for AI2:** Don't start until research complete
2. **Read guides:** All 8 guides in `/guides` folder
3. **Follow checkpoints:** Implement in order (CP1 → CP10)
4. **Validate each CP:** Before moving to next
5. **Deploy:** Production deployment

### If you're reviewing:
- **START_HERE.md** - AI2's mission brief (comprehensive)
- **Research files** - AI2's findings (once complete)
- **Guides** - Implementation instructions (once complete)
- **Checkpoints** - Step-by-step implementation plan (once complete)

---

## 🔗 Related Documents

**In this repo:**
- `../MULTI_AI_DEV_EFFORTS/FOUNDATIONAL_EFFORTS_OVERVIEW.md` - All 3 efforts
- `../MULTI_AI_DEV_EFFORTS/WALLET_PER_CUSTOMER_DECISION.md` - Architecture rationale
- `../MULTI_AI_DEV_EFFORTS/AI2_RESEARCH_BRIEF_EFFORT1.md` - Original research brief
- `../research/TDK_CORE_START_HERE.md` - AI1's initial research

**External repos:**
- `/home/user/tetto-sdk` - SDK codebase (reference)
- `/home/user/warmmemory-plugin` - Plugin example (reference)
- `tetto-portal` - Must be cloned by AI2 (contains existing schema)

---

## ⏱️ Time Estimates

**AI2 (Research & Guide Creation):** 40-50 hours
- Research: 20-25 hours
- Guides: 15-20 hours
- Checkpoints: 5 hours

**AI3 (Implementation):** 35-45 hours
- CP1-CP10: 3-5 hours each
- Testing: 5 hours
- Deployment: 3 hours

**Total Effort:** 80-95 hours from start to production

---

## 📞 Questions?

**User:** Review documents and provide feedback
**AI2:** Start with `START_HERE.md`, ask questions if blocked
**AI3:** Wait for AI2 to complete, then begin implementation

---

**Let's build the TDK Backend Foundation! 🚀**
