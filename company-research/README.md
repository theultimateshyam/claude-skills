# Company Research Skill

A Claude skill that produces a structured intel report on any company, designed to help product managers make informed decisions about whether to join.

## What It Does

Say "research [company name]" and get an 8-section report covering:

1. **Company Snapshot** -- funding, stage, revenue model, headcount
2. **Product Landscape** -- every product, target user, maturity, recent launches
3. **Founders and Leadership** -- backgrounds, domain expertise, org signals
4. **Culture Signal: Tech-Led vs Product-Led** -- the make-or-break section for PMs. Assesses where the company sits on the eng-led / product-led / sales-led spectrum with evidence
5. **Social Pulse (Last 30 Days)** -- Twitter/X, LinkedIn, Reddit, HN sentiment. Red and green flags
6. **Employee Reviews** -- Glassdoor, Blind, PM-specific signals
7. **Strategic Position** -- competitors, moat, market momentum
8. **AI Moat Risk Assessment** -- how likely are advances in model capabilities to erode this company's core value proposition in the next 2-3 years

## Usage

```
"Research Notion for me"
"Should I join Linear? I'm looking at a PM role"
"Company deep dive on Figma"
"What's Amplitude like as a workplace?"
```

## Structure

```
company-research/
├── SKILL.md                              # Main skill definition
└── references/
    ├── research-protocol.md              # Step-by-step search queries and research process
    └── sentiment-analysis.md             # Social media analysis protocol with scoring rubric
```

## Planned Additions

- [ ] Role-specific variant (Engineering version with different culture assessment lens)
- [ ] `config.yaml` support for personal background so the report can include a fit assessment
- [ ] Interview question suggestions based on findings
