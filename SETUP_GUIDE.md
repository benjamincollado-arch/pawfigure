# pawfigure Setup Guide

This guide walks you from a fresh computer to your first stock analysis. It
assumes no programming background. Plan on about 30 minutes the first time.

pawfigure is a copy of [TradingAgents](https://github.com/TauricResearch/TradingAgents).
It runs a team of AI agents on one stock for one date:

1. **Analysts** read the price chart, company financials, news and social media.
2. **Bull and bear researchers** argue for and against the stock.
3. **A trader** proposes a trade.
4. **Risk managers and a portfolio manager** review the trade and give the final
   rating: Buy, Overweight, Hold, Underweight or Sell.

> **This is a research tool, not financial advice.** The AI can be wrong, and
> it sounds confident even when it is. Test it on past dates before you trust
> it with real money.

---

## Step 1. What you need

| Item | Cost | Where to get it |
|---|---|---|
| A Windows, Mac or Linux computer | You have it | |
| Python 3.10 or newer (3.12 recommended) | Free | [python.org/downloads](https://www.python.org/downloads/) |
| Git | Free | [git-scm.com/downloads](https://git-scm.com/downloads) |
| An API key from one AI company | Pay per use | See Step 4 |

**Windows tip:** when the Python installer opens, check the box **"Add
python.exe to PATH"** before you click Install.

Stock prices, news and company data come from Yahoo Finance by default. That's
free and needs no key.

---

## Step 2. Download the code

Open a terminal:

- **Windows:** press the Windows key, type `PowerShell`, press Enter.
- **Mac:** press Cmd+Space, type `Terminal`, press Enter.

Then run these two commands, one at a time:

```bash
git clone https://github.com/benjamincollado-arch/pawfigure.git
cd pawfigure
```

If this code isn't on your main branch yet, add this command after them:

```bash
git checkout claude/trading-agents-repo-2k8veq
```

---

## Step 3. Install it

A "virtual environment" is a private folder for this project's Python
packages, so they don't clash with anything else on your computer.

**Windows (PowerShell):**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -e .
```

If PowerShell says running scripts is disabled, run this once, then try the
activate line again:
`Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`

**Mac or Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
```

The install takes a few minutes. When it finishes, check it worked:

```bash
tradingagents --help
```

You should see a help screen that lists a `backtest` command.

**Every time you open a new terminal**, go to the `pawfigure` folder and run
the activate line again (`.venv\Scripts\Activate.ps1` on Windows,
`source .venv/bin/activate` on Mac). Your prompt shows `(.venv)` when it's
active.

---

## Step 4. Get an AI API key

The agents need an AI model to think with. You pay the AI company directly for
what you use. Pick **one** provider to start:

| Provider | Sign-up page | Name in `.env` |
|---|---|---|
| Anthropic (Claude) | [console.anthropic.com](https://console.anthropic.com/) | `ANTHROPIC_API_KEY` |
| OpenAI (GPT) | [platform.openai.com](https://platform.openai.com/) | `OPENAI_API_KEY` |
| Google (Gemini) | [aistudio.google.com](https://aistudio.google.com/) | `GOOGLE_API_KEY` |

Create an account, add a payment method, and create an API key. Treat the key
like a password: anyone who has it can spend your money.

**Set a monthly spending limit** in the provider's billing settings before
your first run. $10 to $20 is a good place to start. A single analysis makes
many AI calls, so costs add up fast with the biggest models and with
backtests.

---

## Step 5. Put your key in the `.env` file

The project reads your keys from a file named `.env`. Copy the example:

- **Windows:** `copy .env.example .env`
- **Mac or Linux:** `cp .env.example .env`

Open `.env` in any text editor (Notepad works) and paste your key after the
matching name, with no spaces or quotes:

```
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

Save the file. `.env` is already listed in `.gitignore`, so git won't upload
it. **Never** paste your key into any other file, a GitHub issue or a chat.

Optional, and free: add your name and email so the SEC can reach you if you
use its company-filing data:

```
SEC_EDGAR_USER_AGENT=Your Name your@email.com
```

---

## Step 6. Run your first analysis

```bash
tradingagents
```

The screen asks you a series of questions. Use the arrow keys and press Enter
to pick an answer:

1. **Ticker:** the stock symbol, such as `AAPL`, `NVDA` or `SPY`.
2. **Analysis date:** press Enter for today, or type a past date as
   `YYYY-MM-DD`.
3. **Output language:** press Enter for English.
4. **Analysts:** keep all four selected.
5. **Research depth:** choose the **shallowest** option for your first runs.
   More depth means more debate rounds, which means a bigger bill.
6. **LLM provider:** the company whose key you added in Step 5.
7. **Models:** for your first runs, pick the cheaper, faster option for both
   the "quick" and "deep" thinking models. Upgrade once you know what a run
   costs.

If the tool can't find your key, it asks you to paste it and saves it to
`.env` for you.

The screen then shows each agent working, and ends with the final rating and
full reports. Runs usually take several minutes.

Next time, the tool remembers your answers, so you can just press Enter
through the questions.

### Where the results go

- Reports: in your home folder, under
  `.tradingagents/logs/<TICKER>/<DATE>/reports`.
- A running log of every decision: `.tradingagents/memory/trading_memory.md`,
  also in your home folder. The next time you analyze the same stock, the
  agents check how their earlier call turned out and learn from it.

---

## Step 7. Test it on the past before you trust it

A backtest runs the agents on past dates and scores their calls against what
the stock actually did afterwards. For example:

```bash
tradingagents backtest AAPL --start 2026-06-01 --end 2026-08-31 --every 14
```

That analyzes Apple every 14 days over three months, which is about 7 full
runs. **Each date costs as much as one normal analysis**, so start with
one ticker, a short date range, and a large `--every` gap.

---

## Troubleshooting

| What you see | What to do |
|---|---|
| `tradingagents: command not found` or "not recognized" | Your virtual environment isn't active. Run the activate line from Step 3. |
| `python: command not found` on Mac | Use `python3` instead of `python`. |
| An error mentioning an API key, `401` or "authentication" | Check `.env`: the right name, no spaces, no quotes, and the file is saved in the `pawfigure` folder. |
| `429` or "rate limit" | You're sending requests faster than your provider allows. Wait a minute and try again, or add `TRADINGAGENTS_LLM_MAX_RETRIES=10` to `.env`. |
| No price data or a Yahoo Finance error | Check your internet connection. Some work or government networks block Yahoo Finance; try a home network. |
| A run crashed halfway | Start it with `tradingagents --checkpoint` next time, and it picks up where it stopped. |

---

## Tested

This setup was checked on September 26, 2026, on Linux with Python 3.11:

- `pip install -e ".[dev]"` installed cleanly.
- `tradingagents --help` and `tradingagents backtest --help` worked.
- The offline test suite passed: 1004 passed, 1 skipped (the skipped test
  needs the optional AWS Bedrock add-on).

A full analysis wasn't run during that check. The test machine had no AI API
key, and its network blocked Yahoo Finance. Your first real run is Step 6.
