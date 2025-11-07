# Effort 2: TDK Client Library

**Status:** 🔬 Research Phase - Awaiting AI2
**Created:** 2025-11-06
**Estimated Implementation Time:** 25-30 hours (after research)

---

## 📁 Folder Structure

```
EFFORT_2_TDK_CLIENT_LIBRARY/
├── START_HERE.md              ← READ THIS FIRST (AI2's mission brief)
├── README.md                  ← You are here
├── HIGH_LEVEL_OVERVIEW.md     ← Quick summary of what gets built
├── research/                  ← AI2 fills this (research findings)
│   ├── npm_package_setup.md
│   ├── api_contract.md
│   ├── typescript_patterns.md
│   ├── error_handling.md
│   ├── testing_strategy.md
│   ├── documentation_strategy.md
│   ├── performance_bundle_size.md
│   ├── publishing_release.md
│   └── risks.md
├── guides/                    ← AI2 creates these (implementation guides)
│   ├── 1_project_setup.md
│   ├── 2_core_class.md
│   ├── 3_http_client.md
│   ├── 4_memory_shortcut.md
│   ├── 5_answers_shortcut.md
│   ├── 6_type_definitions.md
│   ├── 7_testing_suite.md
│   ├── 8_documentation_examples.md
│   └── 9_npm_publishing.md
└── checkpoints/               ← AI2 outlines these (for AI3 implementation)
    ├── CP1_scaffolding.md
    ├── CP2_core_class.md
    ├── CP3_http_client.md
    ├── CP4_memory_shortcut.md
    ├── CP5_answers_shortcut.md
    ├── CP6_error_handling.md
    ├── CP7_types.md
    ├── CP8_testing.md
    ├── CP9_documentation.md
    └── CP10_npm_publish.md
```

---

## 🎯 What This Effort Builds

**TDK Client Library (@tetto/tdk)** - TypeScript/JavaScript library for calling Tetto agents.

### Key Features
- ✅ NPM package: `@tetto/tdk`
- ✅ Hybrid approach: Shortcuts + generic `callAgent()`
- ✅ Works in Node.js AND browser
- ✅ Full TypeScript support
- ✅ Automatic retries with exponential backoff
- ✅ Zero dependencies (minimal bundle size)

### Example Usage
```typescript
import { TettoDK } from '@tetto/tdk';

const tdk = new TettoDK({ apiKey: process.env.TETTO_API_KEY });

// Built-in shortcuts
await tdk.memory.set('key', 'value');
await tdk.answers.ask('What is the capital of France?');

// Generic method
await tdk.callAgent('sentiment-analyzer', { text: 'Amazing!' });
```

### Critical Decision: Hybrid Approach

**Built-in shortcuts** for popular agents (memory, answers)
**Generic callAgent()** for everything else

**Why?** Discoverable + Extensible + Clean

**Read full rationale:** `START_HERE.md` (Architecture Decision section)

---

## 📋 Current Status

**Phase:** Research (AI2 not started yet)

**What AI1 completed:**
- ✅ Researched SDK patterns (Stripe, OpenAI, Anthropic)
- ✅ Validated hybrid approach (shortcuts + generic)
- ✅ Identified 8 critical research gaps
- ✅ Created comprehensive START_HERE for AI2

**What AI2 must do:**
1. Fill 8 research gaps (10-12 hours)
2. Create 9 implementation guides (12-15 hours)
3. Outline 10 checkpoints for AI3 (3 hours)
4. Document all risks

**What AI3 will do:**
- Build NPM package
- Implement checkpoints
- Publish to NPM
- Validate with real usage

---

## 🚀 How to Use This Folder

### If you're AI2 (Guide Creator):
1. **Start here:** Read `START_HERE.md` completely
2. **Study successful SDKs:** Stripe, OpenAI, Anthropic
3. **Coordinate with Effort 1:** Get API contract
4. **Fill research gaps:** Create files in `/research` folder
5. **Create guides:** Detailed implementation guides in `/guides` folder
6. **Outline checkpoints:** For AI3 in `/checkpoints` folder

### If you're AI3 (Implementer):
1. **Wait for AI2:** Don't start until research complete
2. **Read guides:** All 9 guides in `/guides` folder
3. **Follow checkpoints:** Implement in order (CP1 → CP10)
4. **Validate each CP:** Before moving to next
5. **Publish:** Beta release to NPM

### If you're reviewing:
- **START_HERE.md** - AI2's mission brief (comprehensive)
- **HIGH_LEVEL_OVERVIEW.md** - Quick summary
- **Research files** - AI2's findings (once complete)
- **Guides** - Implementation instructions (once complete)

---

## 🔗 Related Documents

**In this repo:**
- `../MULTI_AI_DEV_EFFORTS/FOUNDATIONAL_EFFORTS_OVERVIEW.md` - All 3 efforts
- `../EFFORT_1_TDK_BACKEND_FOUNDATION/` - Backend foundation (dependency)

**External repos:**
- `/home/user/tetto-sdk` - Existing SDK (reference only, DON'T copy)
- Stripe SDK - Study for patterns
- OpenAI SDK - Study for simplicity

---

## ⏱️ Time Estimates

**AI2 (Research & Guide Creation):** 25-30 hours
- Research: 10-12 hours
- Guides: 12-15 hours
- Checkpoints: 3 hours

**AI3 (Implementation):** 25-30 hours
- CP1-CP10: 2-3 hours each
- Testing: 5 hours
- Publishing: 2 hours

**Total Effort:** 50-60 hours from start to NPM

---

## 🔗 Dependencies

**Blocked by:**
- Effort 1 API contract (need exact request/response formats)

**Can parallelize:**
- Build scaffolding while Effort 1 defines contract
- Study SDK patterns independently
- Design types without backend running

**Blocks:**
- Effort 3 (plugins need TettoDK class with plugin interface)

---

## 📞 Questions?

**User:** Review documents and provide feedback
**AI2:** Start with `START_HERE.md`, coordinate with Effort 1 team
**AI3:** Wait for AI2 to complete, then begin implementation

---

**Let's build the TDK Client Library! 🚀**
