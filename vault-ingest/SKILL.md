---
name: vault-ingest
description: >
  Autonomously ingest any content into an Obsidian-style knowledge graph vault.
  Accepts URLs, pasted text, uploaded files (.md, .txt, .pdf, .html, .docx).
  Outputs entry notes, atomic concept notes, and MOC updates written directly into the vault.
  Use this skill when the user says "ingest this", "add to vault", "add to knowledge graph",
  "capture this", "save this article", "save this to my second brain", "process this into my vault",
  "add this to obsidian", or pastes a URL/article text and asks to save, capture, or process it,
  or uploads a document and asks to add it to their knowledge graph.
---

# Vault Ingest -- Obsidian Knowledge Graph Builder

Autonomously ingest any content into an Obsidian-style knowledge graph vault.

**Accepts:** URLs, pasted text, uploaded files (.md, .txt, .pdf, .html, .docx)
**Outputs:** Entry notes, atomic concept notes, MOC updates -- written directly into the vault.

---

## Step 0: Clarify the ingestion request

**Before doing any work**, ask the user what they need. Adapt based on what they already provided.

### If the user provided content (URL, text, file):

Ask how they want it processed:
- **Quick capture** -- extract key thoughts and links, auto-tag
- **Deep capture** -- extract thoughts, links, and create detailed concept notes with related connections
- **Bulk mode** -- multiple sources, process them all sequentially
- **Custom** -- user picks the tags and concepts themselves

### If the user said "ingest" or "capture" but gave no content:

Ask what they want to add:
- A URL / web article
- Text / notes (pasted or typed)
- A file (.md, .txt, .pdf, .docx)
- An original idea or thought
- Multiple sources (batch)

### If no vault exists yet (no `_vault_index.json` found):

Ask first:
- Create a new vault in the current working directory
- Let the user specify a folder
- Point to an existing Obsidian vault

Then continue with the content question.

### If the user chose "Custom":

Present extracted thoughts, suggested tags, and suggested concepts before writing any files. Let them approve, modify, or reject each.

---

## Step 1: Detect or bootstrap the vault

1. **Search for an existing vault index.** Look for `_vault_index.json` in the working directory and common locations. This file tracks vault state: last entry number, existing concepts, tags, folder paths.

2. **If found:** Read it. Proceed to Step 2.

3. **If NOT found and user chose to create a new vault:**
   Bootstrap the vault structure:

```
{vault_root}/
  entries/         -- one .md file per ingested source
  concepts/        -- one .md file per concept (atomic notes)
  MOC-Entries.md   -- hub linking all entries
  MOC-Tags.md      -- hub grouping concepts by category tag
  _vault_index.json
```

Create the initial `_vault_index.json`:
```json
{
  "vault_path": "{absolute path}",
  "last_entry_number": 0,
  "entry_count": 0,
  "concept_count": 0,
  "existing_tags": [],
  "existing_concepts": [],
  "existing_entries": [],
  "entry_prefix": "Entry"
}
```

Create empty MOC files with headers:
- `MOC-Entries.md`: `# Map of Content -- Entries`
- `MOC-Tags.md`: `# Map of Content -- Tags`

4. **If user pointed to an existing Obsidian vault:**
   - Read a few existing files to understand conventions (folder names, frontmatter fields, link style, naming patterns)
   - Adapt output format to match their conventions instead of imposing this template
   - Create `_vault_index.json` for state tracking
   - Confirm the adapted structure with the user before writing

---

## Step 2: Extract source content

| Input | Action |
|-------|--------|
| **URL** | Fetch the page content. Extract: title, date, main text, links. Strip nav/ads/boilerplate. |
| **Pasted text** | Use directly. Ask for a title if not obvious from the content. |
| **Uploaded file** | Read it. For PDFs, extract text (use the pdf skill if available). For .docx, use the docx skill if available. For .md/.txt/.html, read directly. |
| **Multiple sources** | Process each as a separate entry with sequential IDs. |

---

## Step 3: Analyze and tag

### Extract thoughts / key ideas
- Break the content into 2-5 distinct ideas or takeaways.
- Each thought should be a self-contained insight.
- **Keep the author's original voice and words where possible.**
- **FACTUAL ONLY. Never add, interpret, editorialize, or hallucinate. Only extract what is explicitly stated.**

### Assign category tags
- Pick 1-3 tags per thought from the vault's existing tag vocabulary (from `_vault_index.json`).
- If the vault is new or the vocabulary is empty, start building it organically.
- Use lowercase, hyphenated tags (e.g., `#product-design`, `#behavioral-economics`).
- Add new tags ONLY if no existing tag fits. Reuse aggressively to build graph density.

### Extract concepts
- Identify 2-5 concepts per thought.
- **Check `existing_concepts` from the index first. Reuse exact names to link to existing nodes.** This is critical for graph connectivity.
- Use Title Case for new concepts.
- Prefer specific over generic (e.g., "Baader-Meinhof Effect" not "Psychology Bias").

### Extract links
- Any URLs mentioned in or related to the content, with descriptive text.

---

## Step 4: Determine entry ID

- Read `last_entry_number` from `_vault_index.json`
- New entry ID = `last_entry_number + 1`
- For multiple sources, increment sequentially
- If the vault uses a different naming convention (check `entry_prefix` field), match it

---

## Step 5: Write entry file

Write to `{vault_path}/entries/{prefix}-{ID}.md`:

```markdown
---
entry: {ID}
title: "{title}"
date: {YYYY-MM-DD}
source_type: {article|book-note|podcast-note|video-note|idea|newsletter|tweet-thread}
source_url: "{url if applicable}"
tags: [{comma-separated tags}]
concepts:
  - "{Concept 1}"
  - "{Concept 2}"
---

# {prefix} {ID}
**{title}** | {date}

## Thoughts

### [{ID}.a]
#{tag1} #{tag2}  [[Concept 1]] [[Concept 2]]

{Thought text -- factual, author's voice preserved.}

### [{ID}.b]
#{tag1} #{tag2}  [[Concept 3]] [[Concept 4]]

{Second thought.}

## Links

- [{link text}]({url}) | #{tag} [[Concept]]

## Source

{source_type}: [{title}]({url}) (accessed {date})

---

Previous: [[{prefix}-{prev}]] | Next: [[{prefix}-{next}]]
[[MOC-Entries]] | [[MOC-Tags]]
```

---

## Step 6: Write concept files (NEW concepts only)

For each concept NOT already in `existing_concepts`, write to `{vault_path}/concepts/{Concept Name}.md`:

```markdown
---
type: concept
tags: [{relevant tags}]
entries: [{entry ID}]
---

# {Concept Name}

## Appears in

- [[{prefix}-{ID}]] (thought {a/b/etc})

## Related

- [[{Co-occurring concept from same entry}]]

---
[[MOC-Tags]] | [[MOC-Entries]]
```

**Sanitize filenames:** Remove characters: `< > : " / \ | ? *`

---

## Step 7: Update EXISTING concept files

For concepts already in the vault that appear in the new content:
- Read the existing concept file
- Add the new entry to `## Appears in`
- Add the entry ID to the `entries:` frontmatter list
- Do NOT duplicate entries that are already listed

---

## Step 8: Append to MOC files

**MOC-Entries.md** -- append:
```markdown
### [[{prefix}-{ID}]] | {date}
{First 80 chars of first thought}...
Concepts: [[Concept 1]], [[Concept 2]], ...
```

**MOC-Tags.md** -- for each tag used:
- Find the `## #{tag}` section (create it if this is a new tag)
- Add `[[{prefix}-{ID}]]` under "Entries:"
- Add any new concepts under "Concepts:"

---

## Step 9: Update vault index

Update `_vault_index.json`:
- Set `last_entry_number` to the new highest ID
- Increment `entry_count`
- Add new entries to `existing_entries`
- Add new concepts to `existing_concepts` (and increment `concept_count`)
- Add any new tags to `existing_tags`

---

## Step 10: Confirm to user

Report back:
- Entry ID and title created
- Number of new concepts added vs. existing concepts linked
- Which tags were used (new vs. reused)
- Sample `[[wikilinks]]` showing connections to existing vault content
- If a new vault was bootstrapped, briefly explain the folder structure

---

## Rules

1. **ASK FIRST.** Always clarify what the user wants before starting work.
2. **FACTUAL ONLY.** Never add, interpret, or hallucinate content. Only extract what is explicitly in the source.
3. **REUSE BEFORE CREATING.** Always check existing concepts and tags before making new ones. Graph density comes from reuse.
4. **KEEP THE AUTHOR'S VOICE.** When extracting thoughts, use the source's own words and phrasing.
5. **ONE CONCEPT = ONE FILE.** Never merge multiple concepts into one note.
6. **BIDIRECTIONAL LINKING.** Entry files link to concepts via `[[wikilinks]]`. Concept files link back to entries. MOCs link to both.
7. **IDEMPOTENT.** Running the skill twice on the same source must not create duplicates. Check `existing_entries` for the source URL or title before writing.
8. **RESPECT EXISTING VAULTS.** If the user has an existing Obsidian vault with different conventions, adapt to their structure rather than imposing this template.
