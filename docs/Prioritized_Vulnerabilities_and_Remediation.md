# Prioritized Vulnerability List & Remediation Plan

**Task:** Week 7 - Task 1 (Advanced Vulnerability Scanning with Nessus)  

---

## 1. Risk-Prioritized Findings Matrix
*Vulnerabilities are ranked using both **CVSS v3.1** base scores and **VPR (Vulnerability Priority Rating)** to reflect active real-world exploitability.*

| Priority | Plugin ID | Vulnerability Name | Affected Host | CVE | CVSS v3.1 | VPR Score | Severity |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **P1 (Immediate)** | 97833 | MS17-010: Security Update for Microsoft Windows SMB Server (EternalBlue) | `192.168.56.102` | CVE-2017-0144 | **8.8** | **9.8** | **Critical** |
| **P1 (Immediate)** | 156001 | Apache Log4j2 Remote Code Execution (Log4Shell) | `192.168.56.103` | CVE-2021-44228 | **10.0** | **9.9** | **Critical** |
| **P2 (Urgent)** | 104743 | TLS Version 1.0 and 1.1 Protocol Detection | `192.168.56.102` | CVE-2011-3389 | **5.3** | **6.4** | **Medium / High Risk** |
| **P2 (Urgent)** | 57608 | SMB Signing Not Required | `192.168.56.102` | CWE-345 | **5.3** | **5.9** | **Medium** |
| **P3 (Scheduled)** | 70658 | SSH Server CBC Mode Ciphers Enabled | `192.168.56.101` | CVE-2008-5161 | **3.7** | **3.2** | **Low** |

---

## 2. Detailed Technical Remediation Recommendations

### 1. Apache Log4j2 RCE / Log4Shell (Plugin 156001 — Critical)
* **Risk Impact:** Unauthenticated attackers can execute arbitrary Java code on the server via crafted HTTP headers (`${jndi:ldap://...}`).
* **Remediation:**
  * Upgrade Apache Log4j2 immediately to version **`2.17.1`** or later.
  * As a temporary mitigation, set the JVM flag `-Dlog4j2.formatMsgNoLookups=true` and remove the `JndiLookup` class from the classpath.

### 2. MS17-010 Windows SMBv1 RCE / EternalBlue (Plugin 97833 — Critical)
* **Risk Impact:** Allows remote attackers to gain SYSTEM-level privileges and spread ransomware across the internal network.
* **Remediation:**
  * Install Microsoft Security Bulletin **MS17-010** via Windows Update / WSUS.
  * Disable legacy SMBv1 completely via PowerShell:
    ```powershell
    Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart
    ```

### 3. SMB Signing Not Required (Plugin 57608 — Medium)
* **Risk Impact:** Enables NTLM relay and man-in-the-middle (MitM) attacks against Windows hosts on the local subnet.
* **Remediation:**
  * Enforce SMB message signing via Group Policy (`Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options > Microsoft network server: Digitally sign communications (always) = Enabled`).

### 4. Deprecated TLS 1.0 / 1.1 Protocols (Plugin 104743 — Medium)
* **Risk Impact:** Exposes encrypted web sessions to cryptographic downgrade attacks (e.g., BEAST/POODLE).
* **Remediation:**
  * Disable TLS 1.0 and TLS 1.1 in IIS / Nginx / Apache configurations and enforce **TLS 1.2** and **TLS 1.3** exclusively.

### 5. SSH Weak CBC Mode Ciphers Enabled (Plugin 70658 — Low)
* **Risk Impact:** Weakens confidentiality of SSH administrative sessions.
* **Remediation:**
  * Edit `/etc/ssh/sshd_config` on `192.168.56.101` to allow only strong CTR/GCM ciphers (`chacha20-poly1305@openssh.com,aes256-gcm@openssh.com`) and restart the SSH service (`systemctl restart sshd`).
