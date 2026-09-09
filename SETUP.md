# SETUP

Instructions for an agent (or human) setting up this repo as a working pi agent
config directory.

## What this is

This repo is the contents of `~/.pi/agent` — the pi coding agent's home
directory: extensions, skills, themes, and TypeScript tooling. It intentionally
does **not** contain `settings.json`, `models.json`, `auth.json`, or `.env`
(those are gitignored because they hold machine-specific config and secrets).

## Prerequisites

- [pi](https://github.com/badlogic/pi-mono) (`~/.local/share/mise/installs/pi/...` via mise works too)
- Node.js >= 22 (v26 used on the source machine)
- bun (optional, faster installs)

## Steps

1. Clone this repo to `~/.pi/agent` (or clone elsewhere and symlink the
   directory):

   ```sh
   git clone <repo-url> ~/.pi/agent
   cd ~/.pi/agent
   bun install   # or: npm install
   ```

2. Install each extension's dependencies (they have their own `package.json`
   and gitignored `node_modules`):

   ```sh
   for d in extensions/*/; do
     [ -f "$d/package.json" ] && (cd "$d" && bun install)
   done
   ```

3. Create `~/.pi/agent/.env` with your Firecrawl key (see `.env.example`):

   ```sh
   cp .env.example .env   # then fill in FIRECRAWL_API_KEY
   ```

4. Create `~/.pi/agent/settings.json` (used by pi, gitignored):

   ```json
   {
     "defaultProvider": "local-spark",
     "defaultModel": "GLM-5.3-Flash-EXL3",
     "defaultThinkingLevel": "high",
     "theme": "omarchy-system",
     "quietStartup": true
   }
   ```

   If you don't have the local inference rig, point `defaultProvider` /
   `defaultModel` at whatever you use (e.g. `openai-codex/gpt-5.6-terra`).

5. Create `~/.pi/agent/models.json` — custom providers. The source machine has
   two local providers (an exllamav2 server on `localhost:8888` and a 2x DGX
   Spark rig at `192.168.20.21:8888`, both OpenAI-compatible completions APIs
   with `apiKey: "local"`). If you have equivalent hardware, define providers
   named `local` and `local-spark` there; otherwise omit this file and use
   built-in providers. See pi's docs on custom providers for the schema
   (`compat`, `thinkingLevelMap`, `chatTemplateKwargs` fields are in use).

6. Authenticate built-in providers (e.g. Codex OAuth) — this writes the
   gitignored `auth.json`:

   ```sh
   pi /login
   ```

7. Verify:

   ```sh
   npm run check        # tsc --noEmit over extensions
   pi                   # agent should start, list extensions on startup
   ```

## Notes for agents

- `settings.json`, `models.json`, `auth.json`, `.env`, `sessions/`,
  `models-store.json`, and `trust.json` are runtime state — never commit them.
  `.gitignore` already covers all of them; keep it that way.
- Extensions live in `extensions/<name>/` and are loaded by pi automatically.
  Shared helpers live in `extensions/shared/`.
- Skills live in `skills/<name>/SKILL.md` and are loaded on demand.
- The Omarchy theme (`themes/omarchy-system.json`) is gitignored — on Omarchy
  machines, recreate it via omarchy's theme switcher or pi's theme picker.
- Formatting/lint: `npm run format` (prettier). Type check: `npm run check`.
