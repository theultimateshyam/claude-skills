---
name: website-builder
description: >
  Agent-based static website builder. Uses a multi-agent iterative pipeline (Architect, Builders,
  Inspector, QA Lead, Guardian) to build a Next.js static site from existing assets or specs, with
  autonomous planning, parallel execution, critic-driven review, and static export. Only use
  this skill when the user explicitly invokes it by name ("website-builder") or asks for
  "agent-based website building". Do NOT trigger on general web dev questions, simple
  code help, or casual mentions of Next.js. The user must clearly want the full orchestrated
  build pipeline.
---

# Website Builder

Build a production-grade static website from existing design assets, wireframes, or a spec document.
The skill orchestrates a multi-agent pipeline that plans, builds, reviews, and iterates until
the site meets quality gates across functionality, responsiveness, accessibility, and performance.

**Technology:** Next.js (App Router) with static export (`output: 'export'`). TypeScript. Tailwind CSS.
No backend, no API routes, no server actions, no database. Pure static HTML/CSS/JS output.

## Before You Start

Resolve these before entering the pipeline. No phases run until answered.

### 1. What are we building from?

You need at least ONE of these from the user:

1. **Assets folder or repo path** -- A local directory or git repo with existing code, screenshots, or design files
2. **Google Drive link** -- A shared folder with mockups, specs, or assets
3. **Spec file** -- A product spec, PRD, or feature doc (markdown, PDF, Google Doc)
4. **Live URL** -- A deployed website to screenshot and reverse-engineer
5. **Figma/design screenshots** -- Uploaded images of the design to match

If the user provides none of these, tell them:

> "To build the site, I need a starting point. Can you share any of these?
> - A folder or repo with existing UI code or screenshots
> - A Google Drive link with design assets or specs
> - A spec document describing the site's pages and flows
> - A live URL I can inspect
> - Screenshots or Figma exports of the target design
>
> If you're starting from scratch with just an idea, I can help you draft a spec first
> (that's a separate workflow). Come back with that and we'll build from it."

Do not enter the pipeline without a valid input source.

### 2. Understand the source

Study the input thoroughly before entering Phase 1:

- If **repo/folder**: Read the file tree, identify pages/routes, component structure, state management, and navigation. Map every user-facing page.
- If **Drive link**: Fetch and catalog all assets. Identify pages, flows, and design patterns.
- If **spec file**: Extract the page inventory, user flows, content structure, and integration points.
- If **live URL**: Take screenshots of every reachable page. Map the navigation graph. Note interactive elements, forms, modals, animations, and responsive behavior.
- If **design screenshots**: Catalog each screen, identify layout patterns, typography, color palette, spacing system, and component hierarchy.

Produce a **Source Audit**:
```
## Source Audit
- Pages found: [list each page with a one-line description]
- Navigation structure: [header nav, footer links, sidebar, how pages connect]
- Key components: [reusable UI pieces -- hero sections, cards, forms, modals]
- Content dependencies: [static data, JSON files, markdown content, images]
- Assets inventory: [icons, images, fonts, colors, branding tokens]
- Responsive requirements: [mobile, tablet, desktop breakpoints observed]
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
| **QA Lead** | `references/qa-lead.md` | Full E2E validation of the complete site |
| **Guardian** | `references/guardian.md` | Performance, accessibility, code quality, and SEO review |

## Commit Message Convention

All commits across the entire pipeline follow this format:

```
<type>(<scope>): <short description>

<optional body with context>
```

**Types:** `feat`, `fix`, `refactor`, `style`, `chore`, `docs`, `test`
**Scope:** The page name, component, or cross-cutting concern (e.g., `layout`, `theme`, `hero`, `nav`, `footer`, `about-page`)

Examples:
- `feat(home): add hero section with CTA`
- `fix(nav): close mobile menu on route change`
- `refactor(theme): consolidate color tokens into tailwind config`
- `chore(deps): remove unused framer-motion dependency`
- `style(blog): adjust card grid spacing on tablet`

Never include agent names, task IDs, or pipeline metadata in commit messages.
Commits should read like a normal human developer wrote them.

## Untracked Pipeline Files

These files live in the repo root but are **git-ignored**. They are working memory for the
pipeline, not part of the shipped site.

| File | Purpose |
|------|---------|
| `PLAN.md` | Central coordination doc: page inventory, task registry, iteration log |
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
- **Deviate from spec**: "Spec shows a 5-column footer, but on mobile that's unreadable so I collapsed it into an accordion"
- **Fill spec gaps**: "Spec didn't specify a 404 page. Added one consistent with the site's visual language"
- **Make UX improvements**: "Source has a multi-page contact form. Consolidated to a single-page form with validation since there are only 5 fields"
- **Choose between alternatives**: "Could use CSS Grid or Flexbox for the card layout. Chose Grid because the layout has both row and column alignment needs"
- **Add features not in spec**: "Added smooth scroll behavior and scroll-to-top on route changes since it's standard web UX"

Format:
```markdown
# Decisions Log

## DECISION-001: <short title>
- **Phase:** Build Loop / E2E / Hardening
- **Task:** TASK-XXX (if applicable)
- **What was specified:** <what the spec/source said, or "nothing -- gap in spec">
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
mkdir <site-name> && cd <site-name>
git init
```

Scaffold the Next.js project:
```bash
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --no-import-alias
```

Then configure for static export. In `next.config.ts`:
```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: "export",
  images: {
    unoptimized: true,
  },
};

export default nextConfig;
```

**Critical static export rules** (enforce throughout the entire pipeline):
- No `use server` directives anywhere
- No API routes (`app/api/` directory must not exist)
- No server actions
- No `next/headers` or `next/cookies` imports
- No dynamic routes without `generateStaticParams`
- All data is static: JSON files in `src/data/`, hardcoded content, or markdown files
- Images use `<img>` tags or Next.js `<Image>` with `unoptimized: true`
- No middleware (`middleware.ts` must not exist)
- Forms use client-side handling only (mailto links, external form services, or just UI)

Commit the scaffold to `main`, then immediately branch:
```bash
git add -A && git commit -m "chore(init): scaffold next.js project"
git checkout -b build/agent-pipeline
```

All subsequent commits happen on `build/agent-pipeline`. The `main` branch stays clean with
only the initial scaffold until the user reviews and merges.

### Initialize Pipeline Files

Create `PLAN.md`, `DECISIONS.md`, `.build-logs/`, and add them all to `.gitignore`.

PLAN.md initial structure:

```markdown
# Site Build Plan

## Source Audit
<paste from Source Audit>

## Page Inventory
| # | Page | Route | Priority | Status | Assigned To | Iteration |
|---|------|-------|----------|--------|-------------|-----------|
| 1 | ...  | /     | P0       | TODO   | -           | -         |

## Iteration Log
### Iteration 1
- **Goal:** <what this iteration aims to complete>
- **Tasks:** <list of task IDs>
- **Status:** IN_PROGRESS / COMPLETE / BLOCKED

## Task Registry
### TASK-001: <title>
- **Page:** <which page or "shared">
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
add dark mode" or "the pricing page needs a toggle" or just "handle the mobile nav."
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

The Architect never writes site code directly. It only plans, coordinates, and updates the plan.

**Task assignment rules:**
- Each task maps to exactly one page or one cross-cutting concern (layout, navigation, theming, shared components)
- Tasks must have explicit file-level scope (which files to create or modify)
- Tasks must not overlap in files they touch (prevents merge conflicts)
- Priority order: layout shell + nav first, then P0-HUMAN, then P0, then P1, then polish

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

1. **Task completion** -- Does the commit deliver what the acceptance criteria specify?
2. **Build health** -- Does `next build` succeed without errors?
3. **Regression absence** -- Do previously completed pages still work?
4. **Responsive correctness** -- Layout works at mobile (375px), tablet (768px), and desktop (1280px)
5. **Static export compliance** -- No server-only features, no API routes, no dynamic imports that break static export

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
complete, the QA Lead does a holistic walkthrough of the entire site.

The QA Lead checks:

1. **Page coverage** -- Every page from the Source Audit exists and is reachable via navigation
2. **Flow completeness** -- Every user flow works end-to-end (navigation, CTAs, forms, modals)
3. **Missing functionality** -- Features present in the source but absent from the build
4. **Responsive behavior** -- Every page looks correct at mobile, tablet, and desktop widths
5. **Visual parity** -- Look and feel matches the source design

The QA Lead produces a **Gap Report** with new tasks added to PLAN.md. Control returns
to the Architect for another build loop if gaps exist.

**Exit condition:** Zero missing pages, zero broken flows, zero critical visual or responsive issues.

## Phase 4: Hardening (Guardian)

Read `references/guardian.md` before spawning. Only runs after Phase 3 passes.

The Guardian reviews four areas:

1. **Performance** -- Bundle size analysis, image optimization, no unused dependencies, efficient CSS
2. **Accessibility** -- Semantic HTML, ARIA labels, keyboard navigation, color contrast, focus management
3. **Code quality** -- Remove dead code, consolidate duplicates, consistent naming, clean file structure
4. **SEO and meta** -- Page titles, meta descriptions, Open Graph tags, structured data, sitemap, robots.txt

The Guardian creates tasks for failures and hands back to the Architect. All decisions
logged to DECISIONS.md.

**Exit condition:** All four areas report clean.

## Phase 5: Static Build

Once Phase 4 passes, run the production build and static export.

```bash
npm run build
```

This produces a static site in the `out/` directory.

Verify the export:
```bash
npx serve out
```

Open in browser and spot-check key pages. If the build fails, create a fix task and return
to the build loop.

## Phase 6: Build Report

After a successful static build, generate `BUILD_REPORT.md` at the repo root. This is the
final deliverable alongside the static output.

### Report Structure

```markdown
# Build Report

## Summary
- **Site name:** <n>
- **Technology:** Next.js (App Router) + TypeScript + Tailwind CSS, static export
- **Branch:** `build/agent-pipeline` (all work here; `main` has only the initial scaffold)
- **Total iterations:** <N>
- **Total tasks completed:** <N>
- **Total tasks reverted:** <N>
- **Total human-added tasks:** <N>
- **Build duration:** <approximate wall-clock time>

## Review and Merge

All pipeline work is on the `build/agent-pipeline` branch. To review and merge:

\`\`\`
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
\`\`\`

## Static Output
- **Output directory:** `out/`
- **Total size:** <size in MB>
- **Page count:** <N>
- **Build time:** <seconds>

## Deployment

The `out/` directory contains the complete static site. Deploy to any static host:

### Vercel (recommended for Next.js)
\`\`\`
npm i -g vercel
vercel --prod
\`\`\`

### Netlify
\`\`\`
npm i -g netlify-cli
netlify deploy --prod --dir=out
\`\`\`

### Cloudflare Pages
\`\`\`
npx wrangler pages deploy out --project-name=<site-name>
\`\`\`

### GitHub Pages
Push to a repo and enable Pages from Settings > Pages > Source: GitHub Actions.

### Any static host
Upload the contents of the `out/` directory. It's plain HTML/CSS/JS.

## Pages Delivered
| # | Page | Route | Status | Iteration Completed |
|---|------|-------|--------|---------------------|
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
- Performance issues found and fixed: <N>
- Accessibility issues found and fixed: <N>
- Code quality issues found and fixed: <N>
- SEO issues found and fixed: <N>

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
2. **No remote pushes.** Everything stays local. No `git push`, no remote repos, no GitHub API calls. The user handles pushing and deployment after the pipeline finishes.
3. **Untracked pipeline files.** PLAN.md, DECISIONS.md, BUILD_REPORT.md, and `.build-logs/` are git-ignored. Only production code, assets, and config are committed.
4. **PLAN.md is the single source of truth.** Every agent reads it before acting and updates it after.
5. **Commits are atomic.** One task = one commit (or one revert). No mixing tasks.
6. **Revert over patch.** Broken builds get reverted cleanly, not patched on top.
7. **Max 2 retries per task.** After 2 failures, escalate to the Architect for re-scoping.
8. **Human tasks are async.** The human can add tasks at any time, any phase. These get queued as `P0-HUMAN` in PLAN.md and the Architect picks them up on its next pass. No interruption to running work.
9. **Human can say "stop" at any time.** Finish current commits, update PLAN.md, generate BUILD_REPORT.md with current state, and present to user.
10. **Log every autonomous decision.** Any spec deviation or improvement goes in DECISIONS.md with reasoning.
11. **Clean commit messages.** No agent names, task IDs, or pipeline metadata in commits.
12. **Static export is sacred.** Every commit must preserve static exportability. No server-only features ever.

## Pipeline Summary

```
Before You Start: Input source + Source Audit
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
Phase 5: Static Build (next build + verify)
    |
Phase 6: Build Report + DECISIONS.md summary
    |
DONE - present out/ + BUILD_REPORT.md + DECISIONS.md
       user reviews diff, merges build/agent-pipeline to main, deploys
```

Each phase gate requires explicit pass before advancing. The loop between Phase 2/3/4
continues until all exit conditions are met or the user says stop.
