## Deployment & Services Guide
Quantum AI Ecosystem — Full Installation & Service Management

This document explains how to deploy, run, and manage the Quantum AI FX ecosystem in a production environment. It covers directory structure, environment variables, ports, systemd service files, log locations, and startup procedures.

## 1. SYSTEM REQUIREMENTS

Hardware:
- 4–8 vCPU
- 8–16 GB RAM
- SSD storage
- Low-latency stable Internet

Software:
- Ubuntu 20.04+ (or Debian-based OS)
- Python 3.10+
- systemd
- SQLite3
- FastAPI + Uvicorn

## 2. DIRECTORY STRUCTURE

Production layout used by the Quantum AI Ecosystem:

/opt/quantum/
    services/
    
- athena_brain.py
        
- hermes_main.py
        
- oracle_main.py
        
- ares.py
        
- erebus.py
        
- prometheus.py
        
- orion.py
        
- cerberus.py

#

    configs/
        .env                (NOT included in repo)
        .env.example        (included template)

    logs/
        athena.log
        hermes.log
        oracle.log
        ares.log
        erebus.log
        prometheus_service.log
        orion.log
        cerberus.log

    quantum.db             (SQLite database)

## 3. ENVIRONMENT VARIABLES (.env.example)

SYSTEM_NAME=QuantumAI_Elite

ENVIRONMENT=production

DB_PATH=/opt/quantum/quantum.db

LOG_DIR=/opt/quantum/logs

CONFIG_DIR=/opt/quantum/configs

# PORTS

###### HERMES_PORT=8200

###### PROMETHEUS_PORT=8300

###### EREBUS_PORT=8400

###### ATHENA_PORT=8580

###### ORION_PORT=8600

###### CERBERUS_PORT=8700

###### ARES_PORT=8900

###### ORACLE_PORT=8750

WEBHOOK_SECRET=your_secret_here

# USER MUST INSERT KEYS
- OANDA_API_KEY=your_oanda_key_here

- OANDA_ACCOUNT_ID=your_oanda_account_here

- OANDA_API_URL=https://api-fxtrade.oanda.com/v3

- OPENAI_API_KEY=your_openai_key_here

- TWELVEDATA_KEY=your_twelvedata_key_here

- NEWSAPI_KEY=your_newsapi_key_here

- FMP_API_KEY=your_fmp_key_here

- TWITTER_BEARER=your_twitter_token_here

- ALPHA_VANTAGE_KEY=your_alpha_key_here

- VIX_SYMBOL=^VIX

## 4. PORTS & SERVICES MAP

Module        | Port | Service Name
--------------|------|-----------------------
Hermes        | 8200 | hermes.service
Prometheus    | 8300 | prometheus.service
Erebus        | 8400 | erebus.service
Athena        | 8580 | athena.service
Orion         | 8600 | orion.service
Cerberus      | 8700 | cerberus.service
Oracle        | 8750 | oracle.service
Ares          | 8900 | ares.service

## 5. SYSTEMD SERVICE FILES

Place files in: /etc/systemd/system/

-----------------------------------------------
athena.service
-----------------------------------------------
[Unit]
Description=Athena – Trading Engine
After=network.target

[Service]
WorkingDirectory=/opt/quantum/services
ExecStart=/usr/bin/python3 /opt/quantum/services/athena_brain.py
Restart=always

[Install]
WantedBy=multi-user.target

-----------------------------------------------
hermes.service
-----------------------------------------------
[Unit]
Description=Hermes – Execution Bridge
After=network.target

[Service]
WorkingDirectory=/opt/quantum/services
ExecStart=/usr/bin/python3 /opt/quantum/services/hermes_main.py
Restart=always

[Install]
WantedBy=multi-user.target

-----------------------------------------------
oracle.service
-----------------------------------------------
[Unit]
Description=Oracle – Microstructure Engine

[Service]
ExecStart=/usr/bin/python3 /opt/quantum/services/oracle_main.py
Restart=always

[Install]
WantedBy=multi-user.target

-----------------------------------------------
erebus.service
-----------------------------------------------
[Unit]
Description=Erebus – Sentiment Engine

[Service]
ExecStart=/usr/bin/python3 /opt/quantum/services/erebus.py
Restart=always

[Install]
WantedBy=multi-user.target

-----------------------------------------------
ares.service
-----------------------------------------------
[Unit]
Description=Ares – Order Block Engine

[Service]
ExecStart=/usr/bin/python3 /opt/quantum/services/ares.py
Restart=always

[Install]
WantedBy=multi-user.target

-----------------------------------------------
prometheus.service
-----------------------------------------------
[Unit]
Description=Prometheus – Real P/L Engine

[Service]
ExecStart=/usr/bin/python3 /opt/quantum/services/prometheus.py
Restart=always

[Install]
WantedBy=multi-user.target

-----------------------------------------------
orion.service
-----------------------------------------------
[Unit]
Description=Orion – Strategy Evolution Engine

[Service]
ExecStart=/usr/bin/python3 /opt/quantum/services/orion.py
Restart=always

[Install]
WantedBy=multi-user.target

-----------------------------------------------
cerberus.service
-----------------------------------------------
[Unit]
Description=Cerberus – Health Monitor

[Service]
ExecStart=/usr/bin/python3 /opt/quantum/services/cerberus.py
Restart=always

[Install]
WantedBy=multi-user.target

## 6. SERVICE MANAGEMENT COMMANDS

- Start a service:

    sudo systemctl start athena.service

- Restart a service:

    sudo systemctl restart hermes.service

- Check status:

    sudo systemctl status prometheus.service

- Stop a service:

    sudo systemctl stop orion.service

- Enable all services on boot:

    sudo systemctl enable *.service

## 7. LOG LOCATIONS

All logs stored at:
/opt/quantum/logs/

Tail logs live:

- tail -f /opt/quantum/logs/athena.log

- tail -f /opt/quantum/logs/hermes.log

- tail -f /opt/quantum/logs/oracle.log

- tail -f /opt/quantum/logs/erebus.log

## 8. DATABASE (SQLite)

Database located at:

- /opt/quantum/quantum.db

List tables:

- sqlite3 /opt/quantum/quantum.db ".tables"

Backup:

- cp /opt/quantum/quantum.db /opt/quantum/backup_$(date +%F).db

## 9. FIRST-TIME BOOT ORDER

Start modules in this order:

1. Ares
2. Oracle
3. Erebus
4. Prometheus
5. Hermes
6. Athena
7. Orion
8. Cerberus


## 10. VERIFY SYSTEM IS RUNNING

- curl localhost:8580/athena/status

- curl localhost:8200/health

- curl localhost:8400/health

- curl localhost:8300/summary

- curl localhost:8750/signal/EUR_USD

If all endpoints respond with JSON, the system is fully operational.

## END OF FILE
