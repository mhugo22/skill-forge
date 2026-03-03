# NotebookLM Research Pipeline

**YouTube search → NotebookLM notebook → AI analysis, all from the terminal.**

Claude Code skills that chain [yt-dlp](https://github.com/yt-dlp/yt-dlp) and [notebooklm-py](https://github.com/seba3y/notebooklm-py) into an automated research workflow. Search a topic, load the videos into NotebookLM as sources, and get an AI-generated analysis — in one command.

## What It Does

1. Searches YouTube for videos on your topic (with date/popularity sorting)
2. Saves structured metadata (title, channel, views, tags, description) to JSON
3. Creates a NotebookLM notebook and adds all videos as sources
4. Generates a thematic summary across all sources
5. Leaves the notebook ready for follow-up questions, mind maps, infographics, etc.

## Skills

### `/research` — Full Pipeline

```bash
/research servicenow cmdb ai --limit 20 --sort newest
```

Runs the entire pipeline: YouTube search → NotebookLM notebook → source ingestion → AI analysis.

**Flags:**
- `--limit N` — Max videos (default: 10)
- `--sort newest|popular` — Sort order (default: newest)

### `/yt-dlp` — YouTube Search Only

```bash
/yt-dlp kubernetes networking --limit 5 --sort popular
```

Standalone YouTube search that saves results to `output/<query>.json`. No NotebookLM involved.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (these are Claude Code skills)
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) (`pip install yt-dlp`)
- [notebooklm-py](https://github.com/seba3y/notebooklm-py) (`pip install notebooklm-py`)
- Authenticated NotebookLM session (`notebooklm login`)

## Setup

```bash
git clone https://github.com/mhugo22/notebooklm.git
cd notebooklm
python3 -m venv .venv
source .venv/bin/activate
pip install yt-dlp notebooklm-py

# Authenticate with NotebookLM (browser-based)
notebooklm login
```

The `.claude/skills/` directory is automatically picked up by Claude Code.

## Project Structure

```
notebooklm/
├── .claude/skills/
│   ├── research/
│   │   └── SKILL.md       # Full pipeline: search → notebook → analysis
│   └── yt-dlp/
│       └── SKILL.md       # YouTube search only
├── output/                 # Saved video metadata (gitignored)
└── .venv/                  # Python environment (gitignored)
```

## Example Output

Running `/research servicenow cmdb ai --limit 10 --sort newest`:

```
| # | Title                                          | Channel              | Date       | Views | Duration |
|---|------------------------------------------------|----------------------|------------|-------|----------|
| 1 | AI Search in ServiceNow | Now Assist for CMDB  | TechTalk with Bill   | 2026-03-01 | 26    | 8:23     |
| 2 | Closing CMDB Data Gaps with Custom AI Agents   | ServiceNow Community | 2026-02-25 | 514   | 32:26    |
| ...                                                                                                        |

Saved 10 videos to output/servicenow-cmdb-ai.json
Created notebook: Servicenow CMDB AI Research (11 sources)
```

Then NotebookLM analyzes all the videos and returns a thematic summary. The notebook stays active for follow-up:

```bash
notebooklm ask "which videos cover CMDB health scoring?"
notebooklm generate mind-map
notebooklm generate infographic "blueprint of key topics"
```

## License

MIT
