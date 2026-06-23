# Hybrid-SOC-Penetration-Testing-Lab-Architecture
I used JavaScript with Node.js and Express.js to build REST APIs for handling security logs and real-time SOC data

                  📊 Complete System Architecture Diagram

                        ┌──────────────────────────────┐
                        │      FRONTEND DASHBOARD      │
                        │ (Windows + VS Code)          │
                        │ HTML + CSS + JS              │
                        │                              │
                        │ • Real-time Alerts           │
                        │ • Attack Visualization       │
                        │ • Network Traffic Graph      │
                        │ • Logs & Charts              │
                        └──────────────┬───────────────┘
                                       │
                                       │ REST API / Socket.io
                                       ▼
        ┌──────────────────────────────────────────────────┐
        │         NODE.JS + EXPRESS (SOC CORE ENGINE)      │ 
        │                                                  │
        │  ┌────────────────────────────────────────────┐  │
        │  │  Log Ingestion API                         │  │
        │  │  /api/nmap  /api/logs  /api/pcap           │  │
        │  └────────────────────────────────────────────┘  │
        │                                                  │
        │  ┌────────────────────────────────────────────┐  │
        │  │  Security Analysis Engine                  │  │
        │  │  - Port scan detection (Nmap)              │  │
        │  │  - Exploit detection (Metasploit)          │  │
        │  │  - Packet inspection (Wireshark)           │  │
        │  └────────────────────────────────────────────┘  │
        │                                                  │
        │  ┌────────────────────────────────────────────┐  │
        │  │  Alert Generation System                   │  │
        │  │  - High / Medium / Low severity alerts     │  │
        │  └────────────────────────────────────────────┘  │
        └──────────────────────┬───────────────────────────┘
                               │
                               │ Structured Logs (JSON)
                               ▼
                ┌──────────────────────────────────┐
                │         MONGODB DATABASE         │
                │                                  │
                │  Collections:                    │
                │  • logs                          │
                │  • alerts                        │
                │  • traffic_data                  │
                └──────────────┬───────────────────┘
                               │
                               │ Data Fetch / Query
                               ▼
        ┌──────────────────────────────────────────────────┐
        │              VISUALIZATION LAYER                 │
        │                                                  │
        │  • Charts (Attack trends)                        │
        │  • IP tracking map                               │
        │  • SOC timeline view                             │
        │  • Live incident feed                            │
        └──────────────────────┬───────────────────────────┘
                               │
                               │ Data Injection (Logs/PCAP)
                               ▼
        ┌──────────────────────────────────────────────────┐
        │             KALI LINUX (ATTACK VM)               │
        │                                                  │
        │  ┌──────────────┐  ┌─────────────────┐           │
        │  │ Nmap         │  │ Metasploit      │           | 
        │  │ (Scanning)   │  │ (Exploitation)  │           │
        │  └──────────────┘  └─────────────────┘           |
        │                                                  │
        │  ┌────────────────────────────────────────────┐  │
        │  │ Wireshark (Packet Capture / PCAP files)    │  │
        │  └────────────────────────────────────────────┘  │
        │                                                  │
        │ Output:                                          │
        │ • Scan results                                   │
        │ • Exploit logs                                   │
        │ • Network packets                                │
        └──────────────────────────────────────────────────┘
