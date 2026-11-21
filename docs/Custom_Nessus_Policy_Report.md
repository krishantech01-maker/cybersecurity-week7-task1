# Custom Nessus Scan Policy & Configuration Report

**Date:** 21-11-2026  
**Analyst:** Krishan Kumar  
**Task:** Week 7 - Task 1 (Advanced Vulnerability Scanning with Nessus)  
**Policy Name:** `Enterprise_Deep_Credentialed_Audit`  
**Target Environment:** `192.168.56.101 - 192.168.56.105` (Windows & Linux Lab Hosts)  

---

## 1. Executive Summary
To overcome the limitations of default basic network scans, a custom **Nessus Advanced Scan Policy** (`Enterprise_Deep_Credentialed_Audit`) was engineered and executed against target lab systems. By enabling authenticated local checks alongside targeted service discovery and safe-check tuning, the scan uncovered deep OS-level missing patches, weak cryptographic protocols, and privilege escalation vectors that unauthenticated scans miss.

---

## 2. Custom Nessus Policy Configuration Details

### A. Discovery & Port Scanning Settings
* **Host Discovery:** Enabled ARP ping, TCP SYN ping (ports `22, 80, 443, 445, 3389`), and ICMP ping with 2 retries.
* **Port Scan Range:** Configured full TCP port range (`1-65535`) using **SYN Stealth Scan** and top `1,000` UDP ports for critical services (DNS, SNMP, NTP).
* **Service Detection:** Enabled *Probe all ports to find services* and *Search for SSL/TLS on all ports* to detect non-standard service bindings.

### B. Assessment & Performance Tuning
* **Safe Checks:** Enabled (`Yes`) to prevent denial-of-service crashes on production-like services while still verifying version-based and banner-based vulnerabilities.
* **Web Application Tests:** Enabled generic HTTP/CGI checks, directory traversal detection, and default web credential audits.
* **Performance Throttling:** Set *Max simultaneous checks per host* to `5` and *Max concurrent hosts* to `10` to avoid network congestion.

### C. Credentialed Scan Configuration (Authenticated Audit)
* **Windows Hosts (SMB/WMI):** Configured domain/local administrator credentials with *Start the Remote Registry service during the scan* enabled to audit missing Windows KBs and registry misconfigurations.
* **Linux Hosts (SSH):** Configured SSH key-based authentication with `sudo` privilege escalation enabled to inspect installed package versions (`dpkg`/`rpm`) and file permissions.

### D. Custom Plugin Family Selection
* **Enabled High-Value Families:** *Windows : Microsoft Bulletins*, *Ubuntu/Debian Local Security Checks*, *Web Servers*, *Databases*, *General*, and *Service Detection*.
* **Disabled Noisy Families:** Disabled *Denial of Service* and *Port Scanners (redundant)* to keep execution fast and safe.

---

## 3. Integration with Other Security Tools
To operationalize Nessus results within a Security Operations Center (SOC):
1. **SIEM Integration (Wazuh / Splunk):** Exported scan results in `.nessus` (XML) and `.csv` formats and ingested them into Wazuh/Splunk to correlate active IDS alerts against known vulnerable hosts.
2. **Ticketing & Patch Management (Jira):** Automated P1/Critical findings (CVSS >= 9.0) into remediation tickets assigned to System Administrators with a 48-hour SLA.
