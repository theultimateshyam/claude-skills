---
name: mobile-app-builder
description: >
  Agent-based Android app builder. Uses a multi-agent iterative pipeline (Architect, Builders,
  Inspector, QA Lead, Guardian) to build an Android app from existing assets or specs, with
  autonomous planning, parallel execution, critic-driven review, and APK bundling. Only use
  this skill when the user explicitly invokes it by name ("mobile-app-builder") or asks for
  "agent-based Android app building". Do NOT trigger on general mobile dev questions, simple
  code help, or casual mentions of Android. The user must clearly want the full orchestrated
  build pipeline.
---

# Mobile App Builder

Build a production-grade Android app from existing web UI, design assets, or a spec document.
The skill orchestrates a multi-agent pipeline that plans, builds, reviews, and iterates until
the app meets quality gates across functionality, UI correctness, security, and bundle optimization.

## Before You Start

Resolve these two things before entering the pipeline. No phases run until both are answered.

### 1. What are we building from?

You need at least ONE of these from the user:

1. **Assets folder or repo path** - A local directory or git repo with the existing web app code, screenshots, or design files
2. **Google Drive link** - A shared folder with mockups, specs, or assets
3. **Spec file** - A product spec, PRD, or feature doc (markdown, PDF, Google Doc)
4. **Live URL** - A deployed web app to screenshot and reverse-engineer

If the user provides none of these, tell them:

> "To build the app, I need a starting point. Can you share any of these?
> - A folder or repo with the existing UI code or screenshots
> - A Google Drive link with design assets or specs
> - A spec document describing the app's screens and flows
> - A live URL I can inspect
>
> If you're starting from scratch with just an idea, I can help you draft a spec first
> (that's a separate workflow). Come back with that and we'll build from it."

Do not enter the pipeline without a valid input source.

### 2. What technology?

Ask the user:

- **Does this app also need to run on web or iOS in the future?**
  - Yes, or unsure -> **React Native** (Expo managed workflow, generates Android APK)
  - No, Android only -> **Kotlin** (Jetpack Compose, native Android)

Default to Kotlin for pure Android. Use React Native only if cross-platform is needed.

### 3. Understand the source

Once you have both answers, study the input thoroughly before entering Phase 1:

- If **repo/folder**: Read the file tree, identify screens/routes, component structure, state management, API calls, and navigation flow. Map every user-facing screen.
- If **Drive link**: Fetch and catalog all assets. Identify screens, flows, and design patterns.
- If **spec file**: Extract the screen inventory, user flows, data models, and integration points.
- If **live URL**: Take screenshots of every reachable screen. Map the navigation graph. Note interactive elements, forms, modals, and transitions.

Produce a **Source Audit**:
```
## Source Audit
- Screens found: [list each screen with a one-line description]
- Navigation flow: [how screens connect]
- Key components: [reusable UI pieces]
- Data/API dependencies: [what data each screen needs]
- Assets inventory: [icons, images, fonts, colors, branding]
- Gaps/ambiguities: [anything unclear, ask the user]
```

Present the Source Audit to the user for confirmation before entering Phase 1.

## Agent Roster

The pipeline uses five specialized agent roles. Read the corresponding reference file in
`references/` before spawning each agent.

| Role | Reference File | Responsibility |
|------|---------------|----------------|
| **Architect** | `references/architect.md` | Plans iterations, splits tasks, coordinates agents, maintains PLAN.md |
| **Builder** | `references/builder.md` | Implements a single task, commits code |
| **Inspector** | `references/inspector.md` | Reviews each commit against five quality dimensions |
| **QA Lead** | `references/qa-lead.md` | Full E2E validation of the complete app |
| **Guardian** | `references/guardian.md` | Security, code quality, bundle optimization, architecture review |

## Commit Message Convention

All commits across the entire pipeline follow this format:

```
<type>(<scope>): <short description>

<optional body with context>
```

**Types:** `feat`, `fix`, `refactor`, `style`, `chore`, `docs`, `test`
**Scope:** The screen name, module, or cross-cutting concern (e.g., `navigation`, `theme`, `auth-screen`, `home`)

Examples:
- `feat(home): add pull-to-refresh on feed list`
- `fix(navigation): prevent back press from exiting app on main tab`
- `refactor(theme): consolidate color tokens into single palette`
- `chore(deps): remove unused image-picker dependency`
- `style(profile): align avatar and username vertically`

Never include agent names, task IDs, or pipeline metadata in commit messages.
Commits should read like a normal human developer wrote them.

## Untracked Pipeline Files

These files live in the repo root but are **git-ignored**. They are working memory for the
pipeline, not part of the shipped app.

| File | Purpose |
|------|---------|
| `PLAN.md` | Central coordination doc: screen inventory, task registry, iteration log |
| `DECISIONS.md` | Log of autonomous decisions, spec deviations, and improvements made by agents |
| `BUILD_REPORT.md` | Final summary generated at the end of the pipeline |
| `.build-logs/` | Directory for per-agent logs collected during the run |

Add all four to `.gitignore` at project setup:
```
PLAN.md
DECISIONS.md
BUILD_REPORT.md
.build-logs/
```

## The Decisions Log

`DECISIONS.md` is one of the most important outputs of this pipeline. It captures every place
where agents exercised judgment beyond what the spec or source material explicitly specified.

Agents write to DECISIONS.md whenever they:
- **Deviate from spec**: "Spec says bottom nav has 4 tabs, but Settings only has 2 items so I merged it into Profile as a section instead"
- **Fill spec gaps**: "Spec didn't specify error states for the feed. Added a retry-with-illustration pattern consistent with the rest of the app"
- **Make UX improvements**: "Source app has a 3-step flow for editing profile. Consolidated to inline editing since mobile patterns favor it"
- **Choose between alternatives**: "Could use either ViewPager2 or HorizontalPager from Compose. Chose HorizontalPager because the project is Compose-first"
- **Add features not in spec**: "Added haptic feedback on pull-to-refresh since it's standard Android UX"

Format:
```markdown
# Decisions Log

## DECISION-001: <short title>
- **Phase:** Build Loop / E2E / Hardening
- **Task:** TASK-XXX (if applicable)
- **What was specified:** <what the spec/source said, or "nothing - gap in spec">
- **What was done instead:** <the actual decision>
- **Reasoning:** <why this is better or necessary>
- **Reversible:** Yes / No
- **Impact:** Low / Medium / High

## DECISION-002: ...
```

This file serves two purposes:
1. The human can review autonomous decisions after the build, understand the reasoning, and revert anything they disagree with
2. It becomes input for improving the spec or source material for future builds

## Phase 1: Project Setup

### Repository

Create a new **local** git repo. All pipeline work happens on a feature branch, never on `main`.
The user merges to `main` after reviewing the final output.

```bash
mkdir <app-name> && cd <app-name>
git init
```

For React Native (Expo):
```bash
npx create-expo-app@latest . --template blank-typescript
```

For Kotlin (Jetpack Compose): scaffold a standard Android project with Gradle, Material3, and
Compose Navigation.

Commit the scaffold to `main`, then immediately branch:
```bash
git add -A && git commit -m "chore(init): scaffold project"
git checkout -b build/agent-pipeline
```

All subsequent commits happen on `build/agent-pipeline`. The `main` branch stays clean with
only the initial scaffold until the user reviews and merges.

### Initialize Pipeline Files

Create `PLAN.md`, `DECISIONS.md`, `.build-logs/`, and add them all to `.gitignore`.

PLAN.md initial structure:

```markdown
# App Build Plan

## Source Audit
<paste from Source Audit>

## Screen Inventory
| # | Screen | Priority | Status | Assigned To | Iteration |
|---|--------|----------|--------|-------------|-----------|
| 1 | ...    | P0       | TODO   | -           | -         |

## Iteration Log
### Iteration 1
- **Goal:** <what this iteration aims to complete>
- **Tasks:** <list of task IDs>
- **Status:** IN_PROGRESS / COMPLETE / BLOCKED

## Task Registry
### TASK-001: <title>
- **Screen:** <which screen>
- **Description:** <what to build>
- **Acceptance criteria:** <specific, testable conditions>
- **Status:** TODO / IN_PROGRESS / REVIEW / DONE / REVERTED
- **Agent:** <assigned Builder ID>
- **Commits:** <list of commit hashes>
- **Inspector notes:** <feedback from review>
```

## Phase 2: The Build Loop

This is the core orchestration engine. It runs iteratively until all quality gates pass.

### Human Tasks

The human can add tasks at any time, during any phase, in any form. They might say "also
add dark mode" or "the onboarding needs a skip button" or just "handle offline state."
This should never interrupt or pause anything that's already running.

How it works:
1. **Acknowledge** the human's input briefly and confirm understanding
2. **Append** new task entries to PLAN.md's Task Registry with status `TODO` and priority `P0-HUMAN`
3. The Architect picks these up on its next planning pass, like any other task in the backlog
4. **Note** in DECISIONS.md that these tasks came from the human (not derived from the spec)

No special ceremony. The pipeline keeps running. The Architect sees `P0-HUMAN` tasks in the
registry and schedules them into the current or next iteration based on scope and dependencies.
If the human's request conflicts with something already built, the Architect creates both a
task for the new work and a note in DECISIONS.md about the conflict.

### Role: Architect

Read `references/architect.md` before spawning. The Architect:

1. **Before every planning round**, reads `git diff` and `git log --oneline` to understand exactly what changed since the last iteration. This is mandatory, not optional. The Architect cannot plan without knowing the current state of the codebase.
2. Reads `PLAN.md` to cross-reference task status with actual commits
3. Breaks the current iteration goal into parallel tasks (max 4 concurrent)
4. Assigns tasks to Builder agents with clear scope boundaries
5. Updates `PLAN.md` after each iteration cycle

The Architect never writes app code directly. It only plans, coordinates, and updates the plan.

**Task assignment rules:**
- Each task maps to exactly one screen or one cross-cutting concern (navigation, theming, state)
- Tasks must have explicit file-level scope (which files to create or modify)
- Tasks must not overlap in files they touch (prevents merge conflicts)
- Priority order: navigation shell first, then P0-HUMAN, then P0, then P1, then polish

### Role: Builder Agents (Parallel)

Read `references/builder.md` before spawning. Spawn up to 4 Builders per iteration. Each:

1. Receives a single task from the Architect
2. Implements the task in the codebase
3. Logs any spec deviations or autonomous decisions to `DECISIONS.md`
4. Commits with a conventional message (see Commit Message Convention above)
5. Does NOT create log files or meta-commentary in the repo

### Role: Inspector

Read `references/inspector.md` before spawning. After Builders commit, the Inspector reviews
each commit across five dimensions:

1. **Task completion** - Does the commit deliver what the acceptance criteria specify?
2. **Build health** - Does the project compile without errors?
3. **Regression absence** - Do previously completed screens still work?
4. **UI correctness** - No broken layouts, overlapping views, hidden content, or untappable elements
5. **Flow integrity** - Navigation works, back button behaves, transitions are smooth

**Inspector verdict matrix:**

| Verdict | Action |
|---------|--------|
| PASS | Mark task DONE in PLAN.md |
| FAIL (fixable) | Add specific feedback, reassign to Builder |
| FAIL (broken) | `git revert <commit>`, mark REVERTED, reassign with notes |

### The Inner Loop

```
Architect reads git diff + git log (mandatory), then plans iteration N
    |
    v
Builders execute tasks in parallel (up to 4)
    |
    v
Each Builder commits
    |
    v
Inspector reviews each commit
    |
    +---> PASS -> mark DONE
    +---> FAIL -> revert or reassign, Builder retries (max 2 retries per task)
    |
    v
Architect reads git diff + git log (mandatory), picks up any new human tasks, plans iteration N+1
```

Repeat until the Architect determines all tasks for the current milestone are DONE.

## Phase 3: E2E Validation (QA Lead)

Read `references/qa-lead.md` before spawning. Once the Architect declares all iterations
complete, the QA Lead does a holistic walkthrough of the entire app.

The QA Lead checks:

1. **Screen coverage** - Every screen from the Source Audit exists and is reachable
2. **Flow completeness** - Every user flow works end-to-end
3. **Missing functionality** - Features present in the source but absent from the build
4. **State management** - Data persists across navigation, back-button works
5. **Visual parity** - Look and feel matches the source app

The QA Lead produces a **Gap Report** with new tasks added to PLAN.md. Control returns
to the Architect for another build loop if gaps exist.

**Exit condition:** Zero missing screens, zero broken flows, zero critical visual issues.

## Phase 4: Hardening (Guardian)

Read `references/guardian.md` before spawning. Only runs after Phase 3 passes.

The Guardian reviews four areas:

1. **Security** - No hardcoded secrets, HTTPS everywhere, secure storage for sensitive data, permissions match usage, input validation
2. **Code simplification** - Remove dead code, consolidate duplicates, simplify nesting, consistent naming
3. **APK bundle optimization** - Enable R8/Proguard, tree-shake deps, compress images (prefer WebP), target under 30MB
4. **Architecture** - Navigation follows Android conventions, state management is predictable, file structure follows community standards, accessibility basics covered

The Guardian creates tasks for failures and hands back to the Architect. All decisions
logged to DECISIONS.md.

**Exit condition:** All four areas report clean.

## Phase 5: APK Build

Once Phase 4 passes, build the release APK.

### React Native
```bash
cd android && ./gradlew assembleRelease
```
APK location: `android/app/build/outputs/apk/release/app-release.apk`

### Kotlin
```bash
./gradlew assembleRelease
```
APK location: `app/build/outputs/apk/release/app-release.apk`

If the build fails, create a fix task and return to the build loop.

## Phase 6: Build Report

After a successful APK build, generate `BUILD_REPORT.md` at the repo root. This is the
final deliverable alongside the APK itself.

### Report Structure

```markdown
# Build Report

## Summary
- **App name:** <name>
- **Technology:** React Native / Kotlin (Jetpack Compose)
- **Branch:** `build/agent-pipeline` (all work here; `main` has only the initial scaffold)
- **Total iterations:** <N>
- **Total tasks completed:** <N>
- **Total tasks reverted:** <N>
- **Total human-added tasks:** <N>
- **Build duration:** <approximate wall-clock time>

## Review and Merge

All pipeline work is on the `build/agent-pipeline` branch. To review and merge:

```
# See everything the pipeline built
git log main..build/agent-pipeline --oneline

# Review the full diff against the initial scaffold
git diff main..build/agent-pipeline

# When satisfied, merge to main
git checkout main
git merge build/agent-pipeline

# If pushing to a remote
git remote add origin <your-repo-url>
git push -u origin main
```

## APK Details
- **File path:** <absolute path to APK>
- **File size:** <size in MB>
- **Min SDK:** <version>
- **Target SDK:** <version>
- **Permissions:** <list of declared permissions>

## Install on Device

### Prerequisites
1. Enable **Developer Options** on your Android phone:
   Settings > About Phone > tap "Build Number" 7 times
2. Enable **USB Debugging**:
   Settings > Developer Options > USB Debugging > ON
3. Connect your phone via USB cable
4. Set USB connection mode to **File Transfer (MTP)** -- not PTP, not charge-only.
   Most phones show a notification when you plug in. Tap it and select "File Transfer / MTP."
   ADB generally works over MTP. PTP (Picture Transfer Protocol) is for cameras and does not
   reliably support ADB on most devices.
5. Install ADB on your computer if not already present:
   - macOS: `brew install android-platform-tools`
   - Linux: `sudo apt install adb`
   - Windows: download from https://developer.android.com/tools/releases/platform-tools

### Install Command
\`\`\`
adb install <absolute-path-to-apk>
\`\`\`

### Verify Connection
\`\`\`
adb devices
\`\`\`
Should show your device serial with "device" status. If it shows "unauthorized", unlock
your phone and accept the USB debugging permission dialog.

### Troubleshooting
- **No devices listed:** try a different USB cable (must be data-capable, not charge-only),
  re-enable USB Debugging, or toggle USB mode
- **INSTALL_FAILED_UPDATE_INCOMPATIBLE:** uninstall existing app first:
  `adb uninstall <package.name>`
- **Wireless debugging (Android 11+):** `adb pair <ip>:<port>` then `adb connect <ip>:<port>`

## Screens Delivered
| # | Screen | Status | Iteration Completed |
|---|--------|--------|---------------------|
<table from PLAN.md>

## Decisions Summary
<count> autonomous decisions were made during this build.
See DECISIONS.md for the full log. Key highlights:
<top 5 highest-impact decisions, one line each>

## Agent Activity Log

### Architect
- Iterations planned: <N>
- Tasks created: <N>
- Re-scopes after failure: <N>

### Builders
- Total commits: <N>
- Commits accepted: <N>
- Commits reverted: <N>

### Inspector
- Reviews performed: <N>
- Pass rate: <X%>
- Most common failure type: <category>

### QA Lead
- E2E rounds: <N>
- Gaps found (first round): <N>
- Final gaps: 0

### Guardian
- Security issues found and fixed: <N>
- Code quality issues found and fixed: <N>
- Bundle size: <before> -> <after>
- Architecture issues found and fixed: <N>

## Detailed Agent Logs
Full per-agent logs are available in `.build-logs/`:
- `.build-logs/architect.log`
- `.build-logs/builder-1.log` through `.build-logs/builder-N.log`
- `.build-logs/inspector.log`
- `.build-logs/qa-lead.log`
- `.build-logs/guardian.log`
```

Present BUILD_REPORT.md and DECISIONS.md to the user at the end.

## Agent Logging

Each agent appends to its log file in `.build-logs/` during execution. These are not
committed to git. They capture:

- What the agent read before acting (which files, which sections of PLAN.md)
- What decisions it made and why
- What actions it took (files created, modified, commands run)
- Duration and outcome

Format each entry with a timestamp:
```
[2025-03-27T14:22:01] ARCHITECT: Reading PLAN.md, iteration 3
[2025-03-27T14:22:03] ARCHITECT: 2 tasks remain from iteration 2 (TASK-007 REVERTED, TASK-009 IN_PROGRESS)
[2025-03-27T14:22:05] ARCHITECT: Planning iteration 3 with 3 new tasks + 1 rework
```

## Orchestration Rules

1. **Work on a branch, not main.** All pipeline work happens on `build/agent-pipeline`. The `main` branch is only touched for the initial scaffold commit. The user merges to `main` after reviewing the final output.
2. **No remote pushes.** Everything stays local. No `git push`, no remote repos, no GitHub API calls. The user handles pushing and PR creation after the pipeline finishes.
2. **Untracked pipeline files.** PLAN.md, DECISIONS.md, BUILD_REPORT.md, and `.build-logs/` are git-ignored. Only production code, assets, and config are committed.
3. **PLAN.md is the single source of truth.** Every agent reads it before acting and updates it after.
4. **Commits are atomic.** One task = one commit (or one revert). No mixing tasks.
5. **Revert over patch.** Broken builds get reverted cleanly, not patched on top.
6. **Max 2 retries per task.** After 2 failures, escalate to the Architect for re-scoping.
7. **Human tasks are async.** The human can add tasks at any time, any phase. These get queued as `P0-HUMAN` in PLAN.md and the Architect picks them up on its next pass. No interruption to running work.
8. **Human can say "stop" at any time.** Finish current commits, update PLAN.md, generate BUILD_REPORT.md with current state, and present to user.
9. **Log every autonomous decision.** Any spec deviation or improvement goes in DECISIONS.md with reasoning.
10. **Clean commit messages.** No agent names, task IDs, or pipeline metadata in commits.

## Pipeline Summary

```
Before You Start: Input source + tech choice + Source Audit
    |
Phase 1: Project Setup (scaffold on main, then branch to build/agent-pipeline)
    |
Phase 2: Build Loop (Architect -> Builders -> Inspector) [repeats]
    |       [Human can queue tasks at any time -- Architect picks them up naturally]
    |
Phase 3: E2E Validation (QA Lead) [may send back to Phase 2]
    |
Phase 4: Hardening (Guardian) [may send back to Phase 2]
    |
Phase 5: APK Build
    |
Phase 6: Build Report + DECISIONS.md summary
    |
DONE - present APK + BUILD_REPORT.md + DECISIONS.md
       user reviews diff, merges build/agent-pipeline to main
```

Each phase gate requires explicit pass before advancing. The loop between Phase 2/3/4
continues until all exit conditions are met or the user says stop.
