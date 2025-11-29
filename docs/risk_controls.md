# Risk Controls & Safety Systems  
### **Athena AI FX Architecture**

This document describes the layered safety and risk-management controls across all modules.  
The system is designed so **no single module can cause catastrophic loss**, and multiple independent layers verify conditions before any trade is executed.

---

# 1. High-Level Risk Architecture

The architecture uses **8 layers** of risk protection:

1. **Sentiment Gate (Erebus)**
2. **Microstructure Gate (Oracle)**
3. **Structure Gate (Ares)**
4. **Trend Strength Filter (Athena)**
5. **ATR Volatility Gate (Athena)**
6. **Position Sizing Engine (Athena)**
7. **Spread / Slippage Validation (Hermes)**
8. **P/L Monitoring & Kill-Switch (Prometheus)**
9. **Safety Watchdog & Auto-Recovery (Cerberus)**

Each layer must pass before reaching execution.

---

# 2. Sentiment Risk Gate — Erebus

Erebus outputs environmental risk classification:

- **RISK_ON** → bias toward BUY setups  
- **RISK_OFF** → bias toward SELL setups  
- **NEUTRAL** → only highest confidence trades allowed  

Athena uses this to automatically:

- Block trades during macro instability  
- Reduce size during uncertainty  
- Boost size during aligned sentiment

This eliminates many false trades during volatile news windows.

---

# 3. Microstructure Gate — Oracle

Oracle enforces **order-flow-informed** risk constraints:

- Detects liquidity sweeps
- Identifies low-liquidity pockets
- Monitors volatility expansion
- Tracks direction memory
- Measures order imbalance

Athena uses Oracle’s flags to:

- Avoid entries during whipsaws  
- Enter only when microstructure is supportive  
- Close early when sweeps indicate reversal  
- Reduce size when volatility is excessive

This dramatically increases win-rate stability.

---

# 4. Structure Gate — Ares (Order Block / Structure Engine)

Ares enforces structural confluence:

- Validates trend structure (HH/HL or LH/LL)
- Filters trades against major order blocks
- Only allows entries near strong OB zones
- Blocks trades inside chop zones

Athena receives structure_score and OB proximity, tightening trade quality.

---

# 5. Trend Strength Filter — Athena

A core filter:

Trade is blocked unless:

trend_strength >= TREND_STRENGTH

Trend strength is computed from:

- Multi-timeframe EMAs
- Market slope
- Direction momentum
- Recent candle bias

This prevents countertrend scalps and random entries.

---

# 6. ATR Volatility Gate — Athena

One of the strongest safety systems:

Trades only occur when:

ATR >= ATR_GATE


Where ATR_GATE is dynamically scaled using:

- Symbol-specific risk tier  
- Session multiplier  
- Volatility class  

Examples:

- Low ATR → Block trade (chop avoidance)
- Mid ATR → Allow small size
- High ATR → Allow regular size  
- Extreme ATR → Reduce size or block entirely

This stops trades during dead markets and panic markets.

---

# 7. Position Sizing Engine — Athena

Dynamic sizing uses:

- ATR class
- Trend confidence
- Session multiplier
- Account equity feedback
- Prometheus performance feedback

Final size is determined through a multi-step scaler:

qty_base
→ ATR scaler
→ trend scaler
→ sentiment scaler
→ session multiplier
→ final size


This ensures size is proportionate to real conditions.

---

# 8. Spread / Slippage Validation — Hermes

Before sending ANY order to OANDA:

Hermes validates:

- spread <= configured limit  
- margin available  
- account health  
- execution cooldown  

If spread is too high:

**Trade is automatically cancelled**, even if Athena gives a valid BUY/SELL.

This prevents precision entries from becoming instant losses.

---

# 9. P/L Monitoring — Prometheus

Prometheus enforces live performance risk rules:

- Stops trading after **daily drawdown threshold**
- Adjusts size after streaks
- Reduces aggression during losing sessions
- Scales aggression during strong sessions
- Tracks:
  - equity curve
  - session win-rate
  - volatility-adjusted P/L
  - drawdown recovery

If performance turns negative:

Prometheus sends:

RISK_DOWNGRADE


Athena automatically reduces size until recovery.

---

# 10. Daily Kill-Switch

If daily drawdown exceeds threshold:

- All existing trades CLOSE  
- New trades are blocked  
- System enters SAFE MODE  
- Prometheus logs the event  
- Cerberus enforces downtime  

This prevents spiraling losses.

---

# 11. Safety Watchdog — Cerberus

Cerberus prevents catastrophic failures:

- Restarts modules if they hang  
- Pings API health  
- Detects deadlocks  
- Enforces CPU/memory safe conditions  
- Recovers from service crashes  
- Auto-restarts Hermes after failed execution cycles  
- Ensures no module can “go dark”

This gives institutional-level reliability.

---

# 12. Crash-Safe Recovery

If Athena, Hermes, Oracle, or any module fails:

- Cerberus restarts it
- Prometheus provides last known P/L + state  
- Athena loads last known configuration  
- Orion rolls back unsafe changes  
- Hermes ensures open orders are synced with OANDA

No human intervention needed.

---

# 13. Combined Risk Flow Diagram

Sentiment Gate (Erebus)
↓ PASS
Microstructure Gate (Oracle)
↓ PASS
Structure Gate (Ares)
↓ PASS
Trend Filter (Athena)
↓ PASS
ATR Volatility Gate (Athena)
↓ PASS
Position Sizing Engine (Athena)
↓ PASS
Spread Check (Hermes)
↓ PASS
OANDA Execution
↓
Prometheus Monitoring
↓
Orion Safe-Upgrades + Long-term supervision


---

# End of Risk Controls Documentation
