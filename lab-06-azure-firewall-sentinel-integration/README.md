# Lab 06 — Azure Firewall Deployment & Microsoft Sentinel Integration

## 🎯 Objective

Deploy and integrate **Azure Firewall Standard** into an existing Microsoft Sentinel lab environment.

The lab focuses on:

* Azure Firewall deployment
* Firewall Policy configuration
* User Defined Routes (UDR)
* Network traffic redirection
* Diagnostic Settings
* Log Analytics integration
* Firewall telemetry preparation for Microsoft Sentinel
* Troubleshooting asymmetric routing

The goal was not only to deploy the firewall, but also to validate the resulting network behavior and troubleshoot a real connectivity issue introduced by the routing configuration.

---

## 🧪 Lab Environment

| Component               | Configuration            |
| ----------------------- | ------------------------ |
| Microsoft Sentinel      | Existing SOC environment |
| Azure Firewall          | Standard                 |
| Firewall Policy         | `fw-soc-lab-policy`      |
| Virtual Network         | `vm-soc-lab-vnet`        |
| VNet Address Space      | `10.0.0.0/16`            |
| Firewall Subnet         | `AzureFirewallSubnet`    |
| Firewall Subnet CIDR    | `10.0.1.0/26`            |
| Firewall                | `azure-firewall`         |
| Region                  | East US                  |
| Public IP               | `fw-soc-lab-pip-v2`      |
| Route Table             | `rt-soc-lab`             |
| Log Analytics Workspace | `law-soc-lab`            |
| Diagnostic Setting      | `diag-fw-soc-lab`        |
| Workload                | Windows Server 2022      |

---

## 🔗 Integration With Previous Labs

This lab extends the existing Microsoft Sentinel environment by introducing **network security controls and firewall telemetry**.

```text
Windows Server
      │
      ▼
Azure Firewall
      │
      ▼
Log Analytics Workspace
      │
      ▼
Microsoft Sentinel
```

The architecture provides a foundation for collecting and analyzing network security telemetry alongside the security data sources configured in previous labs.

---

# 🏗️ Architecture

```text
                         Internet
                            │
                            ▼
                    ┌─────────────────┐
                    │  Azure Firewall  │
                    │     Standard     │
                    └────────┬────────┘
                             │
                             ▼
                    AzureFirewallSubnet
                       10.0.1.0/26
                             │
                             ▼
                    vm-soc-lab-vnet
                       10.0.0.0/16
                             │
                             ▼
                       Default Subnet
                             │
                             ▼
                     Windows Server 2022
                          vm-soc-lab
```

---

# 🛠️ Deployment

## 1. Virtual Network

The existing Sentinel lab virtual network was used as the foundation for the deployment.

**Virtual Network**

```text
vm-soc-lab-vnet
```

**Address Space**

```text
10.0.0.0/16
```

The existing workload subnet was preserved for the Windows Server VM.

---

## 2. Azure Firewall Subnet

Azure Firewall requires a dedicated subnet named:

```text
AzureFirewallSubnet
```

The following subnet was created:

```text
AzureFirewallSubnet
10.0.1.0/26
```

This isolates the firewall infrastructure from the workload subnet.

---

## 3. Azure Firewall

The firewall was deployed with the following configuration:

```text
Name:
azure-firewall

SKU:
Standard

Region:
East US

Firewall Policy:
fw-soc-lab-policy

Public IP:
fw-soc-lab-pip-v2
```

The firewall provides the centralized network security control point for the environment.

---

# 🔥 Firewall Policy

A dedicated Firewall Policy was created and associated with the Azure Firewall:

```text
fw-soc-lab-policy
```

The policy provides centralized management for future firewall configurations, including:

* Network Rules
* Application Rules
* NAT Rules
* Threat Intelligence configuration

At this stage, the main objective was establishing the firewall and telemetry foundation rather than implementing an extensive rule set.

---

# 🌐 User Defined Route (UDR)

A Route Table was created to force outbound traffic from the workload subnet through Azure Firewall.

**Route Table**

```text
rt-soc-lab
```

**Default Route**

```text
Address Prefix:
0.0.0.0/0

Next Hop Type:
Virtual Appliance

Next Hop IP:
10.0.1.4
```

The address:

```text
10.0.1.4
```

is the private IP address assigned to the Azure Firewall.

### Routing Flow

```text
VM
 │
 │ 0.0.0.0/0
 ▼
Route Table
 │
 ▼
Virtual Appliance
 │
 ▼
10.0.1.4
 │
 ▼
Azure Firewall
 │
 ▼
Internet
```

The route table was associated with the workload subnet:

```text
Default Subnet
      │
      ▼
rt-soc-lab
```

---

# 🧪 Routing Validation

After associating the UDR with the workload subnet, outbound traffic was configured to use Azure Firewall as the next hop.

This validated the intended traffic path:

```text
vm-soc-lab
     │
     ▼
UDR
     │
     ▼
Azure Firewall
     │
     ▼
Internet
```

However, the change introduced an unexpected connectivity problem.

---

# ⚠️ Operational Challenge — Asymmetric Routing

After the UDR was associated with the workload subnet, **Remote Desktop connectivity to the VM was interrupted**.

This became an important troubleshooting scenario in the lab.

## Observed Traffic Flow

Inbound RDP traffic was still reaching the VM through its public IP:

```text
Internet
    │
    ▼
VM Public IP
    │
    ▼
vm-soc-lab
```

However, the VM's outbound traffic was now affected by the UDR:

```text
vm-soc-lab
    │
    ▼
UDR
    │
    ▼
Azure Firewall
    │
    ▼
Internet
```

This resulted in different paths being used for inbound and outbound traffic.

---

# 🧠 Root Cause Analysis

The issue was caused by an **asymmetric routing condition**.

The inbound RDP connection was entering directly through the VM's public IP, while the VM's response traffic was being redirected through Azure Firewall.

```text
Inbound

Internet
   │
   ▼
VM Public IP
   │
   ▼
VM
```

```text
Outbound

VM
   │
   ▼
UDR
   │
   ▼
Azure Firewall
   │
   ▼
Internet
```

Instead of maintaining a consistent traffic path, the connection was using different paths for the request and response.

This demonstrated an important networking security concept:

> Routing changes can affect existing management connectivity even when the firewall itself is operating correctly.

---

# 🔧 Resolution

To restore administrative access to the VM, the UDR association was temporarily removed from the workload subnet.

```text
Default Subnet
      │
      ▼
Remove UDR Association
      │
      ▼
RDP Connectivity Restored
```

The firewall deployment and integration components were preserved while administrative connectivity was restored.

This allowed the lab to continue without leaving the VM inaccessible.

---

# 📡 Diagnostic Settings

Azure Firewall diagnostic settings were configured to send firewall telemetry to the existing Log Analytics Workspace.

**Diagnostic Setting**

```text
diag-fw-soc-lab
```

**Destination**

```text
law-soc-lab
```

**Logs**

```text
allLogs
```

### Telemetry Flow

```text
Azure Firewall
      │
      ▼
Diagnostic Settings
      │
      ▼
Log Analytics
      │
      ▼
Microsoft Sentinel
```

This establishes the telemetry pipeline required for future firewall monitoring and detection engineering.

---

# 🛡️ Security Concepts Demonstrated

This lab provided hands-on experience with:

* Azure Firewall
* Azure Firewall Policy
* User Defined Routes (UDR)
* Virtual Appliances
* Network Segmentation
* Secure Traffic Routing
* Log Analytics
* Diagnostic Settings
* Firewall Telemetry
* Network Security Architecture
* Asymmetric Routing
* Connectivity Troubleshooting
* Security Monitoring Foundations

---

# 📚 SC-200 Alignment

This lab supports several Microsoft Security Operations Analyst concepts, particularly around security data ingestion, network security telemetry, and investigation.

### Relevant Topics

* Configure Azure Firewall
* Integrate Azure security data sources
* Configure Log Analytics ingestion
* Understand security telemetry pipelines
* Investigate network connectivity problems
* Analyze routing behavior
* Prepare firewall telemetry for Microsoft Sentinel
* Understand how network infrastructure contributes to security monitoring

The lab also reinforces the operational mindset required in a SOC environment: **deploy → validate → observe → troubleshoot → recover**.

---

# 📊 Lab Outcome

| Component                    | Result |
| ---------------------------- | :----: |
| Azure Firewall Deployment    |    ✅   |
| Firewall Policy              |    ✅   |
| Public IP Configuration      |    ✅   |
| AzureFirewallSubnet          |    ✅   |
| Route Table (UDR)            |    ✅   |
| Traffic Redirection          |    ✅   |
| Log Analytics Integration    |    ✅   |
| Diagnostic Settings          |    ✅   |
| Routing Validation           |    ✅   |
| Connectivity Troubleshooting |    ✅   |
| Asymmetric Routing Analysis  |    ✅   |

---

# ✅ Status

**Completed**

Azure Firewall Standard was successfully deployed and integrated into the existing Microsoft Sentinel lab environment.

The lab included Firewall Policy configuration, dedicated firewall subnet deployment, User Defined Routes, Diagnostic Settings, and Log Analytics integration.

During validation, the UDR configuration introduced an **asymmetric routing condition that affected RDP connectivity**. The issue was analyzed by comparing inbound and outbound traffic paths, identifying the routing asymmetry, and temporarily removing the UDR association to restore administrative access.

The lab therefore covered not only the **deployment of Azure Firewall**, but also the **validation and troubleshooting of a real network security configuration issue**.

---

## 🔭 Next Steps

Potential extensions for this environment include:

* Create Azure Firewall Network Rules
* Create Application Rules
* Configure DNAT rules
* Enable Threat Intelligence
* Query Firewall logs with KQL
* Create Microsoft Sentinel analytics rules
* Build firewall-focused workbooks
* Create incident investigation scenarios
* Integrate firewall alerts into SOC workflows

---

## 🧩 Key Takeaway

This lab demonstrates the relationship between **network security infrastructure and security operations**.

Deploying a firewall is only one part of the process. Understanding how routing changes affect connectivity, validating traffic paths, analyzing failures, and establishing telemetry for Microsoft Sentinel are equally important skills in a Cloud Security / SOC environment.
