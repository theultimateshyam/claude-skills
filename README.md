# Claude Skills

A growing collection of Claude skills I've built for my own workflows. Accumulated over time across product management, research, content creation, and trust and safety work.

Built for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and [Cowork](https://claude.ai).

## Skills

| Skill | Category | What it does |
| --- | --- | --- |
| **[company-research](./company-research)** | Career | Deep-dive company research for evaluating whether a company is worth joining. Covers products, funding, founders, culture (tech-led vs product-led), social sentiment, employee reviews, and an AI moat risk assessment. |

More coming. Skills I use regularly for PM work, fraud investigation, content writing, and automation will land here over time.

## Installation

### Claude Code

```bash
git clone https://github.com/<your-username>/claude-skills.git

# Copy the skill(s) you want
cp -r claude-skills/company-research ~/.claude/skills/
```

### Cowork

Drop the skill folder into your mounted `.skills/skills/` directory.

### Claude.ai (via Skills UI)

Copy the contents of `SKILL.md` into a new skill in Settings > Skills.

## How Skills Work

Each skill is a folder with a `SKILL.md` file (the main instructions) and an optional `references/` directory for supporting docs that get loaded on demand. No code, no dependencies. Just markdown that teaches Claude specialized workflows.

```
skill-name/
├── SKILL.md              # Main instructions (always loaded)
├── README.md             # What it does, how to use it
└── references/           # Supporting docs (loaded as needed)
    ├── protocol-a.md
    └── protocol-b.md
```

## Status

Work in progress. Skills here are ones I actively use, but they evolve as I find better patterns.

## License

MIT
