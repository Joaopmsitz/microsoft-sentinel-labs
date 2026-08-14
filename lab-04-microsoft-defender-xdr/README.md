# Lab 04 — Microsoft Defender XDR

## 🎯 Objective

Deploy and validate Microsoft Defender XDR capabilities for endpoint security monitoring and incident investigation.

The lab focuses on onboarding a Windows Server endpoint into Microsoft Defender for Endpoint, validating endpoint telemetry, investigating process execution activity, creating and tuning a detection rule in Microsoft Sentinel, and analyzing the resulting incidents through the Microsoft Defender XDR portal.

---

## 🧪 Lab Environment

* Microsoft Defender XDR
* Microsoft Defender for Endpoint
* Microsoft Defender for Cloud
* Microsoft Sentinel
* Azure Virtual Machine
* Windows Server 2022
* Log Analytics
* KQL
* SecurityEvent
* DeviceProcessEvents
* Microsoft Defender Portal

---

## 🏗️ Architecture

```text
Windows Server 2022 VM
        ↓
Microsoft Defender for Endpoint
        ↓
Microsoft Defender XDR
        ↓
Microsoft Sentinel
        ↓
Analytics Rule
        ↓
Alert
        ↓
Incident
        ↓
Investigation & Response
```

Microsoft Defender for Endpoint provides endpoint security telemetry and investigation capabilities through Microsoft Defender XDR. Microsoft Sentinel can integrate with Microsoft Defender XDR to correlate security alerts and incidents with other security data sources.

---

## ⚙️ Defender for Cloud Configuration

The Azure virtual machine was protected by Microsoft Defender for Servers.

The following Defender for Cloud capabilities were reviewed during the lab:

* Defender for Servers
* Defender for Key Vault
* Defender CSPM
* Endpoint Protection
* Vulnerability Assessment
* Guest Configuration
* Agentless Scanning
* File Integrity Monitoring

The VM was also evaluated through Microsoft Defender for Cloud security recommendations.

---

## 🔐 Network Security Hardening

A Defender for Cloud recommendation identified exposed remote management ports on the virtual machine.

The VM's Network Security Group (NSG) was reviewed.

The original RDP rule allowed inbound TCP/3389 traffic from any source. The rule was modified to restrict access to the administrator's public IP address.

### Before

| Setting  | Value |
| -------- | ----- |
| Priority | 300   |
| Name     | RDP   |
| Port     | 3389  |
| Protocol | TCP   |
| Source   | Any   |
| Action   | Allow |

### After

| Setting  | Value                   |
| -------- | ----------------------- |
| Priority | 300                     |
| Name     | RDP                     |
| Port     | 3389                    |
| Protocol | TCP                     |
| Source   | Administrator Public IP |
| Action   | Allow                   |

This reduced exposure to Internet-based brute-force attacks while preserving administrative access.

---

## 🖥️ Microsoft Defender for Endpoint Onboarding

The Windows Server 2022 virtual machine was onboarded into Microsoft Defender for Endpoint.

The Defender deployment package was generated from the Microsoft Defender portal and transferred to the virtual machine.

After executing the onboarding process, the endpoint appeared in Microsoft Defender XDR Device Inventory.

The device status changed from:

```text
Can be onboarded
```

to:

```text
Onboarded
Active
```

The endpoint was successfully recognized as an onboarded server by Microsoft Defender XDR.

---

## 🔎 Endpoint Validation

The Defender XDR Device Inventory was used to validate the endpoint.

### Observed Device Information

| Property   | Value               |
| ---------- | ------------------- |
| Device     | vm-soc-lab          |
| OS         | Windows Server 2022 |
| IP         | 10.0.0.4            |
| Platform   | Azure               |
| Status     | Active              |
| Onboarding | Onboarded           |

The endpoint subsequently became available for investigation through Microsoft Defender XDR.

---

## 🕵️ Advanced Hunting

Microsoft Defender XDR Advanced Hunting was used to inspect endpoint process creation telemetry through the `DeviceProcessEvents` table.

Example telemetry fields include:

* `DeviceName`
* `AccountName`
* `FileName`
* `ProcessCommandLine`
* `InitiatingProcessFileName`

This endpoint telemetry was reviewed alongside Windows Event ID 4688 data ingested into Microsoft Sentinel through the `SecurityEvent` table.

### Example Advanced Hunting Query

```kusto
DeviceProcessEvents
| where isnotempty(ProcessCommandLine)
| project
    Timestamp,
    DeviceName,
    AccountName,
    FileName,
    ProcessCommandLine,
    InitiatingProcessFileName
| order by Timestamp desc
```

This demonstrated the difference between endpoint telemetry available through Microsoft Defender XDR Advanced Hunting and Windows security events ingested into Microsoft Sentinel.

---

## 🚨 Initial Detection Rule

The original Lab 03 analytics rule generated alerts for collected Windows Event ID 4688 process creation events.

```kusto
SecurityEvent
| where EventID == 4688
| project
    TimeGenerated,
    Computer,
    Account,
    NewProcessName,
    ParentProcessName,
    CommandLine
| order by TimeGenerated desc
```

Event ID 4688 represents Windows process creation events when Process Creation Auditing is enabled and the events are successfully collected by Microsoft Sentinel.

Although useful for telemetry validation, this approach generated significant alert noise because normal administrative activity such as:

* `cmd.exe`
* `powershell.exe`
* Windows services
* System processes

also generated events.

The rule therefore required tuning.

---

## 🛠️ Detection Rule Tuning

The original rule was modified to focus on potentially suspicious command-line activity rather than generating alerts for every collected process creation event.

### Rule

**Name:** Suspicious Process Execution - Windows

| Setting           | Value            |
| ------------------ | ---------------- |
| Severity           | Medium            |
| Frequency          | Every 5 minutes   |
| Lookup Period       | Last 10 minutes  |
| Incident Creation  | Enabled           |
| Alert Grouping     | Enabled           |

> **Note:** Lookup Period was set wider than the query Frequency (10 minutes vs. 5 minutes) to create overlap between consecutive runs. This avoids gaps caused by ingestion delay, where an event could otherwise fall between two scheduled runs and never be evaluated.

### Detection Logic

```kusto
SecurityEvent
| where EventID == 4688
| where isnotempty(CommandLine)
| where CommandLine has_any (
    "-enc",
    "-encodedcommand",
    "DownloadString",
    "Invoke-WebRequest",
    "FromBase64String",
    "IEX",
    "mshta",
    "rundll32",
    "regsvr32",
    "certutil"
)
| project
    TimeGenerated,
    Computer,
    Account,
    NewProcessName,
    ParentProcessName,
    CommandLine
| order by TimeGenerated desc
```

The rule was designed to reduce false positives while preserving visibility into potentially suspicious command-line execution patterns.

---

## 🧪 Detection Testing

A controlled test was performed against the Windows Server endpoint.

The test was designed to validate the command-line matching logic rather than simulate a real attack technique.

The following PowerShell command intentionally contained the `IEX` keyword monitored by the analytics rule:

```powershell
Write-Host "IEX test for Sentinel Lab 03"
```

The resulting process creation event was observed in Microsoft Sentinel.

### Observed Event

**Process**

```text
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

**Parent Process**

```text
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

**Command Line**

```text
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -Command "Write-Host 'IEX test for Sentinel Lab 03'"
```

This confirmed that command-line telemetry was available for detection and investigation.

> **Note:** The test did not execute `Invoke-Expression`. The `IEX` string was intentionally included only to validate the detection rule's command-line matching logic.

---

## 🚨 Incident Investigation

The detection generated a Microsoft Sentinel incident.

| Property      | Value                                  |
| ------------- | --------------------------------------- |
| Incident Name | Suspicious Process Execution - Windows |
| Incident ID   | Lab-generated                          |
| Severity      | Medium                                 |
| Category      | Execution                              |
| Status        | Active                                 |
| Device        | vm-soc-lab                             |
| User          | vm-soc-lab\analyst01                   |

The incident was investigated by reviewing the affected device, process execution details, command-line activity, parent process, user context, and the surrounding telemetry.

The activity was identified as an intentional test generated during the lab rather than a real malicious execution.

The incident was classified as a **Benign Positive** and subsequently resolved.

---

## 🔬 False Positive Analysis

During testing, another incident was generated by:

**Process**

```text
C:\Windows\System32\rundll32.exe
```

**Parent Process**

```text
C:\Windows\System32\svchost.exe
```

**Command Line**

```text
"C:\Windows\system32\rundll32.exe" /d acproxy.dll,PerformAutochkOperations
```

The event demonstrated an important SOC investigation principle:

> A process name alone is not sufficient to determine whether activity is malicious.

Although `rundll32.exe` is frequently associated with Living-off-the-Land techniques, it is also a legitimate Windows component commonly invoked by operating system services and signed Microsoft binaries.

The complete process context, parent process, command line, user, and surrounding activity must therefore be evaluated before classifying the event as malicious.

This activity was identified as legitimate Windows behavior and used to improve detection tuning.

---

## 🧠 Investigation Workflow

```text
Endpoint Telemetry
        ↓
Process Creation Event
        ↓
KQL Detection
        ↓
Alert
        ↓
Incident
        ↓
Entity & Evidence Analysis
        ↓
Command-Line Investigation
        ↓
False Positive / Benign Activity Assessment
        ↓
Incident Resolution
```

---

## 🛡️ Security Operations Concepts Demonstrated

* Microsoft Defender XDR
* Microsoft Defender for Endpoint
* Microsoft Defender for Cloud
* Endpoint Onboarding
* Device Inventory
* Endpoint Telemetry
* Advanced Hunting
* `DeviceProcessEvents`
* Microsoft Sentinel Integration
* `SecurityEvent`
* KQL
* Windows Event ID 4688
* Command-Line Investigation
* Detection Engineering
* Analytics Rule Tuning
* Alert Noise Reduction
* False Positive Analysis
* Incident Investigation
* Incident Classification
* Incident Resolution
* Network Security Group Hardening
* SOC Investigation Workflow

---

## 📚 SC-200 Alignment

This lab supports SC-200 preparation in areas including:

* Investigate incidents in Microsoft Defender XDR
* Investigate Microsoft Defender for Endpoint alerts
* Investigate devices and users
* Use Advanced Hunting
* Analyze endpoint telemetry
* Create and tune Microsoft Sentinel analytics rules
* Use KQL for security investigations
* Investigate process execution
* Analyze command-line activity
* Reduce false positives
* Investigate and respond to security incidents
* Apply security recommendations
* Perform incident response

---

## 📸 Evidence

### 1. Defender XDR — Endpoint Onboarded

Screenshot showing `vm-soc-lab` successfully onboarded and active in Microsoft Defender XDR Device Inventory.

<img width="1439" height="814" alt="Microsoft Defender XDR endpoint onboarded" src="https://github.com/user-attachments/assets/112533b6-b923-48a2-b61e-b203a734e83b" />

### 2. Command-Line Evidence & Resolved Incident

Screenshot showing process execution evidence, command-line telemetry, and the incident resolved after investigation.

<img width="1439" height="814" alt="Command-line evidence and resolved incident" src="https://github.com/user-attachments/assets/d206a69e-086e-48de-9f70-b80569db2290" />

---

## ✅ Status

**Completed**

The lab successfully demonstrated endpoint onboarding into Microsoft Defender for Endpoint, device visibility through Microsoft Defender XDR, endpoint telemetry investigation using Advanced Hunting, Microsoft Sentinel integration, KQL-based detection, analytics rule tuning, command-line investigation, incident generation, false-positive analysis, incident classification, and incident resolution within a modern SOC workflow.
