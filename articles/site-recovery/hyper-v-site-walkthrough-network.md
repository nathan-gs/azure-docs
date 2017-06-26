---
title: Plan networking for Hyper-V (without System Center VMM) to Azure replication with Azure Site Recovery | Microsoft Docs
description: This article discusses network planning required when replicating Hyper-V VMs (without VMM) to Azure
services: site-recovery
documentationcenter: ''
author: rayne-wiselman
manager: carmonm
editor: ''

ms.assetid: 5ca12254-975d-48e8-a84d-422825dac9b2
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 06/21/2017
ms.author: raynew

---

# Step 4: Plan networking for Hyper-V to Azure replication

This article summarizes network planning considerations when replicating on-premises Hyper-V VMs (without System Center VMM) to Azure using the [Azure Site Recovery](site-recovery-overview.md) service.

Post any comments at the bottom of this article, or ask questions in the [Azure Recovery Services Forum](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr).


## Connecting to replica VMs after failover

When planning your replication and failover strategy, one of the key questions is how to connect to the Azure VM after failover. There are a couple of choices when designing your network strategy for replica Azure VMs:

- **Different IP address**: You can select to use a different IP address range for the replicated Azure VM network. In this scenario the VM gets a new IP address after failover, and a DNS update is required. [Learn more](site-recovery-test-failover-vmm-to-vmm.md#prepare-the-infrastructure-for-test-failover)
- **Same IP address**: You might want to use the same IP address range in Azure after failover, as you have in your primary on-premises site. In a normal scenario, you would have to update routes with the new location of the IP addresses. However, if you have a stretched VLAN deployed between the primary site and Azure, retaining the IP addresses for the virtual machines becomes a valid option. Keeping the same IP addresses simplifies the recovery by reducing network related issues after failover.


## Retain IP addresses

From a disaster recovery perspective, using fixed IP addresses seems to be the simplest method, but there are a number of potential challenges. Site Recovery provides the capability to retain the IP addresses when failing over to Azure, with subnet failover.


### Subnet failover

In this scenario, a specific subnet is present at Site 1 or Site 2, but never at both sites simultaneously. In order to maintain the IP address space in the event of a failover, you programmatically arrange for the router infrastructure to move the subnets from one site to another. During failover, the subnets move with the associated protected VMs. The main drawback to this approach is in the event of a failure you have to move the whole subnet, which might affect failover granularity considerations.


### Failover example

Let's look at an example for failover to Azure.

- A ficticious company, Woodgrove Bank, has an on-premises infrastructure hosting their business apps. Their mobile applications are hosted on Azure.
- Connectivity between Woodgrove Bank VMs in Azure and on-premises servers is provided by a site-to-site (VPN) connection between the on-premises edge network and the Azure virtual network.
- This VPN means that the company's virtual network in Azure appears as an extension of their on-premises network.
- Woodgrove wants to use Site Recovery to replicate on-premises workloads to Azure.
 - Woodgrove has to deal with applications and configurations which depend on hard-coded IP addresses, and thus need to retain IP addresses for their applications after failover to Azure.
 - Woodgrove has assigned IP addresses from range 172.16.1.0/24, 172.16.2.0/24 to its resources running in Azure.


For Woodgrove to be able to replicate its VMS to Azure while retaining the IP addresses, here's what the company needs to do:

1. Create an Azure virtual network. It should be an extension of the on-premises network, so that applications can fail over
seamlessly.
2. Azure allows you to add site-to-site VPN connectivity, in addition to point-to-site connectivity to the virtual networks created in Azure.
3. When setting up the site-to-site connection, in the Azure network, you can route traffic to the on-premises location (local-network) only if the IP address range is different from the on-premises IP address range.
    - This is because Azure doesn’t support stretched subnets. So if you have subnet 192.168.1.0/24 on-premises, you can’t add a local-network 192.168.1.0/24 in the Azure network.
    - This is expected because Azure doesn’t know that there are no active VMs in the subnet, and that the subnet is being created for disaster recovery only.
    - To be able to correctly route network traffic out of an Azure network the subnets in the network and the local-network mustn't conflict.

![Before subnet failover](./media/hyper-v-site-walkthrough-network/network-design7.png)

### Before failover

1. Create an additional network (for example Recovery Network). This is the network in which failed over VMs are created.
2. To ensure that the IP address for a VM is retained after a failover, in the VM properties > **Configure**, specify the same IP address that the VM has on-premises, and click **Save**.
3. When the VM is failed over, Azure Site Recovery will assign the provided IP address to it.

    ![Network properties](./media/hyper-v-site-walkthrough-network/network-design8.png)

4. After failover is trigger is triggered, and the VMs are created in Azure with the required IP address, you can connect to the network using a [Vnet to Vnet vonnection](../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md). This action can be scripted.
5. Routes need to be appropriately modified, to reflect that 192.168.1.0/24 has now moved to Azure.

    ![After subnet failover](./media/hyper-v-site-walkthrough-network/network-design9.png)

### After failover

If you don't have an Azure network as illustrated above, you can create a site-to-site VPN connection between your primary site and Azure, after failvoer.

## Change IP addresses

This [blog post](http://azure.microsoft.com/blog/2014/09/04/networking-infrastructure-setup-for-microsoft-azure-as-a-disaster-recovery-site/) explains how to set up the Azure networking infrastructure when you don't need to retain IP addresses after failover. It starts with an application description, looks at how to set up networking on-premises and in Azure, and concludes with information about running failovers.  

## Next steps

Go to [Step 5: Prepare Azure](hyper-v-site-walkthrough-prepare-azure.md)
