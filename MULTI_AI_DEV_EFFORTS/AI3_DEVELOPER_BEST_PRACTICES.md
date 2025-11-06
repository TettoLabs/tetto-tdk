# AI3 Developer Best Practices

**Audience:** AI developers (primarily AI 3 implementers) working on Tetto platform
**Context:** Git workflow, commits, testing, and deployment practices
**Updated:** 2025-10-24 (post ENV_SETUP)

---

## 🎯 The Golden Rule

**"Move fast without breaking things"**

Balance between:
- ✅ Quick iteration (don't over-engineer process)
- ✅ Safe rollback (granular commits when needed)
- ✅ Production stability (test before push)

---

## 🌐 Understanding the Environments

### The Three Environments

| Environment | Domain | Branch | Database | Purpose | Audience |
|-------------|--------|--------|----------|---------|----------|
| **Production** | www.tetto.io | `main` | Production | Live mainnet agents | Public |
| **Development** | dev.tetto.io | `main` | Production | Free devnet testing | Public |
| **Staging** | staging.tetto.io | `staging` | Staging | Pre-production testing | Internal (auth required) |

### Key Concepts

**1. Same Branch, Different Filtering:**
- www.tetto.io and dev.tetto.io BOTH use `main` branch
- Same code, different runtime behavior (domain detection)
- Network filtering happens at API level

**2. Staging is Your Safety Net:**
- Isolated database (can break without affecting production)
- Test risky changes here first
- Password protected (Vercel team only)

**3. Localhost:**
- Defaults to devnet filtering
- Uses production database (read-only mindset)
- Safe for development

---

## 🔀 Git Workflow

### Standard Feature Flow

```bash
# 1. Start from main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/add-new-dashboard

# 3. Develop locally, commit frequently (see commit guide below)
git add <files>
git commit -m "Add dashboard skeleton"
# ... more commits ...

# 4. Push and PR to staging (test first!)
git push origin feature/add-new-dashboard
# Create PR: feature/add-new-dashboard → staging

# 5. After merge, test on staging.tetto.io
# - Log in with Vercel
# - Verify feature works
# - Check for issues

# 6. If good, PR staging → main
# Create PR: staging → main

# 7. After merge to main, sync staging back
git checkout staging
git pull origin staging
git merge main
git push origin staging
```

### Hotfix Flow (Production Emergencies)

```bash
# Critical bugs can skip staging
git checkout -b hotfix/critical-payment-bug main

# Fix, test locally
git commit -m "Fix payment bug in escrow verification"

# Push and PR directly to main
git push origin hotfix/critical-payment-bug
# PR: hotfix → main

# After merge, sync staging
git checkout staging
git merge main
git push origin staging
```

### When to Use Each Flow

**Use standard flow (feature → staging → main) for:**
- ✅ New features
- ✅ Refactoring
- ✅ Non-critical bug fixes
- ✅ Database migrations
- ✅ API changes

**Use hotfix flow (direct to main) for:**
- ❌ Production is broken
- ❌ Security vulnerability
- ❌ Data loss risk
- ❌ Payment failures

**Rule:** If in doubt, use staging first.

---

## 📝 Commit Practices

### ⚠️ FIRST: Git Branch Safety

**✅ SOLVED: We now use separate dev environments (tetto-dev1, tetto-dev2)**

**The Old Problem (Now Solved):**
When multiple devs shared the same working directory (`tetto/`), git state was shared. Dev A creates a branch → Dev B's terminal switches to it automatically. Branch confusion chaos.

**The Solution (Current Setup):**
```
~/Desktop/eudaimonia/ai_coding/
├── tetto/         ← Original (reference only)
├── tetto-dev1/    ← Dev A / AI instance 1
│   ├── tetto-portal/
│   ├── tetto-sdk/
│   └── ... (all 6 repos)
└── tetto-dev2/    ← Dev B / AI instance 2
    ├── tetto-portal/
    ├── tetto-sdk/
    └── ... (all 6 repos)
```

**Each environment is completely isolated - no branch conflicts possible!**

**If using tetto-dev1 or tetto-dev2:** You're safe! Branch switching won't happen.

**Start every work session with:**
```bash
cd ~/Desktop/eudaimonia/ai_coding/tetto-dev1  # (or tetto-dev2)
./sync.sh  # Syncs all 6 repos with remote
```

**Then `git status` before commits is just good practice** (not paranoid necessity).

**If still using shared `tetto/` directory:** ⚠️ Read the paranoid checklist below (you'll need it).

---

#### Pre-Commit Checklist (For Shared Directory Users Only)

**Before EVERY commit:**

```bash
# 1. Check current branch (ALWAYS!)
git status

# 2. Read the output carefully:
# On branch feature/xyz ← Is this YOUR branch?

# 3a. If it's YOUR branch:
git checkout feature/your-branch  # Paranoid: checkout again to be sure
git commit -m "Your message"

# 3b. If it's SOMEONE ELSE's branch:
# Check if your branch exists:
git branch --list feature/your-branch

# If it exists (you created it earlier):
git checkout feature/your-branch
git commit -m "Your message"

# If it doesn't exist (first time working):
git checkout staging
git pull origin staging
git checkout -b feature/your-branch
git commit -m "Your message"
```

---

#### Why This Matters

**Your branch can change:**
- ✅ Even if you just created it
- ✅ Even if you just checked it out
- ✅ Anytime another dev creates their branch in the same directory

**The fix:** `git status` before EVERY commit (make it muscle memory)

---

#### Push Your Branch Early (Safety Backup)

**As soon as you create your branch, push it:**
```bash
git checkout -b feature/your-work
# Make first change
git add .
git commit -m "Initial commit"
git push origin feature/your-work -u  # ✅ Push immediately!
```

**Benefits:**
- ✅ Branch saved on remote (can't lose it)
- ✅ Easy to recover if local gets confused
- ✅ Other devs can see what you're working on
- ✅ CI/CD can run tests on your branch

**Then push frequently:** After every 1-3 commits

---

### Commit Frequency: The Middle Ground

**❌ Too Frequent (Avoid):**
```bash
git commit -m "Add import"
git commit -m "Add function declaration"
git commit -m "Add function body"
git commit -m "Add export"
```
**Problem:** 50 micro-commits = impossible to rollback meaningfully

**❌ Too Infrequent (Avoid):**
```bash
# One commit after 4 hours of work
git commit -m "Implement entire bounty system"
```
**Problem:** Can't rollback partially, lose granularity

**✅ Just Right (Aim For):**
```bash
git commit -m "Add bounty creation UI components"
git commit -m "Add bounty creation API endpoint"
git commit -m "Connect UI to API and test flow"
git commit -m "Add validation and error handling"
```
**Sweet spot:** ~30-60 minutes of focused work per commit

### When to Commit

**Commit when you complete a "unit of work":**
- ✅ One API endpoint fully implemented
- ✅ One component created and integrated
- ✅ One bug fixed and tested
- ✅ One feature working end-to-end (if small)
- ✅ Database migration applied and tested

**Don't commit:**
- ❌ Broken code (doesn't compile)
- ❌ Code you haven't tested
- ❌ Partial implementations (half-done feature)
- ❌ Just to "save progress" (use git stash instead)

**Exception:** Long-running work (4+ hours)
- Commit intermediate states with clear WIP markers
- `git commit -m "WIP: Bounty UI skeleton (not functional)"`
- Squash WIP commits before PR

### Commit Message Format

**Standard format:**
```
<type>: <subject>

<optional body>

<optional footer>
```

**Examples:**
```bash
# Good ✅
git commit -m "Add network validation to agent detail endpoint

- Import getDomainConfig for network detection
- Validate agent network matches domain
- Return 404 if mismatch with helpful hint
- Database switching based on domain"

# Also good ✅
git commit -m "Fix TypeScript linting error in test-env endpoint"

# Bad ❌
git commit -m "updates"
git commit -m "fix bug"
git commit -m "changes to api"
```

**Types:** feat, fix, refactor, docs, test, chore

**Rule:** Subject line should complete: "This commit will..."

### ⚠️ NO Co-Author Tags

**Don't add:**
```bash
# ❌ Don't do this:
git commit -m "Add feature

Co-Authored-By: Claude <noreply@anthropic.com>"
```

**Why:**
- Clutters git history
- Obvious AI involvement anyway
- Focus on technical content, not attribution

**Exception:** Human collaboration (pair programming with another dev)

---

## 🚀 Push & Deploy Strategy

### When to Push

**Push after each logical checkpoint:**
- ✅ After 1-3 related commits
- ✅ After completing a sub-feature
- ✅ Before switching contexts
- ✅ End of work session
- ✅ Before asking for review

**Example sequence:**
```bash
# Work session: Add network filtering
git commit -m "Add network column to database"
git commit -m "Update agent API with network filter"
git commit -m "Test network filtering"
git push origin feature/network-filtering  # ✅ Push checkpoint
```

**Don't push:**
- ❌ After every single commit (creates noise)
- ❌ Broken/untested code
- ❌ Mid-refactor (wait until it works)

### Deployment Triggers

**Vercel auto-deploys when you push:**
- Push to `staging` branch → deploys to staging.tetto.io
- Push to `main` branch → deploys to www.tetto.io + dev.tetto.io

**Strategy:**
1. **Test locally first** (npm run dev, verify it works)
2. **Push to feature branch** (no deployment)
3. **Merge to staging** (deploys to staging.tetto.io)
4. **Test on staging** (validate in deployed environment)
5. **Merge to main** (deploys to production)
6. **Monitor production** (check for errors)

### Sync staging After Main Merges

**Critical:** Keep staging close to main

```bash
# After EVERY merge to main (yours or others'):
git checkout staging
git pull origin staging
git merge main
git push origin staging
```

**Why:** Prevents staging from drifting too far from production

**Frequency:** After each main merge (could be daily if active development)

---

## 🧪 Testing Requirements

### Before Each Commit

**Minimum checks:**
- ✅ Code compiles (`npm run build` or type check)
- ✅ No obvious linting errors
- ✅ Basic manual test (does it work?)

**Don't need:**
- ❌ Full test suite (save for pre-push)
- ❌ E2E tests (save for staging)
- ❌ Perfect polish (iterate)

### Before Each Push

**Required:**
```bash
# 1. Build succeeds
npm run build

# 2. Tests pass (if exists)
npm run test

# 3. Type check (TypeScript projects)
npm run type-check  # or tsc --noEmit
```

**If any fail:** Fix before pushing, or commit as WIP and note in PR

### Before Merging to Staging

**Required:**
- ✅ All tests pass locally
- ✅ Build succeeds
- ✅ Feature works in local testing
- ✅ No console errors
- ✅ PR description explains changes

**Optional but recommended:**
- ✅ Test edge cases
- ✅ Check mobile view (if UI changes)
- ✅ Review your own diff (catch silly mistakes)

### Before Merging to Main (Production)

**REQUIRED:**
- ✅ Feature tested on staging.tetto.io
- ✅ No staging errors for 30+ minutes
- ✅ Edge cases validated
- ✅ Breaking changes documented
- ✅ Database migrations tested (if applicable)

**Critical path verification:**
```bash
# After deploy to staging, test core flows:
1. Can users sign in? (if auth changes)
2. Can agents be called? (if payment changes)
3. Do API endpoints work? (curl test each one)
4. Does dashboard load? (if UI changes)
```

---

## 🎯 Rollback Strategy

### Why Granular Commits Matter

**Scenario:** You make 5 changes, push to main, #4 breaks production

**With granular commits:**
```bash
git log
a1b2c3d (HEAD) Change 5: Add feature E
d4e5f6g Change 4: Refactor module D ← BUG HERE
g7h8i9j Change 3: Add feature C
j1k2l3m Change 2: Add feature B
m4n5o6p Change 1: Add feature A

# Revert just change 4:
git revert d4e5f6g
git push origin main
# ✅ Changes 1,2,3,5 remain, only 4 removed
```

**With one big commit:**
```bash
git log
a1b2c3d (HEAD) Add features A, B, C, D, E ← BUG IN D

# Must revert everything:
git revert a1b2c3d
# ❌ Lose A, B, C, E along with D
```

### Commit Granularity Guide

**One commit should contain:**
- ✅ One complete feature (if small, <30 min)
- ✅ One bug fix
- ✅ One API endpoint
- ✅ One component + integration
- ✅ Changes that belong together logically

**One commit should NOT contain:**
- ❌ Unrelated changes ("fix bug + add feature + update docs")
- ❌ Multiple files that can be separated
- ❌ Experimental code + production code
- ❌ "Everything I did today"

**Example from ENV_SETUP (Good ✅):**
```
CP2.1: Add network filtering to agent list endpoint
CP2.2: Fix token mint bug and auto-set network in registration
CP2.3: Add network validation to agent detail endpoint
CP2.4: Add network validation to build-transaction endpoint
```
Each commit = one endpoint = easy to rollback individually

---

## 🔍 Code Review & Testing Checklist

### Self-Review Before Pushing

**Ask yourself:**
- [ ] Did I test this locally?
- [ ] Does the build succeed?
- [ ] Are there any hardcoded values that should be env vars?
- [ ] Did I remove console.logs? (or make them intentional)
- [ ] Are variable names clear?
- [ ] Did I handle errors properly?
- [ ] Are there any TODOs I should address now?

### Staging Validation Checklist

**After deploying to staging.tetto.io:**
- [ ] Visit the URL, does it load?
- [ ] Test the specific feature you added
- [ ] Check browser console (any errors?)
- [ ] Check Vercel logs (any server errors?)
- [ ] Test on mobile (if UI changes)
- [ ] Try to break it (edge cases, invalid input)

**Time required:** 10-20 minutes per deployment

**If issues found:** Fix on feature branch, push again, re-test

---

## 🏗️ Environment-Specific Practices

### Understanding Our Two Databases

**⚠️ CRITICAL: Two Completely Separate Supabase Projects**

| Database | Project ID | Used By | Data |
|----------|-----------|---------|------|
| **Production** | ranrnjskpimzzpgrodxx | www.tetto.io + dev.tetto.io | 11 live agents, real transactions |
| **Staging** | nhirvkprspzfyilhldme | staging.tetto.io | Empty/test data, safe to break |

**Key points:**
- These are SEPARATE Supabase instances (not the same DB!)
- www and dev share production DB, but filter by network at runtime
- Changes to staging DB don't affect production (isolated)
- Staging can be reset/broken without consequence

**Rules for Production Database:**
- ✅ Test ALL migrations on staging database first (never prod first!)
- ✅ HUMAN runs all migrations (never automated!)
- ✅ Never DELETE or TRUNCATE in production
- ✅ Backup before risky operations (export data)

**Quick reference:**
```bash
# Staging DB:  nhirvkprspzfyilhldme (test here first!)
# Production DB: ranrnjskpimzzpgrodxx (apply here after staging test)
```

**See "Database Migrations" section below for complete workflow.**

### Using Staging Environment

**staging.tetto.io = Safe to break**

**Use staging for:**
- ✅ Testing database migrations
- ✅ Breaking changes validation
- ✅ New feature integration testing
- ✅ Performance testing
- ✅ Learning new patterns

**Staging database can be:**
- Reset anytime (isolated from production)
- Seeded with test data
- Broken without consequence

**Access staging:**
- Requires Vercel team authentication
- Log in via browser automatically if on team

---

## 📦 Multi-File Changes: Commit Strategy

### Scenario 1: Related Changes (Single Commit)

**Example:** Adding network filtering to one endpoint

**Files changed:**
- app/api/agents/route.ts (add network filter)
- lib/config/domain.ts (already exists, no change)

**Commit:**
```bash
git add app/api/agents/route.ts
git commit -m "Add network filtering to agent list endpoint"
```

**One commit** because it's one logical change.

---

### Scenario 2: Multiple Independent Changes (Multiple Commits)

**Example:** Updating 7 API endpoints for ENV_SETUP CP2

**Approach:**
```bash
# Commit 1
git add app/api/agents/route.ts
git commit -m "CP2.1: Add network filtering to agent list endpoint"

# Commit 2
git add app/api/agents/register/route.ts
git commit -m "CP2.2: Fix token mint bug in registration"

# Commit 3
git add app/api/agents/[id]/route.ts
git commit -m "CP2.3: Add network validation to agent detail"

# ... etc (7 separate commits)
```

**Why separate:** If one endpoint breaks, rollback that commit only

---

### Scenario 3: New Feature (Bundle Logically)

**Example:** Adding environment banner

**Files changed:**
- components/EnvironmentBanner.tsx (new component)
- app/layout.tsx (integrate component)

**Commit:**
```bash
git add components/EnvironmentBanner.tsx app/layout.tsx
git commit -m "Add environment banner component

- Create EnvironmentBanner.tsx (yellow for staging, blue for dev)
- Integrate into app/layout.tsx
- No banner shown on production"
```

**One commit** because component + integration is one feature unit.

---

## ⚡ Push Frequency

### The Push Cadence

**Good rhythm:**
- ✅ Every 1-3 commits
- ✅ Every 1-2 hours of work
- ✅ After completing a sub-feature
- ✅ Before context switching
- ✅ End of work session

**Example:**
```bash
# Morning: Work on feature
10:00 - commit "Add API endpoint"
10:30 - commit "Add UI component"
11:00 - commit "Connect UI to API"
11:15 - git push  # ✅ Push checkpoint (3 commits)

# Afternoon: Different feature
14:00 - commit "Fix bug in dashboard"
14:30 - git push  # ✅ Push (1 commit, but switching context)
```

### Why Push Regularly

**Benefits:**
1. **Backup** - Code saved remotely
2. **Collaboration** - Others can see progress
3. **CI/CD** - Triggers builds and tests
4. **Rollback points** - Clear checkpoints in history

**Don't push:**
- ❌ Broken code (WIP is okay if marked, but should compile)
- ❌ After every single commit (creates noise)
- ❌ Before testing locally

---

## 🐛 When Things Go Wrong

### Build Fails on Vercel

**Scenario:** Push to staging, Vercel build fails

**Response:**
```bash
# 1. Check build logs in Vercel dashboard
# 2. Fix locally
git add <files>
git commit -m "Fix build error: TypeScript strict mode issue"
git push origin feature/your-branch

# 3. Wait for rebuild
# 4. If succeeds, continue
```

**Don't:** Revert commits unless error is unfixable quickly

---

### Feature Works Locally But Breaks on Staging

**Scenario:** Deployed to staging, getting errors

**Response:**
```bash
# 1. Check Vercel logs (real-time)
# 2. Check browser console (client errors)
# 3. Identify root cause

# 4. If quick fix (< 15 min):
# - Fix locally, commit, push, re-test

# 5. If complex issue:
# - Revert merge to staging
git checkout staging
git revert HEAD
git push origin staging
# - Fix on feature branch properly
# - Re-merge when ready
```

---

### Production is Broken

**Scenario:** Merged to main, production has critical issue

**Response (Fast):**
```bash
# Option A: Revert merge
git checkout main
git revert <commit-hash>
git push origin main
# ✅ Instant rollback

# Option B: Hotfix forward
git checkout -b hotfix/fix-production
# Fix issue quickly
git commit -m "Hotfix: Fix critical payment bug"
git push origin hotfix/fix-production
# PR to main immediately
```

**Rule:** Revert first, fix later (unless fix is 5-minute obvious)

---

## 📊 Incremental Deployment Pattern

### The Checkpoint Approach

**Example:** ENV_SETUP had CP0, CP1, CP2, CP3

**Each checkpoint:**
1. Implemented separately
2. Tested on staging
3. Deployed to main
4. Verified in production
5. Then move to next checkpoint

**Benefits:**
- ✅ Know exactly what broke (if anything)
- ✅ Can rollback single checkpoint
- ✅ Reduce blast radius
- ✅ Incremental risk

**From ENV_SETUP:**
```
Day 1:
├─ CP0 + CP1 → staging → main → verify ✅
├─ Monitor 30 min ✅
└─ Continue to CP2

Day 1 (continued):
├─ CP2.1 (endpoint 1) → main → verify ✅
├─ CP2.2 (endpoint 2) → main → verify ✅
├─ ... (7 separate endpoint commits)
└─ All working ✅

Day 2:
└─ CP3 → staging → test → main ✅
```

**Pattern:** Small, tested, deployable chunks

---

## 🎯 Special Cases

### Database Migrations

**⚠️ CRITICAL: We Have Two Separate Databases**

| Database | Supabase Project | Used By | Purpose |
|----------|------------------|---------|---------|
| **Staging** | nhirvkprspzfyilhldme | staging.tetto.io | Safe testing, can break |
| **Production** | ranrnjskpimzzpgrodxx | www.tetto.io + dev.tetto.io | Live data, NEVER break |

**These are completely separate instances - changes to one don't affect the other!**

---

#### Migration Workflow (HUMAN-RUN ONLY)

**🚨 AI 3: You WRITE the SQL, HUMAN RUNS it**

**Phase 1: Test on Staging Database**
```bash
# 1. AI writes migration SQL (save to file)
# Example: supabase/migrations/add_new_column.sql

# 2. HUMAN applies to staging database
# - Open Supabase console (staging project)
# - Go to SQL Editor
# - Paste migration SQL
# - Run manually
# - Verify schema correct in Table Editor

# 3. AI deploys code to staging branch
git checkout staging
git add <migration-related-files>
git commit -m "Add feature that uses new column"
git push origin staging

# 4. Test on staging.tetto.io (both HUMAN and AI)
# - Does site load?
# - Does feature work?
# - Any errors in console?
# - Test for 30+ minutes

# 5. If issues found: Fix and repeat Phase 1
```

**Phase 2: Apply to Production Database**

**⏰ TIMING IS CRITICAL: Right Before Code Merge**

```bash
# 6. Code is ready on staging branch, tested, working
# 7. HUMAN applies migration to production database
#    (Do this IMMEDIATELY before step 8!)
#    - Open Supabase console (production project)
#    - Go to SQL Editor
#    - Paste THE SAME migration SQL
#    - Run manually
#    - Verify schema correct

# 8. IMMEDIATELY merge code to main (AI can do PR)
git checkout main
git merge staging
git push origin main
# This deploys to www.tetto.io + dev.tetto.io

# 9. Verify production IMMEDIATELY (AI + HUMAN)
curl https://www.tetto.io/api/agents | jq '.count'
# - Visit www.tetto.io (does it work?)
# - Check for errors
# - Test the feature that uses new schema

# 10. Monitor for 30 minutes
# - Watch Vercel logs
# - Check error rates
# - Be ready to rollback if issues
```

**Why this order matters:**
- ✅ Code depends on schema (must migrate DB first)
- ✅ Minimal time gap between migration and deploy (seconds, not hours)
- ✅ If migration fails, code isn't deployed yet
- ✅ If code fails, can rollback without schema issues

---

#### Critical Rules

**NEVER:**
- ❌ Apply migration to production first (always staging first!)
- ❌ Let AI run migrations (humans only - too risky!)
- ❌ Deploy code before migration (will break production!)
- ❌ Wait hours between migration and deploy (coordination matters!)
- ❌ Skip staging test (catches schema issues early!)

**ALWAYS:**
- ✅ Test migration on staging database first
- ✅ Test code with new schema on staging.tetto.io
- ✅ Have HUMAN run production migration
- ✅ Deploy code immediately after production migration
- ✅ Monitor production for 30+ minutes after deploy

---

#### Rollback Plan for Migration Issues

**If migration breaks production:**

```bash
# Option 1: Rollback migration (if possible)
# HUMAN runs rollback SQL in Supabase console
# Example: DROP COLUMN, ALTER TABLE to previous state

# Option 2: Rollback code deploy
git revert <merge-commit>
git push origin main
# Reverts to code that works with old schema

# Option 3: Fix forward
# If rollback not possible, fix the migration
# Apply corrective migration
# Redeploy fixed code
```

**Best practice:** Always write rollback SQL before applying migration

**Example:**
```sql
-- Migration: add_new_column.sql
ALTER TABLE agents ADD COLUMN new_field TEXT;

-- Rollback: remove_new_column.sql (write this FIRST!)
ALTER TABLE agents DROP COLUMN new_field;
```

---

#### Example from ENV_SETUP

**What we did:**
1. ✅ Wrote SQL: `ALTER TABLE agents ADD COLUMN network TEXT`
2. ✅ Human applied to staging database
3. ✅ Tested on staging.tetto.io (worked perfectly)
4. ✅ Human applied to production database
5. ✅ Deployed code to main immediately
6. ✅ Verified: 11 agents visible, network='mainnet'
7. ✅ Zero downtime

**Result:** Clean migration, zero issues, production stable

---

### Breaking Changes

**If change breaks existing functionality:**

**1. Feature flags (preferred):**
```typescript
const useNewFlow = process.env.ENABLE_NEW_FLOW === 'true';
if (useNewFlow) {
  // New code
} else {
  // Old code (fallback)
}
```

**2. Parallel deployment:**
- Deploy new endpoint alongside old
- Migrate consumers gradually
- Deprecate old endpoint later

**3. Coordinated deployment:**
- Deploy backend first (backward compatible)
- Then deploy frontend (uses new backend)
- Two separate PRs

---

### Emergency Rollback

**Production is on fire:**

```bash
# FAST: Revert last merge
git checkout main
git log --oneline -5
# Find the bad commit
git revert <bad-commit-hash>
git push origin main
# ✅ Deployed in 60 seconds

# VERIFY:
curl https://www.tetto.io/api/agents | jq '.count'
# Should work again
```

**After rollback:**
- Fix issue properly on feature branch
- Test extensively on staging
- Re-deploy when safe

---

## 📋 Pre-Deploy Checklist

### Before Merging to Staging

- [ ] All commits have clear messages
- [ ] Code builds locally (`npm run build`)
- [ ] Tests pass locally (if exist)
- [ ] No console.errors in browser
- [ ] PR description explains what changed
- [ ] Breaking changes noted (if any)

### Before Merging to Main

- [ ] Feature tested on staging.tetto.io
- [ ] No errors for 30+ minutes on staging
- [ ] Critical path tested (payments, auth, etc.)
- [ ] Database migrations applied (if any)
- [ ] Rollback plan ready (know how to revert)
- [ ] Team aware (if risky change)
- [ ] Monitoring plan (watch for issues)

---

## 🎓 Lessons from ENV_SETUP

### What Worked

**1. Incremental Commits:**
- 15 commits total for full project
- Each commit deployable and testable
- Easy to track what changed when

**2. Test After Each Deploy:**
- Every commit to main was tested immediately
- Caught issues fast (hardcoded URLs found late, but fixable)

**3. Database Migration Safety:**
- Tested on staging first
- Zero production downtime
- No schema issues

**4. Prod-First Approach (Advanced):**
- AI 3 deployed directly to main (with backup)
- Worked because: careful commits, instant testing, easy rollback
- Not recommended for beginners

### What Could Improve

**1. Earlier Comprehensive Search:**
- Hardcoded URL bug found late (during testing phase)
- Should have grepped for hardcoded URLs in CP1
- **Lesson:** Do thorough code searches before starting

**2. Build Testing:**
- Hit Next.js 15 async headers() issue during deploy
- Could have caught with local build test
- **Lesson:** Run `npm run build` before pushing

**3. Assumptions:**
- Assumed frontend used correct API URLs
- Didn't validate until late
- **Lesson:** Validate assumptions early

---

## 🏆 The Ideal Workflow (Summary)

### Daily Development Rhythm

```
Morning:
├─ git checkout main
├─ git pull origin main
├─ git checkout -b feature/new-thing
├─ CODE (1-2 hours)
├─ Commit (logical unit)
├─ CODE (1-2 hours)
├─ Commit (logical unit)
├─ npm run build && npm run test
├─ git push origin feature/new-thing
└─ Create PR to staging

Afternoon:
├─ PR merged to staging
├─ Test on staging.tetto.io (30 min)
├─ Verify works ✅
├─ Create PR: staging → main
├─ PR merged to main
├─ Monitor production (30 min)
├─ Sync staging with main
└─ Feature complete ✅
```

**Total time:** 4-6 hours (2-3 hours coding, 1 hour testing, 1 hour deploy/monitor)

**Commits:** 2-4 (logical chunks)

**Pushes:** 1-2 (after commits, before PR)

**Deploys:** 2 (staging, then main)

---

## ⚠️ Common Mistakes to Avoid

### Git Anti-Patterns

**❌ Don't:**
```bash
git commit -am "updates"  # Vague message
git push --force  # Rewrites history (dangerous!)
git commit --amend  # After pushing (creates divergence)
git merge main  # While on main (should be fast-forward)
```

**✅ Do:**
```bash
git commit -m "Clear, specific message"
git push origin feature-branch  # Normal push
git revert <hash>  # Undo cleanly
git pull --rebase  # Keep history clean (optional)
```

### Deployment Anti-Patterns

**❌ Don't:**
- Deploy on Friday evening (no time to fix issues)
- Deploy all checkpoints at once (can't isolate problems)
- Skip staging (even for "small" changes)
- Deploy without testing locally
- Deploy and walk away (monitor for 30 min)

**✅ Do:**
- Deploy Tuesday-Thursday (time to fix if needed)
- Deploy incrementally (checkpoint by checkpoint)
- Always use staging first
- Test locally before every push
- Monitor after deploy (Vercel logs, error rates)

---

## 📊 Success Metrics

### Good Developer Practices

**You're doing it right if:**
- ✅ Commits are clear and logical
- ✅ Can rollback any single commit without losing other work
- ✅ Staging always close to main (synced regularly)
- ✅ Production stable (no emergency rollbacks)
- ✅ Deployments feel routine (not scary)

**Warning signs:**
- ❌ Git history is messy (100+ commits for simple feature)
- ❌ Can't remember what commit does what
- ❌ Afraid to deploy (commits too large)
- ❌ Frequent rollbacks (need better testing)
- ❌ Staging and main diverged significantly

---

## 🎯 Quick Reference

### Commit Size Guide

| Scenario | Commits | Example |
|----------|---------|---------|
| Small bug fix | 1 commit | Fix typo in error message |
| New API endpoint | 1 commit | Add /api/agents/stats endpoint |
| New UI component + integration | 1 commit | Add EnvironmentBanner to layout |
| Multiple API endpoints | N commits | CP2: 7 endpoints = 7 commits |
| Full feature (large) | 3-5 commits | Add bounty system: DB, API, UI, tests, docs |

### Push Timing

| Scenario | Push When |
|----------|-----------|
| Normal development | After 1-3 commits |
| End of work session | Always push before closing |
| Switching contexts | Push current work first |
| Before asking for review | Push to feature branch |
| Need backup | Push to remote |

### Testing Requirements

| Phase | Tests Required |
|-------|----------------|
| Before commit | Code compiles, basic manual test |
| Before push | Build succeeds, tests pass |
| Before staging merge | Local testing complete |
| Before main merge | Staging tested 30+ min, critical paths verified |

---

## 🔧 Tools & Commands

### Useful Git Commands

```bash
# See what changed
git status
git diff
git log --oneline -10

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo changes to file (DESTRUCTIVE)
git restore <file>

# See commit details
git show <commit-hash>

# Check if staging is synced with main
git log main..staging  # Should be empty if synced

# Sync staging with main
git checkout staging && git merge main && git push origin staging
```

### Useful Vercel Commands

```bash
# Check deployment status (if vercel CLI installed)
vercel ls

# View logs
vercel logs www-tetto-io

# Doesn't have vercel CLI? Use dashboard:
# https://vercel.com/dashboard → project → deployments
```

---

## 🎊 Remember

**Quality > Speed (but both matter)**

**The goal:**
- Fast iteration (don't overthink commits)
- Safe rollback (granular when needed)
- Production stability (test before deploy)
- Clear history (future you will thank you)

**When in doubt:**
- Commit more frequently (can always squash)
- Test more thoroughly (prevents rollbacks)
- Use staging (it's free!)
- Ask for review (fresh eyes catch issues)

---

**Created:** 2025-10-24
**Context:** Post ENV_SETUP implementation
**Proven:** 15 commits, 0 rollbacks, 0 downtime
**For:** AI 3 implementers and all developers on Tetto platform

**MOVE FAST. TEST WELL. DEPLOY SAFELY.** 🚀
