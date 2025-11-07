# Effort 3: Plugin System & First Plugins - High Level Overview

**Status:** 🔬 Research Phase (AI2 not started)
**Implementation Time:** 20-25 hours
**Complexity:** Low-Medium (Plugin architecture design, 2 plugins)

---

## 🎯 What It Does (One Sentence)

Enables developers to extend TettoDK with additional agent shortcuts through a simple plugin system, and provides 2 official plugins as examples.

---

## 🏗️ What Gets Built

### **Plugin System for TettoDK**

A lightweight plugin architecture that lets developers add agent shortcuts to TettoDK.

**Before Plugin System:**
```typescript
// TettoDK core has ALL shortcuts built-in (bloated!)
import { TettoDK } from '@tetto/tdk';

const tdk = new TettoDK({ apiKey: '...' });

// Built-in (always loaded, even if unused)
await tdk.memory.set('key', 'value');
await tdk.answers.ask('question');
await tdk.sentiment.analyze('text');  // +1 KB
await tdk.summarizer.summarize('text');  // +1 KB
await tdk.translator.translate('text');  // +1 KB
// ... 100+ agents = 100 KB bundle! 😱
```

**After Plugin System:**
```typescript
// TettoDK core is MINIMAL (only plugin interface)
import { TettoDK } from '@tetto/tdk';
import { MemoryPlugin } from '@tetto/plugin-memory';  // +0.5 KB
import { AnswersPlugin } from '@tetto/plugin-answers';  // +0.5 KB

const tdk = new TettoDK({ apiKey: '...' });

// Load only what you need
tdk.use(MemoryPlugin);
tdk.use(AnswersPlugin);

await tdk.memory.set('key', 'value');  // Works!
// Total bundle: Core (10 KB) + Memory (0.5 KB) + Answers (0.5 KB) = 11 KB ✅
```

**Benefits:**
- ✅ **Smaller bundles** - Only load plugins you use
- ✅ **Extensible** - Community can create plugins for any agent
- ✅ **Independently versioned** - Update plugins without updating core
- ✅ **Tree-shakeable** - Unused plugins = 0 bytes in bundle

---

## 🔑 Key Components

### 1. **Plugin Interface** (in TettoDK Core)

**Simple, minimal interface:**
```typescript
// @tetto/tdk/src/types.ts
export interface TDKPlugin {
  name: string;           // Namespace (e.g., 'memory')
  version: string;        // Plugin version
  init(tdk: TettoDK): void | Promise<void>;
  destroy?(): void | Promise<void>;
}

export type PluginFunction = (options?: any) => TDKPlugin;
```

**Usage in TettoDK:**
```typescript
// @tetto/tdk/src/index.ts
export class TettoDK {
  private plugins: Map<string, TDKPlugin> = new Map();

  use(pluginFn: PluginFunction, options?: any): this {
    const plugin = pluginFn(options);

    // Check namespace collision
    if (plugin.name in this) {
      throw new Error(`Namespace '${plugin.name}' already in use`);
    }

    // Register and initialize
    this.plugins.set(plugin.name, plugin);
    plugin.init(this);

    return this;  // Chainable
  }

  async destroy(): Promise<void> {
    for (const plugin of this.plugins.values()) {
      await plugin.destroy?.();
    }
    this.plugins.clear();
  }
}
```

### 2. **Official Plugin: @tetto/plugin-memory**

**Wraps WarmMemory agent:**
```typescript
// @tetto/plugin-memory/src/index.ts
import type { TDKPlugin, TettoDK, PluginFunction } from '@tetto/tdk';

export const MemoryPlugin: PluginFunction = (options = {}) => {
  return {
    name: 'memory',
    version: '1.0.0',

    init(tdk: TettoDK) {
      // Extend TettoDK with memory methods
      tdk.memory = {
        async set(key: string, value: any): Promise<void> {
          await tdk.callAgent('warmmemory', {
            action: 'store',
            key,
            value: JSON.stringify(value)
          });
        },

        async get<T = any>(key: string): Promise<T | null> {
          const result = await tdk.callAgent('warmmemory', {
            action: 'retrieve',
            key
          });
          return result.output.exists
            ? JSON.parse(result.output.value)
            : null;
        },

        async list(prefix = ''): Promise<string[]> {
          const result = await tdk.callAgent('warmmemory', {
            action: 'list',
            prefix
          });
          return result.output.items || [];
        },

        async delete(key: string): Promise<void> {
          await tdk.callAgent('warmmemory', {
            action: 'delete',
            key
          });
        }
      };
    }
  };
};

// TypeScript declaration merging (IntelliSense!)
declare module '@tetto/tdk' {
  interface TettoDK {
    memory: {
      set(key: string, value: any): Promise<void>;
      get<T = any>(key: string): Promise<T | null>;
      list(prefix?: string): Promise<string[]>;
      delete(key: string): Promise<void>;
    };
  }
}
```

**Usage:**
```typescript
import { TettoDK } from '@tetto/tdk';
import { MemoryPlugin } from '@tetto/plugin-memory';

const tdk = new TettoDK({ apiKey: '...' });
tdk.use(MemoryPlugin);

// IntelliSense works! Type-safe!
await tdk.memory.set('user:prefs', { theme: 'dark' });
const prefs = await tdk.memory.get<UserPrefs>('user:prefs');
```

### 3. **Official Plugin: @tetto/plugin-answers**

**Wraps WarmAnswers agent:**
```typescript
// @tetto/plugin-answers/src/index.ts
export const AnswersPlugin: PluginFunction = (options = {}) => {
  return {
    name: 'answers',
    version: '1.0.0',

    init(tdk: TettoDK) {
      tdk.answers = {
        async teach(fact: string): Promise<void> {
          await tdk.callAgent('warmanswers', {
            action: 'teach',
            question: fact  // WarmAnswers uses 'question' param
          });
        },

        async ask(question: string): Promise<string> {
          const result = await tdk.callAgent('warmanswers', {
            action: 'ask',
            question
          });
          return result.output.answer;
        },

        async update(oldFact: string, newFact: string): Promise<void> {
          await tdk.callAgent('warmanswers', {
            action: 'update',
            old_fact: oldFact,
            new_fact: newFact
          });
        },

        async forget(fact: string): Promise<void> {
          await tdk.callAgent('warmanswers', {
            action: 'forget',
            question: fact
          });
        }
      };
    }
  };
};
```

### 4. **Plugin Template Repository**

**GitHub template for community plugins:**
```
tdk-plugin-template/
├── src/
│   └── index.ts           # Plugin skeleton with TODOs
├── test/
│   └── index.test.ts      # Test template
├── README.md              # Documentation template
├── package.json           # Configured for TDK plugins
├── tsconfig.json
├── .github/
│   └── workflows/
│       └── ci.yml         # Auto-test on PR
└── LICENSE
```

**Features:**
- Click "Use this template" on GitHub → instant plugin repo
- Pre-configured TypeScript, testing, CI/CD
- TODOs guide developers step-by-step
- Documentation template filled out

### 5. **Plugin Developer Guide**

**Comprehensive guide for creating plugins:**
```markdown
# Plugin Developer Guide

## Quick Start (5 minutes)
1. Clone template: github.com/TettoLabs/tdk-plugin-template
2. Install: npm install
3. Implement: src/index.ts
4. Test: npm test
5. Publish: npm publish

## Your First Plugin
[Step-by-step tutorial creating a sentiment plugin]

## API Reference
[All plugin interfaces, methods, hooks]

## Best Practices
- Keep plugins small (< 1 KB)
- Document all methods
- Handle errors gracefully
- Test thoroughly

## Publishing Checklist
- [ ] Tests pass (npm test)
- [ ] Documentation complete
- [ ] Examples included
- [ ] Version bumped
- [ ] Published to NPM
```

### 6. **TypeScript Declaration Merging**

**How IntelliSense works:**
```typescript
// After: tdk.use(MemoryPlugin)
// Developer types: tdk.mem...
// IntelliSense shows:
//   - memory.set()
//   - memory.get()
//   - memory.list()
//   - memory.delete()

// How it works:
// 1. Plugin exports declaration merging:
declare module '@tetto/tdk' {
  interface TettoDK {
    memory: MemoryAPI;
  }
}

// 2. TypeScript merges interfaces automatically
// 3. IntelliSense sees extended TettoDK interface
// 4. Autocomplete works! 🎉
```

---

## 📦 What Gets Published

**3 NPM packages:**

1. **@tetto/tdk@1.1.0** (updated with plugin interface)
   - Adds `use()` method
   - Adds `TDKPlugin` type
   - Size: Still ~10 KB (plugin interface is tiny)

2. **@tetto/plugin-memory@1.0.0** (new)
   - WarmMemory shortcuts
   - Size: ~0.5 KB
   - Peer dep: `@tetto/tdk@^1.1.0`

3. **@tetto/plugin-answers@1.0.0** (new)
   - WarmAnswers shortcuts
   - Size: ~0.5 KB
   - Peer dep: `@tetto/tdk@^1.1.0`

**Installation:**
```bash
npm install @tetto/tdk
npm install @tetto/plugin-memory @tetto/plugin-answers
```

---

## 🔄 Plugin Ecosystem

**Official Plugins (by Tetto):**
- `@tetto/plugin-memory` - WarmMemory agent
- `@tetto/plugin-answers` - WarmAnswers agent

**Community Plugins (by developers):**
```bash
# Naming convention: tdk-plugin-{name}
npm install tdk-plugin-sentiment
npm install @acme/tdk-plugin-translator
npm install tdk-plugin-summarizer
```

**Discovery:**
- NPM search: `keywords: ["tdk-plugin"]`
- GitHub topics: `tdk-plugin`
- Future: Plugin directory on tetto.io

---

## 📋 Implementation Checkpoints (8 CPs)

AI3 will implement in this order:

1. **CP1: Plugin Interface** (3-4h)
   - Add plugin types to TettoDK
   - Implement use() method
   - Add tests

2. **CP2: Memory Plugin** (3h)
   - Create @tetto/plugin-memory package
   - Implement all methods
   - Add tests

3. **CP3: Answers Plugin** (3h)
   - Create @tetto/plugin-answers package
   - Implement all methods
   - Add tests

4. **CP4: Plugin Template** (2h)
   - Create GitHub template repo
   - Add skeleton code
   - Add CI/CD workflow

5. **CP5: Developer Guide** (3h)
   - Write comprehensive guide
   - Step-by-step tutorial
   - Best practices

6. **CP6: Testing** (2h)
   - Test plugin system
   - Test both plugins
   - Integration tests

7. **CP7: NPM Publish** (2h)
   - Publish updated @tetto/tdk
   - Publish @tetto/plugin-memory
   - Publish @tetto/plugin-answers

8. **CP8: Documentation** (2h)
   - Update TettoDK README
   - Plugin READMEs
   - Examples

**Total:** 20-25 hours

---

## 📊 Before vs. After

### Before Effort 3
```typescript
// Developers use generic callAgent() for everything
import { TettoDK } from '@tetto/tdk';

const tdk = new TettoDK({ apiKey: '...' });

// Verbose, error-prone
await tdk.callAgent('warmmemory', {
  action: 'store',
  key: 'user:prefs',
  value: JSON.stringify({ theme: 'dark' })
});

// No IntelliSense for agent inputs ❌
// Must remember exact action names ❌
// No type safety ❌
```

### After Effort 3
```typescript
// Developers use plugins for shortcuts
import { TettoDK } from '@tetto/tdk';
import { MemoryPlugin } from '@tetto/plugin-memory';

const tdk = new TettoDK({ apiKey: '...' });
tdk.use(MemoryPlugin);

// Elegant, type-safe
await tdk.memory.set('user:prefs', { theme: 'dark' });

// IntelliSense works! ✅
// Type-safe parameters ✅
// Clean, readable code ✅
```

---

## 🎯 Success Metrics

**After Effort 3 is complete:**

✅ TettoDK supports plugins via use() method
✅ 2 official plugins published to NPM
✅ Plugin template repository available on GitHub
✅ Plugin developer guide comprehensive
✅ IntelliSense works for plugin methods
✅ Community can create plugins easily
✅ Bundle size stays small (tree-shakeable)

**Community adoption:**
- 5+ community plugins within 3 months
- 100+ plugin downloads per week
- Positive developer feedback on DX

---

## 🚧 What It DOESN'T Include

**Not in Effort 3:**
- ❌ Plugin marketplace (future enhancement)
- ❌ Plugin verification/certification (future)
- ❌ Plugin discovery dashboard (future)
- ❌ More official plugins (only memory + answers)
- ❌ Plugin dependencies (plugins can't depend on other plugins)
- ❌ Plugin configuration UI (CLI-based only)

**Effort 3 = Foundation only.** Ecosystem grows organically.

---

## ⚠️ Top 5 Risks

1. **Namespace collisions** → SDK breaks
   - Mitigation: Detect in use(), throw clear error

2. **Breaking changes in TettoDK** → All plugins break
   - Mitigation: Semver, peer dependencies, test matrix

3. **Type safety issues** → No IntelliSense
   - Mitigation: Declaration merging, test in real projects

4. **Low community adoption** → No plugins created
   - Mitigation: Excellent docs, easy template, promote on socials

5. **Low quality community plugins** → Bad reputation
   - Mitigation: Official plugins set high bar, quality guidelines

---

## 💰 Bundle Size Impact

**Without plugins (Effort 2):**
- Core with built-in shortcuts: 15 KB
- All agents always loaded (bloat)

**With plugins (Effort 3):**
- Core with plugin interface: 10 KB
- Memory plugin: +0.5 KB (if used)
- Answers plugin: +0.5 KB (if used)
- **Total (using both):** 11 KB
- **Total (using neither):** 10 KB

**Savings:** 4 KB if you don't use shortcuts (tree-shaking works!)

**At scale (50 plugins available):**
- Core: Still 10 KB
- Load 2 plugins: 11 KB
- Load 50 plugins: 35 KB (but why would you?)
- **Key:** Developer chooses what to bundle

---

## 🔗 Dependencies

**Blocked by:**
- Effort 2 must be complete (@tetto/tdk must exist)

**Can start:**
- Plugin interface design (independent research)
- Template repo setup (independent)

**Blocks:**
- Nothing! Final foundational effort

**Timeline:**
```
Week 1-2: Effort 2 builds core
Week 3: Effort 3 adds plugin interface (patch release: 1.0.0 → 1.1.0)
Week 4: Effort 3 builds official plugins
Week 5: Effort 3 publishes everything
```

---

## 📊 Comparison with tetto-sdk Plugins

**tetto-sdk plugins (for agents):**
```typescript
// Complex: PluginAPI security boundary
export const MyPlugin: Plugin = (api: PluginAPI, options) => {
  // api is restricted (can't access secrets)
  return {
    name: 'myPlugin',
    method() {
      api.callAgent(agentId, input, wallet);  // Must pass wallet
    }
  };
};
```

**@tetto/tdk plugins (for developers):**
```typescript
// Simple: Direct access to callAgent
export const MyPlugin: PluginFunction = (options) => {
  return {
    name: 'myPlugin',
    init(tdk) {
      tdk.myPlugin = {
        method() {
          tdk.callAgent(agentId, input);  // No wallet needed!
        }
      };
    }
  };
};
```

**Key difference:**
- tetto-sdk = Security boundary (agents are untrusted)
- @tetto/tdk = No boundary (plugins extend user's SDK, already trusted)

**Both are valid!** Different use cases.

---

## 🎯 Why This Matters

**Problem:** Effort 2 provides core SDK, but all shortcuts built-in = bloated.

**Solution:** Effort 3 enables plugins for extensibility.

**Impact:**
- Smaller core (10 KB vs 15 KB)
- Tree-shakeable (unused = 0 bytes)
- Extensible (community adds agents)
- Sustainable (don't need to update core for every agent)

**This makes TDK extensible forever. Community can add any agent without touching core.** 🌟

---

## 📖 Where to Learn More

- **Detailed mission:** `START_HERE.md` (comprehensive for AI2)
- **All 3 efforts:** `../MULTI_AI_DEV_EFFORTS/FOUNDATIONAL_EFFORTS_OVERVIEW.md`
- **TettoDK core:** `../EFFORT_2_TDK_CLIENT_LIBRARY/`

---

**Effort 3 is about ecosystem. Build the foundation, empower the community, watch it grow.** 🌱
