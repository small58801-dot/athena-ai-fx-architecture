# Data Flow & Execution Pipeline – Athena AI FX Architecture

This document explains the full lifecycle of a trade, from data ingestion → analysis → decision → execution → feedback → adaptation.

It shows how each microservice collaborates to produce stable, high-quality trades in real time.

---

# 1. High-Level Data Flow Diagram

    ┌──────────┐
    │  Erebus  │  Sentiment (RISK_ON / RISK_OFF)
    └────┬─────┘
         ▼
    ┌──────────┐
    │  Oracle  │  Microstructure (Sweeps, Liquidity, Volatility)
    └────┬─────┘
         ▼
    ┌──────────┐
    │  Ares    │  Structure / OB Zones
    └────┬─────┘
         ▼
    ┌───────────────┐
    │   Athena      │  Strategy Brain
    │  (Fusion)     │
    │  - Trend      │
    │  - ATR Gate   │
    │  - Confidence │
    └────┬──────────┘
         ▼
    ┌──────────┐
    │  Hermes  │  Execution → OANDA
    └────┬─────┘
         ▼
    ┌───────────────┐
    │   OANDA API    │  Fills, Rejections, Spread
    └────┬──────────┘
         ▼
    ┌───────────────┐
    │  Prometheus    │  P/L, Win Rate, Equity Curve
    └────┬──────────┘
         ▼
    ┌──────────┐
    │  Orion   │  Safe Upgrades + Learning
    └──────────┘


---

# 2. Detailed Pipeline Description

## Step 1 — **Erebus (Sentiment Feed)**
- Classifies the environment as:
  - RISK_ON
  - RISK_OFF
  - NEUTRAL
- Integrates optional news + volatility factors.
- Produces a **sentiment packet**.

**Output →** Sentiment bias → Athena + Oracle

---

## Step 2 — **Oracle (Microstructure Engine)**
- Uses real OANDA price feed.
- Applies synthetic orderbook modeling.
- Detects:
  - Liquidity sweeps
  - Volatility expansions
  - Imbalance
  - Direction memory

**Output →**  
- microstructure_bias  
- sweep_flag  
- volatility_profile  

Sent to Athena (strategy brain).

---

## Step 3 — **Ares (Structure / OB Engine)**
- Detects fresh & historical order blocks.
- Identifies structure breaks.
- Evaluates OB strength.

**Output →**  
- OB Zones  
- Market structure trend  

Sent to Athena.

---

## Step 4 — **Athena (Strategy Brain)**
Athena fuses **all upstream data**:

| Input | Contribution |
|-------|--------------|
| Oracle | Microstructure bias + sweep confidence |
| Erebus | Global sentiment (risk filtering) |
| Ares | Structural confluence / OB validations |
| OANDA prices | Trend metrics + ATR |
| Prometheus | P/L performance risk scaling |

Athena computes:
- Trend strength
- ATR volatility gates
- Confidence score
- Position size
- BUY/SELL/HOLD decision

If validated:

**Output →** Execution packet → Hermes

---

## Step 5 — **Hermes (Execution Layer)**
- Pre-validates spread
- Checks margin availability
- Sends market order to OANDA
- Handles rejections & retry logic
- Saves execution logs

**Output →**  
- Position open / close  
- Fill details  
- Error packets  

Sent to Prometheus.

---

## Step 6 — **OANDA Broker**
- Provides authoritative execution and fills.
- Returns:
  - Fill price
  - Spread
  - Slippage
  - Live P/L stream

---

## Step 7 — **Prometheus (Performance Engine)**
- Tracks:
  - Real-time P/L
  - Win rate
  - Bias overperformance
  - Drawdown patterns
- Detects:
  - Bad sessions
  - Over-trading
  - Risk anomalies

**Output →** Adaptive learning metrics → Athena & Orion.

---

## Step 8 — **Orion (Safe Upgrade System)**
- Applies system upgrades gradually.
- Uses performance metrics to evaluate new configurations.
- Rolls back if performance degrades.
- Keeps version logs for the investor.

**Output →**  
Stable configuration + upgrade history.

---

# 3. End-to-End Example (BUY Flow)

1. Oracle detects bullish liquidity imbalance.  
2. Erebus outputs RISK_ON.  
3. Ares confirms bullish order block + BOS.  
4. Athena:
   - Trend strength: strong  
   - ATR volatility: acceptable  
   - Confidence = 0.82  
   - Position size = 0.0012  
5. Hermes sends BUY → OANDA.  
6. OANDA fills, spread 0.6 pips.  
7. Prometheus logs P/L and updates equity.  
8. Orion logs results for future upgrades.

---

# 4. End-to-End Example (CLOSE Flow)

1. Prometheus detects +18 pips unrealized.  
2. Oracle shows weakening microstructure.  
3. Athena triggers QUANTUM CLOSE.  
4. Hermes closes via OANDA.  
5. Prometheus archives trade with performance tags.  
6. Orion sees positive metrics → approves config.

---

# 5. Why This Pipeline Matters for Buyers

- Full modular microservice design  
- Extremely rare in retail FX systems  
- Clear separation of concerns  
- Automatic fail-safes  
- Scalable for hedge fund environments  
- Easy to extend (modules are isolated)

---

# End of Data Flow & Execution Documentation
