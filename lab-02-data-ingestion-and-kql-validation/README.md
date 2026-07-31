# 🔎 Lab 02 - First Data Source Integration and KQL Exploration

## 🎯 Objective

Configure Azure Activity Logs as the first telemetry source for Microsoft Sentinel and validate log ingestion using KQL queries against live Azure data.

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

**Destination**

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

To validate that log collection was working correctly, administrative actions were performed within the Azure environment.

A temporary Resource Group was created and later removed to generate management-plane activity.

Additional validation actions included:

- Creating Resource Groups
- Updating Resource Groups
- Adding Tags
- Removing Tags
- Creating Diagnostic Settings
- Deleting Test Resources

These actions were used to produce Azure Activity events for ingestion testing.

---

### 3. Validated Log Ingestion

After configuring the Diagnostic Setting and generating Azure activity, events began appearing in the **AzureActivity** table.

Validation query:

```kusto
AzureActivity
| sort by TimeGenerated desc
| take 20
```

---

## 📸 Screenshot

<img width="1439" height="774" alt="image" src="https://github.com/user-attachments/assets/eb82b866-599d-495c-a914-3be13967dc10" />


Example query used:

```kusto
AzureActivity
| sort by TimeGenerated desc
| take 20
```

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
- Diagnostic Settings are required to export Activity Logs into Log Analytics.
- Newly configured log sources may take time before data becomes available.
- KQL queries can be used to validate successful telemetry ingestion.
- Administrative changes within Azure generate valuable audit events for investigation and monitoring.

---

## ✅ Outcome

Successfully integrated Azure Activity Logs with Microsoft Sentinel and validated data ingestion through Log Analytics using KQL queries.

The environment now receives management activity telemetry and is ready for advanced KQL analysis, threat hunting, analytics rules, and future SC-200 security operations exercises.
