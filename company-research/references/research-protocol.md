# Research Protocol

Step-by-step web research protocol for building the company report. Follow these in order.
Each step maps to a section in the output.

## General Rules

- Use web_search for every factual claim. Don't rely on training data for anything time-sensitive.
- Use web_fetch to read full pages when search snippets aren't enough (especially for blog posts, about pages, investor pages).
- Run at least 10-15 searches per company. More for complex/large companies.
- Prefer primary sources (company blog, investor announcements, SEC filings, official press releases) over aggregator sites.
- When sources conflict, note the conflict. Don't silently pick one.
- For every section, if you can't find reliable info, mark as N/A rather than guessing.
- Adapt search queries to the user's specified role (replace "product manager" with the relevant role throughout).

## Step 1: Company Basics

Search queries to run:
- `[company] about`
- `[company] crunchbase funding`
- `[company] series funding round 2025 2026`
- `[company] revenue business model`
- `[company] employee count headcount`

Fetch the company's own About page and Crunchbase profile if available.

## Step 2: Product Landscape

Search queries:
- `[company] products`
- `[company] product launch 2025 2026`
- `[company] changelog updates`
- `[company] pricing` (pricing pages reveal product lines)
- `[company] integrations ecosystem`

Also fetch their main website and any product pages linked from it.
Look for: product announcements on their blog, Product Hunt launches, feature comparison pages.

## Step 3: Funding History

Search queries:
- `[company] crunchbase`
- `[company] funding rounds`
- `[company] series A B C funding`
- `[company] seed round investors`
- `[company] valuation`
- `[company] fundraise 2025 2026`
- `[company] bootstrapped` OR `[company] profitable`
- `[company] investors board members`

Fetch the Crunchbase profile and any TechCrunch/press coverage of funding announcements.
Look for: individual round amounts, lead investors, notable angels, any down-round or bridge signals.
If bootstrapped, look for revenue/profitability claims and how they fund growth.

## Step 4: Founders and Leadership — Deep Dive

This step requires researching the founders and CEO **as people**, not just listing titles.

Search queries:
- `[company] founders`
- `[company] CEO background`
- `"[CEO/founder name]" interview`
- `"[CEO/founder name]" podcast`
- `"[CEO/founder name]" Twitter OR X` (what they post about reveals priorities)
- `"[CEO/founder name]" LinkedIn`
- `"[CEO/founder name]" previous company`
- `[company] co-founder departure` (check for any splits)
- `[company] CTO CPO CRO leadership team`
- `[company] VP [user's role function] head of [user's role function]`

For each founder, build a mini-profile:
- Career trajectory: where they worked before, what they built, how they got here
- Public voice: what topics do they talk about in interviews/social? Tech? Business? Users? Revenue?
- Pattern: consulting/finance background → likely sales-led org. Engineering background → likely eng-led. PM/design background → likely product-led. Domain expert → domain-led.
- Any red flags: controversies, lawsuits, messy company failures, negative press

Also look for podcast/interview appearances — founders reveal their true priorities in long-form conversations far more than in press releases.

## Step 5: Culture Assessment (Tech-Led vs Product-Led vs Sales-Led)

This requires triangulating multiple signals. Search:
- `[company] engineering blog`
- `[company] [user's role] job description`
- `[company] how we build products`
- `[company] [user's role] role` (on job boards)
- `[company] org structure`
- `[company] culture values`
- `site:glassdoor.com [company] [user's role] review`
- `site:reddit.com [company] [user's role] experience`

Cross-reference with founder research from Step 4 — the founder's background is the strongest culture signal.

Read the engineering blog vs product blog ratio. Check if the company talks publicly about product process (discovery, user research, experimentation) or mostly about infrastructure and technical achievements or revenue milestones.

Check job descriptions for the user's role carefully:
- "Write requirements and specs for engineering" = eng-led signal
- "Own the roadmap, drive discovery, define strategy" = product-led signal
- "Work with sales to prioritize customer requests" = sales-led signal

## Step 6: Social Media Pulse (Last 30 Days)

See `sentiment-analysis.md` for the detailed protocol. Key searches:

- `[company] site:twitter.com OR site:x.com` (recent mentions)
- `[company] site:reddit.com` (recent threads)
- `[company] site:news.ycombinator.com` (HN discussions)
- `[company] site:linkedin.com` (employee posts, company posts)
- `[company] news 2026` or `[company] news March 2026` (recent press)
- `[company] layoffs 2026` (check for any workforce changes)
- `[company] controversy` (any recent issues)

## Step 7: Employee Reviews

Search queries:
- `[company] glassdoor reviews`
- `[company] glassdoor rating`
- `[company] blind reviews`
- `[company] levels.fyi reviews`
- `[company] work life balance reviews`
- `[company] [user's role] review glassdoor`
- `site:reddit.com "working at [company]"`
- `site:reddit.com [company] [user's role]`

Look for patterns, not individual reviews. What do 3+ reviews say about the same thing?
Pay special attention to reviews mentioning the user's function, role, or leadership quality.

## Step 8: Strategic Position

Search queries:
- `[company] competitors`
- `[company] vs [competitor]` (for top 2-3 competitors)
- `[company] market share`
- `[company] competitive advantage moat`
- `[company] customers case studies`
- `[company] partnerships`
- `[company] market size TAM`

## Step 9: AI Moat Risk

Search queries:
- `[company] AI strategy`
- `[company] artificial intelligence`
- `[company] product` + think critically about what AI models can now do
- `AI replacing [company's product category]`
- `[company's product category] AI startup`

For this section, combine search results with your own analytical reasoning. Ask:
1. Is the core product primarily a software layer that organizes/presents info? (HIGH risk -- AI can do this)
2. Does the product depend on proprietary data that's hard to replicate? (lowers risk)
3. Does the product have strong network effects? (lowers risk)
4. Is the product in a regulated industry with compliance requirements? (lowers risk)
5. Could an AI agent replace the core workflow the product enables? (raises risk)
6. Is the company already building with AI, or will they be disrupted by it? (critical question)
