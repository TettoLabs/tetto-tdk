# Effort 2: TDK Client Library - High Level Overview

**Status:** 🔬 Research Phase (AI2 not started)
**Implementation Time:** 25-30 hours
**Complexity:** Medium (TypeScript library, no backend complexity)

---

## 🎯 What It Does (One Sentence)

Builds a TypeScript/JavaScript library that developers install from NPM to call Tetto agents with simple, elegant code.

---

## 🏗️ What Gets Built

### **NPM Package: @tetto/tdk**

A zero-dependency client library that wraps the TDK API (from Effort 1).

**Before TDK Client Library:**
```typescript
// Developers need to:
import fetch from 'node-fetch';

const response = await fetch('https://api.tetto.io/v1/call', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer tk_abc123...',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    agent_id: 'warmmemory',
    input: { action: 'store', key: 'test', value: 'data' }
  })
});

const result = await response.json();
if (!result.ok) {
  throw new Error(result.error);
}
// Manual error handling, no retries, no types... 😰
```

**After TDK Client Library:**
```typescript
// Install: npm install @tetto/tdk
import { TettoDK } from '@tetto/tdk';

const tdk = new TettoDK({ apiKey: process.env.TETTO_API_KEY });

// Elegant shortcuts
await tdk.memory.set('test', 'data');  // ✨

// Automatic retries, error handling, TypeScript types! 🎉
```

---

## 🔑 Key Features

### 1. **Hybrid API Design** ⭐ CORE DECISION

**Two ways to call agents:**

**Option 1: Built-in Shortcuts (Elegant)**
```typescript
// Memory operations
await tdk.memory.set('user:prefs', { theme: 'dark' });
const prefs = await tdk.memory.get('user:prefs');
const keys = await tdk.memory.list('user:');
await tdk.memory.delete('user:old');

// Q&A operations
await tdk.answers.teach('The capital of France is Paris');
const answer = await tdk.answers.ask('What is the capital of France?');
await tdk.answers.update('Actually, Paris is the capital');
await tdk.answers.forget('outdated fact');
```

**Option 2: Generic Method (Extensible)**
```typescript
// Call ANY agent
const sentiment = await tdk.callAgent('sentiment-analyzer', {
  text: 'I love this product!'
});

const summary = await tdk.callAgent('summarizer', {
  text: 'Long article...',
  max_length: 100
});

const custom = await tdk.callAgent('your-custom-agent-id', {
  custom: 'input'
});
```

**Why hybrid?**
- ✅ Shortcuts = Discoverable (IntelliSense autocomplete)
- ✅ Generic = Extensible (any agent, even new ones)
- ✅ Clean = Only popular agents get shortcuts
- ✅ Future-proof = Plugins can add more shortcuts (Effort 3)

### 2. **Full TypeScript Support**

**Type safety everywhere:**
```typescript
interface MemoryAPI {
  set(key: string, value: any): Promise<void>;
  get<T = any>(key: string): Promise<T | null>;
  list(prefix?: string): Promise<string[]>;
  delete(key: string): Promise<void>;
}

interface AnswersAPI {
  teach(fact: string): Promise<void>;
  ask(question: string): Promise<string>;
  update(oldFact: string, newFact: string): Promise<void>;
  forget(fact: string): Promise<void>;
}

// Generic method supports any type
const result = await tdk.callAgent<SentimentOutput>(
  'sentiment-analyzer',
  { text: 'Great!' }
);
console.log(result.output.score); // TypeScript knows this exists!
```

### 3. **Automatic Retries**

**Transient failures handled automatically:**
```typescript
// Developer writes:
await tdk.callAgent('agent-id', input);

// SDK does:
1. Try request
2. If 429 (rate limit) or 5xx (server error):
   → Wait 1 second
   → Retry (attempt 2)
3. If still fails:
   → Wait 2 seconds (exponential backoff)
   → Retry (attempt 3)
4. If still fails:
   → Throw error (after 3 attempts)

// Non-retryable errors (401, 400, 404):
→ Throw immediately (no retries)
```

**Algorithm:** Exponential backoff with jitter
- Attempt 1: Immediate
- Attempt 2: 1 second + random(0-500ms)
- Attempt 3: 2 seconds + random(0-1000ms)
- Max attempts: 3

### 4. **Rich Error Handling**

**Custom error classes:**
```typescript
class TettoDKError extends Error {
  code: string;
  statusCode?: number;
  details?: any;
}

class AuthenticationError extends TettoDKError {
  // 401 errors
}

class RateLimitError extends TettoDKError {
  // 429 errors
  retryAfter?: number;  // Seconds to wait
}

class InvalidInputError extends TettoDKError {
  // 400 errors
  validationErrors?: string[];
}

// Usage:
try {
  await tdk.callAgent('agent-id', input);
} catch (error) {
  if (error instanceof AuthenticationError) {
    console.error('Invalid API key!');
  } else if (error instanceof RateLimitError) {
    console.log(`Rate limited. Retry in ${error.retryAfter}s`);
  }
}
```

### 5. **Works Everywhere**

**Node.js:**
```typescript
// server.ts
import { TettoDK } from '@tetto/tdk';

const tdk = new TettoDK({ apiKey: process.env.TETTO_API_KEY });
await tdk.memory.set('key', 'value');
```

**Browser:**
```typescript
// React component
import { TettoDK } from '@tetto/tdk';

function MyComponent() {
  const tdk = new TettoDK({ apiKey: 'tk_...' });

  const handleSubmit = async () => {
    const answer = await tdk.answers.ask('What is AI?');
    console.log(answer);
  };

  return <button onClick={handleSubmit}>Ask</button>;
}
```

**Edge Runtime (Vercel, Cloudflare Workers):**
```typescript
export default async function handler(request: Request) {
  const tdk = new TettoDK({ apiKey: process.env.TETTO_API_KEY });
  const result = await tdk.callAgent('agent-id', { ... });
  return new Response(JSON.stringify(result));
}
```

### 6. **Zero Dependencies**

**Minimal bundle size:**
- Uses native `fetch` API (built into Node 18+, all modern browsers)
- No axios, no node-fetch, no heavy dependencies
- Minified: < 10 KB
- Gzipped: < 3 KB

**Comparison:**
- AWS SDK: ~300 KB 😱
- OpenAI SDK: ~15 KB
- **@tetto/tdk: < 10 KB** ✨

### 7. **Usage Tracking Helper**

**Check your usage:**
```typescript
const usage = await tdk.getUsage();
console.log(`Operations: ${usage.operations}`);
console.log(`Cost: $${usage.cost_usd}`);
console.log(`Remaining: ${usage.remaining} ops`);
```

---

## 📦 Package Structure

**Published to NPM as `@tetto/tdk`:**

```
@tetto/tdk/
├── dist/
│   ├── index.js         # CommonJS (Node.js)
│   ├── index.mjs        # ESM (modern)
│   └── index.d.ts       # TypeScript types
├── examples/
│   ├── basic.ts         # Simple example
│   ├── node-script.ts   # Node.js CLI
│   └── react-app.tsx    # React component
├── src/
│   ├── index.ts         # Main entry
│   ├── client.ts        # HTTP client
│   ├── errors.ts        # Error classes
│   ├── memory.ts        # Memory shortcut
│   ├── answers.ts       # Answers shortcut
│   └── types.ts         # TypeScript types
├── package.json
├── README.md
└── LICENSE
```

**Installation:**
```bash
npm install @tetto/tdk
```

**Import (ESM):**
```typescript
import { TettoDK } from '@tetto/tdk';
```

**Import (CommonJS):**
```javascript
const { TettoDK } = require('@tetto/tdk');
```

---

## 🔄 Architecture

**Simple layered design:**

```
Developer Code
      ↓
┌─────────────────────────────────┐
│   TettoDK Class (index.ts)      │
│   - Constructor(config)          │
│   - callAgent()                  │
│   - getUsage()                   │
│   - memory: MemoryAPI            │
│   - answers: AnswersAPI          │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│   HTTP Client (client.ts)       │
│   - fetch wrapper                │
│   - Retry logic                  │
│   - Error parsing                │
│   - Auth header injection        │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│   Shortcuts (memory.ts, etc)    │
│   - map to callAgent()           │
│   - convenience methods          │
└──────────────┬──────────────────┘
               ↓
      TDK Backend API
  (from Effort 1)
```

**Data flow:**
```
Developer calls: tdk.memory.set('key', 'value')
     ↓
MemoryAPI: Maps to callAgent('warmmemory', { action: 'store', ... })
     ↓
HTTP Client: Adds auth, retries on failure
     ↓
POST https://api.tetto.io/v1/call
     ↓
TDK Backend: Uses dedicated wallet, calls WarmMemory agent
     ↓
Response flows back up the stack
     ↓
Developer receives: Promise<void> (resolved)
```

---

## 📋 Implementation Checkpoints (10 CPs)

AI3 will implement in this order:

1. **CP1: Project Scaffolding** (2h)
   - package.json, tsconfig.json, build config
   - Testing setup (Vitest)
   - Linting (ESLint, Prettier)

2. **CP2: Core TettoDK Class** (3h)
   - Constructor, config validation
   - Base HTTP client skeleton

3. **CP3: HTTP Client + Auth** (3h)
   - Fetch wrapper with retries
   - Error parsing and mapping
   - Auth header injection

4. **CP4: Memory Shortcut** (2h)
   - set(), get(), list(), delete()
   - Error handling
   - Tests

5. **CP5: Answers Shortcut** (2h)
   - teach(), ask(), update(), forget()
   - Error handling
   - Tests

6. **CP6: Error Handling** (2h)
   - Custom error classes
   - Error mapping from HTTP codes
   - Retry logic refinement

7. **CP7: TypeScript Types** (2h)
   - Complete type definitions
   - JSDoc comments
   - Export structure

8. **CP8: Testing Suite** (4h)
   - Unit tests (all methods)
   - Integration tests (mock server)
   - E2E tests (staging API)

9. **CP9: Documentation** (3h)
   - README.md
   - Examples (5-10 examples)
   - API reference

10. **CP10: NPM Publish** (2h)
    - Build for production
    - Publish beta to NPM
    - Verify installation

**Total:** 25-30 hours

---

## 📊 Before vs. After

### Before Effort 2
```typescript
// Developers need to manually:
- Make HTTP requests with fetch ❌
- Handle authentication headers ❌
- Parse responses and errors ❌
- Implement retry logic ❌
- Remember agent input formats ❌
- Write TypeScript types themselves ❌

Code: ~50 lines per agent call
Time to integrate: 2-3 hours
Error-prone: High
```

### After Effort 2
```typescript
// Developers just:
npm install @tetto/tdk

import { TettoDK } from '@tetto/tdk';
const tdk = new TettoDK({ apiKey: '...' });
await tdk.memory.set('key', 'value');  // ✨

Code: 3 lines per agent call
Time to integrate: 5 minutes
Error-prone: Low (SDK handles everything)
```

---

## 🎯 Success Metrics

**After Effort 2 is complete:**

✅ Published to NPM as `@tetto/tdk`
✅ Installs in 5 seconds (`npm install @tetto/tdk`)
✅ Works in Node.js, browser, and edge runtimes
✅ Bundle size < 10 KB (minified)
✅ Full TypeScript support (types autocomplete)
✅ Automatic retries (transient failures handled)
✅ Clear error messages (developers know what went wrong)
✅ Comprehensive documentation (examples for common use cases)
✅ 90%+ test coverage

**Developer experience:**
```typescript
// From signup to first API call:
1. npm install @tetto/tdk
2. import { TettoDK } from '@tetto/tdk';
3. const tdk = new TettoDK({ apiKey: 'tk_...' });
4. await tdk.memory.set('key', 'value');

Total time: 2 minutes
```

---

## 🚧 What It DOESN'T Include

**Not in Effort 2:**
- ❌ Plugin system (that's Effort 3)
- ❌ More shortcuts beyond memory & answers (Effort 3 via plugins)
- ❌ CLI tool (future enhancement)
- ❌ React hooks (future enhancement)
- ❌ Caching layer (future enhancement)

**Effort 2 = Core library only.** Plugins and enhancements come later.

---

## ⚠️ Top 5 Risks

1. **API contract mismatch** → SDK doesn't work
   - Mitigation: Coordinate with Effort 1, integration tests

2. **Bundle size too large** → Slow downloads
   - Mitigation: Zero dependencies, tree-shaking, monitor size

3. **TypeScript types wrong** → IntelliSense breaks
   - Mitigation: Study successful SDKs, validate with real usage

4. **Browser compatibility issues** → Doesn't work in Safari
   - Mitigation: Test on all major browsers, polyfill if needed

5. **Breaking changes in agents** → Shortcuts break
   - Mitigation: Version shortcuts separately (via plugins in Effort 3)

---

## 💰 Cost to Use (Developers)

**Free!**
- Open source library (MIT License)
- No licensing fees
- No vendor lock-in

**Developers only pay:**
- TDK backend usage (from Effort 1)
- Agent costs + TDK markup
- Standard metered billing

**Example:**
```
10,000 calls/month to WarmMemory ($0.001 each):
- Agent cost: $10
- TDK markup (50%): $5
- Total: $15/month

Compare to building in-house:
- Engineering time: 40 hours × $150/hr = $6,000
- Maintenance: 10 hours/month × $150/hr = $1,500/month

TDK saves: $6,000 upfront + $1,485/month ongoing
```

---

## 🔗 Dependencies

**Blocked by:**
- Effort 1 API contract (need exact formats)

**Can start:**
- Scaffolding (independent)
- Type design (independent)
- Study successful SDKs (independent)

**Blocks:**
- Effort 3 (plugins need TettoDK class)

**Timeline:**
```
Week 1: Effort 1 defines API contract
Week 2: Effort 2 builds client library (parallel with Effort 1 testing)
Week 3: Effort 2 publishes to NPM
Week 4: Effort 3 builds plugins (uses Effort 2)
```

---

## 📊 Comparison with Existing SDK

**Existing tetto-sdk (Agent-focused):**
```typescript
import { TettoSDK } from 'tetto-sdk';

// For agents calling agents
const tetto = new TettoSDK({ ... });
await tetto.callAgent(agentId, input, wallet);  // Needs wallet!
```

**New @tetto/tdk (Developer-focused):**
```typescript
import { TettoDK } from '@tetto/tdk';

// For developers calling agents
const tdk = new TettoDK({ apiKey: '...' });
await tdk.callAgent(agentId, input);  // No wallet needed!
```

**Key difference:**
- `tetto-sdk` = For building agents (complex, crypto-native)
- `@tetto/tdk` = For using agents (simple, no crypto)

**Both will coexist!**

---

## 🎯 Why This Matters

**Problem:** Effort 1 provides REST API, but raw HTTP is still tedious.

**Solution:** Effort 2 wraps API with elegant SDK.

**Impact:**
- 100x easier to integrate (3 lines vs. 50 lines)
- Better DX (IntelliSense, types, error handling)
- Faster adoption (npm install = instant start)
- Less support burden (SDK handles edge cases)

**This turns TDK from "usable" to "delightful." ✨**

---

## 📖 Where to Learn More

- **Detailed mission:** `START_HERE.md` (comprehensive for AI2)
- **All 3 efforts:** `../MULTI_AI_DEV_EFFORTS/FOUNDATIONAL_EFFORTS_OVERVIEW.md`
- **Backend foundation:** `../EFFORT_1_TDK_BACKEND_FOUNDATION/`

---

**Effort 2 is about developer experience. Make it elegant, make it fast, make it delightful.** 🎨
