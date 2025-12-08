# Athena – Multi-Module AI FX Trading Architecture  
**Risk Engine • Microstructure • Sentiment • Execution • Strategy Brain**

This repository documents a complete multi-service AI FX trading architecture built over two years of engineering, live testing, and system iteration.  
It is not a traditional EA or a single-file bot — it is a modular, full-stack architecture designed for expansion, research, and real-time trading.

---

## 🔷 System Overview

The architecture is composed of independently running services communicating through REST APIs.  
Each module is responsible for a distinct layer of market understanding, execution, or analytics:

### **1. Athena — Strategy & Decision Layer**
- Signal generation from multi-timeframe price action  
- Trend analysis, volatility filters, ATR gates  
- Entry scoring, confidence scoring, execution routing to Hermes  
- Avoids over-trading and adapts to market sessions  

### **2. Orion — Risk Intelligence**
- Multi-factor risk scoring  
- Volatility/ATR gating  
- Trend confirmation  
- Over-extension protection  
- Position sizing engine (dynamic tiers)

### **3. Oracle — Institutional Microstructure Engine**
- Synthetic + live depth fusion  
- Liquidity sweeps  
- Micro-imbalances  
- Direction memory  
- JPY pip correction  
- Session-weighted microstructure

### **4. Erebus — Sentiment Engine**
- Macro sentiment (risk-on/risk-off)  
- Bias scoring  
- Market regime classification  

### **5. Hermes — Execution Layer**
- Routes market orders  
- Slippage control  
- Retry logic  
- Spread awareness  
- Trade confirmation logging  

### **6. Prometheus — Analytics Layer**
- Performance logging  
- PnL tracking  
- Trade journaling  
- Daily summaries  
- Adaptive feedback for Athena

### **7. Ares — Structure / Order Block Model**
- Structure detection  
- Fair value gaps  
- Order block analysis  
- Market structure shift identification

---

## 🔷 What This Architecture Is Designed For

- Real-time FX trading  
- Multi-layer signal confirmation  
- Institutional-style microstructure + macro + technical fusion  
- Extensible research platform for quant teams  
- Low-latency Python-service pipeline  
- Modular components that can be independently upgraded

This repository does **not** include the proprietary code — only documentation, structure, service descriptions, and system overview.  
The full production codebase is available for licensing or acquisition.

---

## 🔷 Licensing / Acquisition

I’m open to discussions with:

- Quant trading teams  
- Hedge funds  
- Prop firms  
- Algo trading researchers  
- Independent quants seeking a full-stack foundation

### **Current licensing structure**
- **$5k upfront** for a non-exclusive license  
- **+ $60k Exclusive License**   

If you want to review, validate, or build on top of the architecture, email:

**📩 small58801@gmail.com**

---

## 🔷 Why This Repo Exists

This repository acts as a **public, transparent portfolio piece** showing:

- The architecture  
- The engineering thought process  
- The modular design philosophy  
- The structure of a production FX AI system  

Serious buyers can request the private full codebase.

---

## 🔷 Contact

For acquisition, licensing, or collaboration:  
📧 **small58801@gmail.com**

