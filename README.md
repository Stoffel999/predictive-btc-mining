# BlockSeer — Predictive Bitcoin Mining via Mempool Lookahead

> **Status:** Live in production on Bitcoin Mainnet via OCEAN Pool (DATUM Gateway)
> **Recorded sample window:** 23–24 June 2026 — median predictive hit-rate **95.3 %** (n=166 blocks, 21.5 h)
> **Current production (rolling 24 h):** **87–100 %**
> **Approach:** Predict which transactions will land in the next Bitcoin block, before it is mined.

---

## Why this matters

Bitcoin block templates are normally built greedily by fee-rate. BlockSeer does better: it **predicts the actual next block content** by combining mempool dynamics, fee-market signals and a custom multi-block lookahead model.

When OCEAN miners build their own templates via DATUM Gateway, BlockSeer's predictions allow:

- **Higher fee capture** per mined block (vs. default `getblocktemplate`)
- **Fewer stale templates** during high-volume periods
- **Quantifiable predictive edge** with reproducible statistics

## Live Numbers (sample window: 23–24 June 2026)

| Metric | Value |
|--------|------:|
| Blocks analyzed | 166 |
| Median Hit-Rate (+1 bucket, outlier-filtered) | **95.3 %** |
| Mean Hit-Rate (+1 bucket, outlier-filtered) | 78.0 % |
| Mempool transactions prioritised | 4,343,476 |
| DATUM-Feeder runs | 7,520 (1 error) |
| Outlier blocks (empty / reorg, < 10 %) | 4 |

Full breakdown and raw numbers: see [`STATS.md`](./STATS.md).

## Architecture

```
┌──────────────┐    ┌──────────────────┐    ┌──────────────┐
│ Bitcoin Node │ →  │ Mempool Streamer │ →  │ Lookahead DB │
└──────────────┘    └──────────────────┘    └──────┬───────┘
                                                    ↓
┌──────────────┐    ┌──────────────────┐    ┌──────────────┐
│ OCEAN Pool   │ ←  │ DATUM Gateway    │ ←  │ Predictive   │
│ (Bitcoin)    │    │ (Template Build) │    │ Feeder       │
└──────────────┘    └──────────────────┘    └──────────────┘
```

- **Mempool Streamer**: snapshots mempool every 30 s with fee-rate ranking, computes next-3-block lookahead buckets
- **Predictive Feeder**: uses `prioritisetransaction` RPC to influence Bitcoin Core's block template
- **Squeeze Mode**: promotes all mempool transactions for max fee capture during low-volume periods
- **Auto-Validator**: cross-checks every newly mined block against the lookahead → measures real-time accuracy

Full architecture overview: see [`ARCHITECTURE.md`](./ARCHITECTURE.md).

## Use Cases

1. **Mining Pools** with DATUM / Stratum V2 → integrate predictive templates as a service
2. **Mining Farms** running solo via Bitcoin Core → improve template freshness and fee capture
3. **Trading Desks** → use confirmation-timing signals
4. **Academic Research** → reproducible dataset of predictive mining performance

## Technical Stack

- **Backend:** Python (FastAPI), MongoDB, Async I/O
- **Bitcoin Integration:** Bitcoin Core / Knots RPC + ZMQ, OCEAN DATUM Gateway
- **AI Analysis:** Claude Sonnet 4.5 weekly stakeholder reports
- **Frontend:** React + dashboards
- **Hardware:** runs on commodity x86_64 server hardware (specs in the licensed install guide)

## Reproducibility & Trust

- Weekly auto-generated **PDF reports** with full statistics
- Outlier transparency (with and without filter)
- All metrics logged in MongoDB for independent audit
- Audit access available to serious counterparties under NDA

## Status & Roadmap

- ✅ Production live on Bitcoin Mainnet via OCEAN
- ✅ Autonomous 24/7 operation with Telegram critical-event alerts
- ✅ Cryptographically signed authenticity-key per deployment
- ✅ EN / DE native UI, 7 additional languages via auto-translate pipeline
- ⏳ Weekly stakeholder reports with AI auto-interpretation
- ⏳ Long-term dataset (target: 4–8 weeks of continuous data)

## Commercial Licensing — Three Models

BlockSeer is sold under three commercial licensing models. Choose the one that fits your situation:

| Model | What you get | Buyers per Version |
|---|---|---|
| **A — Full Exclusive, Self-Maintained** | Own the codebase outright, develop it further yourself | 1 (exclusive) |
| **B — Full Exclusive, Vendor-Maintained** | Exclusive use, vendor continues development against agreed update fees | 1 (exclusive) |
| **C — Single-Version License** | One working production licence, may run in parallel with up to 2 other model-C buyers | up to 3 |

Side-by-side comparison: see [`LICENSE_MODELS.md`](./LICENSE_MODELS.md).
Full contract terms, support window, day-rate and pricing are disclosed after a signed NDA — see [`CONTACT.md`](./CONTACT.md).

## Interested?

Suitable counterparties:

- Bitcoin mining pool operators
- Mid-size mining farms (1–10 EH/s)
- Mining-infrastructure investors
- Crypto research groups / academic institutions

**Contact:** see [`CONTACT.md`](./CONTACT.md) — Telegram, Reddit, Bitcoin Talk and GitHub Issues.

---

*This repository contains documentation, architecture and live-stat samples only.
The full production codebase, install guide and licence-key system are private and shared with serious counterparties after a signed NDA.*
