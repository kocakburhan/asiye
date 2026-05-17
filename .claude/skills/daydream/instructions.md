# Vault Daydream — Implementation Instructions

You are running the Vault Daydream skill. Follow these steps precisely.

## Step 0: Resolve Vault Root

`VAULT_ROOT` is `asiye-obsidian/` relative to the project root (`D:\Projects\asiye\asiye-obsidian\`).

## Constants

```
VAULT_ROOT = D:\Projects\asiye\asiye-obsidian
DAYDREAMS_DIR = Daydreams
DIGESTS_DIR = Daydreams/digests
HISTORY_FILE = ai-research/daydream/history.json
DAILY_DIR = Daily
EXCLUDE_DIRS = .obsidian, templates, node_modules, Daydreams, .claude, .git
LOOKBACK_DAYS = 120
PAIR_COUNT = 50
BATCH_SIZE = 5
SCORE_THRESHOLD = 7.0
MAX_EXCERPT_WORDS = 500
```

---

## Step 1: Scan Vault

### 1a. Get today's date
```powershell
Get-Date -Format "yyyyMMdd"
```
Store as `TODAY` (e.g., `20260517`).

### 1b. Find recent notes
Use Bash (PowerShell) to list all .md files modified in the last 120 days, excluding certain directories:

```powershell
Get-ChildItem -Path $VAULT_ROOT -Recurse -Filter "*.md" -File |
  Where-Object {
    $_.LastWriteTime -gt (Get-Date).AddDays(-120) -and
    $_.FullName -notmatch '\\\.obsidian\\' -and
    $_.FullName -notmatch '\\templates\\' -and
    $_.FullName -notmatch '\\node_modules\\' -and
    $_.FullName -notmatch '\\Daydreams\\' -and
    $_.FullName -notmatch '\\\.claude\\' -and
    $_.FullName -notmatch '\\\.git\\'
  } |
  Sort-Object LastWriteTime -Descending |
  Select-Object FullName, LastWriteTime
```

Parse into a list of `{path, timestamp}` objects.

### 1c. Load history for dedup
Read `ai-research/daydream/history.json` if it exists (relative to VAULT_ROOT). This contains previously sampled pairs:
```json
{
  "sampled_pairs": [
    {"a": "Note A.md", "b": "Note B.md", "date": "20260213"}
  ]
}
```
If file doesn't exist, treat as empty: `{"sampled_pairs": []}`.

---

## Step 2: Extract Excerpts

For each note found in Step 1 (up to 200 notes — if more, take the 200 most recent):

1. **Read the file** using the `read` tool
2. **Strip YAML frontmatter**: Remove everything between the first `---` and the second `---` (inclusive)
3. **Strip noise**: Remove code blocks, raw URLs, wikilink brackets (keep display text), image embeds (![[...]]), HTML tags
4. **Extract title**: Use the first `# ` heading, or the filename (without .md extension) if no heading
5. **Take first 500 words** of the cleaned text
6. **Compute days_ago**: `(now_timestamp - file_timestamp) / 86400`

Store as array:
```json
{
  "path": "relative/path.md",
  "title": "Note Title",
  "excerpt": "First 500 words...",
  "days_ago": 5
}
```

**Important**: Read in parallel batches using multiple `read` tool calls in a single message.

---

## Step 3: Generate Weighted Random Pairs

### 3a. Assign weights
For each note, assign a sampling weight based on recency:
- **days_ago <= 7**: weight = 3
- **days_ago <= 30**: weight = 2
- **days_ago > 30**: weight = 1

### 3b. Weighted random sampling
Generate 50 unique pairs using weighted random sampling with Python:

```powershell
python -c "
import json, random, sys

notes = json.loads(sys.stdin.read())
history_pairs = set()

# Build weighted pool
pool = []
for i, note in enumerate(notes):
    w = 3 if note['days_ago'] <= 7 else (2 if note['days_ago'] <= 30 else 1)
    pool.extend([i] * w)

pairs = set()
attempts = 0
while len(pairs) < 50 and attempts < 500:
    a = random.choice(pool)
    b = random.choice(pool)
    if a != b:
        pair = (min(a, b), max(a, b))
        pair_key = (notes[pair[0]]['path'], notes[pair[1]]['path'])
        if pair not in pairs and pair_key not in history_pairs:
            pairs.add(pair)
    attempts += 1

result = [{'a': notes[p[0]], 'b': notes[p[1]]} for p in pairs]
print(json.dumps(result))
"
```

Pipe the notes JSON from Step 2 into this script. Incorporate history dedup by loading known pairs from history.json.

### 3c. Output
Store the pairs array. Each pair has:
```json
{
  "a": {"path": "...", "title": "...", "excerpt": "..."},
  "b": {"path": "...", "title": "...", "excerpt": "..."}
}
```

---

## Step 4: Synthesize Connections (Parallel Task Agents)

### 4a. Read synthesizer prompt
Read the file `.claude/skills/daydream/synthesizer-prompt.md`.

### 4b. Batch pairs
Split the pairs into batches of 5. This yields up to 10 batches.

### 4c. Launch Task agents in parallel
For each batch, launch a `task` agent with `subagent_type: general`:

```
task(
  subagent_type: "general",
  description: "Synthesize daydream batch N",
  prompt: "[SYNTHESIZER PROMPT with pairs filled in]"
)
```

**Launch all Task agents in a SINGLE message** (all in parallel).

### 4d. Collect results
Parse the JSON arrays returned by each Task agent. Combine into one flat array of synthesis results. Skip malformed JSON batches.

---

## Step 5: Critique and Score (Parallel Task Agents)

### 5a. Read critic prompt
Read the file `.claude/skills/daydream/critic-prompt.md`.

### 5b. Batch synthesis results
Split synthesis results into batches of 5.

### 5c. Launch Task agents in parallel
For each batch, launch a `task` agent with `subagent_type: general`:

```
task(
  subagent_type: "general",
  description: "Critique daydream batch N",
  prompt: "[CRITIC PROMPT with insights filled in]"
)
```

**Launch all Task agents in a SINGLE message** (all in parallel).

### 5d. Filter results
Combine all critic responses. Keep only entries where `verdict == "accept"` AND `average >= 7.0`.

---

## Step 6: Write Outputs

### 6a. Create directories
```powershell
New-Item -ItemType Directory -Path "$VAULT_ROOT\Daydreams\digests" -Force
```

### 6b. Write individual insight notes
For each accepted insight, create a file:

**Filename**: `Daydreams\{TODAY}-{slug}.md`

Generate `slug` from `suggested_title`: lowercase, replace spaces with hyphens, remove special characters, max 50 chars.

**Content template**:
```markdown
---
created_date: '[[{TODAY}]]'
type: daydream
source_notes:
  - '[[{title_a}]]'
  - '[[{title_b}]]'
scores:
  novelty: {novelty}
  coherence: {coherence}
  usefulness: {usefulness}
  average: {average}
---

# {suggested_title}

> Connection between [[{title_a}]] and [[{title_b}]]

{synthesis}

## Implication

{implication}

## Critic's Note

{reason}
```

Write each file using the `write` tool.

### 6c. Write daily digest
**File**: `Daydreams\digests\{TODAY}-digest.md`

```markdown
---
type: daydream-digest
date: {TODAY}
pairs_sampled: {total_pairs}
insights_accepted: {accepted_count}
---

# Daydream Digest -- {TODAY}

## Stats
- Pairs sampled: {total_pairs}
- Insights generated: {total_synthesized}
- Accepted (avg >= 7.0): {accepted_count}
- Acceptance rate: {acceptance_rate}%

## Top Insights

{For each accepted insight, sorted by average score descending:}

### {rank}. {suggested_title} (avg: {average})
**Connecting**: [[{title_a}]] <-> [[{title_b}]]
{connection}

```

### 6d. Update daily note
Read `Daily\{TODAY}.md`. If it doesn't exist, create it. Check if a `## Daydream` section already exists — if so, **replace** it. If not, append it.

---

## Step 7: Log History

Update `ai-research/daydream/history.json` with all sampled pairs and run metadata.

---

## Step 8: Summary

Output a concise summary to the user:

```
Vault Daydream complete -- {TODAY}

{accepted_count}/{total_pairs} pairs produced insights (acceptance rate: {acceptance_rate}%)

Top insights:
1. {title} (avg: {score}) -- {title_a} <-> {title_b}
2. ...
3. ...

Files: Daydreams/{TODAY}-*.md
Digest: Daydreams/digests/{TODAY}-digest.md
```

---

## Error Handling

- **No notes found**: Tell user "No notes modified in last 120 days."
- **Fewer than 10 notes**: Warn user, proceed with fewer pairs
- **Task agent returns bad JSON**: Skip that batch, continue
- **All insights rejected**: Write digest with 0 accepted, inform user
- **History file corrupt**: Start fresh, warn user
