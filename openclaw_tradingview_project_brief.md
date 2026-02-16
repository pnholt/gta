# OpenClaw × TradingView — Clarification Questions + Execution-Ready Project Brief

## Clarification Questions for You

Before a long autonomous run, please confirm these so execution is unambiguous:

1. **Exact repository location**
   - Confirm the target repo URL/path is your **Moondove AI agents** GitHub.
   - Confirm work must stay inside: `Agents/OpenClaw/TradingView/`.

2. **Scope of TradingView items**
   - Confirm we process **Community → Editor’s Picks, Top, Trending** under **Indicators only** (not Strategies for now).
   - Confirm whether to include duplicates that appear in multiple tabs, or deduplicate by script name/ID.

3. **Skip rules**
   - Confirm skip criteria are exactly:
     - invite-only / no source visible,
     - source extraction failure,
     - non-backtestable visualization-only scripts after a “loose” attempt.
   - Confirm skipped items should be logged with reason in a separate CSV/JSON.

4. **Backtest translation standard**
   - Confirm Python conversion target is `backtesting.py`.
   - Confirm indicator calculations can use **pandas-ta** first, then **TA-Lib** fallback, then custom implementation.

5. **Dataset + symbol/timeframe policy**
   - Confirm data source is `moondove.com/data` (not Yahoo).
   - Confirm primary symbol(s) and required timeframe(s): you mentioned **6h** and **1h**. Should both be run for every indicator?
   - Confirm date range (e.g., full available history vs fixed start/end).

6. **Execution cadence / anti-rate-limit behavior**
   - Confirm delay between TradingView script pulls should be **10–15 seconds**.
   - Confirm retry policy (e.g., 3 retries with exponential backoff).

7. **Output naming and folder layout**
   - Confirm structure:
     - `Agents/OpenClaw/TradingView/pinescript/<script_name>.pine`
     - `Agents/OpenClaw/TradingView/backtesting/<script_name>_backtest.py`
     - `Agents/OpenClaw/TradingView/results/master_results.csv`
     - `Agents/OpenClaw/TradingView/results/skipped_scripts.csv`

8. **Metrics requirements**
   - Confirm notes block in each backtest file should include **full raw stats dump** from terminal output.
   - Confirm master CSV should include at least:
     - `script_name`, `tab_source`, `roi`, `max_drawdown`, `sharpe`, `sortino`, `expected_value`, `trades`, `status`.

9. **Commit policy**
   - Confirm commit frequency is **after each completed backtest** (or every N scripts if batching is acceptable).
   - Confirm commit message format preference (e.g., `feat(openclaw): add <script_name> pine + backtest + metrics`).

10. **Runtime supervision**
    - You asked to avoid long silent windows (e.g., 24h timeout). Confirm preferred heartbeat interval (e.g., status update every 10–20 minutes or every 5 scripts).

11. **Safety and credentials**
    - Confirm API keys must never be committed and should be loaded from env variables / local secrets file.

12. **Definition of “backtestable enough”**
    - For non-signal indicators, confirm fallback policy:
      - derive signals via threshold/cross logic,
      - if still ambiguous, mark as `needs_manual_signal_design` and skip execution.

---

## Execution-Ready Project Description (for OpenClaw Agent)

### Objective
Build an autonomous pipeline that iterates through TradingView Community indicator lists (Editor’s Picks, Top, Trending), copies source Pine scripts, converts each candidate to Python `backtesting.py` strategy code, runs baseline backtests, and logs standardized metrics.

### Mission Rules
- No shortcutting to only “easy” indicators.
- Process indicator-by-indicator from all three tabs.
- If a script cannot be used, log a concrete skip reason.
- Keep commits frequent and progress visible.
- Work only inside the designated OpenClaw folder.

### Directory Structure

```text
Agents/OpenClaw/TradingView/
  README.md
  pinescript/
    <script_name>.pine
  backtesting/
    <script_name>_backtest.py
  results/
    master_results.csv
    skipped_scripts.csv
  logs/
    run_log.md
```

### Workflow (Per Script)
1. Open TradingView → Indicators → Community tab (Editor’s Picks / Top / Trending).
2. Open script source code.
3. Save raw Pine source to `pinescript/<script_name>.pine`.
4. Assess backtestability:
   - If viable: translate to Python strategy.
   - If ambiguous: attempt loose signal derivation.
   - If impossible/invite-only: skip with reason.
5. Save Python strategy to `backtesting/<script_name>_backtest.py`.
6. Run baseline backtest (no optimization).
7. Add top-of-file notes block containing full run stats.
8. Append key metrics to `results/master_results.csv`.
9. Commit + push.

### CSV Schema (`master_results.csv`)

Suggested headers:

```csv
timestamp,script_name,tab_source,status,symbol,timeframe,start_date,end_date,roi,max_drawdown,sharpe,sortino,expected_value,trades,notes
```

### Skip Log Schema (`skipped_scripts.csv`)

```csv
timestamp,script_name,tab_source,reason,detail
```

### Backtest Defaults (until overridden)
- Engine: `backtesting.py`
- Data source: `moondove.com/data`
- Timeframes: `1h`, `6h` (if required by your final confirmation)
- Run type: single baseline run, no optimization
- Delay between source pulls: 10–15s

### Definition of Done
- All reachable scripts from all three indicator tabs are either:
  - converted + backtested + logged, **or**
  - recorded in skipped list with explicit reason.
- Master CSV is complete and continuously append-only.
- Every completed script has:
  - `.pine` source,
  - Python backtest file,
  - embedded stats notes,
  - committed history.

### Recommended README Statement
> This project intentionally avoids shortcut sampling. It processes TradingView community indicators systematically, records skips transparently, and logs repeatable baseline backtest metrics for downstream research.

