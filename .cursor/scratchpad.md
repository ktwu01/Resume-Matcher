# Background and Motivation

Goal: Run `ai-resume-refiner` (Resume-Matcher) locally on macOS. Evaluate if `setup.sh` is sufficient to bootstrap backend (FastAPI) and frontend (Next.js) with Ollama and SQLite, and define minimal steps to succeed.

# Key Challenges and Analysis

- Backend and frontend `.env.sample` are referenced by `setup.sh` but not present in this repo snapshot. Without `.env`, backend DB URLs and session key are missing; frontend lacks `NEXT_PUBLIC_API_URL`.
- Python version: backend requires Python >= 3.12. `setup.sh` checks for `python3` but not its version.
- Ollama: script may install but does not start the daemon; provider pulls models only if `.env` exists. Need `ollama serve` and required models available.
- Frontend API base: app uses `NEXT_PUBLIC_API_URL` directly; must be set to `http://localhost:8000`.

Conclusion: `setup.sh` helps (installs deps, runs uv sync, npm ci) but is not sufficient alone on macOS for this snapshot. Manual `.env` creation and starting Ollama are required.

# High-level Task Breakdown

1) Verify prerequisites
- Ensure: `node -v` ≥ 18, `npm -v`, `python3 --version` ≥ 3.12, `uv --version`, `ollama --version`.

2) Run setup script
- `chmod +x setup.sh && ./setup.sh`
- Expect: root deps installed, backend `uv sync`, frontend `npm ci`.

3) Create backend `.env` at `apps/backend/.env`
- Minimal:
  - `SYNC_DATABASE_URL=sqlite:///./app.db`
  - `ASYNC_DATABASE_URL=sqlite+aiosqlite:///./app.db`
  - `SESSION_SECRET_KEY=change-me`
  - `LLM_PROVIDER=ollama`
  - `LL_MODEL=gemma3:4b`
  - `EMBEDDING_PROVIDER=ollama`
  - `EMBEDDING_MODEL=dengcao/Qwen3-Embedding-0.6B:Q8_0`

4) Create frontend `.env` at `apps/frontend/.env`
- `NEXT_PUBLIC_API_URL=http://localhost:8000`

5) Start Ollama and ensure models
- Run `ollama serve` in a separate terminal (or open the macOS app)
- Optionally pre-pull: `ollama pull gemma3:4b` and `ollama pull dengcao/Qwen3-Embedding-0.6B:Q8_0`

6) Run in development
- From repo root: `npm run dev`
- Verify: `http://localhost:8000/api/docs` and `http://localhost:3000`

7) Production (optional)
- `npm run build` then `npm run start`

# Project Status Board

- [ ] Create backend `.env`
- [ ] Create frontend `.env`
- [ ] Start `ollama serve` and pull required models
- [ ] Run `./setup.sh`
- [ ] Start dev: `npm run dev`
- [ ] Verify health endpoints and UI flows

# Current Status / Progress Tracking

✅ **GIT COMMIT AND PUSH COMPLETED**

- Successfully committed all changes (commit b751d87)
- Pushed to GitHub repository: https://github.com/ktwu01/Resume-Matcher.git
- 8 files changed: API LLM support, deployment docs, project documentation
- Repository is now up to date with all local changes

✅ **DEPLOYMENT SUCCESSFUL**

- Backend running on http://localhost:8000 with Anthropic API
- Frontend running on http://localhost:3000 
- Health check passes: `{"message":"pong","database":"reachable"}`
- API endpoints responding correctly
- No Ollama required - using API-based LLM

**Access Points:**
- Frontend: http://localhost:3000
- Backend API docs: http://localhost:8000/api/docs
- Health check: http://localhost:8000/ping

✅ **BUILD DEBUG COMPLETED**

- Issue: `npm run build` failed with "sh: uv: command not found"
- Root cause: Temporary PATH issue in npm script environment
- Investigation: Found proper PATH setup already exists in `~/.local/bin/env` (sourced by `.zshrc`)
- Solution: The PATH configuration was already correct; issue was likely a temporary shell environment problem
- Result: Build now completes successfully without manual PATH export
- Status: `npm run build` works reliably in current shell setup

✅ **PRODUCTION START DEBUG COMPLETED**

- Issue: `npm run start` failed with "address already in use" errors on ports 3000 and 8000
- Root cause: Development servers were still running from earlier `npm run dev` commands
- Solution: Killed existing dev processes (PIDs 54790, 51823) before starting production
- Result: Production servers now running successfully
- Status: 
  - Frontend (production): http://localhost:3000 ✅
  - Backend (production): http://localhost:8000 ✅
  - Health check: `{"message":"pong","database":"reachable"}` ✅

# Executor's Feedback or Assistance Requests

- If `python3 --version` < 3.12, install `brew install python@3.12` and configure `uv` to use it.
- Prefer pre-pulling models up-front to avoid first-request latency and errors.

# Lessons

- Missing `.env.sample` requires manual `.env` creation in this snapshot.
- Backend DB URLs must be set; engines create at import-time.
- **uv PATH Setup**: The project uses `~/.local/bin/env` (sourced by `.zshrc`) to properly manage PATH for `~/.local/bin`. This setup automatically handles uv availability without manual PATH exports.
- **Port Conflicts**: When switching between dev and production modes, always check for existing processes on ports 3000/8000. Use `lsof -ti:PORT` and `kill PID` to resolve conflicts.

# Plan: Use API LLM (Anthropic via OpenAI-compatible endpoint)

Assumption: The endpoint at `https://anyrouter.top` exposes an OpenAI-compatible API (often requires `/v1`). If it needs `/v1`, include it in BASE_URL.

Backend `.env` changes (replace existing values):
- `LLM_PROVIDER=llama_index.llms.openai_like.OpenAILike`
- `LLM_API_KEY=<your-anthropic-token>`
- `LLM_BASE_URL=https://anyrouter.top`  (use `https://anyrouter.top/v1` if required)
- `LL_MODEL=claude-3-5-haiku-20241022`

Embeddings without local models (preferred):
- `EMBEDDING_PROVIDER=llama_index.embeddings.openai_like.OpenAILikeEmbedding`
- `EMBEDDING_API_KEY=<your-anthropic-token>`
- `EMBEDDING_BASE_URL=https://anyrouter.top` (or `/v1` if required)
- `EMBEDDING_MODEL=text-embedding-3-small` (ensure the router supports this; otherwise pick a supported embedding model)

Notes:
- Anthropic does not provide embeddings natively; ensure the router supports an OpenAI-like embedding model. If not, switch `EMBEDDING_*` to a provider that does (e.g., OpenAI) or temporarily keep Ollama embeddings.
- No need to run `ollama serve` when using API for both LLM and embeddings.

Verification checklist:
- Backend starts, `/api/docs` loads.
- A resume upload triggers LLM calls successfully (no 401/404).
- Structured JSON responses parse without errors in `JSONWrapper`.

# Fork Synchronization Plan

**Situation Analysis:**
- Current fork status: 1 commit ahead, 22 commits behind `srbhr/Resume-Matcher:main`
- Your commit (b751d87): Contains valuable API LLM support and deployment docs
- Strategy: Sync with upstream while preserving your contributions

**High-level Sync Task Breakdown:**

1) **Add upstream remote** (if not already added)
   - Add `srbhr/Resume-Matcher` as upstream remote
   - Success criteria: `git remote -v` shows upstream URL

2) **Fetch latest upstream changes**
   - Fetch all branches and commits from upstream
   - Success criteria: No errors, upstream/main branch available locally

3) **Merge upstream into local main**
   - Merge upstream/main into your local main branch
   - Handle any merge conflicts if they occur
   - Success criteria: Local main contains both your commit and upstream changes

4) **Push synchronized branch to your fork**
   - Push updated main branch to your GitHub fork
   - Success criteria: GitHub shows fork is up-to-date with upstream

5) **Verify application still works**
   - Test that the app still runs correctly after merge
   - Verify API LLM functionality is preserved
   - Success criteria: Both dev and production modes work as before

# Project Status Board (Fork Sync)
- [ ] Add upstream remote repository
- [ ] Fetch latest changes from upstream
- [ ] Merge upstream/main into local main branch
- [ ] Resolve any merge conflicts (if needed)
- [ ] Push synchronized branch to fork
- [ ] Test application functionality post-merge
- [ ] Verify API LLM features still work
