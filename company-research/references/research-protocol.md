# Research Protocol

Step-by-step web research protocol for building the company report. Follow these in order.
Each step maps to a section in the output.

## General Rules

- Use web_search for every factual claim. Don't rely on training data for anything time-sensitive.
- Use web_fetch to read full pages when search snippets aren't enough (especially for blog posts, about pages, investor pages).
- Run at least 8-12 searches per company. More for complex/large companies.
- Prefer primary sources (company blog, investor announcements, SEC filings, official press releases) over aggregator sites.
- When sources conflict, note the conflict. Don't silently pick one.
- For every section, if you can't find reliable info, say "Could not verify" rather than guessing.

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

## Step 3: Founders and Leadership

Search queries:
- `[company] founders`
- `[company] CEO background`
- `[company] CTO CPO leadership team`
- `[company] VP product head of product`
- `"[founder name]" LinkedIn background` (for each founder)

Look for: LinkedIn profiles (can't always access, but search results often have summaries), podcast interviews, conference talks, prior company history.

## Step 4: Culture Assessment (Tech-Led vs Product-Led)

This requires triangulating multiple signals. Search:
- `[company] engineering blog`
- `[company] product management job description`
- `[company] how we build products`
- `[company] product manager role` (on job boards)
- `[company] org structure`
- `[company] culture values`
- `site:glassdoor.com [company] product manager review`
- `site:reddit.com [company] product manager experience`

Read the engineering blog vs product blog ratio. Check if the company talks publicly about product process (discovery, user research, experimentation) or mostly about infrastructure and technical achievements.

Check PM job descriptions carefully:
- "Write requirements and specs for engineering" = eng-led signal
- "Own the roadmap, drive discovery, define strategy" = product-led signal
- "Work with sales to prioritize customer requests" = sales-led signal

## Step 5: Social Media Pulse (Last 30 Days)

See `sentiment-analysis.md` for the detailed protocol. Key searches:

- `[company] site:twitter.com OR site:x.com` (recent mentions)
- `[company] site:reddit.com` (recent threads)
- `[company] site:news.ycombinator.com` (HN discussions)
- `[company] site:linkedin.com` (employee posts, company posts)
- `[company] news 2026` or `[company] news March 2026` (recent press)
- `[company] layoffs 2026` (check for any workforce changes)
- `[company] controversy` (any recent issues)

## Step 6: Employee Reviews

Search queries:
- `[company] glassdoor reviews`
- `[company] glassdoor rating`
- `[company] blind reviews`
- `[company] levels.fyi reviews`
- `[company] work life balance reviews`
- `[company] product manager review glassdoor`
- `site:reddit.com "working at [company]"`

Look for patterns, not individual reviews. What do 3+ reviews say about the same thing?
Pay special attention to reviews mentioning the product org, PM role, or leadership quality.

## Step 7: Strategic Position

Search queries:
- `[company] competitors`
- `[company] vs [competitor]` (for top 2-3 competitors)
- `[company] market share`
- `[company] competitive advantage moat`
- `[company] customers case studies`
- `[company] partnerships`
- `[company] market size TAM`

## Step 8: AI Moat Risk

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
