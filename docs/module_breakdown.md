Module Breakdown

This architecture is composed of eight core modules working together as a fully autonomous FX trading ecosystem. Each module is isolated, single-responsibility, and communicates through lightweight HTTP APIs and the shared SQLite event database.

1. Athena — Strategy & Decision Engine

File: athena_brain.py
Athena is the “brain” of the trading ecosystem.
It ingests multi-timeframe indicators, microstructure signals, sentiment, trend scoring, and session-filtered volatility before generating BUY/SELL/HOLD decisions.

Primary Responsibilities

* Multi-TF technical signal generation (1m → 15m)

* ATR gating & volatility normalization

* Trend strength scoring

* Confidence scoring

* Session-aware trading activation

* Risk-tier & position size calculation

* Strict overtrading prevention & daily kill-switch

* Sending executable orders to Hermes

* Writing structured trade objects to athena_trades

2. Orion — Strategy Architect & Evolution Engine

File: orion.py 

orion


Orion acts as the in-system quant engineer. It evaluates real trade history, detects weaknesses, and produces human-reviewable strategy upgrades (never applied automatically for safety).

Primary Responsibilities

* Full trade-history ingestion and statistics engine

* Strategy weakness detection (win-rate, drawdown, avg P/L)

* Monte Carlo simulation (2,000 runs) for long-term edge estimation

* Code analyzer for Athena logic

* Generates improved strategy files on command

* Provides /orion/strategy_report and /orion/upgrade_strategy APIs

* Always SAFE: no disk writes, no service restarts, no auto-apply

3. Oracle — Microstructure Engine

File: oracle_main.py
Oracle replicates institutional liquidity signals normally available only to HFT desks. It generates synthetic depth, sweep detection, and micro-pulse directional bias.

Primary Responsibilities

* Real and synthetic order-book modeling

* JPY pip correction logic

* Spread/volatility expansion modeling

* Sweep detection (liquidity vacuum movements)

* Direction memory (to prevent flip-flopping)

* Session-weighted liquidity pressure

* Produces a unified microstructure rating for Athena

* Stores stateful microstructure memory in SQLite

4. Erebus — Sentiment Engine

File: erebus.py
Your sentiment engine that transforms news, macro indicators, and local heuristics into a simple bullish/bearish/neutral “risk regime.”

Primary Responsibilities

* Local rule-based sentiment scoring

* RISK-ON / RISK-OFF classification

* Filtering microstructure noise during macro-heavy periods

* API endpoint providing sentiment to Athena

* Light-weight and low-latency

5. Hermes — Execution Router

File: hermes_main.py
The dedicated execution engine responsible for routing orders to OANDA with strict safety, clean formatting, and no duplication.

Primary Responsibilities

* Receive trade commands from Athena

* Format/validate OANDA order payloads

* Submit orders through REST

* Retry logic and durable error handling

* Ensures Athena cannot spam OANDA

* Logs outbound execution attempts

6. Prometheus — Real P/L & Trade Integrity Engine

File: prometheus.py 

prometheus


Prometheus is the real-time P/L engine that pulls actual account fills from OANDA and writes finalized entry/exit rows into athena_trades.

Primary Responsibilities

* Poll OANDA transactions safely

* Detect real entry vs real exit fills

* Prevent duplicate fills (ID-set tracking)

* Calculate realized P/L, slippage, duration

* Update equity curve and daily P/L

* Ensure database integrity across sessions

* The single source of truth for P/L and trade outcomes

7. Ares — Order-Block & Structure Engine

File: ares.py
Lightweight module that identifies higher-timeframe structure and order-blocks used as contextual bias.

Primary Responsibilities

* Identify market structure (HH, HL, LL, LH)

* Detect supply/demand zones

* Provide confluence for directional bias

* Feed structure hints to Athena

8. Cerberus — System Guardian / Watchdog

File: cerberus.py
The system’s safety watchdog that ensures no module gets stuck, crashes silently, or violates runtime stability.

Primary Responsibilities

* Health-checks for all microservices

* Watchdog timers

* Automatic restart triggers (safe and isolated)

* Logs system-wide performance events

* Protects against latency spikes or deadlocks

Summary Diagram (Text-Based)
----------------------------

```text
Erebus (Sentiment)  ──►  Oracle (Microstructure)  ──►  Athena (Strategy Brain)  ──►  Hermes (Execution)  ──►  OANDA
                                                                                     │
                                                                                     ▼
                                                                   Prometheus (P&L / Feedback)  ──►  Orion (Upgrades)
              ┌───────────────────────┐
              │       Erebus         │
              │   (Sentiment AI)     │
              └─────────┬────────────┘
                        │  Bias / regime
                        ▼
              ┌───────────────────────┐
              │       Oracle         │
              │  (Microstructure)    │
              └─────────┬────────────┘
                        │  Liquidity / sweeps
                        ▼
        ┌───────────────────────┐        ┌───────────────────────┐
        │       Athena          │◄──────►│         Ares          │
        │   (Strategy Brain)    │  OB ctx│   (Structure / OB)    │
        └─────────┬────────────┘        └───────────────────────┘
                  │  Trade decisions
                  ▼
              ┌───────────────────────┐
              │       Hermes         │
              │   (Execution API)    │
              └─────────┬────────────┘
                        │  Orders / fills
                        ▼
                     OANDA (Broker)

                        ▲
                        │  Real P/L, equity, stats
              ┌─────────┴────────────┐
              │     Prometheus       │
              │ (Analytics / QA AI)  │
              └─────────┬────────────┘
                        │  Feedback / config proposals
                        ▼
              ┌───────────────────────┐
              │        Orion          │
              │  (Upgrade Orchestrator│
              │       – SAFE)         │
              └───────────────────────┘
