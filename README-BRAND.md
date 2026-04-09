# LLM Philosopher Chat

Internal AI assistant for the team. Think of it as a private ChatGPT, pre-configured
with our API key and a curated set of models.

## What it does

A branded chat interface where colleagues can pick a model, send messages, and get
streaming responses — with conversation history that persists between sessions.
Under the hood it's a rebranded [Open WebUI](https://github.com/open-webui/open-webui)
fork tailored for our TV production workflow.

## How it works

- Open WebUI (Svelte + FastAPI) handles the chat UI, user management, and history.
- The backend proxies requests to an OpenAI-compatible endpoint using our shared API key.
- Branding (name, favicon, colours, hidden features) is layered on top via
  `Dockerfile.brand` + `branding/custom.css` so rebuilds stay clean.

## Installation (admin / operator)

1. **Clone and switch to the rebrand branch**
   ```bash
   git clone <repo-url> && cd open-webui-ls
   git checkout feature/rebrand-llm-philosopher
   ```
2. **Create your `.env`**
   ```bash
   cp .env.brand.example .env
   # Edit .env — set OPENAI_API_KEY and WEBUI_SECRET_KEY (openssl rand -hex 32).
   ```
3. **Swap placeholder branding assets** in `branding/` for real favicon files.
4. **Build and launch**
   ```bash
   docker compose -f docker-compose.brand.yaml up -d --build
   ```
5. **First-run admin setup**, open <http://localhost:3000>:
   - First signup becomes admin — create your account.
   - Admin Panel → Users → create colleague accounts (signup is disabled).
   - Admin Panel → Settings → Models → hide models you don't want exposed.
   - Admin Panel → Settings → Interface → paste `branding/custom.css` into
     *Custom CSS*, set the default system prompt, pick default model.

## Usage (colleagues)

1. Open the URL your admin shared (e.g. `http://server:3000`).
2. Log in with the credentials you were given.
3. Pick a model from the dropdown, type, press Enter.

**Please don't** paste confidential contracts, client NDAs, or anything you
wouldn't send over email. Questions → ping the admin.

## Testing checklist

- [ ] `docker compose -f docker-compose.brand.yaml up -d --build` starts cleanly
- [ ] `http://localhost:3000` shows "LLM Philosopher Chat", not "Open WebUI"
- [ ] Favicon is the custom one
- [ ] Custom CSS colours applied (via Admin Panel)
- [ ] Admin + colleague accounts log in
- [ ] Model dropdown only shows curated models
- [ ] Streaming response works
- [ ] History persists across `docker compose down && up -d`
- [ ] Signup is blocked for non-admins
- [ ] System prompt is applied (ask "what are you?")

## Gotchas

- **PersistentConfig:** Admin Panel settings override `.env` after first save.
  To force `.env` to win, set `RESET_CONFIG_ON_START=true` for one restart, then remove.
- **`WEBUI_NAME` misses some strings.** Use `branding/custom.css` to hide residual
  "Open WebUI" text surfaced in hardcoded Svelte components.
- **Image size:** Upstream image is ~5GB; first pull is slow, subsequent builds cached.
- **API key security:** `.env` is gitignored. Never commit real keys.

## File layout

```
.env.brand.example        # Template — copy to .env
Dockerfile.brand          # Branding layer (FROM upstream image)
docker-compose.brand.yaml # Compose config for the rebrand
branding/
  favicon.svg/.png/.ico   # Placeholder icons — replace
  custom.css              # Theme overrides
  README.md               # Asset notes
README-BRAND.md           # This file
```

The upstream `Dockerfile`, `docker-compose.yaml`, and `README.md` are left untouched
so we can still pull updates from the upstream Open WebUI project.
