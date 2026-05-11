# 🏰 Hybrid Active Directory Attack & Defense Lab

> Built by John Victory | Junior SOC Analyst
> GitHub: github.com/john-opsec34g

---

## Lab Overview

A hybrid Active Directory home lab simulating real enterprise
attack scenarios and defensive detection using Microsoft
Sentinel SIEM connected to a live domain controller.

This lab mirrors how modern organisations operate — combining
on-premise Active Directory with cloud-based security
monitoring through Azure.

## Lab Architecture
┌─────────────────────────────────────────────────────┐
│                    CLOUD (AZURE)                     │
│                                                      │
│    ┌─────────────────────────────────────────┐      │
│    │         Microsoft Sentinel SIEM          │      │
│    │    Log Analytics Workspace               │      │
│    │    KQL Detection Rules                   │      │
│    │    Automated Alerts                      │      │
│    └──────────────────┬──────────────────────┘      │
└───────────────────────│─────────────────────────────┘
│ Azure Monitor Agent
│ (forwards all Windows
│  Security Events)
│
┌───────────────────────▼─────────────────────────────┐
│                 ON-PREMISE LAB                       │
│                                                      │
│  ┌──────────────────┐    ┌──────────────────────┐   │
│  │  Windows Server  │    │    Kali Linux VM     │   │
│  │  2019 VM         │◄───│                      │   │
│  │  Domain: lab.local│   │  Attack Tools:       │   │
│  │  DC01            │    │  - Responder         │   │
│  │  IP: 192.168.43.10│   │  - Impacket          │   │
│  └──────────────────┘    │  - Mimikatz          │   │
│                           │  - BloodHound        │   │
│  ┌──────────────────┐    └──────────────────────┘   │
│  │  HP Workstation  │                                │
│  │  Windows 10      │                                │
│  │  Domain-joined   │                                │
│  │  IP: 192.168.43.20│                               │
│  └──────────────────┘                                │
└─────────────────────────────────────────────────────┘


---

## Environment Details

| Component | Details |
|---|---|
| Domain Controller | Windows Server 2019 — lab.local |
| Workstation | Windows 10 — domain joined |
| Attack Machine | Kali Linux — Impacket, Responder, Mimikatz |
| SIEM | Microsoft Sentinel via Azure Monitor Agent |
| Detection Language | KQL (Kusto Query Language) |
| Network | Phone hotspot — all machines same subnet |

---

## Attacks Simulated

### 1. LLMNR/NBT-NS Poisoning — T1557.001
**Tool:** Responder
**What happens:** Kali intercepts LLMNR broadcast traffic
and captures NTLMv2 hashes from the Windows workstation.

**Detection Event IDs:**
- 4625 — Failed logon attempt
- 4648 — Logon with explicit credentials

**KQL Detection:**
```kql
SecurityEvent
| where EventID == 4625
| summarize count() by IpAddress, Account
| where count_ > 5
| order by count_ desc
```

**Status:** ⏳ In Progress

---

### 2. Kerberoasting — T1558.003
**Tool:** Impacket GetUserSPNs.py
**What happens:** Requests Kerberos service tickets for
service accounts and attempts offline cracking.

**Detection Event IDs:**
- 4769 — Kerberos Service Ticket Request (RC4 encryption 0x17)

**KQL Detection:**
```kql
SecurityEvent
| where EventID == 4769
| where TicketEncryptionType == "0x17"
| project TimeGenerated, Account, ServiceName, IpAddress
```

**Status:** ⏳ In Progress

---

### 3. Pass-the-Hash — T1550.002
**Tool:** Impacket psexec.py with NTLM hash
**What happens:** Uses captured NTLM hash to authenticate
without knowing the plaintext password.

**Detection Event IDs:**
- 4624 — Logon Type 3 from unexpected source
- 4672 — Special privileges assigned

**KQL Detection:**
```kql
SecurityEvent
| where EventID == 4624
| where LogonType == 3
| where AccountName !endswith "$"
| project TimeGenerated, AccountName, IpAddress
```

**Status:** ⏳ In Progress

---

### 4. PsExec Lateral Movement — T1570
**Tool:** Impacket psexec.py
**What happens:** Remotely executes commands by installing
a temporary service on the target machine.

**Detection Event IDs:**
- 7045 — New service installed (PSEXESVC)
- 4688 — Process creation

**KQL Detection:**
```kql
SecurityEvent
| where EventID == 7045
| project TimeGenerated, ServiceName, ServiceFileName, SubjectAccount
```

**Status:** ⏳ In Progress

---

### 5. Scheduled Task Persistence — T1053.005
**Tool:** Windows schtasks command
**What happens:** Creates a scheduled task that runs
automatically — simulating attacker persistence mechanism.

**Detection Event IDs:**
- 4698 — Scheduled task created

**KQL Detection:**
```kql
SecurityEvent
| where EventID == 4698
| project TimeGenerated, SubjectAccount, TaskName
```

**Status:** ⏳ In Progress

---

## MITRE ATT&CK Coverage

| Tactic | Technique | ID | Status |
|---|---|---|---|
| Credential Access | LLMNR/NBT-NS Poisoning | T1557.001 | ⏳ |
| Credential Access | Kerberoasting | T1558.003 | ⏳ |
| Lateral Movement | Pass-the-Hash | T1550.002 | ⏳ |
| Lateral Movement | Lateral Tool Transfer | T1570 | ⏳ |
| Persistence | Scheduled Task | T1053.005 | ⏳ |

---

## Microsoft Sentinel Integration

All Windows Security Events from the domain controller
are forwarded to Microsoft Sentinel via Azure Monitor Agent.

Custom analytics rules created for each attack type:
- Run every 5 minutes
- Look back 1 hour
- Alert threshold: Greater than 0 events

Detection rules fire when attacks are simulated, enabling
real SOC analyst investigation and response practice.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Windows Server 2019 | Domain Controller |
| Kali Linux | Attack simulation |
| Responder | LLMNR/NBT-NS poisoning |
| Impacket Suite | Kerberoasting, PTH, PsExec |
| Mimikatz | Credential dumping simulation |
| BloodHound | AD attack path mapping |
| Microsoft Sentinel | Cloud SIEM — log collection |
| Azure Monitor Agent | Log forwarding to Sentinel |
| KQL | Detection query language |

---

## Documentation Structure
active-directory-lab/
├── README.md (this file)
├── 01-LLMNR-Poisoning/
│   ├── README.md
│   └── screenshots/
├── 02-Kerberoasting/
│   ├── README.md
│   └── screenshots/
├── 03-Pass-the-Hash/
│   ├── README.md
│   └── screenshots/
├── 04-PsExec-Lateral-Movement/
│   ├── README.md
│   └── screenshots/
└── 05-Persistence-Scheduled-Task/
├── README.md
└── screenshots/

Each attack folder contains:
- Full investigation writeup
- Screenshots of attack execution
- Screenshots of logs generated
- Screenshots of Sentinel alerts fired
- KQL queries used
- Remediation recommendations

---

## Lab Status

| Phase | Task | Status |
|---|---|---|
| Phase 1 | Network setup and connectivity | ✅ Complete |
| Phase 2 | Domain Controller configuration | ✅ Complete |
| Phase 3 | HP Workstation domain join | ✅ Complete |
| Phase 4 | Azure Sentinel connection | ⏳ In Progress |
| Phase 5 | LLMNR Poisoning simulation | ⏳ In Progress |
| Phase 6 | Kerberoasting simulation | ⏳ In Progress |
| Phase 7 | Pass-the-Hash simulation | ⏳ In Progress |
| Phase 8 | PsExec simulation | ⏳ In Progress |
| Phase 9 | Persistence simulation | ⏳ In Progress |
| Phase 10 | Full documentation on GitHub | ⏳ In Progress |

*This repository is actively being built.
Documentation is added as each phase is completed.*

---

## Related Projects

- **SOC Investigations:** github.com/john-opsec34g/soc-analyst-labs
- **AI SOC Triage Tool:** github.com/john-opsec34g/ai-soc-triage-tool

---

## Author

**John Victory** — Junior SOC Analyst in Training
Nigeria | Open to Remote Roles Globally

Certifications: SC-200 | AZ-500 | SC-100 | AZ-900 |
Cisco CyberOps | Security+ | Network+ | ISC2 CC

GitHub: github.com/john-opsec34g
LinkedIn: linkedin.com/in/john-victory

---

## Disclaimer

All attacks documented in this repository are conducted
in a controlled private lab environment for educational
and defensive security purposes only.

---

## Lab Architecture
