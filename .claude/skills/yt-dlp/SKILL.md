---
name: yt-dlp
description: Search YouTube by topic and extract video metadata using yt-dlp. Use when the user wants to search YouTube, scrape video data, or find videos on a topic.
argument-hint: <search query> [--limit N] [--sort newest|popular]
allowed-tools: Bash, Write, Read
---

# YouTube Search & Scrape Skill

Search YouTube for videos on a topic and save structured metadata to a JSON file.

## Argument Parsing

Parse `$ARGUMENTS` for:
- **Search query**: everything that isn't a flag (required)
- **--limit N**: max number of videos to return (default: 20)
- **--sort newest|popular**: sort order (default: newest)

## Step 1: Run yt-dlp

The command depends on the sort order:

**For `--sort popular` (or default relevance):**

```bash
yt-dlp --dump-json --no-download --playlist-end <LIMIT> "ytsearch<LIMIT>:<QUERY>" 2>/dev/null
```

**For `--sort newest`:**

Use YouTube's search URL with the upload date filter parameter (`sp=CAISBAgDEAE%253D` filters to "this month" sorted by upload date):

```bash
yt-dlp --dump-json --no-download --playlist-end <LIMIT> "https://www.youtube.com/results?search_query=<URL_ENCODED_QUERY>&sp=CAISBAgDEAE%253D" 2>/dev/null
```

Replace spaces in the query with `+` for URL encoding.

This outputs one JSON object per line to stdout. Capture all output.

**Important:** This takes ~2-3 seconds per video. For large limits, warn the user it may take a minute.

## Step 2: Post-process with Python

Write a small inline Python script (via bash) that:

1. Reads the JSON lines from stdin or a temp file
2. Extracts these fields from each video object:
   - `title`
   - `url` (use `webpage_url`)
   - `channel`
   - `upload_date` (format: YYYYMMDD — convert to YYYY-MM-DD for readability)
   - `duration_string`
   - `view_count`
   - `like_count`
   - `comment_count`
   - `tags` (list of strings)
   - `categories` (list of strings)
   - `description`
3. Sorts results:
   - If `--sort newest`: sort by `upload_date` descending
   - If `--sort popular`: sort by `view_count` descending
4. Writes the result as a formatted JSON array to `output/<sanitized-query>.json`
   - Sanitize the query for use as a filename (lowercase, replace spaces with hyphens, remove special chars)

## Step 3: Display Summary

Show the user a markdown table with columns: **#**, **Title** (truncated to 60 chars), **Channel**, **Date**, **Views**, **Duration**

At the bottom, print:
- Total videos found
- Path to the saved JSON file
- A note that URLs are ready for NotebookLM

## Example

User runs: `/yt-dlp kubernetes networking --limit 5 --sort popular`

Output:

| # | Title | Channel | Date | Views | Duration |
|---|-------|---------|------|-------|----------|
| 1 | Kubernetes Networking Explained... | TechWorld | 2024-06-15 | 523,400 | 12:34 |
| ... | ... | ... | ... | ... | ... |

Saved 5 videos to `output/kubernetes-networking.json`
