# Predictive Bitcoin Mining — Mempool Lookahead Templates

> **Status:** Live in production on Bitcoin Mainnet via OCEAN Pool (DATUM Gateway)
> **Performance:** Median Predictive Hit-Rate **95.3%** (n=166 blocks, 21.5h window)
> **Approach:** Predict which transactions will land in the next Bitcoin block, before it is mined.

---

## Why this matters

Bitcoin block templates are normally built greedily by fee-rate. We do better: we **predict the actual next block content** by combining mempool dynamics, fee market signals and a custom lookahead model.

When OCEAN miners build their own templates via DATUM Gateway, our predictions allow:

- **Higher fee capture** per mined block (vs. default `getblocktemplate`)
- **Fewer stale templates** during high-volume periods
- **Quantifiable predictive edge** with reproducible statistics

## Live Numbers (sample window: 23-24 June 2026)

| Metric | Value |
|--------|------:|
| Blocks analyzed | 166 |
| Median Hit-Rate (+1 bucket, outlier-filtered) | **95.3%** |
| Mean Hit-Rate (+1 bucket, outlier-filtered) | 71.9% |
| Mempool transactions priorititized | 4,343,476 |
| DATUM-Feeder runs | 7,520 (1 error) |
| Outlier blocks (empty / reorg, < 10%) | 4 |

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

- **Mempool Streamer**: Snapshots mempool every 30s with fee-rate ranking, computes next-3-block lookahead buckets
- **Predictive Feeder**: Uses `prioritisetransaction` RPC to influence Bitcoin Core's block template
- **Squeeze Mode**: Promotes all mempool transactions for max fee capture during low-volume periods
- **Auto-Validator**: Cross-checks every newly mined block against the lookahead → measures real-time accuracy

## Use Cases

1. **Mining Pools** with DATUM/Stratum V2 → integrate predictive templates as a service
2. **Mining Farms** running solo via Bitcoin Core → improve template freshness and fee capture
3. **Trading Desks** → use confirmation timing signals
4. **Academic Research** → reproducible dataset of predictive mining performance

## Technical Stack

- **Backend:** Python (FastAPI), MongoDB, Async I/O
- **Bitcoin Integration:** Bitcoin Core RPC + ZMQ, OCEAN DATUM Gateway
- **AI Analysis:** Claude Sonnet 4.5 weekly reports
- **Frontend:** React + Chart.js dashboards
- **Hardware:** Tested on consumer GPU rigs (7× P106-100) + ASICs

## Reproducibility & Trust

- Weekly auto-generated **PDF reports** with full statistics
- Outlier transparency (with + without filter)
- All metrics logged in MongoDB for independent audit
- Source-controlled (commit history available on request)

## Status & Roadmap

- ✅ Production live on Bitcoin Mainnet via OCEAN
- ✅ Autonomous 24/7 operation with Telegram critical-event alerts
- ✅ Multi-puzzle GPU coverage sweep (Anti-Herd strategy)
- ⏳ Weekly stakeholder reports with KI auto-interpretation
- ⏳ Long-term dataset (target: 4-8 weeks of continuous data)

## Interested?

This project is being prepared for sale or licensing. Suitable counterparties:

- Bitcoin mining pool operators
- Mid-size mining farms (1–10 EH/s)
- Mining infrastructure investors
- Crypto research groups / academic institutions

**Contact:** Telegram @Stoffel999

---

*This repository contains documentation and high-level architecture only.
The full production codebase is private and available under NDA for serious counterparties.*
