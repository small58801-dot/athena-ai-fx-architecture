# API Endpoints
### Quantum AI FX Architecture — Full Internal API Specification

This document lists every REST endpoint exposed across all microservices in the Quantum AI Ecosystem.

Each module is a FastAPI service running locally, communicating over internal ports:

| Module       | Port | Purpose                               |
|--------------|------|-----------------------------------------|
| Athena       | 8580 | Core trading logic engine               |
| Hermes       | 8200 | Execution bridge (OANDA)                |
| Erebus       | 8400 | Sentiment & macro engine                |
| Prometheus   | 8300 | Real P/L + equity engine                |
| Ares         | 8900 | Order-block / CHoCH engine              |
| Oracle       | 8750 | Microstructure + liquidity              |
| Orion        | 8600 | Strategy evolution engine               |
| Cerberus     | 8700 | Health monitoring (uses Orion’s port)   |

---

# 1. Athena (Port 8580)

### **1.1 GET /athena/status**
- Returns heartbeat, runtime info, and uptime.

### **1.2 POST /athena/sentiment**
- Sentiment push from Erebus.

# Payload:

{
  "macro_signal": "RISK_ON",
  "mood_index": 0.42,
  "vix": 14.5
}

## 2. Hermes – Execution Bridge (Port 8200)
### **2.1 GET /health**
- Returns Hermes status.

### **2.2 GET /account_state**
- Fetches OANDA account summary (NAV, margin, balance).

### **2.3 GET /open_positions**
- Returns open positions from OANDA.

### **2.4 POST /execute**
- Main execution endpoint called by Athena.

# Payload:

{
  "secret": "ysm_jmk",
  "symbol": "EUR_USD",
  "action": "buy",
  "qty": 120
}

### **Hermes will:**

* Filter non-trade actions (hold/skip/wait)

* Auto-close opposite positions

* Submit order to OANDA

* Log trade to DB

* Notify Prometheus

# 3. Erebus – Sentiment Engine (Port 8400)
### **3.1 GET /health**

- Healthcheck endpoint.

### **3.2 GET /sentiment**

- Main public endpoint Athena calls.

### **Response:**

{
  "macro_signal": "RISK_ON",
  "mood_index": 0.31,
  "vix": 15.0
}

### **3.3 POST /broadcast**

- Internal pipeline to send updates to Athena/Hermes.

# 4. Ares v12 – Order Block Engine (Port 8900)
### **4.1 GET /context/{symbol}**

Example: /context/EUR_USD

Returns:

* CHoCH / BOS pattern

* order blocks

* liquidity sweeps

* trend direction

* reversal/breakout info

* combined strength score

### **Response example:**

{
  "pattern": "BOS_UP",
  "direction": "UP",
  "strength": 0.71,
  "reversal": false,
  "breakout": false
}

# 5. Oracle v3.6 – Microstructure Engine (Port 8750)
### **5.1 GET /signal/{symbol}**

Example: /signal/EUR_USD

Returns:

* bias (BUY / SELL / NEUTRAL)

* strength

* sweep + direction

* micro_score

* spread

* synthetic depth map

* volatility estimate

Used by Athena to refine entries/exits.

# 6. Prometheus – Real P/L & Equity Engine (Port 8300)
### **6.1 GET /summary**

Returns:

* daily performance

* daily P/L

* realized wins/losses

* performance trajectory

### **6.2 POST /log**

- Hermes sends real-time execution logs here.

- Background Internal Calls

### **(Not public endpoints):**

* /transactions

* /accounts/{id}

Used to reconcile OANDA-filled orders.

# 7. Orion Omega v9.0 – Strategy Engine (Port 8600)
### **7.1 GET /orion/health**

- Healthcheck.

### **7.2 GET /orion/strategy_report**

Returns:

* trade stats (win rate, avg P/L)

* Monte Carlo simulation

* upgrade recommendations

### **7.3 POST /orion/upgrade_strategy**

Returns reconstructed strategy code for Athena.

# IMPORTANT:
Orion does NOT apply upgrades — only returns files for you to manually review.

# 8. Cerberus – Health Orchestrator (Port 8600)

### **Cerberus does not expose many endpoints.**
It pulls from other modules:

* GET Athena status

* GET Erebus health

* GET Hermes health

* GET Prometheus health

* GET Orion health

### **8.1 POST /athena/broadcast**

- Cerberus sends messages to Athena.

Cerberus is mostly an internal scheduler + watchdog.

# 9. System-Wide Interaction Map

| From      | To                           | Endpoint                     | Purpose                      |
|-----------|-------------------------------|-------------------------------|------------------------------|
| Athena    | Hermes                       | `/execute`                   | Execute trades               |
| Athena    | Oracle                       | `/signal/{symbol}`           | Microstructure data          |
| Athena    | Erebus                       | `/sentiment`                 | Macro sentiment              |
| Athena    | Ares                         | `/context/{symbol}`          | Order block + CHoCH          |
| Athena    | Prometheus                   | `/summary`                   | Performance feedback         |
| Hermes    | OANDA                        | `/v3/orders`                 | Live execution               |
| Prometheus| OANDA                        | `/transactions`              | Fill reconciliation          |
| Erebus    | Athena                       | `/athena/sentiment`          | Push sentiment updates       |
| Cerberus  | Athena                       | `/athena/broadcast`          | Alerts / motivation          |
| Orion     | User (external)              | `/orion/upgrade_strategy`    | Strategy file generation     |

# End of File
