# PsExec-Lateral-Movement-Detection-T1021.002-
Successfully executed and detected PsExec‑based lateral movement in a Windows domain environment. Configured Sysmon on both endpoints to capture process, file, registry, and network telemetry. Enabled SMB and Admin$ access, corrected domain authentication issues, and applied GPO rights required for remote service creation. Executed PsExec from a compromised workstation to the Domain Controller, captured Event 1, 3, 11, and 13, and validated Sigma detection in Security Onion. Built a complete incident timeline aligned with MITRE ATT&CK T1021.002. Demonstrated strong skills in detection engineering, Windows internals, and SOC analysis.

## 1. Scenario Overview

This lab simulates a real‑world attacker performing lateral movement inside a Windows domain using PsExec,
a legitimate Sysinternals tool often abused by threat actors (e.g., FIN7, Conti, APT29).

The goal was to:

- Execute PsExec from a compromised workstation
- Gain remote command execution on the Domain Controller
- Capture Sysmon telemetry
- Validate Sigma detection
- Build a complete incident timeline

---

## 2. Lab Architecture
### Attacker Machine: WLAB01 (Domain‑joined)
### Target Machine: DC1 (Domain Controller)
### Tools:

- PsExec v2.43
- Sysmon (custom config)
- Winlogbeat → Elastic → Security Onion
- Sigma detection rules

## 3. Execution Steps
### 3.1 Initial Issue — PsExec Failing
PsExec initially failed with:

- “Network path not found”
- “Access is denied”
- “Logon failure: user not granted requested logon type”

These errors revealed:
- The target was originally outside the domain
- SMB/445 was blocked
- Admin$ share inaccessible
- User rights missing on the DC
- Sysmon config missing on the DC

## 3.2 Fixes Applied
### A. Network & SMB

- Enabled File & Printer Sharing firewall group
- Verified Admin$ share
- Confirmed SMB connectivity

### B. Authentication
- Used domain‑qualified credentials: DCT-LTD\hemiezan

- Granted required rights via GPO:
    - Log on as a batch job
    - Log on as a service
    - Access this computer from the network

### C. Sysmon Visibility
- Installed/updated Sysmon on the DC

- Applied custom Sysmon config to log:

    - ProcessCreate
    - FileCreate
    - RegistryEvent
    - NetworkConnect

## 3.3 Successful PsExec Execution
Final working command: <img width="579" height="53" alt="cmd" src="https://github.com/user-attachments/assets/b155f230-74cf-453f-afa0-393ad6ebec65" />

Result:

- Remote shell opened on the Domain Controller
- Verified with hostname and whoami

This confirmed successful lateral movement.

## 4. Detection & Telemetry Collected
### 4.1 Sysmon Events (DC)
<img width="628" height="204" alt="Telemetry" src="https://github.com/user-attachments/assets/e9eb4e0a-136c-4c9f-91fa-06f6308004fa" />

### 4.2 Sigma Detection
Once Sysmon was updated, Sigma rules triggered for:

   - PsExec remote execution
   - Suspicious service creation
   - SMB admin share abuse

### 4.3 Security Onion Correlation
Security Onion correlated:

  - Sysmon logs
  - Zeek SMB logs
  - Suricata alerts

Resulting in a complete MITRE ATT&CK T1021.002 detection.

## 5. Incident Timeline (Analyst View)
  1. Attacker initiates PsExec from WLAB01
  2. SMB connection established to DC1
  3. PSEXESVC.exe dropped on DC1 (Sysmon Event 11)
  4. Service created in registry (Event 13)
  5. Remote cmd.exe launched (Event 1)
  6. Analyst detects suspicious service creation
  7. Correlation engine maps activity to T1021.002
  8. Incident escalated as lateral movement attempt

## 6. Key Learning Outcomes
- Deep understanding of PsExec internals
- Ability to troubleshoot authentication, SMB, and GPO issues
- Mastery of Sysmon configuration for detection engineering
- Ability to validate Sigma rules and Elastic mappings
- End‑to‑end SOC workflow: execution → detection → correlation → documentation
