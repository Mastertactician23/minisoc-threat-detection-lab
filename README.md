# MiniSOC: Threat Detection Lab
### A Home SOC Environment for Simulated Attack Detection & Incident Analysis

**Author:** Kofi Asibey-Kitiabi  
**GitHub:** [Mastertactician23](https://github.com/Mastertactician23/)  
**LinkedIn:** [asibey-kitiabi](https://www.linkedin.com/in/asibey-kitiabi/)  
**Date:** June 2026  **Status:** Completed  


---

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [Objectives](#2-objectives)
3. [Lab Architecture](#3-lab-architecture)
4. [Tools & Technologies](#4-tools--technologies)
5. [Project Phases](#5-project-phases)
6. [Attack Simulation](#6-attack-simulation)
7. [Detection & Analysis](#7-detection--analysis)
8. [Key Findings](#8-key-findings)
9. [MITRE ATT&CK Mapping](#9-mitre-attck-mapping)
10. [Challenges & How I Solved Them](#10-challenges--how-i-solved-them)
11. [Skills Demonstrated](#11-skills-demonstrated)
12. [Evidence](#12-evidence)
13. [Recommendations](#13-recommendations)
14. [What I Would Do Differently](#14-what-i-would-do-differently)
15. [Next Steps](#15-next-steps)

---

## 1. Project Overview

MiniSOC is a self-built, containerised Security Operations Centre (SOC) lab environment designed to simulate real-world cyber attack scenarios and practice blue team detection skills. The project was built entirely on a personal laptop using Docker, without the need for expensive hardware or cloud infrastructure.

The lab replicates a core SOC workflow: an attacker performs reconnaissance, exploits a misconfiguration, and launches a brute force attack — while a SIEM (Elastic Stack) captures and indexes all network traffic, allowing the analyst to detect, investigate, and document the attack using the same tools used in professional SOC environments.

This project was built as part of an active transition from IT and SAP support into cybersecurity, specifically targeting SOC Analyst and Security Analyst roles.

---

## 2. Objectives

- Build a functional SOC lab environment using only free, open-source tools
- Simulate a realistic multi-phase attack against a deliberately vulnerable target
- Detect attack activity using a SIEM and network traffic analysis
- Produce professional incident documentation matching real-world SOC output
- Demonstrate hands-on ability with tools directly listed in SOC Analyst job descriptions: Elastic Stack, Nmap, Wireshark-class traffic capture, and incident reporting

---

## 3. Lab Architecture

```
┌─────────────────────────────────────────────────────┐
│              Docker Bridge Network: soc-lab          │
│                   Subnet: 172.18.0.0/16              │
│                                                      │
│  ┌─────────────────┐      ┌──────────────────────┐  │
│  │   ctf-solver    │      │       target         │  │
│  │  Kali Linux     │─────▶│  Metasploitable2     │  │
│  │  172.18.0.3     │      │  172.18.0.2          │  │
│  │                 │      │                      │  │
│  │  [Attacker]     │      │  [Victim]            │  │
│  │  Nmap           │      │  FTP (vsftpd 2.3.4)  │  │
│  │  Hydra          │      │  SSH (OpenSSH 4.7)   │  │
│  │  Packetbeat ────┼──────┼──▶ Traffic capture   │  │
│  └─────────────────┘      └──────────────────────┘  │
│                                      │               │
│                    ┌─────────────────▼────────────┐  │
│                    │      Elastic Stack           │  │
│                    │  Elasticsearch 8.12.0        │  │
│                    │  Kibana 8.12.0               │  │
│                    │  [SIEM / Detection Layer]    │  │
│                    └──────────────────────────────┘  │
└─────────────────────────────────────────────────────┘

Host: Windows 11 + Docker Desktop (WSL2 backend)
RAM allocated: ~2GB for ELK (512MB heap for Elasticsearch)
```

**Why Docker instead of virtual machines?**  
The host machine has 8GB RAM and a dual-core i5 processor — traditional VM-based SOC labs typically require 16GB+ RAM to run multiple VMs simultaneously. Using Docker containers on an isolated bridge network achieves the same network segmentation and service isolation at a fraction of the resource cost, making this approach accessible to anyone building a home lab on modest hardware.

---

## 4. Tools & Technologies

| Category | Tool | Version | Purpose |
|----------|------|---------|---------|
| Containerisation | Docker Desktop | Latest | Lab environment management |
| Attacker platform | Kali Linux | 2026.2 (rolling) | Offensive tooling |
| Vulnerable target | Metasploitable2 | — | Intentionally vulnerable Linux host |
| SIEM — Search & Storage | Elasticsearch | 8.12.0 | Log indexing and querying |
| SIEM — Visualisation | Kibana | 8.12.0 | Dashboard, Discover, KQL queries |
| Network traffic collector | Packetbeat | 8.12.0 | Packet capture and shipping to ELK |
| Reconnaissance | Nmap | 7.99 | Port scanning, service version detection |
| Credential attack | Hydra | 9.6 | FTP brute force simulation |
| Detection query language | KQL (Kibana Query Language) | — | Log search and filtering |
| Documentation | Markdown + GitHub | — | Portfolio and writeup publication |

---

## 5. Project Phases

### Phase 1 — Environment Setup (Week 1)

**Goal:** Build a fully isolated, functional three-container lab network.

**Steps completed:**
- Created an isolated Docker bridge network (`soc-lab`) with no external internet exposure for inter-container traffic
- Deployed Metasploitable2 as the target container, resolving a container exit loop caused by the init script completing without a persistent foreground process — fixed by appending `tail -f /dev/null` to the run command
- Deployed Elasticsearch and Kibana via Docker Compose, resolving a Windows port-binding conflict on port 9200 by removing the host port mapping (Elasticsearch remains reachable internally via Docker DNS at `elasticsearch:9200`)
- Connected the existing Kali container (`ctf-solver`) to the `soc-lab` network as a second network interface (`eth1`), preserving existing connectivity on `eth0`
- Installed and configured Packetbeat 8.12.0 on Kali to sniff traffic on `eth1` and ship events to Elasticsearch

**Key challenge solved:**  
Metasploitable2 runs Ubuntu 8.04 (glibc 2.7), which is incompatible with modern Filebeat 8.x binaries (require glibc 2.17+). Rather than attempting to run a log shipper on the legacy target, Packetbeat was deployed on the modern Kali container to capture all network traffic passing through the `soc-lab` interface — a more architecturally realistic approach that mirrors how SOC teams monitor traffic from network taps or gateway sensors rather than relying on agents on every endpoint.

---

### Phase 2 — Attack Simulation (Week 2)

**Goal:** Execute a realistic multi-phase attack and generate detectable log evidence.

Three attack phases were conducted in sequence, with Packetbeat capturing all traffic throughout.

---

### Phase 3 — Detection & Analysis (Week 2)

**Goal:** Switch to analyst role, query the SIEM, reconstruct the attack timeline.

---

### Phase 4 — Documentation (Week 2)

**Goal:** Produce a professional incident report and project writeup.

---

## 6. Attack Simulation

### 6.1 Phase 1 — Reconnaissance (Nmap Port Scan)

**Time:** 2026-06-23 08:33 UTC  
**Tool:** Nmap 7.99  
**Command:**
```bash
nmap -sV -sC -O 172.18.0.2 -oN /tmp/scan_results.txt
```

**Flags explained:**
- `-sV` — Service version detection (identifies what software is running on each port)
- `-sC` — Default script scan (runs NSE scripts to extract additional info like FTP anonymous access, SSL cert details, SMB signing status)
- `-O` — OS fingerprinting
- `-oN` — Save output to file for evidence

**Result:** 20 open TCP ports identified in 175 seconds, including:

| Port | Service | Version | Risk |
|------|---------|---------|------|
| 21 | FTP | vsftpd 2.3.4 | Critical — anonymous login + CVE-2011-2523 backdoor |
| 22 | SSH | OpenSSH 4.7p1 | Medium — legacy cipher support only |
| 23 | Telnet | Linux telnetd | High — plaintext protocol |
| 80 | HTTP | Apache 2.2.8 | Medium — outdated version |
| 139/445 | SMB | Samba 3.0.20 | High — message signing disabled |
| 1524 | Ingreslock | Root shell | Critical — unauthenticated root shell on connection |
| 3306 | MySQL | 5.0.51a | High — potential anonymous access |
| 6667 | IRC | UnrealIRCd 3.2.8.1 | Critical — known backdoored version |

**Notable:** Port 1524 returned a root shell (`root@bc5e1f9f146a:/#`) to Nmap's probe without any authentication — an ingreslock backdoor present on all Metasploitable2 builds.

---

### 6.2 Phase 2 — Initial Access (Anonymous FTP Login)

**Time:** 2026-06-23 12:49 UTC  
**Tool:** ftp (CLI client)  
**Command:**
```bash
ftp 172.18.0.2
# Username: anonymous
# Password: [blank]
```

**Result:**  
```
220 (vsFTPd 2.3.4)
230 Login successful.
```

Access was granted immediately with no password required. A directory listing was performed and a file retrieval was attempted — demonstrating that an unauthenticated attacker can enumerate and access the FTP service with zero credentials.

**Security finding:** Anonymous FTP access is a CWE-284 (Improper Access Control) misconfiguration. In a real environment this allows an attacker to enumerate file shares, potentially read sensitive files, or use the FTP service as a staging point for further exploitation.

---

### 6.3 Phase 3 — Credential Attack (FTP Brute Force)

**Time:** 2026-06-23 12:49–12:51 UTC  
**Tool:** Hydra 9.6  
**Command:**
```bash
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt \
  172.18.0.2 ftp -t 4 -V -o /tmp/hydra_ftp_results.txt
```

**Flags explained:**
- `-l msfadmin` — Single username to target
- `-P rockyou.txt` — Password wordlist (14.3 million entries)
- `-t 4` — 4 parallel threads
- `-V` — Verbose output (shows each attempt)

**Result:** 201 authentication attempts in approximately 90 seconds before manual termination. All attempts failed against FTP — the attack was designed to generate detectable log noise rather than achieve a successful login, demonstrating the brute force detection pattern.

**Note on SSH brute force:** A MAC algorithm incompatibility between modern Hydra/OpenSSH clients and the ancient SSH server on Metasploitable2 (OpenSSH 4.7 only supports deprecated HMAC algorithms) prevented SSH brute force execution. FTP was used as an equivalent service for generating the detection pattern.

---

## 7. Detection & Analysis

All detection was performed in Kibana using KQL (Kibana Query Language) against the `packetbeat-*` data stream.

### 7.1 Detecting the Port Scan

**KQL Query:**
```
network.transport: tcp AND destination.ip: 172.18.0.2
```
**Result:** Dense spike of TCP connection events across multiple destination ports in a short time window, originating from a single source IP — the classic port scan signature.

---

### 7.2 Detecting the Anonymous FTP Login

**KQL Query:**
```
destination.port: 21 AND source.ip: 172.18.0.3
```
**Result:** 28 hits, all within a 1-second window (12:49:19.000–12:49:20.000). The compression of 28 FTP connection events into under one second confirms automated or scripted activity rather than a human session.

---

### 7.3 Detecting the Brute Force Attack

**KQL Query:**
```
destination.port: 21 AND network.transport: tcp AND event.type: connection
```
**Result:** 408 connection events to port 21 between 12:47–12:52, forming a clear histogram spike that drops abruptly — matching the exact start and stop of the Hydra attack. This pattern (high-volume, single-port, single-source, short duration) is the textbook brute force signature that automated SIEM rules are written to detect.

---

## 8. Key Findings

| # | Finding | Severity | Evidence |
|---|---------|----------|----------|
| 1 | Anonymous FTP access enabled | Critical | Kibana Query 2, FTP session logs |
| 2 | vsftpd 2.3.4 backdoor present (CVE-2011-2523) | Critical | Nmap service detection |
| 3 | Unauthenticated root shell on port 1524 | Critical | Nmap script scan |
| 4 | UnrealIRCd 3.2.8.1 backdoor (CVE-2010-2075) | Critical | Nmap service detection |
| 5 | No account lockout policy on FTP | High | Hydra brute force — 201 attempts, no lockout |
| 6 | SMB message signing disabled | Medium | Nmap smb-security-mode script |
| 7 | SSL certificates expired since 2010 | Medium | Nmap ssl-cert script |
| 8 | Legacy SSH ciphers only (no modern MACs) | Medium | Hydra kex error output |
| 9 | Telnet service exposed (plaintext) | High | Nmap port 23 |
| 10 | 20 open ports — excessive attack surface | High | Full Nmap scan output |

---

## 9. MITRE ATT&CK Mapping

| Tactic | Technique | ID | Evidence |
|--------|-----------|-----|---------|
| Reconnaissance | Active Scanning: Port Scanning | T1595.001 | Nmap scan, Packetbeat flow spike |
| Discovery | Network Service Discovery | T1046 | 20 open ports enumerated |
| Initial Access | Valid Accounts: Default Accounts | T1078.001 | Anonymous FTP login (user: anonymous) |
| Credential Access | Brute Force: Password Guessing | T1110.001 | 201 Hydra FTP attempts |
| Discovery | Remote System Discovery | T1018 | Target host enumeration via Nmap |

---

## 10. Challenges & How I Solved Them

| Challenge | Root Cause | Solution |
|-----------|-----------|----------|
| Metasploitable2 container kept exiting | Docker init script completed, no persistent PID 1 | Appended `tail -f /dev/null` to container run command to keep it alive |
| Elasticsearch port 9200 binding failed on Windows | Windows reserved port range conflict (WSL2/Hyper-V) | Removed host port mapping — Elasticsearch only exposed internally on Docker network |
| YAML docker-compose errors | Incorrect indentation when pasting via VS Code | Rewrote file using PowerShell here-string (`@"..."@ \| Out-File`) for clean UTF-8 output |
| Filebeat incompatible with Metasploitable2 | Metasploitable2 runs glibc 2.7; Filebeat 8.x requires glibc 2.17+ | Deployed Packetbeat on Kali container instead — sniffs all soc-lab traffic on eth1 |
| Packetbeat connecting to localhost instead of Elasticsearch | Default config not overwritten correctly | Verified file with `wc -l` before running; rewrote config cleanly |
| SSH brute force failed (MAC algorithm mismatch) | OpenSSH 4.7 only supports deprecated HMAC algorithms; modern clients reject these | Switched brute force target to FTP — identical detection pattern, no crypto negotiation |
| Hydra Medusa dependency failures | Two packages returning 404 from Kali repo | Skipped Medusa entirely, used Hydra against FTP as equivalent |

---

## 11. Skills Demonstrated

**Blue Team / SOC:**
- SIEM deployment and configuration (Elastic Stack 8.x)
- KQL query writing for threat detection
- Network traffic analysis using Packetbeat
- Alert triage and event correlation
- Attack timeline reconstruction from log data
- Incident report writing (executive summary, IOCs, MITRE mapping, recommendations)

**Red Team / Offensive (for detection context):**
- Network reconnaissance with Nmap (service detection, OS fingerprinting, NSE scripts)
- Anonymous service exploitation (FTP misconfiguration)
- Credential brute force with Hydra
- Vulnerability identification from scan output

**Infrastructure & Engineering:**
- Docker networking (isolated bridge networks, multi-container communication)
- Docker Compose (service orchestration, dependency management)
- Linux CLI (Kali, Debian, Ubuntu environments)
- Windows PowerShell (container management, file creation)
- Troubleshooting across OS, network, and application layers

---

## 12. Evidence

```
soc-home-lab/
├── README.md                        ← This file
├── docker-compose.yml               ← ELK stack configuration
├── packetbeat.yml                   ← Packetbeat configuration
├── reports/
│   └── incident_report.md           ← Full incident report
├── scans/
│   └── scan_results.txt             ← Full Nmap output
├── screenshots/
│   ├── 01_lab_architecture.png      ← Docker containers running
│   ├── 02_kibana_data_view.png      ← Packetbeat data view in Kibana
│   ├── 03_nmap_portscan.png         ← Port scan detection in Kibana
│   ├── 04_ftp_anonymous_login.png   ← Anonymous FTP detection (28 hits)
│   └── 05_ftp_bruteforce.png        ← Brute force spike (408 hits)
└── results/
    └── hydra_ftp_results.txt        ← Hydra output log
```

---

## 13. Recommendations

1. **Disable anonymous FTP immediately** — migrate to SFTP with key-based authentication only
2. **Decommission or patch vsftpd 2.3.4** — contains a confirmed remote root backdoor (CVE-2011-2523)
3. **Remove port 1524 (ingreslock backdoor)** — grants unauthenticated root shell on connection
4. **Patch UnrealIRCd** — CVE-2010-2075 backdoor allows remote code execution
5. **Implement fail2ban** — 5-attempt lockout with 10-minute ban on all authentication services
6. **Disable Telnet** — replace with SSH using modern cipher suites only
7. **Enable SMB message signing** — prevents NTLM relay attacks
8. **Replace expired SSL certificates** — all certificates expired April 2010
9. **Enforce network segmentation** — no host should expose 20 services simultaneously on a flat network
10. **Implement SIEM alerting rules** — automate detection of port scan bursts, anonymous logins, and brute force patterns instead of relying on manual querying

---

## 14. What I Would Do Differently

- **Add Logstash** to the pipeline for log parsing and enrichment before indexing into Elasticsearch — would enable field extraction from raw FTP log lines
- **Build Kibana detection rules** (SIEM → Rules) to trigger automated alerts rather than running manual KQL queries — closer to real SOC workflow
- **Use a dedicated log collector** (Syslog-ng or rsyslog) on a separate container as a log aggregation point
- **Capture a Wireshark PCAP** of the FTP session to show the anonymous login credentials transmitted in plaintext — powerful visual for the report
- **Exploit CVE-2011-2523** (vsftpd backdoor) with Metasploit to demonstrate the full attack chain from reconnaissance to root access

---

## 15. Next Steps

- [ ] Add Kibana alerting rules for automated brute force detection
- [ ] Exploit vsftpd 2.3.4 backdoor (CVE-2011-2523) and document full attack chain
- [ ] Add a Windows target container for Active Directory attack simulation
- [ ] Study for and sit CompTIA Security+ SY0-701
- [ ] Publish writeup on Medium / LinkedIn article

---

*Built on: Windows 11 + Docker Desktop | Kali Linux 2026.2 | Elastic Stack 8.12.0*  
*Network: Isolated Docker bridge — no internet exposure to attack traffic*  
*Purpose: Educational portfolio project — all activity contained within lab environment*
