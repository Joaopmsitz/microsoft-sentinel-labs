# 🔰 Lab 01 - Microsoft Sentinel Deployment

## Objective

Deploy a Microsoft Sentinel environment in Azure for hands-on SOC Analyst training and SC-200 preparation.

## Environment

| Component | Name |
|------------|------------|
| Resource Group | rg-soc-lab |
| Log Analytics Workspace | law-soc-lab |
| SIEM Platform | Microsoft Sentinel |
| Region | Brazil South |

## Architecture

Azure Subscription
↓
Resource Group
↓
Log Analytics Workspace
↓
Microsoft Sentinel
↓
Microsoft Defender Portal

## Result

The Microsoft Sentinel workspace was successfully deployed and integrated with Microsoft Defender Portal as the primary workspace.

## Screenshot

<img width="1439" height="812" alt="image" src="https://github.com/user-attachments/assets/bd777a6c-75ce-4301-9513-a7bb40fc15f0" />

## Skills Developed

- Resource Group Management
- Log Analytics Workspace Deployment
- Microsoft Sentinel Deployment
- Defender Portal Integration
- Cloud SIEM Fundamentals
- Azure Cost Management

## Lessons Learned

- Microsoft Sentinel relies on a Log Analytics Workspace for data storage and analysis.
- Microsoft Defender Portal integrates with Sentinel workspaces for centralized security operations.
- Cost monitoring should be configured before deploying cloud resources.
- A properly configured workspace serves as the foundation for future data ingestion, analytics rules, threat hunting, and incident investigation activities.

## Outcome

Successfully deployed a Microsoft Sentinel environment and established the foundation for future SOC operations, threat hunting, and SC-200 practical exercises.
