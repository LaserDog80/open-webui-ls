# LLM Philosopher Chat — Deployment & Rebrand Plan

## DPICK: Fork & Rebrand Open WebUI for Internal Team Use

-----

## D — Discover

### What We Have

- A private fork of Open WebUI (latest) on GitHub
- An existing LLM Philosopher dialogue app (Python/LangChain/Streamlit) that generates curated Socrates vs Confucius dialogues
- An OpenAI-compatible API key that colleagues will use to access models
- A team of under 50 users (internal use only)

### What We Need

- A ChatGPT-style chat interface branded as “LLM Philosopher” (or similar) so colleagues have a familiar, polished chatbot experience alongside the dialogue generation tool
- Pre-configured with our API key and a curated set of models so it works out of the box
- A sensible default system prompt tailored to the team’s TV production context
- Easy for non-technical colleagues to use — no setup on their end

### Key Findings

- **Licensing is clear.** Deployments of 50 or fewer users can fully rebrand Open WebUI on the latest version. No enterprise licence needed. Internal only, private repo — no issues.
- **Branding is done via three routes:** the `WEBUI_NAME` environment variable sets the app name; a custom CSS file injected into the Docker image handles colours/styling; and favicon/logo files can be replaced directly in the build at `/app/build/static/`.
- **OpenAI-compatible endpoint config** uses `OPENAI_API_BASE_URL` and `OPENAI_API_KEY` environment variables. Multiple endpoints can be semicolon-separated via `OPENAI_API_BASE_URLS` and `OPENAI_API_KEYS`.
- **Model curation** is handled via the Admin Panel after first launch — you can restrict which models appear in the dropdown.
- **The Dockerfile approach** (custom CSS + favicon replacement) ensures branding survives container rebuilds.

### Decision: Why Not Build From Scratch?

Open WebUI already provides conversation history, streaming responses, model switching, system prompts, markdown rendering, mobile-responsive layout, and admin controls. Building even a basic version of this in Streamlit or Gradio would take weeks and still feel worse than what Open WebUI gives you on day one. The right move is to stand on someone else’s shoulders.

-----

## P — Plan

### Phase 1: Environment & Docker Setup (1 hour)

Create a `.env` file in the repo root with the following:

```env
# === BRANDING ===
WEBUI_NAME=LLM Philosopher Chat

# === API CONNECTION ===
# Your OpenAI-compatible endpoint (e.g. OpenAI, Ollama, LM Studio, vLLM)
OPENAI_API_BASE_URL=https://api.openai.com/v1
OPENAI_API_KEY=sk-your-key-here

# If using multiple providers, semicolon-separate them:
# OPENAI_API_BASE_URLS=https://api.openai.com/v1;http://localhost:11434/v1
# OPENAI_API_KEYS=sk-your-openai-key;not-needed

# === SECURITY ===
WEBUI_SECRET_KEY=generate-a-strong-random-key-here
ENABLE_SIGNUP=false
DEFAULT_USER_ROLE=user

# === OPTIONAL: Disable Ollama if not using it ===
ENABLE_OLLAMA_API=false
```

Create or update `docker-compose.yml`:

```yaml
services:
  llm-philosopher-chat:
    build: .
    container_name: llm-philosopher-chat
    ports:
      - "3000:8080"
    env_file:
      - .env
    volumes:
      - llm-philosopher-data:/app/backend/data
    restart: unless-stopped

volumes:
  llm-philosopher-data:
```

**Note:** Using `build: .` instead of the stock image so our Dockerfile with custom branding assets gets used.

### Phase 2: Visual Rebrand (2–3 hours)

#### 2a. Dockerfile for Branding

Create a `Dockerfile` in the repo root:

```dockerfile
FROM ghcr.io/open-webui/open-webui:main

# Replace favicon and logo
COPY branding/favicon.svg /app/build/static/favicon.svg
COPY branding/favicon.png /app/build/static/favicon.png
COPY branding/favicon.ico /app/build/static/favicon.ico

# Inject custom CSS
COPY branding/custom.css /app/build/static/custom.css
```

#### 2b. Create Branding Assets

Create a `branding/` directory in the repo root containing:

|File         |Purpose                  |Notes                                 |
|-------------|-------------------------|--------------------------------------|
|`favicon.svg`|Browser tab icon (vector)|LLM Philosopher logo, keep it simple  |
|`favicon.png`|Browser tab icon (raster)|32x32 or 64x64 PNG                    |
|`favicon.ico`|Legacy browser support   |Can generate from PNG                 |
|`custom.css` |Theme overrides          |Colours, fonts, hide unwanted elements|

#### 2c. Custom CSS Approach

The `custom.css` file should cover:

- **Colour scheme** — Override CSS variables to match LLM Philosopher / your brand palette
- **Hide “Open WebUI” text** — Use CSS selectors to replace or hide default branding text where `WEBUI_NAME` doesn’t reach
- **Optional: Hide features you don’t need** — e.g. if you don’t want image generation or RAG document upload visible to colleagues, hide those UI elements with `display: none`

Start minimal. You can always refine later.

#### 2d. Logo in the UI

The sidebar/header logo can be customised by replacing the relevant SVG file in the build output. Since you’ve forked the repo, you can also edit the Svelte source directly if needed — but the Dockerfile COPY approach is simpler and doesn’t require rebuilding the frontend from source.

### Phase 3: Pre-Configuration (1 hour)

After first launch with `docker compose up`:

1. **Create the admin account** — First user to sign up gets admin rights. Use your own email/password.
1. **Create colleague accounts** — Since `ENABLE_SIGNUP=false`, you’ll create accounts for colleagues manually via Admin Panel > Users.
1. **Curate the model list** — Go to Admin Panel > Settings > Models. Hide or disable any models you don’t want colleagues to see. Only show the models you’ve selected.
1. **Set a default system prompt** — Go to Admin Panel > Settings > Interface. Set a default system prompt, something like:

> You are a helpful AI assistant for a TV production team. You help with research, script development, brainstorming, factual queries, and general production tasks. Be concise, practical, and clear. If you’re unsure about something, say so rather than guessing.

1. **Set default model** — Use `DEFAULT_MODELS` env var or the Admin Panel to set which model colleagues see first when they open a new chat.

### Phase 4: Documentation (30 mins)

Create a simple `README.md` (or a one-page PDF) for colleagues covering:

- **What is this?** — “LLM Philosopher Chat is our internal AI assistant. Think of it as a private ChatGPT.”
- **How to access it** — URL, e.g. `http://your-server:3000`
- **How to log in** — Credentials you’ve set up for them
- **How to use it** — Pick a model from the dropdown, type a message, press Enter. Mention the system prompt is already set.
- **What NOT to do** — Don’t share API keys, don’t paste confidential client contracts, etc. (your call on data policy)
- **Who to contact** — You, if something breaks

### Phase 5: Testing Checklist (30 mins)

Before handing to colleagues:

- [ ] Docker container starts cleanly with `docker compose up -d`
- [ ] Navigating to `http://localhost:3000` shows “LLM Philosopher Chat” branding, not “Open WebUI”
- [ ] Favicon is the custom one, not the Open WebUI default
- [ ] Custom CSS colours are applied
- [ ] Admin can log in and access settings
- [ ] Colleague account can log in (non-admin)
- [ ] Model dropdown shows only curated models
- [ ] Sending a message returns a streamed response
- [ ] Conversation history persists between sessions
- [ ] System prompt is applied (test by asking “What are you?” — should reference TV production context)
- [ ] Colleague cannot sign up without admin creating their account
- [ ] App survives `docker compose down && docker compose up -d` (data persists in volume)

-----

## I — Implement (Build Order)

1. **Set up `.env`** with API key, branding name, security settings
1. **Create `branding/` directory** with favicon files and `custom.css`
1. **Create `Dockerfile`** that layers branding onto the stock image
1. **Create/update `docker-compose.yml`**
1. **Build and launch** — `docker compose up -d --build`
1. **First-run admin setup** — Create admin account, colleague accounts, curate models, set system prompt
1. **Run through the testing checklist**
1. **Write the colleague README**
1. **Push everything to private repo** (excluding `.env` with real keys — add to `.gitignore`)

-----

## C — Check

### It’s Working When…

- Colleagues open the URL and see a branded, ChatGPT-like interface
- They can chat immediately without any setup
- The model dropdown shows only what you’ve chosen
- Conversations persist and feel responsive (streaming)
- Nobody sees “Open WebUI” anywhere in the interface

### Known Gotchas

- **`WEBUI_NAME` doesn’t update everywhere.** Some UI strings are hardcoded in the Svelte frontend. Custom CSS can hide/replace these, or you can edit the source files in your fork. Start with CSS — only touch source if something really bothers you.
- **Admin settings override `.env` after first run.** Open WebUI uses “PersistentConfig” — once you save a setting via the Admin Panel, it’s stored in the database and takes precedence over the `.env` value. If you need to force a reset, use `RESET_CONFIG_ON_START=true` in `.env` for one restart.
- **Docker image size.** The Open WebUI image is ~5GB. First pull takes a few minutes. Subsequent builds using the `FROM` layer cache are fast.
- **API key security.** The `.env` file contains your API key. Add it to `.gitignore`. For production, consider Docker secrets or a vault.

-----

## K — Knowledge

### Architecture (for your reference, not colleagues)

```
┌─────────────────────────────────────────┐
│          LLM Philosopher Chat           │
│         (Open WebUI, rebranded)         │
│                                         │
│  ┌─────────┐  ┌──────────┐  ┌────────┐ │
│  │ Chat UI │  │  Admin   │  │  User  │ │
│  │(Svelte) │  │  Panel   │  │ Mgmt   │ │
│  └────┬────┘  └────┬─────┘  └───┬────┘ │
│       │            │             │       │
│       └────────────┼─────────────┘       │
│                    │                     │
│            ┌───────▼────────┐            │
│            │  Python Backend │            │
│            │   (FastAPI)     │            │
│            └───────┬────────┘            │
│                    │                     │
└────────────────────┼─────────────────────┘
                     │
            ┌────────▼─────────┐
            │  OpenAI-Compatible│
            │     Endpoint      │
            │  (OpenAI / Ollama │
            │   / LM Studio /  │
            │     vLLM etc.)   │
            └──────────────────┘
```

### Key Environment Variables Reference

|Variable               |What It Does                                                                       |
|-----------------------|-----------------------------------------------------------------------------------|
|`WEBUI_NAME`           |Sets the app name shown in the UI                                                  |
|`OPENAI_API_BASE_URL`  |Your OpenAI-compatible endpoint URL                                                |
|`OPENAI_API_KEY`       |API key for the endpoint                                                           |
|`ENABLE_SIGNUP`        |`false` = only admin can create accounts                                           |
|`DEFAULT_USER_ROLE`    |`user` or `pending` (pending requires admin approval)                              |
|`ENABLE_OLLAMA_API`    |`false` if you’re only using OpenAI-compatible endpoints                           |
|`WEBUI_SECRET_KEY`     |Session encryption key — generate a strong random string                           |
|`RESET_CONFIG_ON_START`|`true` to force `.env` values to override database settings (use once, then remove)|

### File Structure (What You’re Adding to the Fork)

```
your-fork/
├── .env                    # API keys, branding name (gitignored)
├── .env.example            # Template without real keys (committed)
├── .gitignore              # Must include .env
├── Dockerfile              # Layers branding onto stock image
├── docker-compose.yml      # Container config
├── branding/
│   ├── custom.css          # Theme overrides
│   ├── favicon.svg         # Vector icon
│   ├── favicon.png         # Raster icon
│   └── favicon.ico         # Legacy icon
└── README.md               # Colleague-facing docs
```

### Future Options (Not Now, But Worth Knowing)

- **Multiple endpoints** — If you later want colleagues to access both OpenAI and a local Ollama instance, use semicolon-separated `OPENAI_API_BASE_URLS` and `OPENAI_API_KEYS`
- **RAG / Document Upload** — Open WebUI supports uploading documents and chatting with them. Could be useful for production research. Already built in, just needs enabling.
- **Custom System Prompts per Model** — Via Admin Panel > Models, you can set different system prompts for different models (e.g. a “researcher” prompt for GPT-4, a “creative brainstorm” prompt for a different model)
- **Integration with Philosopher App** — The philosopher dialogue tool and this chat tool are separate. If you later want to link them (e.g. “chat with Socrates” as a model in Open WebUI), you could create a custom Pipe Function in Open WebUI that calls your philosopher pipeline.

-----

*Plan prepared: April 2026. Fork source: github.com/open-webui/open-webui (latest). Licence: Under 50 users, internal only — full rebrand permitted.*
