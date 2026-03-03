---
name: research
description: Search YouTube for videos on a topic, create a NotebookLM notebook, add videos as sources, and generate an initial analysis. Combines yt-dlp search with NotebookLM in one command.
argument-hint: <search query> [--limit N] [--sort newest|popular]
allowed-tools: Bash, Write, Read
---

# YouTube → NotebookLM Research Skill

Search YouTube, create a NotebookLM notebook, add all found videos as sources, and generate an initial analysis — all in one command.

## Argument Parsing

Parse `$ARGUMENTS` for:
- **Search query**: everything that isn't a flag (required)
- **--limit N**: max number of videos (default: 10)
- **--sort newest|popular**: sort order (default: newest)

## Step 1: Search YouTube (yt-dlp)

Run yt-dlp to find videos. The command depends on sort order:

**For `--sort popular`:**
```bash
yt-dlp --dump-json --no-download --playlist-end <LIMIT> "ytsearch<LIMIT>:<QUERY>" 2>/dev/null
```

**For `--sort newest` (default):**
```bash
yt-dlp --dump-json --no-download --playlist-end <LIMIT> "https://www.youtube.com/results?search_query=<URL_ENCODED_QUERY>&sp=CAISBAgDEAE%253D" 2>/dev/null
```

Replace spaces in the query with `+` for URL encoding.

**Important:** This takes ~2-3 seconds per video. Warn the user for large limits.

### Post-process with Python

Write an inline Python script (via bash) that:
1. Reads the JSON lines from stdin or a temp file
2. Extracts: `title`, `url` (from `webpage_url`), `channel`, `upload_date` (convert YYYYMMDD → YYYY-MM-DD), `duration_string`, `view_count`, `like_count`, `comment_count`, `tags`, `categories`, `description`
3. Sorts by `upload_date` desc (newest) or `view_count` desc (popular)
4. Writes formatted JSON array to `output/<sanitized-query>.json`
   - Sanitize: lowercase, replace spaces with hyphens, remove special chars

### Display Summary Table

Show a markdown table: **#**, **Title** (truncated to 60 chars), **Channel**, **Date**, **Views**, **Duration**

Print total videos found and path to JSON file.

## Step 2: Create NotebookLM Notebook & Add Sources

All notebooklm commands require activating the venv first:
```bash
source /home/matt/projects/notebooklm/.venv/bin/activate
```

### Create the notebook
```bash
notebooklm create "<Query> Research" --json
```

Extract the notebook ID from the JSON output (the `id` field).

### Set it as active
```bash
notebooklm use <notebook_id>
```

### Add each video URL as a source

Loop through the URLs from the saved JSON file. For each URL:
```bash
notebooklm source add "<url>" --json
```

Extract the source ID from the JSON output. Report progress to the user as each source is added (e.g., "Added source 3/10: <title>").

**Important:** Sources need time to process. After adding all sources, wait for the last source to finish processing:
```bash
notebooklm source wait <last_source_id> --timeout 120
```

If any source fails to add (e.g., video has no captions), log it and continue with the rest.

## Step 3: Generate Initial Analysis

Once sources are processed, ask NotebookLM for a summary:
```bash
notebooklm ask "What are the key topics and themes discussed across these videos?"
```

Display the full response to the user.

### Wrap-up message

After displaying the analysis, tell the user:
- The notebook name and how many sources were added
- They can ask follow-up questions with: `notebooklm ask "your question"`
- They can view sources with: `notebooklm source list`
- The raw video metadata is saved at `output/<query>.json`

## Error Handling

- If yt-dlp finds no results, stop and tell the user
- If `notebooklm create` fails, stop and suggest running `notebooklm login`
- If a source fails to add, skip it and continue (some videos lack captions)
- If the final `ask` fails, still report success for the notebook creation and suggest the user try manually

## Example

User runs: `/research servicenow scripting --limit 5 --sort newest`

Expected flow:
1. Searches YouTube for "servicenow scripting", newest first, 5 results
2. Saves to `output/servicenow-scripting.json`
3. Creates notebook "Servicenow Scripting Research"
4. Adds 5 video URLs as sources
5. Asks for thematic summary
6. Displays everything to the user
