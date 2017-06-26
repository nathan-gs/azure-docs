---
title: Different ways to create a Linux VM with Azure CLI 1.0 | Microsoft Docs
description: Learn the different ways to create a Linux virtual machine on Azure, including links to tools and tutorials for each method.
services: virtual-machines-linux
documentationcenter: ''
author: iainfoulds
manager: timlt
editor: ''
tags: azure-resource-manager

ms.assetid: f38f8a44-6c88-4490-a84a-46388212d24c
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: 
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 05/11/2017
ms.author: iainfou

---
# Different ways to create a Linux virtual machine in Azure
You have the flexibility in Azure to create a Linux virtual machine (VM) using tools and workflows comfortable to you. This article summarizes these differences and examples for creating your Linux VMs.

## Azure CLI
You can create VMs in Azure using one of the following CLI versions:

- Azure CLI 1.0 – our CLI for the classic and resource management deployment models (this article)
- [Azure CLI 2.0](../windows/creation-choices.md) - our next generation CLI for the resource management deployment model

The Azure CLI 1.0 is available across platforms via an npm package, distro-provided packages, or Docker container. You can read more about [how to install and configure the Azure CLI](../../cli-install-nodejs.md). The following tutorials provide examples on using the Azure CLI 1.0. Read each article for more details on the CLI quick-start commands shown:

* [Create a Linux VM from the Azure CLI for dev and test](quick-create-cli-nodejs.md)
  
  * The following example creates a CoreOS VM using a public key named *azure_id_rsa.pub*:
    
    ```azurecli
    azure vm quick-create -ssh-publickey-file ~/.ssh/azure_id_rsa.pub \
      --image-urn CoreOS
    ```
* [Create a secured Linux VM using an Azure template](create-ssh-secured-vm-from-template.md)
  
  * The following example creates a VM using a template stored on GitHub:
    
    ```azurecli
    azure group create --name myResourceGroup --location eastus 
      --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json
    ```
* [Create a complete Linux environment using the Azure CLI](create-cli-complete-nodejs.md)
  
  * Includes creating a load balancer and multiple VMs in an availability set.
* [Add a disk to a Linux VM](add-disk.md)
  
  * The following example adds a *5* Gb disk to an existing VM named *myVM*:
    
    ```azurecli
    azure vm disk attach-new \
        --resource-group myResourceGroup \
        --vm-name myVM \
        --size-in-GB 5
    ```

## Azure portal
The [Azure portal](https://portal.azure.com) allows you to quickly create a VM since there is nothing to install on your system. Use the Azure portal to create the VM:

* [Create a Linux VM using the Azure portal](quick-create-portal.md) 

## Operating system and image choices
When creating a VM, you choose an image based on the operating system you want to run. Azure and its partners offer many images, some of which include applications and tools pre-installed. Or, upload one of your own images (see [the following section](#use-your-own-image)).

### Azure images
Use the `azure vm image` CLI commands to see what's available by publisher, distro release, and builds.

List available publishers as follows:

```azurecli
azure vm image list-publishers --location eastus
```

List available products (offers) for a given publisher as follows:

```azurecli
azure vm image list-offers --location eastus --publisher Canonical
```

List available SKUs (distro releases) of a given offer as follows:

```azurecli
azure vm image list-skus --location eastus --publisher Canonical --offer UbuntuServer
```

List all available images for a given release follows:

```azurecli
azure vm image list --location eastus --publisher Canonical --offer UbuntuServer --sku 16.04.0-LTS
```

For more examples on browsing and using available images, see [Navigate and select Azure virtual machine images with the Azure CLI](cli-ps-findimage.md#use-azure-cli-10).

The `azure vm quick-create` and `azure vm create` commands have aliases you can use to quickly access the more common distros and their latest releases. Using aliases is often quicker than specifying the publisher, offer, SKU, and version each time you create a VM:

| Alias | Publisher | Offer | SKU | Version |
|:--- |:--- |:--- |:--- |:--- |
| CentOS |OpenLogic |Centos |7.2 |latest |
| CoreOS |CoreOS |CoreOS |Stable |latest |
| Debian |credativ |Debian |8 |latest |
| openSUSE |SUSE |openSUSE |13.2 |latest |
| RHEL |Redhat |RHEL |7.2 |latest |
| SLES |SLES |SLES |12-SP1 |latest |
| UbuntuLTS |Canonical |UbuntuServer |14.04.4-LTS |latest |

### Use your own image
If you require specific customizations, you can use an image based on an existing Azure VM by *capturing* that VM. You can also upload an image created on-premises. For more information on supported distros and how to use your own images, see the following articles:

* [Azure endorsed distributions](endorsed-distros.md)
* [Information for non-endorsed distributions](create-upload-generic.md)
* [Upload and create a Linux VM from custom disk image](upload-vhd.md)
* [How to capture a Linux virtual machine as a Resource Manager template](capture-image.md).
  
  * Quick-start example commands to capture an existing VM:
    
    ```azurecli
    azure vm deallocate --resource-group myResourceGroup --vm-name myVM
    azure vm generalize --resource-group myResourceGroup --vm-name myVM
    azure vm capture --resource-group myResourceGroup --vm-name myVM --vhd-name-prefix myCapturedVM
    ```

## Next steps
* Create a Linux VM from the [portal](quick-create-portal.md), with the [CLI](quick-create-cli.md), or using an [Azure Resource Manager template](../windows/cli-deploy-templates.md).
* After creating a Linux VM, [add a data disk](add-disk.md).
* Quick steps to [reset a password or SSH keys and manage users](using-vmaccess-extension.md)

