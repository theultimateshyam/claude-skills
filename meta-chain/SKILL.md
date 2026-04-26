---
name: meta-chain
description: >
  Recursively explains any idea by spawning a chain of zero-context agents,
  each one explaining the previous level's output without access to anything
  else. Produces a depth-configurable "5 whys"-style causal regress that
  bottoms out at first principles. Use this skill whenever the user says
  "meta-chain this", "run meta-chain on", "5 whys on", "go meta on",
  "recursive why", "why why why on", "keep asking why about", "drill into
  the root of", "causal regress on", or "what's underneath this idea".
  Also trigger when the user pastes a sentence and says "go deeper" or "keep
  going" in a way that implies recursive explanation. Depth defaults to 5
  but is configurable — honor any number the user specifies.
---

# Meta-Chain Skill

You are running a recursive causal-regress engine. Each level is a
zero-context agent — it receives only the previous level's text and one
instruction: "Why is this true? Explain. Do not repeat any sentence from
the input." The chain bottoms out when depth is reached or when the agent
starts producing content indistinguishable from the level before it.

---

## Inputs

Collect these before starting. If the user's message already contains them,
do not ask again.

| Input | Required | Default | Notes |
|-------|----------|---------|-------|
| **Seed** | Yes | — | The statement, idea, or claim to recurse on. Can be a sentence, phrase, or paragraph. |
| **Depth** | No | 5 | How many zero-context agents to chain. Accepts any positive integer. Warn above 8 (cost + diminishing returns). |
| **Mode** | No | `causal` | See Modes below. |

If the seed is missing, ask for it. If depth is missing, use 5 silently.

---

## Modes

| Mode | Agent prompt | When to use |
|------|-------------|-------------|
| `causal` (default) | "Why is this true? Explain causally. Do not repeat input sentences." | Root-cause analysis, idea regress, 5-whys |
| `steelman` | "What is the strongest possible argument for why this is true or valuable? Do not repeat input sentences." | Strengthening claims, debate prep |
| `counter` | "What breaks or complicates this? Explain the most substantive objections. Do not repeat input sentences." | Devil's advocate, stress-testing |
| `abstraction` | "What more general principle does this exemplify? Generalize upward. Do not repeat input sentences." | Theory building, pattern extraction |

The user can specify mode explicitly ("meta-chain this in steelman mode") or
implicitly ("stress-test this recursively" → `counter` mode). If ambiguous,
use `causal`.

---

## Execution

### In Claude.ai (browser)

Claude.ai artifacts cannot call the Anthropic API directly (no API key in
browser context, CORS blocks it). Deliver a **React artifact** that runs the
full chain client-side via the `window.claude.complete()` bridge. The
artifact must:

1. Show the seed as Level 0.
2. For each subsequent level, call `window.claude.complete(prompt)` where
   `prompt` is the previous level's full text concatenated with the mode's
   agent instruction. The bridge returns a string with the completion.
3. Display each level as it completes (streaming UX — don't wait for all
   levels before showing any).
4. Show word count and char count per level.
5. Color-code levels with distinct but legible palette (use CSS variables).
6. Show a "Save as HTML" button at the end that downloads the full chain
   with styling preserved.

Use the artifact template at the bottom of this skill.

### In Claude Code / Cowork

Use bash + curl to call the API level by level. Write each level to a
numbered file (`level_0.txt`, `level_1.txt`, ...). After all levels, present
the files. Use `$ANTHROPIC_API_KEY` from the environment.

```bash
call_agent() {
  local content="$1"
  local prompt="$2"
  python3 -c "
import urllib.request, json, sys
payload = json.dumps({
  'model': 'claude-sonnet-4-6',
  'max_tokens': 1000,
  'system': '',
  'messages': [{'role': 'user', 'content': sys.argv[1] + '\n\n---\n' + sys.argv[2]}]
}).encode()
req = urllib.request.Request(
  'https://api.anthropic.com/v1/messages',
  data=payload,
  headers={
    'Content-Type': 'application/json',
    'x-api-key': '$ANTHROPIC_API_KEY',
    'anthropic-version': '2023-06-01'
  }
)
resp = urllib.request.urlopen(req)
data = json.loads(resp.read())
print(data['content'][0]['text'])
" "$content" "$prompt"
}
```

---

## What to tell the user before running

One sentence: what the chain will do, the depth, and the mode. Example:

> Running a 5-level causal regress on your seed. Each agent sees only the
> previous level — no shared context. Starting now.

Do not over-explain. Start building immediately after that line.

---

## Interpreting the output

After the chain completes, add 2–3 sentences of meta-observation. Note:

- Where the language migrated (e.g., "by level 3 we're in evolutionary
  psychology territory")
- Where the chain appeared to bottom out (first principles, tautology, or
  circular reasoning)
- Whether depth was sufficient or the user might want to go deeper / switch
  modes

Do not summarize or paraphrase the levels themselves. The files are the
output; your note is just a pointer.

---

## Depth guidance

| Depth | What happens |
|-------|-------------|
| 1–2 | Surface explanation. Good for quick clarification. |
| 3–4 | Reaches mechanism-level causes. Good for most ideas. |
| 5 (default) | Reaches cognitive / systemic roots. The sweet spot. |
| 6–7 | Philosophical / first-principles territory. Interesting but slow. |
| 8+ | Warn the user. Diminishing returns + cost. Offer to proceed anyway. |

---

## React Artifact Template

Use this as the base for Claude.ai execution. Customize the SEED, DEPTH,
and MODE constants before rendering.

```jsx
import { useState } from "react";

const MODE_PROMPTS = {
  causal: "Why is this true? Explain causally. Do NOT repeat any sentence from the input. Only write new explanatory sentences.",
  steelman: "What is the strongest argument for why this is true or valuable? Do NOT repeat any sentence from the input. Only write new sentences.",
  counter: "What breaks or complicates this? Give the most substantive objections. Do NOT repeat any sentence from the input. Only write new sentences.",
  abstraction: "What more general principle does this exemplify? Generalize upward. Do NOT repeat any sentence from the input. Only write new sentences.",
};

const LEVEL_COLORS = [
  { bg: "#F1EFE8", border: "#B4B2A9", label: "#444441" },
  { bg: "#E6F1FB", border: "#85B7EB", label: "#0C447C" },
  { bg: "#EEEDFE", border: "#AFA9EC", label: "#3C3489" },
  { bg: "#E1F5EE", border: "#5DCAA5", label: "#085041" },
  { bg: "#FAECE7", border: "#F0997B", label: "#712B13" },
  { bg: "#FAEEDA", border: "#EF9F27", label: "#633806" },
  { bg: "#FBEAF0", border: "#ED93B1", label: "#72243E" },
  { bg: "#EAF3DE", border: "#97C459", label: "#27500A" },
];

async function callAgent(content, modePrompt) {
  const prompt = `${content}\n\n---\n${modePrompt}`;
  const result = await window.claude.complete(prompt);
  return result;
}

export default function MetaChain() {
  // CONFIGURE THESE for each invocation:
  const SEED = "YOUR_SEED_HERE";
  const DEPTH = 5;
  const MODE = "causal";

  const [files, setFiles] = useState([{ level: 0, content: SEED }]);
  const [running, setRunning] = useState(false);
  const [currentLevel, setCurrentLevel] = useState(0);
  const [done, setDone] = useState(false);
  const [error, setError] = useState(null);

  async function runChain() {
    setRunning(true);
    setDone(false);
    setError(null);
    let chain = [{ level: 0, content: SEED }];
    setFiles(chain);

    for (let i = 1; i <= DEPTH; i++) {
      setCurrentLevel(i);
      try {
        const result = await callAgent(chain[i - 1].content, MODE_PROMPTS[MODE]);
        chain = [...chain, { level: i, content: result }];
        setFiles([...chain]);
      } catch (e) {
        setError(`Level ${i} failed: ${e.message}`);
        break;
      }
    }
    setRunning(false);
    setDone(true);
    setCurrentLevel(0);
  }

  function downloadHTML() {
    const levelRows = files.map(({ level, content }) => {
      const col = LEVEL_COLORS[level % LEVEL_COLORS.length];
      const label = level === 0 ? "Level 0 — seed" : `Level ${level} — agent ${level}`;
      const words = content.split(/\s+/).length;
      const escaped = content.replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/>/g,"&gt;");
      return `
        <div style="background:${col.bg};border:1px solid ${col.border};border-radius:10px;padding:1rem 1.25rem;margin-bottom:1.25rem;">
          <div style="font-size:11px;font-weight:600;letter-spacing:0.06em;color:${col.label};margin-bottom:8px;text-transform:uppercase;">${label}</div>
          <div style="font-size:15px;color:#1a1a1a;line-height:1.75;white-space:pre-wrap;">${escaped}</div>
          <div style="font-size:11px;color:${col.label};opacity:0.65;margin-top:8px;">${words} words · ${content.length} chars</div>
        </div>`;
    }).join("");

    const html = `<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>meta-chain — ${SEED.slice(0, 60)}</title>
<style>
  body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; max-width: 760px; margin: 3rem auto; padding: 0 1.5rem; background: #fafafa; color: #1a1a1a; }
  h1 { font-size: 20px; font-weight: 500; margin: 0 0 0.25rem; }
  .meta { font-size: 13px; color: #666; margin: 0 0 2rem; }
</style>
</head>
<body>
  <h1>meta-chain</h1>
  <p class="meta">${DEPTH} levels · ${MODE} mode · generated ${new Date().toLocaleString()}</p>
  ${levelRows}
</body>
</html>`;

    const blob = new Blob([html], { type: "text/html" });
    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "meta-chain.html";
    a.click();
  }

  return (
    <div style={{ padding: "1.5rem 0", fontFamily: "var(--font-sans)" }}>
      <h2 style={{ fontSize: 18, fontWeight: 500, color: "var(--color-text-primary)", margin: "0 0 0.25rem" }}>
        meta-chain
      </h2>
      <p style={{ fontSize: 13, color: "var(--color-text-secondary)", margin: "0 0 1.25rem" }}>
        {DEPTH} levels · {MODE} mode · zero-context agents
      </p>

      {!running && !done && (
        <button onClick={runChain} style={{ marginBottom: "1.5rem", padding: "8px 20px", fontSize: 14 }}>
          Run ↗
        </button>
      )}

      {running && (
        <div style={{ marginBottom: "1.25rem", display: "flex", alignItems: "center", gap: 10 }}>
          <div style={{ width: 8, height: 8, borderRadius: "50%", background: "#378ADD", animation: "pulse 1s ease infinite" }} />
          <style>{`@keyframes pulse{0%,100%{opacity:1}50%{opacity:0.25}}`}</style>
          <span style={{ fontSize: 13, color: "var(--color-text-secondary)" }}>Agent {currentLevel} of {DEPTH} thinking...</span>
        </div>
      )}

      {error && (
        <div style={{ background: "#FCEBEB", border: "0.5px solid #F09595", borderRadius: 8, padding: "10px 14px", fontSize: 13, color: "#791F1F", marginBottom: "1rem" }}>
          {error}
        </div>
      )}

      <div style={{ display: "flex", flexDirection: "column", gap: "1rem" }}>
        {files.map(({ level, content }) => {
          const col = LEVEL_COLORS[level % LEVEL_COLORS.length];
          return (
            <div key={level} style={{ background: col.bg, border: `0.5px solid ${col.border}`, borderRadius: 10, padding: "1rem 1.25rem" }}>
              <div style={{ fontSize: 11, fontWeight: 500, letterSpacing: "0.06em", color: col.label, marginBottom: 8, textTransform: "uppercase" }}>
                {level === 0 ? "Level 0 — seed" : `Level ${level} — agent ${level}`}
              </div>
              <div style={{ fontSize: 14, color: "var(--color-text-primary)", lineHeight: 1.75, whiteSpace: "pre-wrap" }}>
                {content}
              </div>
              <div style={{ fontSize: 11, color: col.label, opacity: 0.65, marginTop: 8 }}>
                {content.split(/\s+/).length} words · {content.length} chars
              </div>
            </div>
          );
        })}
      </div>

      {done && (
        <div style={{ marginTop: "1.5rem", display: "flex", gap: 12, alignItems: "center" }}>
          <button onClick={downloadHTML} style={{ padding: "8px 20px", fontSize: 14 }}>
            Save as HTML ↗
          </button>
          <button onClick={() => { setFiles([{ level: 0, content: SEED }]); setDone(false); }} style={{ padding: "8px 20px", fontSize: 14 }}>
            Reset
          </button>
        </div>
      )}
    </div>
  );
}
```

---

## Examples

**Minimal:**
> "meta-chain: attention is the scarcest resource in modern life"

→ depth 5, mode `causal`, runs immediately.

**With depth:**
> "meta-chain depth 3: NFTs failed"

→ depth 3, mode `causal`.

**With mode:**
> "steelman meta-chain on open-source AI, depth 4"

→ depth 4, mode `steelman`.

**On a previous output:**
> "now counter meta-chain the level 3 output"

→ take the level 3 text from the previous chain, run a new chain in
`counter` mode from that seed.
