# Deploying `deriv_multisymbol_bot.py` to Railway

## Files in this bundle
- `deriv_multisymbol_bot.py` — the bot
- `requirements.txt` — pinned Python deps
- `railway.json` — Railway build/deploy config (Nixpacks builder, start command, always-restart)
- `nixpacks.toml` — pins the Python version Nixpacks builds with
- `Procfile` — fallback start command (not required if `railway.json` is present, kept for portability)
- `.env.example` — list of env vars to set in Railway, not a real secrets file
- `.gitignore`

## Steps
1. Push these files (same folder as the bot) to a GitHub repo, or use `railway up` from this
   directory with the Railway CLI.
2. In Railway: **New Project → Deploy from GitHub repo** (or confirm the CLI-deployed service).
3. Railway auto-detects Python via Nixpacks and installs `requirements.txt`.
4. Under your service → **Variables**, add the variables from `.env.example` with your real
   values:
   - `DERIV_APP_ID`, `DERIV_API_TOKEN`, `DERIV_ACCOUNT_ID`, `DERIV_ACCOUNT_TYPE`
   - `SUPABASE_URL`, `SUPABASE_KEY` (optional — bot runs fine without persistence, just logs a
     warning and skips trade-log storage)
   - `VERBOSE_LOGS` (optional)
5. Deploy. This is a long-running worker process (no HTTP server, no port binding needed) —
   Railway will keep it alive under `restartPolicyType: ALWAYS`. The bot also has its own
   in-process crash-recovery (`os.execv` restart) for unhandled exceptions, so Railway-level
   restarts should only trigger on hard container failures.
6. Check the **Deploy Logs** tab for the startup calibration output
   (`deep_startup_calibration()` across `TRADE_SYMBOLS`) before it begins placing trades.

## Notes
- `DERIV_APP_ID=1089` in `.env.example` is Deriv's public demo app ID — replace with your own
  registered app ID for production use.
- Set `DERIV_ACCOUNT_TYPE=demo` first and confirm behavior on a demo account before pointing
  this at `real`.
- Railway's free/hobby tiers sleep or cap runtime differently than paid plans — an
  always-on trading bot needs a plan that supports persistent, non-sleeping workers.
