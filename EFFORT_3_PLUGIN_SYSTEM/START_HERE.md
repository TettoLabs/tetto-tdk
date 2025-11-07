# START_HERE: Effort 3 - Plugin System & First Plugins

**Version:** 1.0
**Created by:** MEGA AI 1 (Researcher)
**Target:** MEGA AI 2 (Guide Creator)
**Estimated Time:** 20-25 hours of implementation (after research)
**Status:** 🔬 Research Phase - AI2 Must Validate & Create Guides

---

## 🎯 Mission for AI2

You are **MEGA AI 2** - the Guide Creator. Your job is to:

1. **Complete research** (fill 6 research gaps specific to plugin architecture)
2. **Design plugin interface** (extend TettoDK from Effort 2)
3. **Create implementation guides** for AI3
4. **Document plugin patterns** (for community plugin developers)
5. **Ensure excellent plugin developer experience**

**Success criteria:** AI3 can build plugin system + 2 official plugins, and community can easily create their own plugins.

---

## 📋 What This Effort Builds

### **Plugin System for TettoDK**

Enables developers to extend TettoDK with additional agent shortcuts.

**Core features:**
- ✅ Plugin interface in `@tetto/tdk` core
- ✅ Plugin registration via `tdk.use(plugin)`
- ✅ Lifecycle hooks (init, destroy)
- ✅ Security boundary (plugins can't access API keys)
- ✅ Official plugin: `@tetto/plugin-memory` (WarmMemory)
- ✅ Official plugin: `@tetto/plugin-answers` (WarmAnswers)
- ✅ Plugin template repo (for community)
- ✅ Plugin documentation (how to build plugins)

**Why plugins?**

**Without plugins:**
```typescript
// TettoDK core has built-in shortcuts (bloated!)
import { TettoDK } from '@tetto/tdk';

const tdk = new TettoDK({ apiKey: '...' });
await tdk.memory.set('key', 'value');     // Built-in
await tdk.answers.ask('question');         // Built-in
await tdk.sentiment.analyze('text');       // Built-in?
await tdk.summarizer.summarize('text');    // Built-in?
// Where does it end? 100+ agents = 100+ built-in shortcuts?
```

**With plugins:**
```typescript
// TettoDK core is MINIMAL (only plugin interface)
import { TettoDK } from '@tetto/tdk';
import { MemoryPlugin } from '@tetto/plugin-memory';
import { AnswersPlugin } from '@tetto/plugin-answers';

const tdk = new TettoDK({ apiKey: '...' });

// Load only what you need
tdk.use(MemoryPlugin);
tdk.use(AnswersPlugin);

await tdk.memory.set('key', 'value');   // From plugin
await tdk.answers.ask('question');       // From plugin

// Community can create more:
import { SentimentPlugin } from '@acme/tdk-plugin-sentiment';
tdk.use(SentimentPlugin);
await tdk.sentiment.analyze('text');     // From community plugin!
```

**Benefits:**
- ✅ **Core stays small** - Only plugin interface in core
- ✅ **Tree-shakeable** - Only bundle plugins you use
- ✅ **Extensible** - Community can add agents
- ✅ **Versioned independently** - Update plugins without updating core
- ✅ **Standard patterns** - All plugins follow same interface

---

## 🏗️ Architecture Decision: Plugin Interface Design

### **How Plugins Work**

**Plugin interface (simple!):**
```typescript
// In @tetto/tdk core
export interface TDKPlugin {
  name: string;           // Namespace on TettoDK (e.g., 'memory')
  version: string;        // Plugin version
  init(tdk: TettoDK): void | Promise<void>;
  destroy?(): void | Promise<void>;
}

// Plugin function signature
export type PluginFunction = (options?: any) => TDKPlugin;
```

**Plugin implementation:**
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
        async set(key: string, value: any) {
          return await tdk.callAgent('warmmemory', {
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
          return result.output.exists ? JSON.parse(result.output.value) : null;
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
    },

    async destroy() {
      // Cleanup if needed
    }
  };
};
```

**Usage:**
```typescript
import { TettoDK } from '@tetto/tdk';
import { MemoryPlugin } from '@tetto/plugin-memory';

const tdk = new TettoDK({ apiKey: '...' });

// Register plugin
tdk.use(MemoryPlugin());

// Use plugin methods
await tdk.memory.set('key', 'value');
```

### **Why This Design?**

**Compared to tetto-sdk plugin system:**
- ✅ **Simpler** - No PluginAPI security boundary (TDK core already handles auth)
- ✅ **Direct access** - Plugins call `tdk.callAgent()` directly
- ✅ **Type-safe** - Plugins extend TettoDK type definition
- ✅ **Lifecycle hooks** - init() for setup, destroy() for cleanup

**Compared to other options:**
- ❌ **Middleware pattern** - Too complex for simple shortcuts
- ❌ **Event-based** - Overkill for static methods
- ✅ **Direct extension** - Simple, intuitive, type-safe

---

## 🔬 CRITICAL: Research Tasks for AI2

AI1 identified **6 research gaps** specific to plugin development.

### Research Gap 1: Plugin Interface Design 🚨 CRITICAL

**What we DON'T know:**
- Should plugins access TettoDK internals? (apiKey, config)
- Security boundary needed? (like tetto-sdk PluginAPI)
- How to handle plugin conflicts? (two plugins want same namespace)
- Plugin initialization timing (sync vs async?)

**Why critical:**
- Wrong interface = breaking changes later
- Security issues if plugins can access API key
- Namespace collisions break SDK

**Research tasks:**
1. Study existing tetto-sdk plugin system:
   - Read `/home/user/tetto-sdk/src/plugin-api.ts`
   - Read `/home/user/tetto-sdk/src/index.ts` (plugin registration)
   - Understand PluginAPI security boundary
2. Compare with other plugin systems:
   - Vite plugins (simple, clean)
   - Webpack plugins (complex, powerful)
   - ESLint plugins (config-based)
   - Prettier plugins (extension-based)
3. Decide on TDK plugin interface:
   - Minimal (just name + init) OR
   - Full (lifecycle hooks, metadata, dependencies)
4. Design conflict resolution:
   - Error if namespace collision? OR
   - Prefix with plugin ID? OR
   - Let later plugin override?
5. Security analysis:
   - Can plugins access `tdk.apiKey`? (NO - dangerous!)
   - Can plugins access `tdk.config`? (Maybe - read-only?)
   - Can plugins intercept requests? (NO - too powerful)

**Deliverable:** `research/plugin_interface_design.md` (500-700 lines)

**Output must include:**
```markdown
# Plugin Interface Design

## Comparison with tetto-sdk Plugins

### tetto-sdk approach (PluginAPI boundary)
[Explain security boundary, why it exists]

### TDK approach (Direct access)
[Explain why TDK plugins are simpler]

## Recommended Interface
[Complete TypeScript interface with rationale]

## Security Boundary
- Can access: [list]
- Cannot access: [list]
- Rationale: [why]

## Conflict Resolution
[How to handle namespace collisions]

## Lifecycle Hooks
- init: [when called, what for]
- destroy: [when called, what for]
- onError: [optional, for error handling]

## Code Examples
[Complete plugin implementation example]
```

---

### Research Gap 2: TypeScript Plugin Types 🚨 CRITICAL

**What we DON'T know:**
- How to make TettoDK extensible via declaration merging?
- How to type plugin methods for IntelliSense?
- Generic plugin types vs. specific plugin types?

**Why critical:**
- Poor types = no IntelliSense = bad DX
- Wrong approach = can't extend TettoDK type

**Research tasks:**
1. Research TypeScript declaration merging:
   ```typescript
   // Plugin declares:
   declare module '@tetto/tdk' {
     interface TettoDK {
       memory: MemoryAPI;
     }
   }
   ```
2. Study how other libraries handle this:
   - Express middleware (extends Request type)
   - Vite plugins (extends UserConfig)
   - Jest matchers (extends Matchers)
3. Design type strategy:
   - Core exports: `TDKPlugin`, `PluginFunction`
   - Plugins export: Types + declaration merging
4. Ensure IntelliSense works:
   - After `tdk.use(MemoryPlugin)`, typing `tdk.mem` autocompletes?
   - Type safety: `tdk.memory.set('key', value)` type-checks?

**Deliverable:** `research/typescript_plugin_types.md` (400-600 lines)

**Output must include:**
```markdown
# TypeScript Plugin Types

## Declaration Merging
[Complete example of extending TettoDK]

## Plugin Type Definitions
[All interfaces plugins must implement]

## IntelliSense Validation
[How to test that autocomplete works]

## Reference Implementations
[How Express, Vite, Jest handle this]

## Recommended Approach
[Concrete recommendation with code]
```

---

### Research Gap 3: Plugin Versioning & Dependencies 🚨

**What we DON'T know:**
- How to version plugins independently?
- Peer dependencies on `@tetto/tdk` (which version?)
- Semantic versioning for plugins (breaking changes)
- Plugin compatibility matrix

**Why critical:**
- Wrong peer deps = installation fails
- No versioning strategy = chaos when plugins update

**Research tasks:**
1. Research peer dependency patterns:
   ```json
   {
     "peerDependencies": {
       "@tetto/tdk": "^1.0.0"
     }
   }
   ```
2. Decide on compatibility:
   - Strict: `@tetto/tdk@1.0.0` only
   - Flexible: `@tetto/tdk@^1.0.0` (any 1.x)
   - Very flexible: `@tetto/tdk@>=1.0.0` (any version)
3. Breaking change strategy:
   - If TettoDK plugin interface changes → Major version
   - If agent API changes → Patch version (plugin handles it)
4. Validate: Install plugin with different TDK versions
   - `@tetto/tdk@1.0.0` + `@tetto/plugin-memory@1.0.0` → works?
   - `@tetto/tdk@1.5.0` + `@tetto/plugin-memory@1.0.0` → works?

**Deliverable:** `research/plugin_versioning.md` (300-400 lines)

---

### Research Gap 4: Plugin Testing Strategy 🚨

**What we DON'T know:**
- How to test plugins in isolation?
- Mock TettoDK for plugin tests?
- Integration tests (plugin + real TettoDK)?

**Why critical:**
- No tests = broken plugins
- Wrong testing approach = flaky tests

**Research tasks:**
1. Design test strategy:
   - Unit tests: Mock TettoDK, test plugin logic
   - Integration tests: Real TettoDK, mock backend
   - E2E tests: Real TettoDK + staging API
2. Create TettoDK mock:
   ```typescript
   const mockTDK = {
     callAgent: jest.fn(),
     config: { apiKey: 'test-key' }
   };
   ```
3. Test plugin lifecycle:
   - Test init() registers methods correctly
   - Test destroy() cleans up
   - Test error handling
4. Test IntelliSense (type checking in tests)

**Deliverable:** `research/plugin_testing.md` (300-500 lines)

---

### Research Gap 5: Plugin Documentation Standards 🚨

**What we DON'T know:**
- What docs should every plugin have?
- README template for plugins
- Example structure for plugin repos

**Why critical:**
- Poor docs = no adoption
- Inconsistent docs = confused developers

**Research tasks:**
1. Study plugin docs from other ecosystems:
   - Vite plugins: Installation, config, examples
   - Webpack loaders: Options, usage, API
   - Babel plugins: Configuration, transforms
2. Create documentation checklist:
   - [ ] Installation instructions
   - [ ] Basic usage example
   - [ ] API reference
   - [ ] TypeScript types
   - [ ] Error handling
   - [ ] Contributing guide
3. Design plugin README template:
   ```markdown
   # @tetto/plugin-{name}

   ## Installation
   npm install @tetto/plugin-{name}

   ## Usage
   [Quick example]

   ## API
   [All methods]

   ## Options
   [Configuration]
   ```

**Deliverable:** `research/plugin_documentation.md` (200-300 lines)

---

### Research Gap 6: Plugin Distribution Strategy 🚨

**What we DON'T know:**
- NPM scope for official plugins (@tetto/plugin-* vs tetto-plugin-*)
- Community plugin naming convention
- Plugin registry/directory (list all available plugins)

**Why critical:**
- Wrong naming = confusion
- No discovery = plugins invisible

**Research tasks:**
1. Decide official naming:
   - Option A: `@tetto/plugin-memory` (scoped, official)
   - Option B: `tetto-plugin-memory` (flat namespace)
   - RECOMMEND: `@tetto/plugin-*` for official
2. Community naming convention:
   - Suggest: `tdk-plugin-{name}` or `@{org}/tdk-plugin-{name}`
   - Document in plugin guide
3. Discovery strategy:
   - NPM search: `keywords: ["tdk-plugin"]`
   - GitHub topic: `tdk-plugin`
   - Plugin directory: Future enhancement (list on website)
4. Quality badges:
   - Official plugins: `@tetto` scope
   - Verified plugins: Reviewed by Tetto team
   - Community plugins: Unverified

**Deliverable:** `research/plugin_distribution.md` (200-300 lines)

---

## 📚 Guides You Must Create

After completing research, create these guides in `/guides`:

### Guide 1: Plugin Interface Implementation

**File:** `guides/1_plugin_interface.md`

**Must include:**
- Modify TettoDK class to support plugins
- `use(plugin)` method implementation
- Plugin registration logic
- Namespace conflict detection
- TypeScript types for plugins
- Tests for plugin system

**Code changes to TettoDK:**
```typescript
// @tetto/tdk/src/index.ts
export class TettoDK {
  private plugins: Map<string, TDKPlugin> = new Map();

  use(pluginFn: PluginFunction, options?: any): this {
    const plugin = pluginFn(options);

    // Check namespace collision
    if (plugin.name in this) {
      throw new Error(`Plugin namespace collision: '${plugin.name}' already exists`);
    }

    // Store plugin
    this.plugins.set(plugin.name, plugin);

    // Initialize plugin
    plugin.init(this);

    return this;  // Allow chaining
  }

  async destroy(): Promise<void> {
    // Call destroy on all plugins
    for (const plugin of this.plugins.values()) {
      if (plugin.destroy) {
        await plugin.destroy();
      }
    }
    this.plugins.clear();
  }
}
```

---

### Guide 2: Memory Plugin Implementation

**File:** `guides/2_memory_plugin.md`

**Must include:**
- Complete `@tetto/plugin-memory` implementation
- Package setup (package.json, tsconfig, build)
- All methods: set(), get(), list(), delete()
- TypeScript types + declaration merging
- Tests (unit + integration)
- Documentation (README, examples)
- NPM publishing steps

**Package structure:**
```
@tetto/plugin-memory/
├── src/
│   ├── index.ts           # Plugin implementation
│   └── types.ts           # Type definitions
├── dist/
│   ├── index.js           # Built JS
│   ├── index.d.ts         # Type declarations
├── test/
│   └── index.test.ts      # Tests
├── package.json
├── tsconfig.json
├── README.md
└── LICENSE
```

---

### Guide 3: Answers Plugin Implementation

**File:** `guides/3_answers_plugin.md`

**Must include:**
- Complete `@tetto/plugin-answers` implementation
- All methods: teach(), ask(), update(), forget()
- Same structure as memory plugin
- Tests
- Documentation
- Publishing

---

### Guide 4: Plugin Template Repository

**File:** `guides/4_plugin_template.md`

**Must include:**
- Create template repo: `tdk-plugin-template`
- Minimal plugin skeleton
- TypeScript setup
- Testing setup
- Documentation template
- GitHub template features (create from template button)

**Template structure:**
```
tdk-plugin-template/
├── src/
│   └── index.ts           # TODO: Implement your plugin
├── test/
│   └── index.test.ts      # TODO: Write tests
├── README.md              # TODO: Document your plugin
├── package.json
├── tsconfig.json
└── .github/
    └── workflows/
        └── ci.yml         # Auto-test on PR
```

---

### Guide 5: Plugin Developer Documentation

**File:** `guides/5_plugin_developer_guide.md`

**Must include:**
- Complete guide for creating plugins
- Step-by-step tutorial
- Best practices
- Common patterns
- Troubleshooting
- Publishing checklist

**Guide outline:**
```markdown
# Plugin Developer Guide

## Getting Started
- Clone template
- Install dependencies
- Understand structure

## Creating Your Plugin
- Choose agent to wrap
- Implement init()
- Add methods
- Type definitions

## Testing Your Plugin
- Unit tests
- Integration tests
- Type checking

## Publishing Your Plugin
- NPM setup
- Versioning
- Distribution

## Best Practices
- Error handling
- Type safety
- Documentation
```

---

### Guide 6: Plugin Testing Suite

**File:** `guides/6_plugin_testing.md`

**Must include:**
- Test setup for plugins
- Mock TettoDK
- Test plugin lifecycle
- Integration tests
- Type checking tests

---

## 📋 Checkpoint Structure for AI3

Create 6-8 checkpoints in `/checkpoints`:

### Example Checkpoint

**File:** `checkpoints/CP1_plugin_interface.md`

```markdown
# CP1: Plugin Interface Implementation

## Goal
Add plugin support to TettoDK core

## Prerequisites
- Effort 2 complete (@tetto/tdk published)
- TypeScript understanding

## Files to Modify
- @tetto/tdk/src/index.ts (add use() method)
- @tetto/tdk/src/types.ts (add plugin types)

## Implementation Steps
1. Add plugin types to types.ts (from Guide 1)
2. Add plugins Map to TettoDK class
3. Implement use() method
4. Implement destroy() method
5. Add tests for plugin system
6. Update documentation

## Validation
- [ ] Can register plugin via use()
- [ ] Plugin init() called on registration
- [ ] Namespace collision throws error
- [ ] destroy() calls plugin cleanup
- [ ] Tests pass

## Estimated Time
3-4 hours

## Definition of Done
- Plugin system works
- Tests pass
- TypeScript types correct
- Can build and test locally
```

**Create 6-8 checkpoints:**
1. Plugin interface in core
2. Memory plugin implementation
3. Answers plugin implementation
4. Plugin template repository
5. Plugin developer guide
6. Testing suite
7. Publish plugins to NPM
8. Documentation & examples

---

## 🚨 Risk Analysis Required

Document risks in `research/risks.md`:

```markdown
# Implementation Risks

## Risk 1: Plugin Namespace Conflicts
**Probability:** Medium
**Impact:** Medium (breaks SDK)
**Mitigation:**
- Detect conflicts in use()
- Throw descriptive error
- Document namespace reservation

## Risk 2: Breaking Changes in TettoDK
**Probability:** Low
**Impact:** High (all plugins break)
**Mitigation:**
- Semver for TettoDK (major version = breaking)
- Peer dependency in plugins (@tetto/tdk@^1.0.0)
- Test plugins against multiple TDK versions

## Risk 3: Type Safety Issues
**Probability:** Medium
**Impact:** Medium (no IntelliSense)
**Mitigation:**
- Use declaration merging correctly
- Test IntelliSense in real projects
- Provide type examples in docs

## Risk 4: Plugin Quality Varies
**Probability:** High
**Impact:** Medium (bad DX for some plugins)
**Mitigation:**
- Official plugins set high bar
- Plugin template enforces best practices
- Quality guidelines in docs
- Consider plugin verification program

[... 6-10 risks total ...]
```

---

## ✅ Deliverables Checklist

Before handing off to AI3:

### Research (in `/research`):
- [ ] `plugin_interface_design.md` - Interface + security boundary
- [ ] `typescript_plugin_types.md` - Declaration merging, types
- [ ] `plugin_versioning.md` - Peer deps, compatibility
- [ ] `plugin_testing.md` - Testing strategy, mocks
- [ ] `plugin_documentation.md` - Doc standards, template
- [ ] `plugin_distribution.md` - Naming, discovery, NPM
- [ ] `risks.md` - All risks with mitigations

### Guides (in `/guides`):
- [ ] `1_plugin_interface.md` - Add plugin support to core
- [ ] `2_memory_plugin.md` - @tetto/plugin-memory
- [ ] `3_answers_plugin.md` - @tetto/plugin-answers
- [ ] `4_plugin_template.md` - Template repository
- [ ] `5_plugin_developer_guide.md` - How to create plugins
- [ ] `6_plugin_testing.md` - Testing suite

### Checkpoints (in `/checkpoints`):
- [ ] `CP1_plugin_interface.md`
- [ ] `CP2_memory_plugin.md`
- [ ] `CP3_answers_plugin.md`
- [ ] `CP4_template_repo.md`
- [ ] `CP5_developer_guide.md`
- [ ] `CP6_testing.md`
- [ ] `CP7_npm_publish.md`
- [ ] `CP8_documentation.md`

### Additional:
- [ ] Plugin template repo (separate GitHub repo)
- [ ] Plugin developer guide (comprehensive docs)

---

## 🔗 Dependencies

**Blocked by:**
- Effort 2 must be complete (@tetto/tdk core must exist)
- Plugin interface must be designed before building plugins

**Blocks:**
- Nothing! Effort 3 is the final foundational effort

**Can parallelize:**
- Design plugin interface while Effort 2 is in testing
- Create plugin template independently

---

## 📖 Reference Materials

**Available:**
- Existing tetto-sdk plugin system (`/home/user/tetto-sdk/src/plugin-api.ts`)
- WarmMemory plugin (`/home/user/warmmemory-plugin/src/index.ts`)
- Effort 2 TettoDK class (once published)

**Study these plugin systems:**
- Vite plugins: Simple, clean, lifecycle hooks
- Webpack plugins: Complex but powerful patterns
- Babel plugins: Transform-based approach
- ESLint plugins: Rule-based extensions
- Express middleware: Functional composition

---

## 🎯 Success Criteria

Your research and guides are complete when:

✅ All 6 research gaps filled
✅ Plugin interface designed and validated
✅ TypeScript declaration merging works
✅ 6 comprehensive guides created
✅ 8 checkpoints outlined
✅ 6-10 risks identified
✅ Plugin template repo designed
✅ Plugin developer guide comprehensive
✅ AI3 can build plugins without questions

---

## ⏱️ Time Budget

**Research Phase:** 8-10 hours
- Gap 1 (Interface design): 2-3 hours
- Gap 2 (TypeScript types): 2 hours
- Gap 3 (Versioning): 1 hour
- Gap 4 (Testing): 1-2 hours
- Gap 5 (Documentation): 1 hour
- Gap 6 (Distribution): 1 hour

**Guide Creation:** 10-12 hours
- 6 guides × 1.5-2 hours each

**Checkpoint Outlining:** 2 hours
- 8 checkpoints × 15 minutes each

**Total:** 20-24 hours for AI2 phase

---

## 🚀 How to Begin

1. **Read Effort 2 code:** Understand TettoDK class structure
2. **Study tetto-sdk plugins:** Reference implementation
3. **Study successful plugin systems:** Vite, Webpack, Babel
4. **Fill gaps 1-6:** Create research docs
5. **Create guides:** After research complete
6. **Outline checkpoints:** After guides complete
7. **Hand off to AI3:** With confidence!

---

## 💡 Key Insights from AI1

**Why plugins matter:**
- Core stays small (< 10 KB without plugins)
- Tree-shakeable (only bundle what you use)
- Extensible (community can add agents)
- Independently versioned (update plugins without core)

**Critical decisions:**
- Simple interface (name + init + destroy)
- No security boundary (TDK already handles auth)
- Direct access to callAgent() (plugins are thin wrappers)
- Declaration merging for types (IntelliSense works)
- Official plugins set quality bar (community follows)

**Success = Community adoption:**
- Easy to create plugins (template + guide)
- Easy to discover plugins (NPM search + keywords)
- Easy to use plugins (tdk.use() + autocomplete)
- High quality bar (official plugins are excellent)

---

**Good luck! You're enabling the plugin ecosystem. Make it simple, make it powerful, make it delightful for plugin developers! 🚀**
