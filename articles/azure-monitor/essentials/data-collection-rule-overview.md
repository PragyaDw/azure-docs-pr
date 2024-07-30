---
title: Data collection rules in Azure Monitor
description: Overview of data collection rules (DCRs) in Azure Monitor including their contents and structure and how you can create and work with them.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/18/2024
ms.reviewer: nikeist
ms.custom: references_regions
---

# Data collection rules in Azure Monitor
Data collection rules (DCRs) are sets of instructions supporting [data collection in Azure Monitor](../essentials/data-collection.md). They provide a consistent and centralized way to define and customize different data collection scenarios. Depending on the scenario, DCRs specify such details as what data should be collected, how to transform that data, and where to send it. 

DCRs are stored in Azure so that you can centrally manage them. Different components of a data collection workflow will access the DCR for particular information that it requires. In some cases, you can use the Azure portal to configure data collection, and Azure Monitor will create and manage the DCR for you. Other scenarios will require you to create your own DCR. You may also choose to customize an existing DCR to meet your required functionality.

For example, the following diagram illustrates data collection for the [Azure Monitor agent](../agents/azure-monitor-agent-overview.md) running on a virtual machine. In this scenario, the DCR specifies events and performance data, which the agent uses to determine what data to collect from the machine and send to Azure Monitor. Once the data is delivered, the data pipeline runs the transformation specified in the DCR to filter and modify the data and then sends the data to the specified workspace and table. DCRs for other data collection scenarios may contain different information.

:::image type="content" source="media/data-collection-rule-overview/overview-agent.png" lightbox="media/data-collection-rule-overview/overview-agent.png" alt-text="Diagram that shows basic operation for DCR using Azure Monitor Agent." border="false":::

## Data collection in Azure Monitor
DCRs are part of a new [ETL](/azure/architecture/data-guide/relational-data/etl)-like data collection pipeline being implemented by Azure Monitor that improves on legacy data collection methods. This process uses a common data ingestion pipeline for all data sources and provides a standard method of configuration that's more manageable and scalable than current methods. Specific advantages of the new data collection include the following:

- Common set of destinations for different data sources.
- Ability to apply a transformation to filter or modify incoming data before it's stored.
- Consistent method for configuration of different data sources.
- Scalable configuration options supporting infrastructure as code and DevOps processes.

When implementation is complete, all data collected by Azure Monitor will use the new data collection process and be managed by DCRs. Currently, only [certain data collection methods](#data-collection-scenarios) support the ingestion pipeline, and they may have limited configuration options. There's no difference between data collected with the new ingestion pipeline and data collected using other methods. The data is all stored together as [Logs](../logs/data-platform-logs.md) and [Metrics](data-platform-metrics.md), supporting Azure Monitor features such as log queries, alerts, and workbooks. The only difference is in the method of collection.


## View data collection rules
There are multiple ways to view the DCRs in your subscription.

### [Portal](#tab/portal)
To view your DCRs in the Azure portal, select **Data Collection Rules** under **Settings** on the **Monitor** menu.

:::image type="content" source="media/data-collection-rule-overview/view-data-collection-rules.png" lightbox="media/data-collection-rule-overview/view-data-collection-rules.png" alt-text="Screenshot that shows DCRs in the Azure portal.":::

Select a DCR to view its details. For DCRs supporting VMs, you can view and modify its associations and the data that it collects. For other DCRs, use the **JSON view** to view the details of the DCR. See [Create and edit data collection rules (DCRs) in Azure Monitor](./data-collection-rule-create-edit.md) for details on how you can modify them.

> [!NOTE]
> Although this view shows all DCRs in the specified subscriptions, selecting the **Create** button will create a data collection for Azure Monitor Agent. Similarly, this page will only allow you to modify DCRs for Azure Monitor Agent. For guidance on how to create and update DCRs for other workflows, see [Create and edit data collection rules (DCRs) in Azure Monitor](./data-collection-rule-create-edit.md).

### [PowerShell](#tab/powershell)
Use [Get-AzDataCollectionRule](/powershell/module/az.monitor/get-azdatacollectionrule) to retrieve the DCRs in your subscription.


```powershell
Get-AzDataCollectionRule 
```

Use [Get-azDataCollectionRuleAssociation](/powershell/module/az.monitor/get-azdatacollectionruleassociation) to retrieve the DCRs associated with a VM. 

```powershell
get-azDataCollectionRuleAssociation -TargetResourceId /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/my-vm | foreach {Get-azDataCollectionRule -RuleId $_.DataCollectionRuleId }
```

### [CLI](#tab/cli)
Use [az monitor data-collection rule](/cli/azure/monitor/data-collection/rule) to work the DCRs using Azure CLI.

Use the following to return all DCRs in your subscription.

```azurecli
az monitor data-collection rule list
```

Use the following to return DCR associations for a VM.

```azurecli
az monitor data-collection rule association list --resource "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/my-vm "
```
---

## Data collection rule associations

Some data collection scenarios will use data collection rule associations (DCRAs), which associate a DCR with an object being monitored. A single object can be associated with multiple DCRs, and a single DCR can be associated with multiple objects. This allows you to manage a single DCR for a group of objects.

For example, the diagram above illustrates data collection for the Azure Monitor agent. When the agent is installed, it connects to Azure Monitor to retrieve any DCRs that are associated with it. You can create an association with to the same DCRs for multiple VMs.

## Data collection scenarios
The following table describes the data collection scenarios that are currently supported using DCR and the new data ingestion pipeline. See the links in each entry for details.

| Scenario | Description |
| --- | --- |
| Virtual machines | Install the [Azure Monitor agent](../agents/agents-overview.md) on a VM and associate it with one or more DCRs that define the events and performance data to collect from the client operating system. You can perform this configuration using the Azure portal so you don't have to directly edit the DCR.<br><br>See [Collect events and performance counters from virtual machines with Azure Monitor Agent](../agents/data-collection-rule-azure-monitor-agent.md). |
| | When you enable [VM insights](../vm/vminsights-overview.md) on a virtual machine, it deploys the Azure Monitor agent to telemetry from the VM client. The DCR is created for you automatically to collect a predefined set of performance data.<br><br>See [Enable VM Insights overview](../vm/vminsights-enable-overview.md). |
| Container insights | When you enable [Container insights](../containers/container-insights-overview.md) on your Kubernetes cluster, it deploys a containerized version of the Azure Monitor agent to send logs from the cluster to a Log Analytics workspace. The DCR is created for you automatically, but you may need to modify it to customize your collection settings.<br><br>See [Configure data collection in Container insights using data collection rule](../containers/container-insights-data-collection-dcr.md).  | 
| Log ingestion API | The [Logs ingestion API](../logs/logs-ingestion-api-overview.md) allows you to send data to a Log Analytics workspace from any REST client. The API call specifies the DCR to accept its data and specifies the DCR's endpoint. The DCR understands the structure of the incoming data, includes a transformation that ensures that the data is in the format of the target table, and specifies a workspace and table to send the transformed data.<br><br>See [Logs Ingestion API in Azure Monitor](../logs/logs-ingestion-api-overview.md). |
| Azure Event Hubs | Send data to a Log Analytics workspace from [Azure Event Hubs](../../event-hubs/event-hubs-about.md). The DCR defines the incoming stream and defines the transformation to format the data for its destination workspace and table.<br><br>See [Tutorial: Ingest events from Azure Event Hubs into Azure Monitor Logs (Public Preview)](../logs/ingest-logs-event-hub.md). |
| Workspace transformation DCR | The workspace transformation DCR is a special DCR that's associated with a Log Analytics workspace and allows you to perform transformations on data being collected using other methods. You create a single DCR for the workspace and add a transformation to one or more tables. The transformation is applied to any data sent to those tables through a method that doesn't use a DCR.<br><br>See [Workspace transformation DCR in Azure Monitor](./data-collection-transformations-workspace.md). |


## Supported regions
Data collection rules are available in all public regions where Log Analytics workspaces and the Azure Government and China clouds are supported. Air-gapped clouds aren't yet supported.

**Single region data residency** is a preview feature to enable storing customer data in a single region and is currently only available in the Southeast Asia Region (Singapore) of the Asia Pacific Geo and the Brazil South (Sao Paulo State) Region of the Brazil Geo. Single-region residency is enabled by default in these regions.

## Data resiliency and high availability
A DCR gets created and stored in a particular region and is backed up to the [paired-region](../../availability-zones/cross-region-replication-azure.md#azure-paired-regions) within the same geography. The service is deployed to all three [availability zones](../../availability-zones/az-overview.md#availability-zones) within the region. For this reason, it's a *zone-redundant service*, which further increases availability.

## Next steps
See the following articles for additional information on how to work with DCRs.

- [Data collection rule structure](data-collection-rule-structure.md) for a description of the JSON structure of DCRs and the different elements used for different workflows.
- [Sample data collection rules (DCRs)](data-collection-rule-samples.md) for sample DCRs for different data collection scenarios.
- [Create and edit data collection rules (DCRs) in Azure Monitor](./data-collection-rule-create-edit.md) for different methods to create DCRs for different data collection scenarios.
- [Azure Monitor service limits](../service-limits.md#data-collection-rules) for limits that apply to each DCR.
