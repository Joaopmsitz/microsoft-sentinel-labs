# Lab 05 — Microsoft Sentinel SOAR & Automated Incident Response

## 🎯 Objective

Implement an automated incident response workflow using **Microsoft Sentinel Automation Rules** and an **Azure Logic Apps Playbook**.

The lab focuses on connecting a Sentinel incident to a playbook, configuring a System Assigned Managed Identity, troubleshooting an **HTTP 403 authorization failure**, and validating the automated response.

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
Automated Response
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
Microsoft Sentinel
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

The Azure Logic Apps Playbook uses a **Microsoft Sentinel Incident** trigger and performs an automated incident enrichment action.

```text
Microsoft Sentinel Incident
        ↓
Microsoft Sentinel Incident Trigger
        ↓
Add Comment To Incident (V3)
```

The comment was used as the initial automated response to validate the integration.

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

## 🧪 Validation

The complete workflow was validated using a controlled Sentinel incident.

```text
Incident Created
        ↓
Automation Rule
        ↓
Playbook Triggered
        ↓
Managed Identity
        ↓
RBAC Authorization
        ↓
Add Comment To Incident
        ↓
Successful Execution
```

The playbook successfully added an automated comment to the incident timeline.

Example:

```text
✅ Automated Incident Response Started

Playbook:
pb-soc-incident-response

Stage:
Initial Validation
```

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
* Automated Incident Response
* Incident Enrichment
* Authorization Troubleshooting
* HTTP 403 Analysis

---

## 📚 SC-200 Alignment

* Configure Microsoft Sentinel automation
* Create and manage Automation Rules
* Create and use Playbooks
* Automate incident response
* Investigate Sentinel incidents
* Troubleshoot automation failures
* Implement SOAR workflows

---

## 📸 Evidence

### Successful Playbook Execution

Screenshot showing the successful Logic App execution after correcting the RBAC permissions.

<img width="1439" height="813" alt="Successful Microsoft Sentinel playbook execution" src="https://github.com/user-attachments/assets/f79ebe83-f240-452f-ab59-3ffaf7a639f7" />

---

## 📊 Lab Outcome

| Component                 | Result |
| ------------------------- | ------ |
| Automation Rule           | ✅      |
| Logic Apps Playbook       | ✅      |
| Managed Identity          | ✅      |
| RBAC Configuration        | ✅      |
| 403 Troubleshooting       | ✅      |
| Automated Incident Action | ✅      |
| End-to-End Validation     | ✅      |

---

## ✅ Status

**Completed**

Implemented and validated a Microsoft Sentinel SOAR workflow using **Automation Rules, Azure Logic Apps, Managed Identity, and Azure RBAC**.

The lab included troubleshooting an initial **HTTP 403 authorization failure** and successfully automating an action against a Sentinel incident after correcting the managed identity permissions.
