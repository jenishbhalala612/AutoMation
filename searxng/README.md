# SearXNG Setup for AutoBlog Engine

SearXNG is a **self-hosted, privacy-respecting** meta search engine that the
AutoBlog pipeline uses to:

| Feature | How SearXNG is Used |
|---------|---------------------|
| **Trend Discovery** | Searches AI/tech news & web for trending topics |
| **Keyword Research** | Autocomplete + search suggestions + title mining |
| **Health Check** | Verifies the service is running every 15 min |

---

## Quick Start (Docker — Recommended)

Make sure **Docker Desktop** is installed and running, then:

```bash
# From project root
cd c:\Jenish\demo\Automation

# Start SearXNG (and all other services)
docker-compose up -d searxng

# Or start everything at once
docker-compose up -d
```

SearXNG will be available at: **http://localhost:8080**

---

## Verify It Works

### 1. Open the UI
Visit **http://localhost:8080** in your browser and search for anything.

### 2. Test from the Pipeline
```bash
cd pipeline

# Test SearXNG with the default query
python main.py --search

# Test with a custom query
python main.py --search "ChatGPT vs Claude 2026"
```

Expected output:
```
SearXNG URL: http://localhost:8080
✅ SearXNG is online!

Searching: best AI tools 2026 (general)
┌─────────────────────────────────┐
│  Web Results: best AI tools 2026│
├───┬─────────────────────────────┤
│ 1 │ Top 10 AI Tools...          │
│ 2 │ Best Free AI Assistants...  │
...
```

---

## Without Docker (Alternative)

If Docker is not available, you can run SearXNG with Python directly:

```bash
pip install searx
searx-run
```

Or use the **pre-built Windows binary** from:
https://github.com/searxng/searxng/releases

---

## Configuration

The pipeline uses these settings (all in `.env`):

```env
SEARXNG_URL="http://localhost:8080"
```

The custom `searxng/settings.yml` file:
- ✅ Enables **JSON API format** (required for pipeline)
- ✅ Configures Google, Bing, DuckDuckGo, HackerNews, Reddit
- ✅ Enables news and IT categories

---

## How It Integrates

```
pipeline/
├── modules/
│   ├── searxng_client.py   ← New: SearXNG API client
│   ├── trend_scanner.py    ← Updated: uses SearXNG as source #1
│   └── keyword_researcher.py ← Updated: full web search for keywords
```

### Data Flow

```
SearXNG (http://localhost:8080)
    │
    ├── /search?format=json&categories=news  → TrendScanner._scan_searxng()
    ├── /search?format=json&categories=general → KeywordResearcher keywords
    └── /autocompleter?q=...                → Keyword suggestions
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `SearXNG not available` | Run `docker-compose up -d searxng` |
| Port 8080 already in use | Change port in `docker-compose.yml` and `SEARXNG_URL` in `.env` |
| No results from Google | SearXNG may be rate-limited; wait a few minutes |
| `format=json not allowed` | The `settings.yml` is not being loaded — check the volume mount |

---

## Useful Docker Commands

```bash
# Check if running
docker ps | grep searxng

# View logs
docker logs autoblog-searxng

# Restart
docker-compose restart searxng

# Stop
docker-compose stop searxng
```
