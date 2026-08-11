# Lab 03 — Endpoint Monitoring

## Objective

Configure Microsoft Sentinel to collect and investigate Windows Security Events from a Windows virtual machine using Azure Monitor Agent (AMA).

The lab focuses on endpoint monitoring, security event ingestion, and KQL-based investigation.

## Environment

* Microsoft Sentinel
* Azure Log Analytics
* Azure Monitor Agent (AMA)
* Windows Server virtual machine
* Data Collection Rules (DCR)
* Windows Security Events
* KQL

## Architecture

```text
Windows VM
    ↓
Azure Monitor Agent
    ↓
Data Collection Rule
    ↓
Microsoft-SecurityEvent
    ↓
Log Analytics
    ↓
SecurityEvent
    ↓
Microsoft Sentinel
```

## Configuration

The **Windows Security Events via AMA** connector was configured in Microsoft Sentinel.

A Data Collection Rule was associated with the Windows VM and configured to collect Windows Security Events.

The DCR uses the `Microsoft-SecurityEvent` stream and sends the collected telemetry to the Log Analytics workspace.

## Validation

Windows Security events were first validated directly on the endpoint.

### Event 4688 — Process Creation

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4688} -MaxEvents 5
```

The command successfully returned Windows Security events with Event ID `4688`, confirming that process creation auditing was generating telemetry on the endpoint.

The same events were then queried in Microsoft Sentinel:

```kusto
SecurityEvent
| where TimeGenerated > ago(1h)
| where EventID == 4688
| order by TimeGenerated desc
```

The query successfully returned the events in the `SecurityEvent` table.

## Security Events Observed

| Event ID | Description                 |
| -------: | --------------------------- |
|     4624 | Successful logon            |
|     4625 | Failed logon                |
|     4634 | Account logoff              |
|     4673 | Privileged service called   |
|     4674 | Privileged object operation |
|     4688 | New process created         |

## Key Takeaways

* Configured Azure Monitor Agent for endpoint telemetry.
* Created and associated a Data Collection Rule.
* Configured Windows Security Events via AMA.
* Validated the `Microsoft-SecurityEvent` data stream.
* Confirmed ingestion into the `SecurityEvent` table.
* Used KQL to investigate Windows endpoint activity.
* Practiced fundamentals relevant to SOC operations and SC-200.

## Status

**Completed** ✅

## SC-200 Alignment

This lab supports the following SC-200 skills:

* Configure Microsoft Sentinel data connectors
* Manage security events
* Investigate endpoint activity
* Use KQL for threat hunting
* Analyze Windows security telemetry
