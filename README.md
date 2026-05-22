# Detection Lab

> Blue Team home lab simulating a real credential theft attack chain, detected using a self-built SIEM environment.

---

## Overview

This lab was built to develop and demonstrate practical cloud security engineering skills. It simulates a full attack lifecycle — from initial access to domain compromise — and detects each technique using custom SIEM rules mapped to the MITRE ATT&CK framework.

Built on a Lenovo Legion Pro 7 running VMware Workstation 26H1 with a fully isolated network. No internet access for attack machines — all traffic contained within the lab.

---

## Environment

| Host | OS | Role | IP |
|---|---|---|---|
| DC01 | Windows Server 2022 Standard | Domain Controller | 192.168.10.10 |
| WIN11-01 | Windows 11 Pro | Victim Endpoint | 192.168.10.20 |
| WAZUH-SIEM | Ubuntu Server 22.04 LTS | Wazuh SIEM | 192.168.10.30 |
| KALI | Kali Linux | Attacker | 192.168.10.40 |

**Network:** VMnet1 host-only — 192.168.10.0/24 — fully isolated  
**Domain:** lab.local  
**Hypervisor:** VMware Workstation 26H1 (free personal use)

---

## Attack Chain Demonstrated

A simulated credential theft and lateral movement attack on a domain-joined Windows endpoint:

1. Attacker gains local admin access to WIN11-01
2. Mimikatz dumps credentials from LSASS memory — extracts NTLM hash for Domain Admin account
3. Pass-the-hash authenticates as Domain Admin without knowing the plaintext password
4. Full domain compromise achieved using only a credential hash

---

## MITRE ATT&CK Coverage

| Technique ID | Name | Description |
|---|---|---|
| T1562.001 | Impair Defenses | Windows Defender disabled prior to attack |
| T1003.001 | OS Credential Dumping: LSASS Memory | Mimikatz extracted NTLM hashes from LSASS |
| T1550.002 | Pass the Hash | Domain Admin session opened using NTLM hash |
| T1078 | Valid Accounts | Legitimate domain admin account used |

---

## Detection

Both primary techniques detected in Wazuh SIEM using custom detection rules built on Sysmon telemetry:

| Rule ID | Level | Trigger | Technique |
|---|---|---|---|
| 100001 | Critical (15) | Non-system process accessed lsass.exe | T1003.001 |
| 100002 | High (12) | LogonType 9 (NewCredentials) authentication | T1550.002 |

Detection rules available in `/detections/local_rules.xml`

---

## Tools Used

| Tool | Purpose |
|---|---|
| VMware Workstation 26H1 | Hypervisor — runs all VMs |
| Windows Server 2022 | Domain Controller, Active Directory |
| Sysmon + SwiftOnSecurity config | Endpoint telemetry and logging |
| Wazuh 4.7 | SIEM — log ingestion, detection, alerting |
| Mimikatz | Credential dumping (attack simulation) |
| MITRE ATT&CK Navigator | Technique mapping and coverage visualization |

---

## Repository Structure

detection-lab/
├── README.md
├── setup/
│   ├── 01-dc01.md               # Domain Controller build
│   ├── 02-win11.md              # Victim endpoint build
│   ├── 03-wazuh.md              # Wazuh SIEM build
│   └── 04-kali.md               # Attacker machine setup
├── attacks/
│   ├── 01-mimikatz.md           # LSASS credential dump walkthrough
│   └── 02-pass-the-hash.md     # Pass-the-hash attack walkthrough
├── detections/
│   └── local_rules.xml          # Custom Wazuh detection rules
├── mitre-mapping/
│   └── attack-mapping.md        # MITRE ATT&CK technique mapping
└── ir-report/
└── IR-001.md                # Incident report in CySA+ format