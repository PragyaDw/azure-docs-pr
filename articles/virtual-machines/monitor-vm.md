---
title: Monitor Azure Virtual Machines
description: Start here to learn how to monitor Azure Virtual Machines and Virtual Machine Scale Sets.
ms.date: 03/27/2024
ms.custom: horz-monitor
ms.topic: conceptual
ms.service: virtual-machines

#customer intent: As a cloud administrator, I want to understand how to monitor Azure virtual machines so that I can ensure the health and performance of my virtual machines and applications.
---

# Monitor Azure Virtual Machines

[!INCLUDE [horz-monitor-intro](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-intro.md)]

This article provides an overview of how to monitor the health and performance of Azure virtual machines (VMs).

>[!NOTE]
>This article provides basic information to help you get started with monitoring Azure Virtual Machines. For a complete guide to monitoring your entire environment of Azure and hybrid virtual machines, see the [Monitor virtual machines deployment guide](/azure/azure-monitor/vm/monitor-virtual-machine).

## Overview: Monitor VM host and guest metrics and logs

You can collect metrics and logs from the **VM host**, which is the physical server and hypervisor that creates and manages the VM, and from the **VM guest**, which includes the operating system and applications that run inside the VM.

VM host and guest data is useful in different scenarios:

| Data type | Scenarios | Data collection | Available data |
|-|-|-|-|
| **VM host data** | Monitor the stability, health, and efficiency of the physical host on which the VM is running.<br>(Optional) [Scale up or scale down](/azure/azure-monitor/autoscale/autoscale-overview) based on the load on your application.| Available by default without any additional setup. |[Host performance metrics](#azure-monitor-platform-metrics)<br><br>[Activity logs](#azure-activity-log)<br><br>[Boot diagnostics](#boot-diagnostics)|
| **VM guest data: overview** | Analyze and troubleshoot performance and operational efficiency of workloads running in your Azure environment. | Install [Azure Monitor Agent](/azure/azure-monitor/agents/agents-overview) on the VM and set up a [data collection rule (DCR)](#data-collection-rules). |See various levels of data in the following rows.|
|**Basic VM guest data**|[VM insights](#vm-insights) is a quick and easy way to start monitoring your VM clients, especially useful for exploring overall VM usage and performance when you don't yet know the metric of primary interest.|[Enable VM insights](/azure/azure-monitor/vm/vminsights-enable-overview) to automatically install Azure Monitor Agent and create a predefined DCR.|[Guest performance counters](/azure/azure-monitor/vm/vminsights-performance)<br><br>[Dependencies between application components running on the VM](/azure/azure-monitor/vm/vminsights-maps)|
|**VM operating system monitoring data**|Monitor application performance and events, resource consumption by specific applications and processes, and operating system-level performance and events. Valuable for troubleshooting application-specific issues, optimizing resource usage within VMs, and ensuring optimal performance for workloads running inside VMs.|Install [Azure Monitor Agent](/azure/azure-monitor/agents/agents-overview) on the VM and set up a [DCR](#data-collection-rules).|[Guest performance counters](/azure/azure-monitor/agents/data-collection-rule-azure-monitor-agent)<br><br>[Windows events](/azure/azure-monitor/agents/data-collection-rule-azure-monitor-agent)<br><br>[Syslog events](/azure/azure-monitor/agents/data-collection-syslog)|
|**Advanced/custom VM guest data**|Monitoring of web servers, Linux appliances, and any type of data you want to collect from a VM. |Install [Azure Monitor Agent](/azure/azure-monitor/agents/agents-overview) on the VM and set up a [DCR](#data-collection-rules).|[IIS logs](/azure/azure-monitor/agents/data-collection-iis)<br><br>[SNMP traps](/azure/azure-monitor/agents/data-collection-snmp-data)<br><br>[Any data written to a text or JSON file](/azure/azure-monitor/agents/data-collection-text-log)|

[!INCLUDE [horz-monitor-insights](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-insights.md)]

### VM insights

VM insights monitors your Azure and hybrid virtual machines in a single interface. VM insights provides the following benefits for monitoring VMs in Azure Monitor:

- Simplified onboarding of the Azure Monitor agent and the Dependency agent, so that you can monitor a virtual machine (VM) guest operating system and workloads.
- Predefined data collection rules that collect the most common set of performance data.
- Predefined trending performance charts and workbooks, so that you can analyze core performance metrics from the virtual machine's guest operating system.
- The Dependency map, which displays processes that run on each virtual machine and the interconnected components with other machines and external sources.

:::image type="content" source="media/monitor-vm/vminsights-01.png" lightbox="media/monitor-vm/vminsights-01.png" alt-text="Screenshot of the VM insights 'Logical Disk Performance' view.":::

:::image type="content" source="media/monitor-vm/vminsights-02.png" lightbox="media/monitor-vm/vminsights-02.png" alt-text="Screenshot of the VM insights 'Map' view.":::

For a tutorial on enabling VM insights for a virtual machine, see [Enable monitoring with VM insights for Azure virtual machine](/azure/azure-monitor/vm/tutorial-monitor-vm-enable-insights). For general information about enabling insights and a variety of methods for onboarding VMs, see [Enable VM insights overview](/azure/azure-monitor/vm/vminsights-enable-overview).

If you enable VM insights, the Azure Monitor agent is installed and starts sending a predefined set of performance data to Azure Monitor Logs. You can create other data collection rules to collect events and other performance data. To learn how to install the Azure Monitor agent and create a data collection rule (DCR) that defines the data to collect, see [Tutorial: Collect guest logs and metrics from an Azure virtual machine](/azure/azure-monitor/vm/tutorial-monitor-vm-guest).

[!INCLUDE [horz-monitor-resource-types](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-resource-types.md)]
For more information about the resource types for Virtual Machines, see [Azure Virtual Machines monitoring data reference](monitor-vm-reference.md).

[!INCLUDE [horz-monitor-data-storage](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-data-storage.md)]

[!INCLUDE [horz-monitor-platform-metrics](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-platform-metrics.md)]
Platform metrics for Azure VMs include important *host metrics* such as CPU, network, and disk utilization. Host OS metrics relate to the Hyper-V session that's hosting a guest operating system (guest OS) session.

Metrics for the *guest OS* that runs in a VM must be collected through one or more agents, such as the [Azure Monitor agent](/azure/azure-monitor/agents/azure-monitor-agent-overview), that run on or as part of the guest OS. Guest OS metrics include performance counters that track guest CPU percentage or memory usage, both of which are frequently used for autoscaling or alerting. For more information, see [Guest OS and host OS metrics](/azure/azure-monitor/reference/supported-metrics/metrics-index#guest-os-and-host-os-metrics).

For detailed information about how the Azure Monitor agent collects VM monitoring data, see [Monitor virtual machines with Azure Monitor: Collect data](/azure/azure-monitor/vm/monitor-virtual-machine-data-collection).

For a list of available metrics for Virtual Machines, see [Virtual Machines monitoring data reference](monitor-vm-reference.md#metrics).

[!INCLUDE [horz-monitor-resource-logs](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-resource-logs.md)]

- For the available resource log categories, their associated Log Analytics tables, and the logs schemas for Virtual Machines, see [Virtual Machines monitoring data reference](monitor-vm-reference.md).

> [!IMPORTANT]
> For Azure VMs, all the important data is collected by the Azure Monitor agent. The resource log categories available for Azure VMs aren't important and aren't available for collection from the Azure portal. For detailed information about how the Azure Monitor agent collects VM log data, see [Monitor virtual machines with Azure Monitor: Collect data](/azure/azure-monitor/vm/monitor-virtual-machine-data-collection).

[!INCLUDE [horz-monitor-activity-log](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-activity-log.md)]

## Data collection rules

[Data collection rules (DCRs)](/azure/azure-monitor/essentials/data-collection-rule-overview) define data collection from the Azure Monitor Agent and are stored in your Azure subscription. For VMs, DCRs define data such as events and performance counters to collect, and specify locations such as Log Analytics workspaces to send the data. A single VM can be associated with multiple DCRs, and a single DCR can be associated with multiple VMs.

### VM insights DCR

VM insights creates a DCR that collects common performance counters for the client operating system and sends them to the [InsightsMetrics](/azure/azure-monitor/reference/tables/insightsmetrics) table in the Log Analytics workspace. For a list of performance counters collected, see [How to query logs from VM insights](/azure/azure-monitor/vm/vminsights-log-query#performance-records). You can use this DCR with other VMs instead of creating a new DCR for each VM.

You can also optionally enable collection of processes and dependencies, which populates the following tables and enables the VM insights Map feature.

- [VMBoundPort](/azure/azure-monitor/reference/tables/vmboundport): Traffic for open server ports on the machine
- [VMComputer](/azure/azure-monitor/reference/tables/vmcomputer): Inventory data for the machine
- [VMConnection](/azure/azure-monitor/reference/tables/vmconnection): Traffic for inbound and outbound connections to and from the machine
- [VMProcess](/azure/azure-monitor/reference/tables/vmprocess): Processes running on the machine

### Collect performance counters

VM insights collects a common set of performance counters in Logs to support its performance charts. If you aren't using VM insights, or want to collect other counters or send them to other destinations, you can create other DCRs. You can quickly create a DCR by using the most common counters.

You can send performance data from the client to either Azure Monitor Metrics or Azure Monitor Logs. VM insights sends performance data to the [InsightsMetrics](/azure/azure-monitor/reference/tables/insightsmetrics) table. Other DCRs send performance data to the [Perf](/azure/azure-monitor/reference/tables/perf) table. For guidance on creating a DCR to collect performance counters, see [Collect events and performance counters from virtual machines with Azure Monitor Agent](/azure/azure-monitor/agents/data-collection-rule-azure-monitor-agent).

[!INCLUDE [horz-monitor-analyze-data](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-analyze-data.md)]

[!INCLUDE [horz-monitor-external-tools](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-external-tools.md)]

### Query logs from VM insights

VM insights stores the data it collects in Azure Monitor Logs, and the insights provide performance and map views that you can use to interactively analyze the data. You can work directly with this data to drill down further or perform custom analyses. For more information and to get sample queries for this data, see [How to query logs from VM insights](/azure/azure-monitor/vm/vminsights-log-query).

[!INCLUDE [horz-monitor-kusto-queries](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-kusto-queries.md)]

To analyze log data that you collect from your VMs, you can use [log queries](/azure/azure-monitor/logs/get-started-queries) in [Log Analytics](/azure/azure-monitor/logs/log-analytics-tutorial). Several [built-in queries](/azure/azure-monitor/logs/queries) for VMs are available to use, or you can create your own queries. You can interactively work with the results of these queries, include them in a workbook to make them available to other users, or generate alerts based on their results.

To access built-in Kusto queries for your VM, select **Logs** in the **Monitoring** section of the left navigation on your VM's Azure portal page. On the **Logs** page, select the **Queries** tab, and then select the query to run.

:::image type="content" source="media/monitor-vm/log-analytics-query.png" lightbox="media/monitor-vm/log-analytics-query.png" alt-text="Screenshot of the 'Logs' pane displaying Log Analytics query results.":::

[!INCLUDE [horz-monitor-alerts](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-alerts.md)]

You can create a single multi-resource alert rule that applies to all VMs in a particular resource group or subscription within the same region. See [Create availability alert rule for Azure virtual machine (preview)](/azure/azure-monitor/vm/tutorial-monitor-vm-alert-availability) for a tutorial using the availability metric.

[!INCLUDE [horz-monitor-insights-alerts](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-recommended-alert-rules.md)]

Recommended alert rules for Azure VMs include the [VM availability metric](monitor-vm-reference.md#vm-availability-metric-preview), which alerts when a VM stops running.

For more information, see [Tutorial: Enable recommended alert rules for Azure virtual machine](/azure/azure-monitor/vm/tutorial-monitor-vm-alert-recommended). 

### Common alert rules

To see common VM log alert rules in the Azure portal, go to the **Queries** pane in Log Analytics. For **Resource type**, enter **Virtual machines**, and for **Type**, enter **Alerts**.

For a list and discussion of common Virtual Machines alert rules, see [Common alert rules](/azure/azure-monitor/vm/monitor-virtual-machine-alerts#common-alert-rules).

[!INCLUDE [horz-monitor-advisor-recommendations](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-advisor-recommendations.md)]

## Other VM monitoring options

Azure VMs has the following non-Azure Monitor monitoring options:

### Boot diagnostics

Boot diagnostics is a debugging feature for Azure VMs that allows you to diagnose VM boot failures by collecting serial log information and screenshots of a VM as it boots up. When you create a VM in the Azure portal, boot diagnostics is enabled by default. For more information, see [Azure boot diagnostics](boot-diagnostics.md).

### Troubleshoot performance issues

[The Performance Diagnostics tool](/troubleshoot/azure/virtual-machines/performance-diagnostics?toc=/azure/azure-monitor/toc.json) helps troubleshoot performance issues on Windows or Linux virtual machines by quickly diagnosing and providing insights on issues it currently finds on your machines. The tool doesn't analyze historical monitoring data you collect, but rather checks the current state of the machine for known issues, implementation of best practices, and complex problems that involve slow VM performance or high usage of CPU, disk space, or memory.

## Related content

- For a reference of the metrics, logs, and other important values for Virtual Machines, see [Virtual Machines monitoring data reference](monitor-vm-reference.md).
- For general details about monitoring Azure resources, see [Monitor Azure resources with Azure Monitor](/azure/azure-monitor/essentials/monitor-azure-resource).
- For guidance based on the five pillars of the Azure Well-Architected Framework, see [Best practices for monitoring virtual machines in Azure Monitor](/azure/azure-monitor/best-practices-vm).
- To get started with VM insights, see [Overview of VM insights](/azure/azure-monitor/vm/vminsights-overview).
- To learn how to collect and analyze VM host and client metrics and logs, see the training course [Monitor your Azure virtual machines with Azure Monitor](/training/modules/monitor-azure-vm-using-diagnostic-data).
- For a complete guide to monitoring Azure and hybrid VMs, see the [Monitor virtual machines deployment guide](/azure/azure-monitor/vm/monitor-virtual-machine).
