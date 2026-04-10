---
name: investigate-signup-spike
description: Investigate abnormal signup volume, user registration spikes, or suspected bot/abuse waves. Produces a detailed report and a one-page summary.
allowed-tools: WebSearch, WebFetch, Bash, Read, Write
---

# Investigate Signup Spike

Use this skill when investigating abnormal signup volume, user registration spikes, or suspected bot/abuse waves. The user may provide a time window, an alert link, or simply describe what they're seeing.

Before starting, ask the user for:
- **Database access method** (tool name or connection details)
- **App/platform name** (for web searches)
- **Time window** to investigate
- **Key table/column names** for users, messages, sessions, and referrals (schemas vary — confirm before querying)

---

## Output

This skill produces **two deliverables**, both saved as markdown files:

### 1. Detailed Report (`signup_spike_investigation_<date>.md`)

A comprehensive investigation covering all phases below. Save to the working directory.

### 2. One-Page Summary (`signup_spike_summary_<date>.md`)

A concise, scannable summary (max ~60 lines). Structure it exactly as:

```
# Signup Spike Summary — <date>

**Window:** <start> - <end> UTC | **Volume:** <N> signups | **Baseline:** ~<N>/hr | **Spike:** ~<N>x

## What Happened
<2-3 sentences: magnitude, who, probable trigger>

## Key Numbers
| Metric | Value |
|--------|-------|
| Total signups | |
| Spike vs baseline | |
| Dominant cohort | |
| Onboarding completed | |
| Active users (sent messages) | |
| Suspected multi-accounts | |

## Cohort Breakdown
<Top 3-5 groups by size with user counts>

## Risk Assessment
- **Bot likelihood:** <None / Low / Medium / High> — <one-line reasoning>
- **Multi-account abuse:** <estimate of clusters and affected users>
- **Referral-driven:** <Yes/No — count>

## Hypothesis: Probable Cause
<2-3 sentences stating the most likely trigger as a hypothesis, not a certainty>

## Recommended Actions
<3-5 bullet points>

## Full Report
See: `signup_spike_investigation_<date>.md`
```

---

## Framing Rules

Follow these rules in BOTH reports:

- **Hypothesis language:** Use "likely", "suggests", "consistent with", "appears to be". Never state unverified conclusions as fact.
- **VPN != fraud:** Only flag VPN usage as suspicious when combined with other signals (shared IP + similar names, multi-account patterns).
- **Shared IP != same person:** Always categorize shared IPs by historical account volume before concluding multi-account abuse. High-traffic VPN exit nodes (20+ total accounts) are likely coincidence.

---

## Investigation Phases

Run phases in order. Each phase informs the next. Parallelize queries within a phase when possible.

---

### Phase 1: Establish the Spike

**Goal:** Confirm the spike, measure magnitude, find the inflection point.

**Data to retrieve:**
- Total signup count within the investigation window
- Hourly signup volume over the past 24h to establish a baseline
- Per-minute signup rate within the window to pinpoint the inflection point

**Record:** spike magnitude (Nx baseline), exact start time, sustained rate (signups/min).

---

### Phase 2: Demographics

**Goal:** Identify geographic concentration, VPN patterns, onboarding completion, profile quality.

**Data to retrieve (run in parallel):**
- Signup count grouped by country
- Signup count grouped by timezone
- Country x timezone cross-tab — mismatches (e.g., country=DE, timezone=Asia/Tehran) indicate VPN usage
- Signup count grouped by onboarding status
- Signup count grouped by email domain

**If a dominant cohort emerges:** run a 24h hourly breakdown filtered to that cohort alongside total signups to see when it started diverging from baseline.

**For users with null country/timezone** (abandoned onboarding): estimate origin using name script patterns (Unicode ranges for Devanagari, Arabic/Persian, Cyrillic, Latin, etc.). Label these as heuristic estimates in the report.

**VPN detection logic:** country is typically sourced from IP geolocation; timezone comes from the browser/OS. A mismatch between the two suggests the user's real location differs from their IP.

---

### Phase 3: Referral Analysis

**Goal:** Determine whether the spike is organic or referral-chain driven. Skip if your platform has no referral system.

**Data to retrieve:**
- Top referrers by number of referred signups within the window
- Referral vs organic breakdown (referred_by null vs not null)

**Key indicators:**
- High referral count = someone is actively incentivizing signups (referral chain or coordinated invite spam)
- Low referral count = externally driven (social media post, viral content, third-party listing)

---

### Phase 4: Usage Patterns — Bot or Human?

**Goal:** Classify users as bots, sign-and-abandon accounts, or genuine users.

**Data to retrieve (run in parallel):**
- Count of users who sent at least one message vs total signups
- Top users by message/session count
- Sample of 30-50 recent messages from new users — eyeball for repetitive, templated, or gibberish content
- Message script/language distribution (especially for users with null country/timezone)

**Bot Indicator Checklist:**

| Signal | Bot | Human |
|--------|-----|-------|
| Message content | Repetitive, templated, gibberish | Diverse topics, multilingual |
| Sessions per user | 1 session | Multiple sessions, different topics |
| Onboarding | Rarely completes | 50%+ complete |
| Language diversity | Single script | Multiple scripts |
| Content quality | Generic, copy-pasted | Contextual, specific questions |

Look specifically for copy-pasted prompts that appear across many users — a strong signal of tutorial-driven or coordinated activity.

---

### Phase 5: Multi-Account Detection

**Goal:** Identify users running multiple accounts.

**Assumes you have a way to map users to IPs, and IP metadata (org, fraud score, network type) from an IP intelligence provider.**

**Data to retrieve — run sequentially:**

1. **Shared IP overview:** Count of IP addresses shared by 2+ new users, and total affected users
2. **Cluster detail:** For each shared IP, retrieve user count, names, emails, IP metadata (org, country, fraud score, network type)
3. **Historical IP volume:** For each shared IP, count total accounts ever seen — used to distinguish real multi-accounts from shared VPN exit nodes
4. **Name duplicates:** Users with identical (normalized) names who signed up in the window
5. **Cluster size distribution:** How many pairs, triples, quads+ exist

**Classify each shared IP by historical volume:**
- 1-5 total accounts ever: high confidence same person
- 6-20 total accounts: uncertain, needs additional signal
- 20+ total accounts: likely shared VPN node, probably coincidence

**Multi-account signal strength (strongest to weakest):**

1. Same name variant + same IP — **definite**
2. Same email stem + same IP — **definite**
3. Residential ISP IP, <=5 total accounts ever — **very likely**
4. VPN IP, <=5 total accounts ever, different names — **probable**
5. Same IP subnet + same name across different IPs — **probable**
6. VPN IP, 6-20 total accounts — **uncertain**
7. VPN IP, 20+ total accounts — **likely coincidence**

For each definite/very likely case, list names, emails, IP, and time between signups.

---

### Phase 6: Source Attribution

**Goal:** Form a hypothesis about what triggered the spike.

**Run web searches** targeting the dominant cohort's likely platforms:

1. `<your_app> site:twitter.com OR site:x.com <language> <month> <year>`
2. `<your_app> telegram <country> <language>`
3. `<your_app> site:youtube.com <language>`
4. `"<your_app>" signup invite <year>`

**Region-specific search tips:**
- **Persian/Iranian:** Search Aparat (`site:aparat.com`), Virgool (`site:virgool.io`), use Persian keywords like `کسب درآمد` (earn money), `رایگان` (free). Persian Telegram channels are NOT web-indexed — recommend manual search.
- **Indian:** Search YouTube, use Hindi keywords
- **Brazilian:** Search YouTube, use Portuguese keywords
- **Nigerian:** Search Twitter/X, YouTube

**Also check:**
- Your app's official social accounts for recent posts
- Your blog/changelog for recent announcements (new features, promotions, partnerships)

State findings as hypotheses. If no definitive source is found, say so and list the most probable candidates.

---

### Phase 7: Write Reports

**Produce both deliverables.**

#### Detailed Report Structure (`signup_spike_investigation_<date>.md`)

1. **Executive Summary** — 2-3 sentences: spike magnitude, probable cause (as hypothesis), bot verdict
2. **Timeline and Volume** — hourly/minute breakdown tables, baseline comparison, inflection point
3. **User Demographics** — geography tables, VPN mismatch analysis, onboarding funnel, name-based estimation (clearly labeled as estimates)
4. **Referral Analysis** — referral vs organic breakdown, top referrers if applicable
5. **Usage Patterns** — activity metrics, message content samples, language distribution, bot assessment checklist
6. **Multi-Account Analysis** — shared IP overview, confidence tiers, definite cases table, cluster size distribution, estimated multi-account rate
7. **Source Attribution** — web search findings, probable trigger hypothesis, what could not be found
8. **Recommendations** — monitoring, abuse prevention, product, growth (4-5 bullets each)

#### One-Page Summary

Follow the template from the Output section. Keep it under 60 lines. Link to the detailed report at the bottom.

**Save both files** to the working directory and inform the user of both file paths.

---

## Tips and Gotchas

- **Confirm schema first:** Table and column names vary significantly across platforms. Always ask or verify before querying — common variations include `registered_at` vs `created_at`, `display_name` vs `name`, `updated_at` vs `modified_at`.
- **Parallelize aggressively:** Within each phase, most data retrieval tasks are independent. Run them in parallel to save time.
- **Read replica lag:** If querying a read replica, data may be slightly behind. Account for this with very recent signups.
- **Large joins may timeout:** Break complex joins into smaller sequential queries if needed.
- **IP data location:** IP-to-user mapping and IP metadata are often in separate tables. Confirm structure before Phase 5.
- **Onboarding nulls:** Users who abandoned onboarding mid-flow will have null country/timezone. Use message content and name scripts to estimate origin.
- **Rate limits:** Log/search tools often have rate limits. If blocked, wait and retry or note the gap in the report.