# BMS Ticket Notifier — Spider-Man: Brand New Day (ScreenX)

Free, simple email alerts when BookMyShow tickets change — a new date opens
or seats free up. Runs on **GitHub Actions cron** (free) and emails via
**Resend** (free tier). Based on
[aviiciii/bms-ticket-notifier](https://github.com/aviiciii/bms-ticket-notifier),
with an added **`BMS_FORMAT`** filter (so you only hear about ScreenX shows)
and a Windows-safe UTF-8 print fix.

## How it works
1. Hits BMS's JSON API for your movie + region + dates.
2. Filters by **theatre**, **date**, **time period**, and **format** (ScreenX).
3. Compares against last run (`bms_state.json`, committed back by the workflow).
4. Emails you when a date opens or a sold-out show becomes available.

---

## Your config (Spider-Man BND, INOX Mall of Asia ScreenX, Aug 8–9)

| Key | Value |
|-----|-------|
| `BMS_URL` | `https://in.bookmyshow.com/movies/bengaluru/spider-man-brand-new-day/buytickets/ET00502684/20260730` |
| `BMS_DATES` | `20260808,20260809` |
| `BMS_THEATRE` | `Mall of Asia` |
| `BMS_FORMAT` | `ScreenX` |
| `BMS_TIME` | *(leave empty for all times, or e.g. `evening,night`)* |

> `BMS_THEATRE` is a case-insensitive substring match on the venue name, so
> `Mall of Asia` catches "INOX: Megaplex Mall of Asia". `BMS_FORMAT` is a
> substring match on the show's screen attribute. If you get zero ScreenX
> hits once tickets are live, run it **without** `BMS_FORMAT` first to see the
> exact format label BMS uses, then set `BMS_FORMAT` to match.

---

## Setup (GitHub Actions — the free, hands-off way)

1. **Create a GitHub repo** and push this folder to it (see git steps below).
2. **Get a free Resend key:** sign up at [resend.com](https://resend.com) →
   API Keys → create one (starts with `re_`). For the sender you can use
   `onboarding@resend.dev` until you verify your own domain.
3. **Add repo Secrets** (Settings → Secrets and variables → Actions → *Secrets*):
   | Secret | Value |
   |--------|-------|
   | `RESEND_API_KEY` | your `re_...` key |
   | `RESEND_FROM_EMAIL` | `onboarding@resend.dev` (or your verified domain email) |
   | `RESEND_TO_EMAIL` | the email you want alerts at |
4. **Add repo Variables** (same page → *Variables* tab): `BMS_URL`,
   `BMS_DATES`, `BMS_THEATRE`, `BMS_FORMAT` from the table above.
5. **Turn on Actions write permission:** Settings → Actions → General →
   *Workflow permissions* → **Read and write** (so it can commit
   `bms_state.json` between runs).
6. **Run it:** Actions tab → *BMS Ticket Checker* → **Run workflow**. After
   that it auto-runs every 30 min. First run just records a baseline (no
   email); you get emails only when something *changes*.

> GitHub's scheduled cron is best-effort and can lag a few minutes under load.
> Every 30 min is plenty for catching a date opening.

---

## Run locally (to test)

Requires Python 3.11+ and [uv](https://docs.astral.sh/uv/).

```bat
uv sync
set BMS_URL=https://in.bookmyshow.com/movies/bengaluru/spider-man-brand-new-day/buytickets/ET00502684/20260730
set BMS_DATES=20260808,20260809
set BMS_THEATRE=Mall of Asia
set BMS_FORMAT=ScreenX
uv run main.py
```

Leave `RESEND_*` unset and it just prints results (no email). Note: BMS's API
is **not reachable from the Walmart VPN/proxy** — test from a personal
network, or just trust the GitHub Actions run.
