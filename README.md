# Claude Skills

A growing collection of Claude skills I've built for my own workflows. Accumulated over time across product management, research, content creation, and trust and safety work.

Built for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and [Cowork](https://claude.ai).

## Skills

| Skill | Category | What it does |
| --- | --- | --- |
| **[company-research](./company-research)** | Career | Deep-dive company research for evaluating whether a company is worth joining. Covers products, funding, founders, culture (tech-led vs product-led), social sentiment, employee reviews, and an AI moat risk assessment. |
| **[linkedin-posting](./linkedin-posting)** | Content | Generate authentic LinkedIn posts with conversational voice, cross-domain thinking, and genuine vulnerability. Crafts content that builds real professional connections through thoughtful observation. |
| **[meta-chain](./meta-chain)** | Thinking | Recursive causal-regress engine. Spawns a chain of zero-context agents that each explain only the previous level's output, producing a depth-configurable "5 whys"-style drill into first principles. Modes: `causal`, `steelman`, `counter`, `abstraction`. |
| **[mobile-app-builder](./mobile-app-builder)** | Engineering | Agent-based Android app builder. Uses a multi-agent iterative pipeline (Architect, Builders, Inspector, QA Lead, Guardian) to build an Android app from assets or specs, with autonomous planning, parallel execution, and APK bundling. |
| **[vault-ingest](./vault-ingest)** | Knowledge Management | Ingest any content (URLs, text, files) into an Obsidian-style knowledge graph vault. Creates entry notes, atomic concept notes, and Map of Content updates with bidirectional wikilinks. Adapts to existing Obsidian vaults. |

More coming. Skills I use regularly for PM work, fraud investigation, and automation will land here over time.

## Installation

### Claude Code

```bash
git clone https://github.com/theultimateshyam/claude-skills.git

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
