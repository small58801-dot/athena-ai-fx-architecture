# TROUBLESHOOTING GUIDE
### **Quantum AI Ecosystem – Diagnostics, Recovery, and Common Fixes**

This guide provides diagnostic procedures, common fixes, recovery steps, and 
verification commands for all modules in the Quantum AI Ecosystem.

Use this file when something fails, a service won’t start, trades do not execute,
or data stops flowing.

### **1. GENERAL DIAGNOSTICS**

-----------------------------------------------
**Check service status**
-----------------------------------------------
- sudo systemctl status athena.service
- sudo systemctl status hermes.service
- sudo systemctl status oracle.service
- sudo systemctl status erebus.service
- sudo systemctl status ares.service
- sudo systemctl status prometheus.service
- sudo systemctl status orion.service
- sudo systemctl status cerberus.service

Look for:
- "active (running)"
- errors
- missing dependencies
- Python path issues
- missing environment variables

-----------------------------------------------
**Restart a module**
-----------------------------------------------
sudo systemctl restart athena.service

-----------------------------------------------
**Start all modules (in correct order)**
-----------------------------------------------
1. sudo systemctl start ares.service
2. sudo systemctl start oracle.service
3. sudo systemctl start erebus.service
4. sudo systemctl start prometheus.service
5. sudo systemctl start hermes.service
6. sudo systemctl start athena.service
7. sudo systemctl start orion.service
8. sudo systemctl start cerberus.service

-----------------------------------------------
**Tail logs (most important debugging tool)**
-----------------------------------------------
- tail -f /opt/quantum/logs/athena.log
- tail -f /opt/quantum/logs/hermes.log
- tail -f /opt/quantum/logs/oracle.log
- tail -f /opt/quantum/logs/erebus.log
- tail -f /opt/quantum/logs/prometheus_service.log

If the logs are empty → the service is not starting.

### **2. COMMON ISSUES & FIXES**

-----------------------------------------------
**(A) Athena is stuck on HOLD / no trades**
-----------------------------------------------
Possible causes:
1. Erebus not returning sentiment
2. Oracle not returning microstructure data
3. Hermes not reachable (poor connectivity triggers HOLD)
4. ATR too low or volatility filters too strict
5. Trend strength failing internal thresholds
6. Overtrading protection triggered

Fixes:
- Check Erebus endpoint: curl localhost:8400/health
- Check Oracle endpoint: curl localhost:8750/signal/EUR_USD
- Check Hermes status: sudo systemctl status hermes.service
- Confirm DB working: sqlite3 /opt/quantum/quantum.db ".tables"

-----------------------------------------------
**(B) Hermes is rejecting orders / no execution**
-----------------------------------------------
Causes:
- Wrong OANDA API key
- Wrong OANDA account ID
- API rate limit
- Market closed or instrument disabled

Fixes:
- Test OANDA auth:
  curl -H "Authorization: Bearer YOUR_KEY" https://api-fxtrade.oanda.com/v3/accounts
- Confirm instrument tradable in your region
- Confirm spread isn't > risk limits

-----------------------------------------------
**(C) Oracle returning "None" or missing fields**
-----------------------------------------------
Causes:
- API timeout to OANDA price feed
- SQLite write/read issues
- Missing granularity data

Fixes:
- Restart Oracle: sudo systemctl restart oracle.service
- Check internet connection
- Clear corrupted rows:
  sqlite3 /opt/quantum/quantum.db "DELETE FROM oracle_liquidity WHERE id > 0;"

-----------------------------------------------
**(D) Erebus sentiment always NEUTRAL or DOWN**
-----------------------------------------------
Causes:
- Missing API keys for sentiment feeds
- Rate limits
- JSON parse failures

Fix:
- Check logs: tail -f /opt/quantum/logs/erebus.log
- Verify external API keys in .env.example

-----------------------------------------------
**(E) Prometheus shows 0 P/L or no updates**
-----------------------------------------------
Causes:
- OANDA transaction endpoint unreachable
- No active positions
- DB locked by another process

Fix:
- Restart Prometheus
- Verify DB access:
  sqlite3 /opt/quantum/quantum.db ".tables"

-----------------------------------------------
**(F) Cerberus spamming alerts**
-----------------------------------------------
Causes:
- Services not responding fast enough
- CPU overload on server
- Memory pressure

Fix:
Check system load: top or htop
Restart cluster in order:
  - sudo systemctl restart ares.service && \
  - sudo systemctl restart oracle.service && \
  - sudo systemctl restart erebus.service && \
  - sudo systemctl restart prometheus.service && \
  - sudo systemctl restart hermes.service && \
  - sudo systemctl restart athena.service && \
  - sudo systemctl restart orion.service && \
  - sudo systemctl restart cerberus.service


### **3. PORT & CONNECTIVITY CHECKS**

-----------------------------------------------
**Test module endpoints directly**
-----------------------------------------------
- Athena      → curl localhost:8580/athena/status
- Hermes      → curl localhost:8200/health
- Erebus      → curl localhost:8400/health
- Prometheus  → curl localhost:8300/summary
- Oracle      → curl localhost:8750/signal/EUR_USD
- Ares        → curl localhost:8900/context/EUR_USD
- Orion       → curl localhost:8600/version
- Cerberus    → curl localhost:8700/status

If any return HTML instead of JSON → service boot failed.

-----------------------------------------------
**Test port availability**
-----------------------------------------------
- sudo lsof -i :8580
- sudo lsof -i :8200
- sudo lsof -i :8400
- sudo lsof -i :8300
- sudo lsof -i :8750
- sudo lsof -i :8900
- sudo lsof -i :8600
- sudo lsof -i :8700

If "no process found" → the service is not running.


### **4. DATABASE FAILURES**

-----------------------------------------------
**Check SQLite file exists**
-----------------------------------------------
ls -l /opt/quantum/quantum.db

-----------------------------------------------
**List all tables**
-----------------------------------------------
sqlite3 /opt/quantum/quantum.db ".tables"

-----------------------------------------------
**Fix locked DB**
-----------------------------------------------
rm -f /opt/quantum/quantum.db-shm
rm -f /opt/quantum/quantum.db-wal

-----------------------------------------------
**Backup DB (recommended weekly)**
-----------------------------------------------
cp /opt/quantum/quantum.db /opt/quantum/backup_$(date +%F).db


### **5. API & NETWORK ISSUES**

-----------------------------------------------
**Test OANDA connectivity**
-----------------------------------------------
curl -H "Authorization: Bearer YOUR_KEY" \
https://api-fxtrade.oanda.com/v3/accounts

If this fails → Hermes cannot execute trades.

-----------------------------------------------
**Test external API dependencies**
-----------------------------------------------
TwelveData  → check key
NewsAPI     → check key
Twitter API → check bearer token
AlphaVantage → check key

-----------------------------------------------
**If these fail, Erebus may output:**
- sentiment = NEUTRAL
- risk_off = false
- news_score = 0

Fix by updating .env keys.


### **6. SERVICE DOES NOT START** 

-----------------------------------------------
**Check systemd logs**
-----------------------------------------------
sudo journalctl -u athena.service -n 50 --no-pager

Look for:
- SyntaxError
- ImportError
- Missing module
- Bad indentation
- Missing environment variables

-----------------------------------------------
**Check Python path**
-----------------------------------------------
which python3

-----------------------------------------------
**Reinstall dependencies**
-----------------------------------------------
pip install fastapi uvicorn requests numpy pandas


### **7. FULL SYSTEM RESET PROCEDURE** 

-----------------------------------------------
**(1) Stop all modules**
-----------------------------------------------
- sudo systemctl stop athena.service
- sudo systemctl stop hermes.service
- sudo systemctl stop oracle.service
- sudo systemctl stop erebus.service
- sudo systemctl stop prometheus.service
- sudo systemctl stop orion.service
- sudo systemctl stop ares.service
- sudo systemctl stop cerberus.service

-----------------------------------------------
**(2) Clear WAL/SHM database files**
-----------------------------------------------
- rm -f /opt/quantum/*.wal
- rm -f /opt/quantum/*.shm

-----------------------------------------------
**(3) Restart in correct order**
-----------------------------------------------
1. sudo systemctl start ares.service
2. sudo systemctl start oracle.service
3. sudo systemctl start erebus.service
4. sudo systemctl start prometheus.service
5. sudo systemctl start hermes.service
6. sudo systemctl start athena.service
7. sudo systemctl start orion.service
8. sudo systemctl start cerberus.service


### **8. WHEN TO CONTACT DEVELOPER** 


If ANY of these occur:

- None of the modules respond to curl  
- Trades execute but P/L stays at 0  
- Erebus or Oracle logs are empty  
- Prometheus stops updating positions  
- Athena crashes during live market volatility  
- DB corruption warnings repeat  
- Cerberus permanently reports failures  

These indicate deeper architectural or system-level issues.

# END OF FILE
