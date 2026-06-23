# Incident Report: Reconnaissance and Unauthorized Access via FTP
**Date:** 2026-06-23  
**Analyst:** Asibey-Kitiabi Koffi Emmanuel
**Severity:** High  
**Status:** Contained (lab environment)  


---

## 1. Executive Summary
A threat actor conducted a full TCP port scan against an internal host 
(172.18.0.2), successfully authenticated to an FTP service anonymously, 
and followed up with an automated credential brute force attack generating 
408 connection attempts over approximately 5 minutes. No confirmed data 
exfiltration occurred. Root cause is misconfigured FTP service allowing 
anonymous access with no account lockout policy.

---

## 2. Timeline of Events
| Time (UTC)       | Event                              | Source IP   | Evidence              |
|------------------|------------------------------------|-------------|-----------------------|
| 08:33:00         | Nmap full TCP scan initiated       | 172.18.0.3  | Packetbeat flow data  |
| 08:36:43         | 20 open ports identified           | 172.18.0.3  | scan_results.txt      |
| 12:49:19         | Anonymous FTP login successful     | 172.18.0.3  | Kibana Query 2 (28 hits)|
| 12:49:19         | FTP directory listing attempted    | 172.18.0.3  | FTP session logs      |
| 12:49:30         | FTP brute force begins (Hydra)     | 172.18.0.3  | Kibana Query 3        |
| 12:51:00         | Brute force stopped (201 attempts) | 172.18.0.3  | Kibana Query 3 (408 hits)|

---

## 3. Indicators of Compromise (IOCs)
- **Attacker IP:** 172.18.0.2 (internal — Kali container on soc-lab network)
- **Target IP:** 172.18.0.2
- **Ports targeted:** 21 (FTP), 22 (SSH), 23, 25, 80, 139, 445 and 13 others
- **Tool signatures:** Nmap 7.99, Hydra v9.6, vsftpd anonymous login
- **Detection:** 408 TCP connection events to port 21 in 5-minute window

---

## 4. Vulnerabilities Identified
| Finding                        | Severity | CVE / Reference         |
|--------------------------------|----------|-------------------------|
| Anonymous FTP login enabled    | Critical | CWE-284                 |
| vsftpd 2.3.4 (backdoored)     | Critical | CVE-2011-2523           |
| No FTP account lockout policy  | High     | CWE-307                 |
| SSH using legacy MAC algorithms| Medium   | NIST SP 800-131A        |
| SMB message signing disabled   | Medium   | CWE-347                 |
| SSL certificates expired 2010  | Medium   | CWE-298                 |

---

## 5. Root Cause
FTP service (vsftpd 2.3.4) configured to allow anonymous authentication 
with no rate limiting or lockout policy. Combined with 20 exposed network 
services, the attack surface is critically over-exposed for any networked 
host.

---

## 6. Recommendations
1. Disable anonymous FTP immediately — migrate to SFTP with key-based auth
2. Patch or replace vsftpd 2.3.4 (contains confirmed backdoor CVE-2011-2523)
3. Implement fail2ban on all authentication services (SSH, FTP) with 
   5-attempt lockout
4. Disable unused services — 20 open ports represents an excessive attack 
   surface
5. Enable SMB message signing to prevent relay attacks
6. Replace expired SSL certificates across all services
7. Implement network segmentation — isolate internet-facing services

---

## 7. MITRE ATT&CK Mapping
| Tactic            | Technique                        | ID          |
|-------------------|----------------------------------|-------------|
| Reconnaissance    | Active Scanning: Port Scanning   | T1595.001   |
| Initial Access    | Valid Accounts: Default Accounts | T1078.001   |
| Credential Access | Brute Force: Password Guessing   | T1110.001   |
| Discovery         | Network Service Discovery        | T1046       |

---

## 8. Lab Architecture
- **Attacker:** ctf-solver (Kali Linux 2026.2) — 172.18.0.3
- **Target:** Metasploitable2 — 172.18.0.2
- **SIEM:** Elastic Stack 8.12.0 (Elasticsearch + Kibana)
- **Collection:** Packetbeat 8.12.0 on eth1 (soc-lab network)
- **Network:** Isolated Docker bridge network (soc-lab)

---

## 9. Evidence
- `screenshots/01_nmap_portscan.png`
- `screenshots/02_ftp_anonymous_login.png`  
- `screenshots/03_ftp_bruteforce.png`
- `scan_results.txt` (full Nmap output)
- `hydra_ftp_results.txt`
