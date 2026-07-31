# 🔎 Lab 02 - First Data Source Integration and KQL Exploration

## 🎯 Objective

Configure Azure Activity Logs as the first telemetry source for Microsoft Sentinel and validate successful log ingestion using KQL queries against live Azure data.

---

## 🏛️ Data Flow

```text
Azure Subscription
        │
        ▼
Azure Activity Logs
        │
        ▼
Diagnostic Settings
        │
        ▼
Log Analytics Workspace
        │
        ▼
Microsoft Sentinel
```

---

## 🛠️ Tasks Performed

### 1. Configured Azure Activity Log Collection

Created a Diagnostic Setting to forward Azure Activity Logs to the Log Analytics Workspace.

**Diagnostic Setting Name**

```text
AzureActivityToSentinel
```

**Destination Workspace**

```text
law-soc-lab
```

**Log Categories Enabled**

- Administrative
- Security
- ServiceHealth
- Alert
- Recommendation
- Policy
- Autoscale
- ResourceHealth

---

### 2. Generated Test Activity

To validate that log collection was working correctly, several administrative actions were performed within the Azure environment.

A temporary Resource Group was created and later deleted to generate management-plane activity.

Additional validation actions included:

- Creating Resource Groups
- Deleting Resource Groups
- Updating Resource Groups
- Adding Tags
- Removing Tags
- Creating Diagnostic Settings
- Modifying Azure Resources

These actions generated Azure Activity events that were later collected by Microsoft Sentinel.

---

### 3. Validated Log Ingestion

After configuring Diagnostic Settings and generating Azure management activity, events began appearing in the **AzureActivity** table.

**Validation Query**

```kusto
AzureActivity
| project TimeGenerated,
          OperationNameValue,
          ActivityStatusValue,
          ResourceProviderValue
| sort by TimeGenerated desc
```

---

## 📸 Screenshot

![Azure Activity Log Ingestion Validation](https://github.com/user-attachments/assets/eb82b866-599d-495c-a914-3be13967dc10)

The screenshot demonstrates successful ingestion of Azure Activity Logs into the Log Analytics Workspace and retrieval through KQL queries.

---

## 📊 Example Events Observed

Examples of ingested activity included:

- Diagnostic Settings Operations
- Resource Group Creation
- Resource Group Deletion
- Resource Group Updates
- Tag Modifications
- Workspace Operations
- Azure Resource Management Activities

---

## 🧠 Skills Developed

- Microsoft Sentinel Data Sources
- Azure Activity Logs
- Azure Monitor
- Diagnostic Settings
- Log Analytics Workspace
- Log Ingestion Validation
- KQL Fundamentals
- Azure Resource Monitoring
- Security Telemetry Collection

---

## 📚 Lessons Learned

- Microsoft Sentinel requires telemetry sources to provide visibility.
- Azure Activity Logs provide management-plane visibility into Azure resources.
- Diagnostic Settings are required to export Activity Logs into Log Analytics Workspace.
- Newly configured log sources may require time before data becomes available.
- KQL queries can be used to validate successful telemetry ingestion.
- Administrative operations performed in Azure create valuable audit records for monitoring and investigation purposes.

---

## ✅ Outcome

Successfully integrated Azure Activity Logs with Microsoft Sentinel and validated data ingestion through Log Analytics using KQL queries.

The environment now receives Azure management activity telemetry and is ready for more advanced Microsoft Sentinel use cases, including threat hunting, analytics rules, incident investigation, and SC-200 practical exercises.
