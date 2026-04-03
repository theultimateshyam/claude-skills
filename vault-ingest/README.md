# Vault Ingest

Autonomously ingest any content into an Obsidian-style knowledge graph vault.

## What it does

Takes URLs, pasted text, or uploaded files and transforms them into a structured knowledge graph:

- **Entry notes** -- one per source, with extracted thoughts, tags, and concept links
- **Concept notes** -- atomic, reusable notes that build graph density over time
- **Map of Content (MOC)** -- hub files that index entries by date and concepts by tag

All notes use `[[wikilinks]]` for bidirectional linking, making the vault navigable in Obsidian or any wikilink-aware editor.

## Trigger phrases

- "ingest this", "add to vault", "add to knowledge graph", "capture this"
- "save this article", "save this to my second brain"
- Paste a URL or text and ask to save/capture/process it
- Upload a document and ask to add it to your vault

## Input types

| Type | What happens |
|------|-------------|
| URL | Fetches page content, extracts title/date/text/links |
| Pasted text | Used directly, asks for title if needed |
| Files (.md, .txt, .pdf, .docx, .html) | Reads and extracts content |
| Multiple sources | Processes each as a separate entry |

## Vault structure

```
vault/
  entries/           # One .md per ingested source
  concepts/          # One .md per concept (atomic notes)
  MOC-Entries.md     # Hub linking all entries
  MOC-Tags.md        # Hub grouping concepts by tag
  _vault_index.json  # State tracker for IDs, tags, concepts
```

## Key principles

- **Factual only** -- extracts what the source says, never interprets or editorializes
- **Reuse over creation** -- checks existing concepts and tags before creating new ones
- **Idempotent** -- won't create duplicates if run twice on the same source
- **Adapts to existing vaults** -- if you already have an Obsidian vault, it matches your conventions
