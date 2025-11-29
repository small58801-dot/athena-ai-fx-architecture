# Athena AI FX Architecture — System Overview
### **1. Introduction**

Athena is a modular, production-tested AI trading architecture designed for real-time FX execution.
Unlike single-script EAs or monolithic algorithms, Athena is a distributed multi-service system composed of independent engines for:

* Signal generation

* Risk analysis

* Market microstructure

* Sentiment interpretation

* Execution routing

* Performance telemetry

Each module runs as its own service with clearly defined boundaries, data contracts, and operational responsibilities.

The architecture is designed to be extensible, interpretable, and suitable for institutional environments where stability, modularity, and resilience matter.

### **2. High-Level System Goals**

Athena is engineered to deliver:

✔ High-quality entries through multi-factor confluence

Trend, volatility, strength, sentiment, and liquidity alignment.

✔ Stable, adaptive exits based on real-time conditions

Dynamic take-profit, adverse-move detection, and microstructure reversals.

✔ Modular design for institutional scaling

Each service can be improved or replaced without breaking the entire system.

✔ Real production behavior

The system runs live against OANDA with real trades, logs, data persistence, and robust error handling.

### **3. Core Components**

Athena is composed of six primary modules:

A. Athena — Strategy & Decision Engine

* Multi-timeframe trend evaluation

* ATR volatility gating

* Confidence scoring

* Oracle + sentiment fusion

* Entry/exit decision logic

* Generates BUY/SELL/HOLD signals

* Sends execution requests to Hermes

Athena is the “brain” of the system.

B. Oracle — Microstructure Engine

* Synthetic liquidity modeling

* Direction memory

* Sweep detection

* Reversal probability scoring

* ATR-aware volatility modifier

* JPY pair correction

* Order-book style premium estimation

Oracle provides the “market internals” layer institutions rely on.

C. Erebus — Sentiment Model

* News sentiment (RISK-ON / RISK-OFF)

* Social sentiment (Bullish / Bearish)

* Macro mood scoring

* Fusion into Athena’s confidence model

Erebus gives the system macro context behind price moves.

D. Orion — Risk/Signal Fusion Layer

* Normalizes scores from multiple modules

* Applies risk factors (ATR, trend, strength)

* Adjusts position size recommendations

* Ensures signals meet minimum thresholds

Orion acts as an intelligent filter before trades ever reach execution.

E. Hermes — Execution Layer

* Routes orders to OANDA

* Validates units, direction, and session timing

* Ensures no duplicate orders

* Handles market rejects and retry logic

* Logs all trade activity to the database

Hermes is the “broker interface” layer.

F. Prometheus — Analytics & Feedback Engine

* Logs performance (P/L, accuracy, session data)

* Monitors adaptation score

* Feeds real-time stats back to Athena

* Generates ongoing improvement signals

Prometheus is the long-term optimization engine.

### **4. Data Architecture**

* Athena uses a centralized SQLite database for:

* Trade history

* Daily P/L aggregation

* Session logs

* Oracle liquidity metrics

* Risk metrics

This allows:

* Stable persistence

* Cross-module read/write access

* Real-time adaptation loops

### **5. Execution Flow (High-Level)**

* Athena scans market

* Pulls sentiment from Erebus

* Pulls microstructure from Oracle

* Fuses everything via Orion

* Computes position size

* Signals BUY/SELL/HOLD

* Hermes executes the order

* Prometheus logs performance

* Athena adapts using feedback

This creates a full-circle decision ecosystem.

### **6. Why This Architecture Matters**

This is not an EA.
This is not an ML toy.
This is not a single-file bot.

Athena is a full commercial architecture, the kind used in:

* Prop firms

* Quant funds

* Institutional execution engines

* AI-driven hedge fund tooling

The system is structured so a buyer can:

* Extend modules

* Replace services

* Add ML/LLM layers

* Scale infrastructure

* Integrate new data feeds

It’s built realistically — the way real firms operate.

### **7. Summary**

Athena AI FX Architecture provides:

* Professional multi-module design

* True real-time operation

* Clear module responsibilities

* Production-level data flows

* Deep expandability

* Strong real-world entry logic

* Improving exit logic with dynamic scoring

This document serves as a high-level guide to the full system and its engineering-grade structure.
