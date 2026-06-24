# Architecture Overview

## System Components

### 1. Mempool Streamer
- Pulls full mempool snapshot via Bitcoin Core RPC (`getrawmempool verbose=true`) every 30s
- Sorts transactions by effective fee-rate (sat/vbyte)
- Builds 3 forward-looking buckets:
  - **+1**: transactions most likely in next block
  - **+2**: most likely in block-after-next
  - **+3**: most likely 3 blocks ahead
- Stores buckets in `mempool_lookahead` with TX-ID lists for matching

### 2. Predictive Feeder (DATUM mode)
- Reads latest lookahead snapshot every 30s
- Calls `prioritisetransaction` RPC for all promoted TX-IDs
- Result: Bitcoin Core's next `getblocktemplate` includes these TXs with elevated effective priority
- DATUM Gateway then publishes this enhanced template to OCEAN's mining pool

### 3. Block Validator
- Subscribes to Bitcoin Core ZMQ `hashblock` notifications
- For each new block:
  - Fetches block transactions (`getblock verbose=2`)
  - Looks up historic lookahead snapshot (`tip_height = block_height − 1`)
  - Computes intersection of (predicted set) ∩ (actual block set)
  - Stores hit-rate per bucket in `block_hit_rates` collection
- Also checks `is_our_block` flag via coinbase-output-address match

### 4. Coverage Sweep (Auto-Executor)
- Optional brute-force GPU search on unsolved Bitcoin puzzles (#71 default)
- Anti-Herd strategy: top-down chunk selection
- Auto-tune: adjusts sub-range size to match GPU performance
- Budget-aware: respects monthly electricity cap

### 5. Weekly Report Service
- Sunday 23:00 (Berlin time) cron-style scheduler
- Aggregates 7 days of statistics
- Calls Claude Sonnet 4.5 for stakeholder analysis
- Generates PDF via Reportlab → sends via Telegram

## Data Flow

```
Bitcoin Core ─┬─ RPC: getrawmempool, getblock, prioritisetransaction
              └─ ZMQ: hashblock notifications
                      │
                      ▼
              ┌──────────────────┐
              │  Mempool Streamer │ (every 30s)
              └──────────────────┘
                      │
                      ▼
                MongoDB: mempool_lookahead, mempool_lookahead_history
                      │
                      ▼
              ┌──────────────────┐
              │ Predictive Feeder │ (every 30s)
              └──────────────────┘
                      │
                      ▼
              Bitcoin Core's getblocktemplate
                      │
                      ▼
              DATUM Gateway → OCEAN Stratum Pool
                      │
                      ▼
              ┌──────────────────┐
              │  Block Validator  │ (on every new block)
              └──────────────────┘
                      │
                      ▼
              MongoDB: block_hit_rates → Weekly Report → Telegram PDF
```

## Stack Choice Rationale

- **MongoDB**: schema-flexible (lookahead buckets vary in size), high write throughput, async via Motor
- **FastAPI**: async-first, perfect for I/O-bound work (Bitcoin RPC + ZMQ + DB)
- **Bitcoin Core (not BTCD)**: only Core implements `prioritisetransaction` reliably and ships with DATUM-Gateway support
- **OCEAN**: only major pool offering DATUM Gateway for user-built templates
- **Claude Sonnet 4.5**: best price/performance for German-language stakeholder reports

## Privacy & Security

- Predictive logic and full source code are private
- This repository contains documentation and high-level architecture only
- Production server is firewalled (Tailscale VPN for remote management)
- No personally identifiable information in mempool data
