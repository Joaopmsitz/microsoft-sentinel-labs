# Lab 05 — Microsoft Sentinel SOAR & Automated Incident Response

## 🎯 Objective

Implement an automated incident response workflow using **Microsoft Sentinel Automation Rules** and an **Azure Logic Apps Playbook**.

The lab focuses on connecting a Sentinel incident to a playbook, configuring a System Assigned Managed Identity, troubleshooting an **HTTP 403 authorization failure**, processing incident entities, and validating automated incident triage.

---

## 🧪 Lab Environment

* Microsoft Sentinel
* Azure Logic Apps
* Automation Rules
* Sentinel Playbooks
* System Assigned Managed Identity
* Azure RBAC
* Windows Server 2022
* Log Analytics
* KQL
* `SecurityEvent`

---

## 🔗 Integration With Previous Labs

This lab builds on the Microsoft Sentinel environment and incident workflow established in the previous labs.

The previous labs provided the telemetry, detection, and incident generation used as the starting point for this automation workflow.

```text
Security Telemetry
        ↓
Analytics Rule
        ↓
Alert
        ↓
Incident
        ↓
Automation Rule
        ↓
Logic Apps Playbook
        ↓
Automated Triage
```

---

## 🏗️ Architecture

```text
Microsoft Sentinel Incident
        ↓
Automation Rule
        ↓
Logic Apps Playbook
        ↓
System Assigned Managed Identity
        ↓
Azure RBAC
        ↓
Entity Extraction
        ↓
Automated Incident Comment
```

---

## 🤖 Automation Rule

A Microsoft Sentinel Automation Rule was created to automatically execute the playbook when a new incident is created.

### Configuration

| Setting  | Value                      |
| -------- | -------------------------- |
| Trigger  | When Incident Is Created   |
| Action   | Run Logic Apps Playbook    |
| Playbook | `pb-soc-incident-response` |

---

## 🛠️ Playbook

### `pb-soc-incident-response`

The Azure Logic Apps Playbook uses a **Microsoft Sentinel Incident** trigger and processes incident entities before recording the result in the incident timeline.

The current workflow extracts host entities from the incident and generates an automated triage comment.

```text
Microsoft Sentinel Incident
        ↓
Entities - Get Hosts
        ↓
For Each
        ↓
Add Comment To Incident (V3)
```

The playbook is designed as a reusable incident-response workflow rather than being tied exclusively to a specific detection type.

### Logic App

<img width="1439" height="816" alt="image" src="https://github.com/user-attachments/assets/5abbe489-5c22-4613-8fa4-3e71848342e2" />

### Sentinel Incident

The successful execution was validated directly in the Sentinel incident timeline.

<img width="1439" height="816" alt="image" src="https://github.com/user-attachments/assets/cf59cf8d-9a61-4bbe-9585-ebd58ecd670b" />

---

## 🔐 Managed Identity & RBAC

The Logic App uses a **System Assigned Managed Identity** to authenticate when interacting with Microsoft Sentinel.

During the first execution, the playbook failed with:

```text
HTTP 403 Forbidden
```

The error occurred while attempting to write a comment to the Sentinel incident:

```text
Microsoft.SecurityInsights/incidents/comments/write
```

The managed identity did not have the required permissions.

### Resolution

The required Microsoft Sentinel permissions were granted to the Logic App's managed identity through **Azure RBAC**.

```text
Logic App
    ↓
System Assigned Managed Identity
    ↓
Azure RBAC
    ↓
Microsoft Sentinel
```

After the role assignment was applied, the playbook was executed again successfully.

---

## 🧪 Automated Incident Triage

The playbook extracts host entities from the Sentinel incident and uses the resulting context to generate an automated triage comment.

Example:

```text
=== SOC Automated Triage ===

Incident:
Suspicious Process Execution - Windows

Host:
vm-soc-lab

Stage:
Entity Extraction

Playbook:
pb-soc-incident-response

Status:
Initial triage completed
```

The host value is obtained dynamically from the incident rather than being hardcoded into the playbook.

This allows the workflow to be reused with incidents involving different hosts.

---

## 🧠 Troubleshooting

The main issue encountered during the lab was an **HTTP 403 authorization failure**.

The troubleshooting process was:

```text
403 Forbidden
      ↓
Execution Analysis
      ↓
Managed Identity Identified
      ↓
RBAC Permissions Reviewed
      ↓
Required Permission Granted
      ↓
Playbook Re-executed
      ↓
Success
```

This demonstrated the importance of validating **identity and authorization** when troubleshooting Sentinel automation.

---

## 🛡️ Security Operations Concepts Demonstrated

* Microsoft Sentinel
* SOAR
* Automation Rules
* Azure Logic Apps
* Sentinel Playbooks
* Managed Identity
* Azure RBAC
* Entity Extraction
* Automated Incident Triage
* Incident Response
* Authorization Troubleshooting
* HTTP 403 Analysis

---

## 📚 SC-200 Alignment

* Configure Microsoft Sentinel automation
* Create and manage Automation Rules
* Create and use Playbooks
* Automate incident response
* Work with Sentinel incident entities
* Investigate Sentinel incidents
* Troubleshoot automation failures
* Implement SOAR workflows

---

## 📊 Lab Outcome

| Component                 | Result |
| ------------------------- | ------ |
| Automation Rule           | ✅      |
| Logic Apps Playbook       | ✅      |
| Managed Identity          | ✅      |
| RBAC Configuration        | ✅      |
| Entity Extraction         | ✅      |
| Automated Incident Triage | ✅      |
| 403 Troubleshooting       | ✅      |
| End-to-End Validation     | ✅      |

---

## ✅ Status

**Completed**

Implemented and validated a Microsoft Sentinel SOAR workflow using **Automation Rules, Azure Logic Apps, Managed Identity, Azure RBAC, and incident entity processing**.

The lab included troubleshooting an initial **HTTP 403 authorization failure**, extracting host information from a Sentinel incident, and successfully generating an automated triage comment in the incident timeline.
