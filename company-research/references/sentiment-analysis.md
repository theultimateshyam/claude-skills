# Sentiment Analysis Protocol

How to assess social media and public sentiment about a company over the last 30 days.

## Channels to Check

### 1. Twitter / X
Search: `[company] site:x.com` or `[company] site:twitter.com`
Also try: `[company] -from:[company_handle]` to filter out the company's own posts

What to look for:
- User complaints vs praise ratio
- Employee posts (people often announce joining/leaving)
- Product feedback and feature requests
- Response from the company's official account (do they engage?)
- Any viral negative threads

### 2. LinkedIn
Search: `[company] site:linkedin.com`
Also search: `[company] layoffs site:linkedin.com`, `[company] hiring site:linkedin.com`

What to look for:
- Employee departure announcements ("excited to share I'm moving on...")
- Employee celebration posts ("thrilled to join..." -- volume indicates hiring momentum)
- Company page posts and engagement levels
- Founder/leadership thought leadership posts
- Comments on company posts (supportive vs skeptical)

### 3. Reddit
Search: `[company] site:reddit.com`
Check subreddits: r/[company] if it exists, r/cscareerquestions, r/ProductManagement, r/startups, r/ExperiencedDevs

What to look for:
- "Working at [company]" threads
- Product complaints from users
- Employee AMAs or candid takes
- Salary/compensation discussions
- Culture descriptions from insiders

### 4. Hacker News
Search: `[company] site:news.ycombinator.com`
Also try: `[company] Show HN`, `[company] Ask HN`

What to look for:
- How the tech community perceives the company
- Reactions to product launches or funding announcements
- Comments from current/former employees (HN users often self-identify)
- Technical criticism or praise
- "Who is hiring" thread mentions

### 5. News and Press
Search: `[company] news 2026`, `[company] announcement 2026`

What to look for:
- Funding announcements
- Product launches
- Partnerships
- Leadership changes
- Layoffs or restructuring
- Legal issues or controversies

## Sentiment Scoring

After collecting signals, score overall sentiment:

**Strong Positive**: Active hiring, positive product momentum, enthusiastic employee posts, good press, community excitement
**Positive**: Generally favorable mentions, steady growth signals, minor complaints but nothing systemic
**Neutral**: Mixed signals, quiet period, not much being said
**Negative**: Notable complaints, employee departures being discussed, product criticism gaining traction, negative press
**Crisis**: Major controversy, layoff wave, leadership scandal, product failure, legal trouble

## Red Flags to Specifically Watch For

- Multiple senior departures in a short window (especially product/eng leaders)
- Glassdoor rating drop of 0.5+ in recent months
- "Open to work" wave from employees on LinkedIn
- Founder/CEO posting defensive content
- Pivots being announced as "exciting new directions"
- Job postings being pulled (hiring freeze signal)
- Customer churn discussions on social media
- "We're not dead" style blog posts

## Green Flags

- Engineers publicly sharing what they built (culture of openness)
- Product people writing about their process (values product craft)
- Healthy debate on the company's subreddit (engaged community)
- Customers defending the product in public forums
- Consistent hiring across teams (not just one function)
- Leadership sharing transparent updates (revenue milestones, user numbers)
