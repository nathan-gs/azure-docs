---
title: Data transfer tools for Azure Stack storage
description: Learn about Azure Stack storage data trandfer tools
services: azure-stack
documentationcenter: ''
author: xiaofmao
manager:
editor: ''

ms.assetid:
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 06/22/2017
ms.author: xiaofmao

---
# Data transfer tools for Azure Stack storage

Microsoft Azure Stack provides a set of the storage services for disks, blobs, tables, queues, and account management functionality. You can use a set of Azure Storage tools if you want to move data to or from Azure Stack Storage. This article provides a quick overview of the tools available.

The tool that works best for you depends on your requirements:
* [Microsoft Storage Explorer (Preview)](#microsoft-azure-storage-explorer)

    An easy to use standalone app with a user interface.
* [AzCopy](#azcopy)

    A storage-specific command-line utility that you can download to copy data from one object to another within your storage account, or between storage accounts.

* [Azure PowerShell](#azure-powershell)

    A task-based command-line shell and scripting language designed especially for system administration.

* [Azure CLI](#azure-cli)

    An open-source, cross-platform tool that provides a set of commands for working with the Azure and Azure Stack platforms. 

Due to the Storage services differences between Azure and Azure Stack, there might be some specific requirements for each tool described in the following sections. For a comparison between Azure Stack storage and Azure storage, see [Azure-consistent storage: differences and considerations](azure-stack-acs-differences.md).


## Microsoft Azure Storage Explorer

Microsoft Azure Storage Explorer is a standalone app from Microsoft. It allows you to easily work with both Azure Storage and Azure Stack Storage data on Windows, macOS and Linux. If you want an easy way to manage your Azure Stack Storage data, then consider using Microsoft Azure Storage Explorer.

For more information about configuring Azure Storage Explorer to work with Azure Stack, see [Connect Storage Explorer to an Azure Stack subscription](azure-stack-storage-connect-se.md).

For more information about Microsoft Azure Storage Explorer, see [Get started with Storage Explorer (Preview)](../vs-azure-tools-storage-manage-with-storage-explorer.md)

## AzCopy
AzCopy is a Windows Command-line utility designed to copy data to and from Microsoft Azure Blob and Table storage using simple commands with optimal performance. You can copy data from one object to another within your storage account, or between storage accounts.
 
### Download and install AzCopy 

[Download](https://aka.ms/azcopyforazurestack) the supported version of AzCopy for Azure Stack. You can install and use AzCopy on Azure Stack the same way as Azure. To learn more, see [Transfer data with the AzCopy Command-Line Utility](../storage/storage-use-azcopy.md). 

### Known issues
* Any AzCopy operation on File storage is not available because File Storage is not yet available in Azure Stack.
* Asynchronous data transfer between Azure Storage and Azure Stack is not supported. You can specify the transfer with the `/SyncCopy` option to copy the data.

## Azure PowerShell
Azure PowerShell is a module that provides cmdlets for managing services on both Azure and Azure Stack. It's a task-based command-line shell and scripting language designed especially for system administration.

Azure Stack compatible Azure PowerShell modules are required to work with Azure Stack. For more information, see [Install PowerShell for Azure Stack](azure-stack-powershell-install.md) and [Configure PowerShell for use with Azure Stack](azure-stack-powershell-configure.md) to learn more.

The current compatible Azure PowerShell module version for Azure Stack is 1.2.9. It’s different from the latest version of Azure PowerShell. This difference impacts storage services operation:

* The return value format of `Get-AzureRmStorageAccountKey` in version 1.2.9 has two properties: `Key1` and `Key2`, while the current Azure version returns an array containing all the account keys.

   ```
   # This command gets a specific key for a Storage account, 
   # and works for Azure PowerShell version 1.4, and later versions.
   (Get-AzureRmStorageAccountKey -ResourceGroupName "RG01" `
   -AccountName "MyStorageAccount").Value[0]

   # This command gets a specific key for a Storage account, 
   # and works for Azure PowerShell version 1.3.2, and previous versions.
   (Get-AzureRmStorageAccountKey -ResourceGroupName "RG01" `
   -AccountName "MyStorageAccount").Key1

   ```

   For more information, see [Get-​Azure​Rm​Storage​Account​Key](https://docs.microsoft.com/en-us/powershell/module/azurerm.storage/Get-AzureRmStorageAccountKey?view=azurermps-4.1.0).

## Azure CLI
The Azure CLI is Azure’s command-line experience for managing Azure resources. You can install it on macOS, Linux, and Windows and run it from the command line. 

Azure CLI is optimized for managing and administering Azure resources from the command line, and for building automation scripts that work against the Azure Resource Manager. It provides many of the same functions found in the Azure Stack portal, including rich data access.

Azure Stack requires Azure CLI version 2.0. For more information about installing and configuring Azure CLI with Azure Stack, see [Install and configure Azure Stack CLI](azure-stack-connect-cli.md).


## Next steps
* [Connect Storage Explorer to an Azure Stack subscription](azure-stack-storage-connect-se.md)
* [Get started with Storage Explorer (Preview)](../vs-azure-tools-storage-manage-with-storage-explorer.md)
* [Azure-consistent storage: differences and considerations](azure-stack-acs-differences.md)
* [Introduction to Microsoft Azure Storage](..\storage\storage-introduction.md)

