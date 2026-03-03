# Skill Forge

**A growing collection of Claude Code skills for automating research and tool-chaining workflows.**

Skills are standalone SKILL.md files that teach Claude Code new slash commands. Build them here, symlink them to `~/.claude/skills/`, and they're available in every project.

## Skills

### `/research` — YouTube → NotebookLM Pipeline

```bash
/research servicenow cmdb ai --limit 20 --sort newest
```

Chains [yt-dlp](https://github.com/yt-dlp/yt-dlp) and [notebooklm-py](https://github.com/seba3y/notebooklm-py) into a single command: search YouTube → create a NotebookLM notebook → add videos as sources → generate AI analysis.

**Flags:**
- `--limit N` — Max videos (default: 10)
- `--sort newest|popular` — Sort order (default: newest)

### `/yt-dlp` — YouTube Search Only

```bash
/yt-dlp kubernetes networking --limit 5 --sort popular
```

Standalone YouTube search. Saves structured metadata to `output/<query>.json`.

## Setup

```bash
git clone https://github.com/mhugo22/skill-forge.git
cd skill-forge
python3 -m venv .venv
source .venv/bin/activate
pip install yt-dlp notebooklm-py

# Authenticate with NotebookLM (browser-based)
notebooklm login

# Symlink skills globally so they work from any project
ln -s ~/projects/skill-forge/.claude/skills/* ~/.claude/skills/
```

## How It Works

Skills live in `.claude/skills/<name>/SKILL.md`. Claude Code auto-discovers them as slash commands.

**Local use:** Run `claude` from this directory and skills are available automatically.

**Global use:** Symlink to `~/.claude/skills/` and they're available everywhere:

```
~/.claude/skills/
├── research -> ~/projects/skill-forge/.claude/skills/research
└── yt-dlp -> ~/projects/skill-forge/.claude/skills/yt-dlp
```

## Project Structure

```
skill-forge/
├── .claude/skills/
│   ├── research/
│   │   └── SKILL.md       # YouTube → NotebookLM full pipeline
│   └── yt-dlp/
│       └── SKILL.md       # YouTube search only
├── output/                 # Saved video metadata (gitignored)
└── .venv/                  # Python environment (gitignored)
```

## Adding New Skills

Create a new directory under `.claude/skills/` with a `SKILL.md`:

```
.claude/skills/my-new-skill/
└── SKILL.md
```

Then symlink it globally:

```bash
ln -s ~/projects/skill-forge/.claude/skills/my-new-skill ~/.claude/skills/my-new-skill
```

See the [Claude Code docs](https://docs.anthropic.com/en/docs/claude-code) for the SKILL.md format.

## License

MIT
