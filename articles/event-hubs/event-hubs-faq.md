---
title: Azure Event Hubs FAQ | Microsoft Docs
description: Azure Event Hubs frequently asked questions (FAQ)
services: event-hubs
documentationcenter: na
author: sethmanheim
manager: timlt
editor: ''

ms.assetid: bfa10984-eb22-4671-861a-f377a90d9372
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/13/2017
ms.author: sethm;shvija

---

# Event Hubs frequently asked questions

## General

### What is the difference between Event Hubs Basic and Standard tiers?
The Standard tier of Azure Event Hubs provides features beyond what is available in the Basic tier. The following features are included with Standard:
* Longer event retention
* Additional brokered connections, with an overage charge for more than the number included
* More than a single Consumer Group
* [Archive](https://docs.microsoft.com/azure/event-hubs/event-hubs-archive-overview)

For a more details regarding pricing tiers, including Dedicated Event Hubs, see the [Event Hubs pricing details](https://azure.microsoft.com/pricing/details/event-hubs/).

### What are Event Hubs throughput units?
You explicitly select Event Hubs throughput units, either through the Azure portal or Event Hubs resource manager templates. Throughput units apply to all Event Hubs in an Event Hubs namespace, and each throughput unit entitles the namespace to the following capabilities:

* Up to 1 MB per second of ingress events (events sent into an Event Hub), but no more than 1000 ingress events, management operations or control API calls per second.
* Up to 2 MB per second of egress events (events consumed from an Event Hub).
* Up to 84 GB of event storage (sufficient for the default 24-hour retention period).

Event Hubs throughput units are billed hourly, based on the maximum number of units selected during the given hour.

### How are Event Hubs throughput unit limits enforced?
If the total ingress throughput or the total ingress event rate across all Event Hubs in a namespace exceeds the aggregate throughput unit allowances, senders will be throttled and will receive errors indicating that the ingress quota has been exceeded.

If the total egress throughput or the total event egress rate across all Event Hubs in a namespace exceeds the aggregate throughput unit allowances, receivers will be throttled and will receive errors indicating that the egress quota has been exceeded. Ingress and egress quotas are enforced separately, so that no sender can cause event consumption to slow down, nor can a receiver prevent events from being sent into an Event Hub.

### Is there a limit on the number of throughput units that can be selected?
There is a default quota of 20 throughput units per namespace. You can request a larger quota of throughput units by filing a support ticket. Beyond the 20 throughput unit limit, bundles are available in 20 and 100 throughput units. Note that using more than 20 throughput units removes the ability to change the number of throughput units without filing a support ticket.

### Can I use a single AMQP connection to send and receive from multiple Event Hubs?
Yes, as long as all of the Event Hubs are in the same namespace.

### What is the maximum retention period for events?
Event Hubs Standard tier currently supports a maximum retention period of 7 days. Note that Event Hubs are not intended as a permanent data store. Retention periods greater than 24 hours are intended for scenarios in which it is convenient to replay an event stream into the same systems; for example, to train or verify a new machine learning model on existing data. If you need message retention beyond 7 days, enabling [Archive](https://docs.microsoft.com/azure/event-hubs/event-hubs-archive-overview) on your Event Hub will pull the data from your Event Hub to the storage of your choosing. Enabling Archive will incur a charge based on your purchased Throughput Unit.

### Where is Azure Event Hubs available?
Azure Event Hubs is available in all supported Azure regions. For a list, visit the [Azure regions](https://azure.microsoft.com/regions/) page.  

## Best practices

### How many partitions do I need?
Please keep in mind that the partition count on an Event Hub cannot be modified after setup. With that in mind, it is important to think about how many partitions you need before getting started. 

Event Hubs is designed to allow a single partition reader per consumer group. In most use cases, the default setting of four partitions is sufficient. If you are looking to scale your event processing, you may want to consider adding additional partitions. There is no specific throughput limit on a partition, however the aggregate throughput in your namespace is limited by the number of throughput units. As you increase the number of throughput units in your namespace, you may want additional partitions to allow concurrent readers to achieve their own maximum throughput.

However, if you have a model in which your application has an affinity to a particular partition, increasing the number of partitions may not be of any benefit to you. For more information on this please see [availability and consistency](event-hubs-availability-and-consistency.md).

## Pricing

### Where can I find more pricing information?
For complete information about Event Hubs pricing, see the [Event Hubs pricing details](https://azure.microsoft.com/pricing/details/event-hubs/).

### Is there a charge for retaining Event Hubs events for more than 24 hours?
The Event Hubs Standard tier does allow message retention periods longer than 24 hours, for a maximum of 7 days. If the size of the total number of stored events exceeds the storage allowance for the number of selected throughput units (84 GB per throughput unit), the size that exceeds the allowance is charged at the published Azure Blob storage rate. The storage allowance in each throughput unit covers all storage costs for retention periods of 24 hours (the default) even if the throughput unit is used up to the maximum ingress allowance.

### How is the Event Hubs storage size calculated and charged?
The total size of all stored events, including any internal overhead for event headers or on disk storage structures in all Event Hubs, is measured throughout the day. At the end of the day, the peak storage size is calculated. The daily storage allowance is calculated based on the minimum number of throughput units that were selected during the day (each throughput unit provides an allowance of 84 GB). If the total size exceeds the calculated daily storage allowance, the excess storage is billed using Azure Blob storage rates (at the **Locally Redundant Storage** rate).

### How are Event Hubs ingress events calculated?
Each event sent to an Event Hub counts as a billable message. An *ingress event* is defined as a unit of data that is less than or equal to 64 KB. Any event that is less than or equal to 64 KB in size is considered to be one billable event. If the event is greater than 64 KB, the number of billable events is calculated according to the event size, in multiples of 64 KB. For example, an 8 KB event sent to the Event Hub is billed as one event, but a 96 KB message sent to the Event Hub is billed as two events.

Events consumed from an Event Hub, as well as management operations and control calls such as checkpoints, are not counted as billable ingress events, but accrue up to the throughput unit allowance.

### Do brokered connection charges apply to Event Hubs?
Connection charges apply only when the AMQP protocol is used. There are no connection charges for sending events using HTTP, regardless of the number of sending systems or devices. If you plan to use AMQP (for example, to achieve more efficient event streaming or to enable bi-directional communication in IoT command and control scenarios), please refer to the [Event Hubs pricing information](https://azure.microsoft.com/pricing/details/event-hubs/) page for details regarding how many connections are included in each service tier.

### How is Event Hubs Archive billed?
Archive is enabled when any Event Hub in the namespace has the Archive feature enabled. Archive is billed hourly per purchased Throughput Unit. As the Throughput Unit count is increased or decreased, Event Hubs Archive billing will reflect these changes in whole hour increments.
please refer to the [Event Hubs pricing information](https://azure.microsoft.com/pricing/details/event-hubs/) page for details regarding Event Hubs Archive billing.

### Will I be billed for the storage account I select for Event Hubs Archive?
Archive uses a storage account you provide when enabled on an Event Hub. As this is your storage account, any changes for this will be billed to your Azure subscription.

## Quotas

### Are there any quotas associated with Event Hubs?
For a list of all Event Hubs quotas, see [quotas](event-hubs-quotas.md).

## Troubleshooting

### What are some of the exceptions generated by Event Hubs and their suggested actions?
For a list of possible Event Hubs exceptions, see [Exceptions overview](event-hubs-messaging-exceptions.md).

### Diagnostic logs
Event Hubs supports two types of [diagnostics logs](event-hubs-diagnostic-logs.md) - Archive error logs and operational logs - both of which are represented in json and can be turned on through the Azure portal.

### Support and SLA
Technical support for Event Hubs is available through the [community forums](https://social.msdn.microsoft.com/forums/azure/home). Billing and subscription management support is provided at no cost.

To learn more about our SLA, see the [Service Level Agreements](https://azure.microsoft.com/support/legal/sla/) page.

## Next steps
You can learn more about Event Hubs by visiting the following links:

* [Event Hubs overview](event-hubs-what-is-event-hubs.md)
* [Create an Event Hub](event-hubs-create.md)
