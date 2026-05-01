#  Cloud Deployment Simulation Lab
**Cloud Computing — GCYS2 | ENSAT 2025/2026**  
*Zakariae Jabiri · Mohamed Nour Hamdane*

---

##  Overview

A local virtualized environment simulating a production public cloud deployment. Four virtual machines replicate the architecture of a real cloud setup — complete with reverse proxying, TLS encryption, load balancing, firewall isolation, and a full observability stack.

> No cloud provider. No shortcuts. Just raw infrastructure from scratch.

---

##  Architecture

```
Internet
   │
   ▼
┌─────────────────────────────────┐
│         nginx_proxy             │  ← Only public-facing machine
│  Reverse Proxy · TLS · Firewall │
│       Load Balancer (RR)        │
└────────────┬────────────────────┘
             │ 10.10.10.0/24 (private)
     ┌───────┴────────┐
     ▼                ▼
┌─────────┐     ┌─────────┐     ┌───────────────┐
│web_srv_1│     │web_srv_2│     │  monitoring   │
│  Flask  │     │  Flask  │     │  Prometheus   │
│  :5000  │     │  :5000  │     │  + Grafana    │
└─────────┘     └─────────┘     └───────────────┘
```

| VM | IP | Role |
|---|---|---|
| `nginx_proxy` | 10.10.10.1 | Reverse proxy, Load balancer, Firewall |
| `web_server_1` | 10.10.10.2 | Flask web application |
| `web_server_2` | 10.10.10.3 | Flask web application |
| `monitoring` | 10.10.10.4 | Prometheus + Grafana |

---

##  Tech Stack

| Tool | Purpose |
|---|---|
| **Nginx** | Reverse proxy + Load balancer |
| **Flask (Python)** | Backend web application |
| **OpenSSL** | Self-signed TLS certificate |
| **UFW** | Firewall rules on all VMs |
| **Prometheus** | Metrics scraping (15s interval) |
| **Node Exporter** | System metrics on port 9100 |
| **Grafana** | Real-time dashboard visualization |
| **ApacheBench** | Load/stress testing |

---

##  Features

- **Reverse Proxy** — Nginx intercepts all traffic; backends are never directly reachable
- **HTTPS** — Self-signed cert via OpenSSL; HTTP → HTTPS redirect (301)
- **Load Balancing** — Round Robin across `web_server_1` and `web_server_2`
- **Firewall** — UFW rules enforce strict port-level isolation per VM
- **Monitoring** — Prometheus scrapes all 4 nodes; Grafana visualizes in real time
- **Stress Test** — 500 requests @ concurrency 10 via ApacheBench; 0 failures

---

##  Stress Test Results

```
ab -n 500 -c 10 https://192.168.111.128/

Complete requests:     500
Failed requests:       0
Requests per second:   969.69 [#/sec]
Time per request:      10.313 ms (mean)
Transfer rate:         309.66 KB/s
```

CPU spike visible on Grafana during the test — confirming live metric capture. 

---

##  Firewall Rules Summary

| VM | Port | Allowed From |
|---|---|---|
| `nginx_proxy` | 22, 80, 443 | Anywhere |
| `nginx_proxy` | 9100 | monitoring only |
| `web_server_1/2` | 5000 | proxy only (10.10.10.1) |
| `web_server_1/2` | 9100 | monitoring only (10.10.10.4) |
| `monitoring` | 3000, 9090, 9100 | Anywhere |

---

##  What We Learned

- How reverse proxies decouple public exposure from backend servers
- TLS termination at the proxy level — backends serve plain HTTP internally
- UFW doesn't flush `iptables` rules on `ufw disable` — had to manually run `iptables -F`
- Grafana rejects proxied origins by default — solved by direct NAT access to monitoring VM
- Real-time observability confirms infrastructure behavior under load

---

##  Full Report

 [cloud_project.pdf](./cloud_project.pdf)

---

> *Built as part of the GCYS2 Cloud Computing curriculum at ENSAT Tanger.*
