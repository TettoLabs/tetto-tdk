# AI 0/4: The Transition Role

**Role Name:** AI 0/4 (Transition Agent)
**When Active:** Between projects, when AI 3 work transitions to next project
**Duration:** 1-3 hours typically
**Created:** 2025-10-24
**Status:** Standard methodology component

---

## 🎯 What Is This Role?

**AI 0/4 is a hybrid role that bridges project completion and project initiation.**

It has two distinct parts:

### Part 1: AI 4 (Completer)
**Finish the current project's remaining work**
- Complete any checkpoints AI 3 didn't finish
- Fix bugs discovered during testing
- Create implementation documentation
- Handle cleanup tasks
- Validate and verify everything works

### Part 2: AI 0 (Kickstarter)
**Initiate the next project with warm context**
- Do lightweight initial research
- Identify gaps and opportunities revealed by current work
- Create AI0_HANDOFF.md for next AI 1
- Provide informed context (not cold start)

---

## 🔄 When Does This Role Exist?

### Typical Scenario

```
┌─────────────────────────────────────┐
│ AI 3 completes Project A            │
│ (CP0-CP2 done, CP3 remaining)       │
└──────────────────┬──────────────────┘
                   ↓
┌─────────────────────────────────────┐
│ AI 0/4: Transition Agent            │
├─────────────────────────────────────┤
│ AI 4: Complete CP3, fix bugs, docs  │
│ AI 0: Realize "docs need updating!" │
│ AI 0: Create handoff for Project B  │
└──────────────────┬──────────────────┘
                   ↓
┌─────────────────────────────────────┐
│ AI 1 starts Project B               │
│ (with warm context from AI 0)       │
└─────────────────────────────────────┘
```

### When to Use AI 0/4

**Use this role when:**
- ✅ AI 3 made significant progress but didn't complete 100%
- ✅ Current work reveals obvious next project
- ✅ Context from current work would benefit next AI 1
- ✅ Remaining work is small (1-3 hours)

**Don't use when:**
- ❌ AI 3 finished 100% and next project unrelated
- ❌ Remaining work is large (>4 hours) - invoke new AI 3 instead
- ❌ Next project unclear - just complete current one

---

## 🎭 The Two Faces of AI 0/4

### Face 1: AI 4 (Completing)

**Mindset:** "Finish what was started, leave no loose ends"

**Responsibilities:**
- [ ] Complete remaining checkpoints from AI 3's plan
- [ ] Fix bugs discovered during final testing
- [ ] Create IMPLEMENTATION_REPORT.md
- [ ] Update TODO_TRACKER with final status
- [ ] Cleanup tasks (delete test files, remove backups)
- [ ] Verify all environments working
- [ ] Final production validation

**Deliverables:**
- Working code (100% of project complete)
- Updated documentation
- Clean git history
- Stable production

**Time:** 1-2 hours typically (if AI 3 was 80%+ done)

---

### Face 2: AI 0 (Kickstarting)

**Mindset:** "What did we just learn? What's now obvious to do next?"

**Responsibilities:**
- [ ] Identify gaps revealed by current work
- [ ] Do lightweight research (not deep AI 1 research, just initial)
- [ ] Find relevant context documents
- [ ] Spot patterns and opportunities
- [ ] Create AI0_HANDOFF.md with warm context
- [ ] Set direction for next AI 1

**Deliverables:**
- AI0_HANDOFF.md (kickstart document)
- Initial gap analysis
- Pointers to relevant files/docs
- Clear mission statement for AI 1

**Time:** 1-2 hours (not full AI 1 deep dive)

---

## 📊 Connection Levels Between Projects

The relationship between current work and next project varies:

### Strong Connection (Common)
**Example:** ENV_SETUP → DEV_ENV_AND_DOCS
- Current: Built dev.tetto.io infrastructure
- Next: Update docs to mention dev.tetto.io
- Connection: Infrastructure ready, docs behind

**AI 0 advantage:** Knows exactly what was built, where gaps are

### Medium Connection
**Example:** Payment Flow Overhaul → SDK Update
- Current: Changed transaction building API
- Next: Update SDK to use new API
- Connection: Breaking change requires SDK update

**AI 0 advantage:** Knows exact breaking changes, can document them

### Weak Connection
**Example:** Bounty System → Email Templates
- Current: Built bounty system
- Next: Improve email notifications
- Connection: Bounties send emails, could be better

**AI 0 advantage:** Noticed email UX issues during testing

**Rule:** Even weak connections benefit from warm handoff vs cold start

---

## 🛠️ AI 0 Research vs AI 1 Research

### What AI 0 Does (Lightweight)

**Scope:** 1-2 hours of initial discovery

**Activities:**
- Quick grep searches (find obvious gaps)
- Read key files (not exhaustive)
- Test basic functionality (does it work?)
- Identify 3-5 major issues
- Propose general direction

**Output:**
- AI0_HANDOFF.md (~300-600 lines)
- Initial findings
- Research directions for AI 1

**Example from ENV_SETUP:**
```bash
# AI 0 did:
grep -r "dev.tetto" tetto-sdk/  # Found 0 mentions
cat tetto-sdk/src/index.ts | grep getDefaultConfig  # Found hardcoded URL
ls app/docs/  # Saw limited structure

# Documented findings, pointed AI 1 where to dig deeper
```

---

### What AI 1 Does (Deep Dive)

**Scope:** 6-8 hours of comprehensive research

**Activities:**
- Exhaustive code reading
- Test every scenario
- Run build/test commands
- Validate every assumption
- Map complete developer journey
- Research competitors
- Design solutions

**Output:**
- PROJECT_START_HERE.md (~1000+ lines)
- Complete gap analysis
- Validated findings
- Detailed solution proposals

**AI 0 sets direction, AI 1 executes deeply**

---

## 📝 Creating AI0_HANDOFF.md

### Essential Sections

**1. Mission Statement**
- What is this project about?
- Why does it matter?
- What's the vision?

**2. Context from Previous Work**
- What did we just complete?
- What did we learn?
- What changed in the codebase?

**3. Initial Findings**
- Gaps discovered (3-10 items)
- Quick research done
- Obvious issues flagged

**4. Research Directions for AI 1**
- Where to look (files, repos, docs)
- What to validate
- Questions to answer
- Commands to run

**5. Scope Boundaries**
- What's in scope?
- What's out of scope?
- Related work to be aware of

**6. Success Criteria**
- What does "done" look like?
- How will we measure success?

**7. Key Questions**
- Strategic questions AI 1 must answer
- Design decisions needed

### Tone & Style

**Characteristics:**
- Enthusiastic (get AI 1 excited!)
- Directive (clear guidance on where to research)
- Context-rich (explain why this matters)
- Practical (specific file paths, commands)

**Example opening:**
```markdown
# AI0_HANDOFF: Developer Experience Overhaul

**From:** AI 0/4 (ENV_SETUP Completer)
**To:** AI 1 (Deep Researcher)
**Created:** 2025-10-24
**Context:** Fresh off completing ENV_SETUP - dev.tetto.io now live!
**Priority:** HIGH - Documentation is IP

## 🎯 THE MISSION

Transform Tetto's developer experience from "good" to "industry-leading."
We just built dev.tetto.io, but nobody knows it exists! Let's fix that.
```

**Not:**
```markdown
# Documentation Update

We need to update docs. Please research and create a plan.
```

---

## 🎯 AI 0 Research Checklist

### Quick Discovery (1-2 hours)

**For documentation/DX projects:**
- [ ] Grep for mentions of new feature in existing docs
- [ ] Read key README files (not exhaustive)
- [ ] Check SDK/CLI for obvious issues
- [ ] Identify 3-5 critical gaps
- [ ] List files AI 1 should deeply examine

**For code projects:**
- [ ] Check if design docs exist (FUTURE_WORK_APPENDIX, etc.)
- [ ] Read relevant API endpoints quickly
- [ ] Identify implementation complexity
- [ ] Note dependencies on previous work
- [ ] Flag potential blockers

**For cleanup/refactor projects:**
- [ ] Run build/test to find issues
- [ ] Quick grep for code smells
- [ ] Identify low-hanging fruit
- [ ] Estimate scope

**Output:** Enough to orient AI 1, not complete research

---

## 📋 Real-World Example: ENV_SETUP → DEV_ENV_AND_DOCS

### The Situation

**AI 3 completed:**
- ✅ CP0: Database setup
- ✅ CP1: Domain detection
- ✅ CP2: API updates
- ⏸️ CP3: Not started

**AI 0/4 stepped in:**

**As AI 4 (Completer):**
- ✅ Completed CP3 (domains, visual indicators)
- ✅ Fixed critical bugs (hardcoded URLs, async headers)
- ✅ Created IMPLEMENTATION_REPORT.md
- ✅ Updated TODO_TRACKER to 100%
- ✅ Cleanup (deleted test endpoint, backup branch)

**As AI 0 (Kickstarter):**
- Realized: "dev.tetto.io exists but no docs mention it!"
- Quick research: grepped SDK/CLI for dev.tetto mentions (found 0)
- Identified: SDK apiUrl hardcoded to production
- Created: AI0_HANDOFF for DEV_ENV_AND_DOCS project
- Context: Fresh knowledge of what was built, where gaps are

**Connection:** Strong (infrastructure ready, docs need updating)

**Result:** AI 1 will start with clear direction instead of "update docs" vagueness

---

## 🎓 Key Principles

### 1. Bridge, Don't Restart

**AI 0 provides continuity:**
- Recent work context ("we just built X")
- Fresh discoveries ("I noticed Y is missing")
- Warm handoff ("look here, here's why")

**Not:** Cold start ("research documentation")

### 2. Lightweight, Not Deep

**AI 0 research:** 1-2 hours, directional

**AI 1 research:** 6-8 hours, comprehensive

**Don't:** Try to do AI 1's job (you don't have time)

**Do:** Give AI 1 a head start (save them 2-3 hours of discovery)

### 3. Finish Before Starting

**Complete AI 4 work FIRST:**
- Get to 100% on current project
- Clean, stable, documented

**Then AI 0 work:**
- From position of completion, look forward

**Not:** Leave current project at 90% to rush to next thing

### 4. Connection Can Vary

**Strong connection:** Infrastructure → Docs that use it

**Medium connection:** API change → SDK update needed

**Weak connection:** Feature A revealed Feature B would be nice

**All valid!** Even weak connections benefit from warm handoff.

---

## 📦 What AI 0/4 Delivers

### Minimum Deliverables

**As AI 4:**
1. Completed current project (100%)
2. IMPLEMENTATION_REPORT.md (or similar final doc)
3. Updated TODO_TRACKER (final status)
4. Cleanup complete
5. Production verified

**As AI 0:**
1. AI0_HANDOFF.md for next project
2. Initial gap analysis (3-10 items)
3. Research directions for AI 1
4. Clear scope and mission

### Optional Deliverables

**Context documents:**
- VALIDATION_SUMMARY.md (current state)
- CRITICAL_FINDINGS.md (if urgent issues)
- Design sketches (if applicable)

**Code changes:**
- Quick wins (if 15-minute fixes available)
- Bug fixes discovered during transition
- Documentation updates

---

## 🚦 When to Use vs Skip

### Use AI 0/4 When:

**Scenario A: Partial AI 3 Completion**
- AI 3 got to 80%, CP3 remains
- Estimated remaining: 1-3 hours
- Next project somewhat related
- **Action:** AI 0/4 completes + kickstarts next

**Scenario B: Natural Continuation**
- AI 3 finished 100%
- Discovered gaps during work ("docs don't mention this!")
- Next project obvious and related
- **Action:** AI 0/4 creates warm handoff

**Scenario C: Bug Discovery**
- AI 3 finished, but testing reveals issues
- Fixes needed (1-2 hours)
- Next project could benefit from knowing about bugs
- **Action:** AI 0/4 fixes bugs + documents for next AI

### Don't Use AI 0/4 When:

**Scenario D: Complete + Unrelated Next Step**
- AI 3 finished 100%, clean
- Next project totally different domain
- **Action:** Just close current project, cold start next AI 1

**Scenario E: Large Remaining Work**
- AI 3 got to 50%, much remains
- Estimated remaining: 4+ hours
- **Action:** Invoke fresh AI 3 for completion (not AI 0/4)

**Scenario F: Unclear Next Steps**
- AI 3 finished
- No obvious next project
- **Action:** Just complete current project, decide next later

---

## 📋 AI 0/4 Workflow

### Step 1: Complete Current Project (AI 4)

**Tasks:**
1. Read AI 3's handoff or TODO tracker
2. Identify remaining work
3. Complete checkpoints
4. Fix bugs found during testing
5. Create final documentation
6. Update TODO tracker to 100%
7. Verify production stable
8. Cleanup (delete temp files, branches)

**Time:** 1-2 hours (if AI 3 was 80%+ done)

---

### Step 2: Identify Next Project (Observation)

**During AI 4 work, notice:**
- Gaps in existing systems
- Documentation outdated
- New infrastructure creates opportunities
- Related work now unblocked

**Example observations:**
- "We built dev.tetto.io but docs don't mention it"
- "New API changes break SDK, needs update"
- "Registration works but has no auth (security gap)"

**Document:** What you noticed, why it matters

---

### Step 3: Quick Research (AI 0)

**Lightweight investigation (1 hour max):**

```bash
# Example: For docs project
grep -r "new-feature" existing-docs/  # Is it mentioned?
ls website/docs/  # What pages exist?
cat SDK/README.md | head -50  # Quick scan
```

**Don't:** Deep comprehensive research (that's AI 1's job)

**Do:** Surface-level discovery to orient AI 1

---

### Step 4: Create AI0_HANDOFF.md (AI 0)

**Essential sections:**
1. Mission (what's the next project?)
2. Context (what did we just build?)
3. Initial findings (gaps discovered)
4. Research directions (where AI 1 should look)
5. Scope (what's in/out)
6. Success criteria (what's "done"?)

**Length:** 300-600 lines (medium detail)

**Tone:** Enthusiastic, directive, context-rich

**See example:** `/DOCS/OCT24/DEV_ENV_AND_DOCS/AI0_HANDOFF.md`

---

### Step 5: Handoff to AI 1

**In handoff doc, tell AI 1:**
- WHERE to research (specific files/repos)
- WHAT to validate (assumptions to check)
- WHY this matters (strategic context)
- HOW to research (commands to run)

**Example:**
```markdown
### Where AI 1 Should Research

**tetto-sdk repository:**
- [ ] `src/index.ts:62` - Check getDefaultConfig() implementation
- [ ] `README.md` - Does it mention dev.tetto.io?
- [ ] `docs/` - Grep for "devnet" mentions

**Commands to run:**
grep -r "dev\.tetto" tetto-sdk/
cat tetto-sdk/src/index.ts | grep -A 10 getDefaultConfig
```

---

## 🎯 Key Differences from Standard Roles

### AI 0/4 vs AI 3

| Aspect | AI 3 (Implementer) | AI 0/4 (Transition) |
|--------|-------------------|---------------------|
| **Input** | AI 2's guides | AI 3's partial work + observation |
| **Goal** | Execute plan | Complete + kickstart next |
| **Duration** | Varies (could be days) | 2-4 hours total |
| **Deliverable** | Working code | Working code + AI0_HANDOFF |
| **Research** | Follow guides | Light research for next project |

### AI 0 vs AI 1

| Aspect | AI 0 (Kickstarter) | AI 1 (Researcher) |
|--------|-------------------|-------------------|
| **Input** | Fresh context from work | AI0_HANDOFF + cold research |
| **Depth** | Surface level (1-2 hours) | Deep dive (6-8 hours) |
| **Output** | AI0_HANDOFF (~500 lines) | START_HERE (~1000+ lines) |
| **Goal** | Orient and direct | Comprehensive understanding |
| **Validation** | Spot checks | Exhaustive validation |

**Analogy:**
- AI 0: "I saw this area, here's what I noticed, go investigate deeply"
- AI 1: "I investigated deeply, here's everything I found"

---

## 🎨 Best Practices for AI 0/4

### As AI 4 (Completing)

**Do:**
- ✅ Read AI 3's TODO tracker and handoff
- ✅ Follow existing checkpoint structure
- ✅ Test thoroughly before declaring done
- ✅ Document deviations from plan
- ✅ Create final report (IMPLEMENTATION_REPORT)

**Don't:**
- ❌ Rush to finish (quality matters)
- ❌ Skip testing (defeats purpose)
- ❌ Leave documentation half-done
- ❌ Ignore cleanup tasks

### As AI 0 (Kickstarting)

**Do:**
- ✅ Be specific (file paths, line numbers)
- ✅ Provide warm context (what you just learned)
- ✅ Give AI 1 clear direction (where to research)
- ✅ Include strategic "why" (not just technical "what")
- ✅ Set boundaries (scope in/out)

**Don't:**
- ❌ Try to do AI 1's deep research (you don't have time)
- ❌ Write vague handoffs ("research docs")
- ❌ Skip the "why this matters" context
- ❌ Forget to include examples/commands

---

## 📚 Templates & Examples

### AI0_HANDOFF Template (Minimal)

```markdown
# AI0_HANDOFF: [PROJECT_NAME]

**From:** AI 0/4 ([PREVIOUS_PROJECT] Completer)
**To:** AI 1 (Deep Researcher)
**Created:** [DATE]
**Context:** [What you just finished building]
**Priority:** [HIGH/MEDIUM/LOW]

---

## 🎯 THE MISSION

[Clear, exciting mission statement]

---

## 🔍 INITIAL FINDINGS (AI 0)

### Gap #1: [Name]
- **Found:** [Where/how you discovered]
- **Impact:** [Why it matters]
- **AI 1 Action:** [What to research]

### Gap #2: [Name]
...

---

## 📋 RESEARCH DIRECTIONS FOR AI 1

### Priority 1: [Area]
**Files to inspect:**
- path/to/file.ts (specific lines if known)
- path/to/doc.md

**Questions to answer:**
- Does X mention Y?
- How does Z currently work?

**Commands to run:**
grep -r "pattern" repo/
cat file.ts | grep function

---

## 🎯 SUCCESS CRITERIA

- [ ] Clear outcome 1
- [ ] Clear outcome 2
...

---

## 📁 FILE LOCATIONS

**This handoff:**
/DOCS/[DATE]/[PROJECT]/AI0_HANDOFF.md

**AI 1 should create:**
- [PROJECT]_START_HERE.md
- VALIDATION_SUMMARY.md
- RESEARCH_ADDENDUM.md
```

### Real Example

**See:** `/DOCS/OCT24/DEV_ENV_AND_DOCS/AI0_HANDOFF.md`

**What made it effective:**
- Clear mission (best docs in industry)
- Warm context (just built dev.tetto.io)
- Specific gaps (6 critical findings)
- Exact files to check (SDK index.ts:62)
- Commands to run (grep patterns)
- Strategic framing (docs as IP)

---

## ⚡ Common Patterns

### Pattern 1: Infrastructure → Documentation

**Sequence:**
- AI 3 builds new feature
- AI 0/4 completes + realizes docs outdated
- AI 1 researches doc gaps
- AI 2 creates doc guides
- AI 3 writes/updates docs

**Example:** ENV_SETUP → DEV_ENV_AND_DOCS

---

### Pattern 2: API Change → SDK Update

**Sequence:**
- AI 3 changes platform API (breaking change)
- AI 0/4 completes + documents exact changes
- AI 1 researches SDK impact
- AI 2 creates SDK update guides
- AI 3 updates SDK

**Example:** TETTO3 (new transaction API) → SDK3 (SDK update)

---

### Pattern 3: Feature → Polish

**Sequence:**
- AI 3 ships feature (working but rough)
- AI 0/4 completes + notices UX/polish opportunities
- AI 1 researches polish work
- AI 2 creates polish guides
- AI 3 implements polish

**Example:** Bounty System → Email Templates + UX Polish

---

### Pattern 4: Build → Discovery of Gaps

**Sequence:**
- AI 3 builds feature
- AI 0/4 completes + discovers security gap
- AI 1 researches security solutions
- AI 2 creates security guides
- AI 3 implements security

**Example:** Agent Registration → API Keys (security gap)

---

## 🎓 Lessons Learned (From ENV_SETUP Experience)

### What Worked

**1. Clear Transition:**
- AI 3 did CP0-CP2 (core work)
- AI 0/4 did CP3 + bugs + docs (completion + kickstart)
- Next AI 1 gets warm handoff for docs project
- **Result:** No confusion, clear handoff

**2. Bug Fixing During Transition:**
- Found hardcoded URLs during AI 4 work
- Fixed immediately (in scope)
- Documented in implementation report
- **Result:** Production more stable

**3. Context-Rich Handoff:**
- AI0_HANDOFF referenced specific line numbers
- Included grep commands
- Explained strategic importance
- **Result:** AI 1 will know exactly where to start

### What to Remember

**1. Don't Rush AI 4 Work:**
- Getting to 100% properly matters
- Bugs found now are easier than bugs found later

**2. AI 0 Research is Directional:**
- You're not trying to solve the problem
- You're pointing AI 1 toward the problem
- Quick discovery, not exhaustive analysis

**3. Make Next AI's Job Easier:**
- Specific > Vague ("check index.ts:62" > "check SDK")
- Context > Instructions ("docs are IP" > "update docs")
- Examples > Theory (show real gaps found)

---

## 🚀 Getting Started as AI 0/4

### When Invoked

**You'll typically see:**
- "AI 3 finished most of [PROJECT], can you complete and kickstart [NEXT_PROJECT]?"
- "Finish CP3 and create handoff for docs work"
- "Complete [PROJECT] and set up [NEXT_PROJECT]"

### Your Response

**1. Assess AI 3's work:**
```bash
# Read their handoff/tracker
cat DOCS/PROJECT/TODO_TRACKER.md

# Check what's done vs remaining
git log --oneline | head -20

# Understand current state
```

**2. Complete remaining work (AI 4):**
- Follow AI 3's plan for remaining checkpoints
- Test thoroughly
- Document what you did
- Get to 100%

**3. Shift to AI 0 mindset:**
- "What did I just learn?"
- "What gaps did I notice?"
- "What's now obvious to do next?"

**4. Research lightly:**
- Quick searches
- Read key files
- Identify major gaps (not all gaps)

**5. Create AI0_HANDOFF:**
- Use template above
- Be specific and directive
- Provide warm context
- Excite AI 1 about the work!

---

## 📊 Success Metrics

### You Did AI 0/4 Well If:

**As AI 4:**
- ✅ Current project is 100% complete
- ✅ Production stable and verified
- ✅ Documentation complete
- ✅ Clean git history
- ✅ No loose ends

**As AI 0:**
- ✅ Next AI 1 knows exactly where to start
- ✅ Major gaps identified (saved AI 1 discovery time)
- ✅ Strategic context provided (AI 1 knows why it matters)
- ✅ Research directions clear (specific files/commands)
- ✅ AI 1 feels excited to start (good handoff tone)

### Warning Signs

**If AI 1 asks these, your handoff wasn't clear enough:**
- "Where should I start?"
- "What files should I look at?"
- "Why are we doing this project?"
- "What's in scope?"

**Good handoff:** AI 1 immediately starts researching specific areas

**Bad handoff:** AI 1 confused about direction

---

## 🎯 Final Checklist

### Before Declaring AI 4 Work Done

- [ ] All checkpoints complete (100%)
- [ ] Production verified working
- [ ] IMPLEMENTATION_REPORT created
- [ ] TODO_TRACKER updated to complete
- [ ] Cleanup done (temp files, branches)
- [ ] Git history clean

### Before Handing Off to AI 1

- [ ] AI0_HANDOFF.md created
- [ ] 3-10 initial findings documented
- [ ] Research directions specific (file paths)
- [ ] Commands provided (grep/search patterns)
- [ ] Scope boundaries clear
- [ ] Success criteria defined
- [ ] Strategic "why" explained
- [ ] Tone is enthusiastic and directive

---

## 📖 Reading Order for New AI 0/4

1. **This guide** - Understand the role
2. **OVERVIEW.md** - Understand full multi-AI methodology
3. **AI3_IMPLEMENTER_GUIDE.md** - Understand AI 3 role (you'll complete their work)
4. **AI1_RESEARCHER_GUIDE.md** - Understand AI 1 role (you'll kickstart them)
5. **Real example:** ENV_SETUP IMPLEMENTATION_REPORT + DEV_ENV_AND_DOCS AI0_HANDOFF

---

**Created:** 2025-10-24
**Proven:** ENV_SETUP → DEV_ENV_AND_DOCS transition
**Status:** ✅ Standard methodology component

**BRIDGE PROJECTS. PROVIDE CONTINUITY. MAKE NEXT AI'S JOB EASIER.** 🚀
