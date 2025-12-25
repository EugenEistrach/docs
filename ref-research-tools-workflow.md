# Research Tools Workflow

## Quick Reference

| Need | Tool | Notes |
|------|------|-------|
| Official docs | `ref_search` → `ref_read_url` | Always first |
| Code examples (concept) | `get_code_context_exa` | Semantic search |
| Real patterns (literal) | `grep:searchGitHub` | Production code |
| Any GitHub file | `ref_read_url` | Works on ANY repo, even new ones |
| StackOverflow | `web_search` → `crawling_exa` | ref blocked |
| Community sentiment | `web_search` | Quotes only |
| Comprehensive research | `.research` | Multi-source synthesis |

## Tool Rules (Absolute)

**GitHub files:** ALWAYS `ref_read_url`, NEVER `crawling_exa` (returns page chrome)

**StackOverflow:** ALWAYS `crawling_exa`, NEVER `ref_read_url` (blocked)

**Reddit:** NO direct access from Claude tools. Options:
- `web_search` for quotes/summaries (usually sufficient)
- User runs curl locally: `curl "https://reddit.com/r/sub/comments/id/title.json"`
- User uses PRAW (Python, 100 req/min free, official API)

## When to Use `.research`

**USE for:**
- Multi-faceted questions needing 5+ sources
- Comparisons ("X vs Y for use case Z")
- Would require 10+ manual tool calls
- Synthesis needed, not just lookup

**DON'T USE for:**
- Simple fact lookups
- Known documentation exists
- Specific file content needed
- Quick answers

## Reddit Workaround

Claude cannot read Reddit directly. Two options:

### Option 1: .json endpoint (quick, no setup)
```bash
curl "https://www.reddit.com/r/nextjs/hot.json?limit=10" | jq
curl "https://www.reddit.com/r/nextjs/comments/abc123/title.json" | jq
```

### Option 2: PRAW (proper, 100 req/min free)
```bash
pip install praw
```

1. Register app: https://old.reddit.com/prefs/apps (5 min)
2. Create "script" type app
3. Use:

```python
import praw

reddit = praw.Reddit(
    client_id="YOUR_ID",
    client_secret="YOUR_SECRET",
    user_agent="research/1.0"
)

# Get thread
post = reddit.submission(url="https://reddit.com/r/nextjs/comments/...")
print(post.title)
print(post.selftext)

for comment in post.comments.list():
    print(comment.body)
```

This is the official API. Free tier = 100 requests/minute. More than enough for research.
