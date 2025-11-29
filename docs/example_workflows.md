# Example Workflows  
Athena AI FX Architecture  

This document provides real-world example workflows demonstrating how the system behaves during live trading.  
Each example shows how data flows across modules, how Athena generates decisions, and how Hermes executes safely.

---

# 1. BUY Trade Workflow (Trend Continuation)

### 1.1 Market Environment
- Sentiment: **RISK_ON**
- Oracle: positive microstructure, liquidity buildup below
- Ares: bullish structure (HL → HH)
- ATR: normal volatility
- Spread: acceptable
- Prometheus: stable P/L (no risk downgrade)

### 1.2 Pipeline Flow

OANDA → Erebus → Oracle → Ares → Athena → Hermes → OANDA → Prometheus


### 1.3 Step-by-Step

1. **Erebus**
   - Detects macro stability  
   - Outputs: `bias=BULL`, `confidence=0.64`

2. **Oracle**
   - Sweep detected below  
   - Expansion forming upward  
   - `oracle_bias=BULL`, `oracle_conf=0.72`

3. **Ares**
   - Pullback into bullish order block  
   - Structure: HL → HH  
   - `structure_conf=0.68`

4. **Athena**
   - Trend filter positive  
   - ATR gate passed  
   - Session risk multiplier = normal  
   - Generates signal:  
     ```
     side = BUY
     confidence = 0.83
     qty = 0.90 * base
     reason = "Trend Continuation + Microstructure Sweep"
     ```

5. **Hermes**
   - Spread OK  
   - Margin OK  
   - No duplicate trade  
   - Executes market order: BUY

6. **OANDA**
   - Fills trade  
   - Returns trade ID and fill price

7. **Prometheus**
   - Logs entry  
   - Tracks P/L in real time  

### 1.4 Exit Behavior
If microstructure momentum weakens or quantum-close triggers:

Athena → CLOSE → Hermes → OANDA


---

# 2. SELL Workflow (Reversal + Liquidity Sweep)

### 2.1 Environment  
- Erebus: `RISK_OFF` (but not blocking)  
- Oracle: strong bearish sweep  
- Ares: break of structure downward  
- ATR: high but within tolerance  

### 2.2 Step-by-Step

1. **Oracle**
   - Detects liquidity sweep above high  
   - Reversal probability spikes  
   - `oracle_bias=SELL`, `oracle_conf=0.81`

2. **Ares**
   - Breaker block forming  
   - Market shifts to lower-low structure  

3. **Athena**
   - Trend filter confirms reversal  
   - Generates:  
     ```
     side = SELL
     confidence = 0.78
     qty = 0.75 * base
     ```

4. **Hermes Execution**
   - Spread validated  
   - Sends SELL order  

5. **Prometheus**
   - Tracks drawdown and session performance  

6. **Exit**
   - If microstructure stall OR structure flips → CLOSE

---

# 3. Quantum-Close Example  

### Trigger Conditions:
- Trade goes +15 to +50 pips  
- Microstructure exhausts  
- Oracle detects compression  
- Ares sees structural slowdown  
- Athena sees confidence drop  
- Prometheus sees session saturation  

### Pipeline:

Oracle → Athena.quantum_should_close() → Hermes → OANDA

### Example Close Log:

quantum_close=True
reason="Momentum Fade + Microstructure Exhaustion"
exit_pips=+32.4

---

# 4. Pullback → Re-entry Workflow  

### Behavior:

1. **Initial BUY taken**
2. Price pulls back 8–12 pips  
3. Athena closes small green or small red  
4. Oracle detects new liquidity sweep  
5. Ares shows order block retest  
6. Athena **re-enters** with stronger confidence  
7. Trade captures full 20–50 pip move  

### Why it’s profitable:
- Avoids holding through unnecessary drawdown  
- Lets the system re-enter when the **real** momentum begins  
- Produces multiple “stacked green wins” instead of one choppy hold

---

# 5. High-Volatility Blocked Trade Workflow  

### Example:
Major news spike → spreads widen → price erratic.

### Chain:

1. Erebus → `RISK_OFF_HIGH`
2. Oracle → volatility expansion  
3. Athena → generates HOLD  
4. Hermes → blocks entry:  

reason="Spread too high"

### Why this matters:
You avoid catastrophic losses during news.

---

# End of Example Workflows
