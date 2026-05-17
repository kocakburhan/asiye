# Vault Daydream Skill

Multi-agent system that mines the Obsidian vault for non-obvious connections between notes, mimicking the brain's default mode network. Samples random note pairs, synthesizes connections, filters with critic.

Inspired by [Gwern's LLM Daydreaming](https://gwern.net/ai-daydreaming).

## Usage

```
/daydream
```

## What it does

1. Uses `asiye-obsidian/` as vault root
2. Scans vault for notes modified in last 120 days
3. Generates 50 recency-weighted random pairs
4. Synthesizes connections (parallel batches)
5. Critiques and scores insights (parallel batches)
6. Filters for quality (average score >= 7.0)
7. Saves insight notes to `Daydreams/` folder
8. Generates daily digest in `Daydreams/digests/`
9. Appends summary to today's daily note

## Output

- **Individual insights**: `Daydreams/YYYYMMDD-slug.md`
- **Daily digest**: `Daydreams/digests/YYYYMMDD-digest.md`
- **Daily note**: Summary appended under `## Daydream`
- **History log**: `ai-research/daydream/history.json`

## Architecture

```
Skill (orchestrator)
  |-- Glob/Read: scan vault, extract excerpts
  |-- Generate 50 random pairs (recency-weighted)
  |-- Task x 10: synthesize connections  <-- parallel
  |-- Task x 10: critique/score insights  <-- parallel
  |-- Filter (avg >= 7.0)
  +-- Write: save insight notes + daily digest
```

No external dependencies -- pure OpenCode tools (glob, read, write, bash, task, question).
