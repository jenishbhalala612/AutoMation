# 🚀 AutoBlog Engine

> Automated blog system: AI-powered content creation, SEO optimization, and self-healing deployment.

---

## Quick Start (Windows 11)

### Prerequisites

1. **Install Docker Desktop** (includes WSL2)
   - Download from https://www.docker.com/products/docker-desktop/
   - During install, enable WSL2 backend
   - Restart your PC after installation

2. **Install Git**
   - Download from https://git-scm.com/download/win
   - Use default settings during install

3. **Install Node.js 20 LTS**
   - Download from https://nodejs.org/
   - Choose the LTS version

4. **Install Python 3.11+**
   - Download from https://www.python.org/downloads/
   - ✅ Check "Add Python to PATH" during install

5. **Install Ollama**
   - Download from https://ollama.com/download
   - After install, pull the required models:
   ```powershell
   # Main content generation model
   ollama pull qwen2.5:14b

   # Fast model for keyword research & summaries
   ollama pull qwen2.5:7b
   ```
   *(Note: If you have limited VRAM/RAM, you can set both to `qwen2.5:7b` in `.env`)*

6. **Install Obsidian**
   - Download from https://obsidian.md/
   - Open the `vault/` folder as an Obsidian vault to view and edit content

---

### Setup

```powershell
# 1. Navigate to the project root directory
cd <path-to-Automation>

# 2. Copy environment configuration
copy .env.example .env

# 3. Edit .env with your settings:
#    - Verify VAULT_PATH, WEBSITE_PATH, and DATABASE_PATH match your project directory
#    - Fill in TELEGRAM_BOT_TOKEN and TELEGRAM_CHAT_ID (see guide below)
#    - (Optional) Add PEXELS_API_KEY or UNSPLASH_API_KEY for featured images

# 4. Install website dependencies
cd website
npm install
cd ..

# 5. Set up Python virtual environment and install dependencies
cd pipeline
python -m venv venv
.\venv\Scripts\activate
pip install -r requirements.txt
cd ..
```

---

### How to Run

#### Option A: Run Everything with Docker (Recommended for Production)

Starts SearXNG, Uptime Kuma, Pipeline (in scheduled mode), and the Monitoring dashboard:

```powershell
docker-compose up -d
```

To stop all containers:
```powershell
docker-compose down
```

#### Option B: Run Without Docker (Development Mode)

If running locally without Docker:

1. *(Optional but recommended)* Start SearXNG for web research:
   ```powershell
   docker compose up searxng -d
   ```

2. **Terminal 1: Website Dev Server**
   ```powershell
   cd website
   npm run dev
   ```

3. **Terminal 2: Run the Automation Pipeline**
   ```powershell
   cd pipeline
   .\venv\Scripts\activate
   python main.py
   ```

4. **Terminal 3 (Optional): Pipeline Monitor Dashboard**
   ```powershell
   cd pipeline
   .\venv\Scripts\activate
   python main.py --monitor
   ```

---

### Service Ports & Dashboards

When services are running, access them at:

| Service | URL | Description |
|---|---|---|
| **Blog Website** | `http://localhost:4321` | Local Astro blog preview |
| **Pipeline Monitor** | `http://localhost:5000` | Real-time pipeline status and logs |
| **Uptime Kuma** | `http://localhost:3001` | Service health & uptime dashboard |
| **SearXNG** | `http://localhost:8080` | Self-hosted meta-search engine |

---

### API Keys & Integrations

#### 1. Telegram Bot Notifications
1. Open Telegram and search for `@BotFather`.
2. Send `/newbot` and follow the prompts to name your bot.
3. Copy the token and paste it in `.env` as `TELEGRAM_BOT_TOKEN`.
4. Send a message to your newly created bot in Telegram, then open in your browser:
   `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates`
5. Locate your `chat_id` in the JSON response and set it in `.env` as `TELEGRAM_CHAT_ID`.

#### 2. Stock Photos (Pexels / Unsplash)
To automatically fetch relevant featured images for blog posts:
* **Pexels (Free)**: Register at [pexels.com/api](https://www.pexels.com/api/) and set `PEXELS_API_KEY` in `.env`.
* **Unsplash (Free)**: Register at [unsplash.com/developers](https://unsplash.com/developers) and set `UNSPLASH_API_KEY` in `.env`.

---

### Pipeline CLI Commands

All commands can be run from the `pipeline/` directory (with virtual environment activated):

| Command | What it does |
|---|---|
| `python main.py` | Run the full pipeline once (Research → Keywords → Write → Images → Publish) |
| `python main.py --schedule` | Start the scheduler (runs tasks daily on configured hours) |
| `python main.py --research` | Run trend and topic research only |
| `python main.py --write` | Write drafts for pending topics only |
| `python main.py --publish` | Publish approved posts only |
| `python main.py --resume` | Resume an interrupted or failed pipeline run from the last step |
| `python main.py --search "query"` | Test SearXNG search directly with a query |
| `python main.py --health` | Run health checks on services, Ollama, and disk space |
| `python main.py --summary` | Generate daily/weekly summary report and send to Telegram |
| `python main.py --monitor` | Start the pipeline web dashboard (default port: `5000`) |
| `cd website && npm run dev` | Start Astro website development server |
| `cd website && npm run build` | Build the Astro website for production |

---

### Free Hosting Setup (Cloudflare Pages)

1. Push this repository to GitHub.
2. Go to [dash.cloudflare.com](https://dash.cloudflare.com/) → **Workers & Pages** → **Create application** → **Pages** → **Connect to Git**.
3. Select your repository.
4. Configure Build settings:
   - **Framework preset**: Astro
   - **Build command**: `cd website && npm run build`
   - **Build output directory**: `website/dist`
5. Click **Save and Deploy** — your site will be live at a free `*.pages.dev` domain.

---

### Windows 11 — Running 24/7

To prevent the computer from sleeping during scheduled night jobs:

```powershell
# Disable PC sleep on AC power
powercfg -change -standby-timeout-ac 0
# Set display turn off after 30 minutes
powercfg -change -monitor-timeout-ac 30

# Set Windows Update active hours (Settings → Windows Update → Active hours)
# Set to 06:00 - 02:00 so it won't restart during night automation

# Auto-start Docker Desktop on login:
# Settings → General → Start Docker Desktop when you sign in
```

---

### Project Structure

```
├── website/             # Astro blog (English + Hindi)
│   ├── src/content/blog # Published blog posts (markdown)
│   └── package.json
├── pipeline/            # Python automation engine
│   ├── modules/         # Pipeline modules (writer, researcher, healer, etc.)
│   ├── main.py          # Main CLI entry point
│   ├── monitor.py       # Live dashboard server
│   ├── scheduler.py     # APScheduler background tasks
│   └── requirements.txt
├── vault/               # Obsidian content vault
│   ├── Research/        # Scraped topics and trend data
│   ├── Keywords/        # Target keyword analyses
│   ├── Drafts/          # Draft articles awaiting approval
│   ├── Approved/        # Ready to publish
│   └── Published/       # Published archive
├── searxng/             # SearXNG configuration (settings.yml)
├── openclaw/            # Chat assistant layer
├── docker-compose.yml   # Multi-container orchestration
├── .env.example         # Template configuration
└── README.md            # Documentation
```

---

### Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| **Ollama not responding** | Service not running | Open terminal and run `ollama serve` |
| **Model not found error** | Missing local LLM model | Run `ollama pull qwen2.5:14b` and `ollama pull qwen2.5:7b` |
| **SearXNG connection error** | Container not running | Run `docker compose up searxng -d` or verify `SEARXNG_URL` in `.env` |
| **Build fails** | Syntax or Astro issue | Check `vault/Logs/` for pipeline error traceback |
| **Telegram messages not sending** | Invalid credentials | Verify `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` in `.env` |
| **Images not downloading** | Missing API key | Add a free Pexels or Unsplash API key to `.env` |
| **Path not found errors** | Mismatched absolute paths | Update `VAULT_PATH` and `WEBSITE_PATH` in `.env` to match your local clone |
| **Docker won't start** | Virtualization disabled | Enable WSL2 and hardware virtualization (VT-x/AMD-V) in BIOS |
