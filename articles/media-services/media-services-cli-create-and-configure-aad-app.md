---
title: Use CLI 2.0 to create AAD app and configure it to access Azure Media Services API | Microsoft Docs
description: This topic shows how to use CLI 2.0 to create AAD app and configure it to access Azure Media Services API.
services: media-services
documentationcenter: ''
author: Juliako
manager: erikre
editor: ''

ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/17/2017
ms.author: juliako

---

# Use CLI 2.0 to create AAD app and configure it to access Azure Media Services API

This topic shows how to use CLI 2.0 to create Azure Active Directory (AAD) application and service principal to access Azure Media Services resources. 

## Prerequisites

- An Azure account. For details, see [Azure Free Trial](https://azure.microsoft.com/pricing/free-trial/). 
- A Media Services account. For more information, see [Create an Azure Media Services account using the Azure portal](media-services-portal-create-account.md)

## Use the Azure Cloud Shell

1. Sign in to the [Azure portal](https://portal.azure.com/).
2. Launch the Cloud Shell from the top navigation of the portal.

	![Cloud shell](./media/media-services-cli-create-and-configure-aad-app/media-services-cli-create-and-configure-aad-app01.png) 

	For more information, see [Overview of Azure Cloud Shell](../cloud-shell/overview.md)

## Create an AAD app and configure access to the media account with CLI 2.0
 
```azurecli
az login
az ad sp create-for-rbac --name <appName> --password <strong password>
az role assignment create -- assignee < user/app id> --role Contributor --scope <subscription/subscription id>
```

For example:

```azurecli
az role assignment create --assignee a3e068fa-f739-44e5-ba4d-ad57866e25a1 --role Contributor --scope /subscriptions/0b65e280-7917-4874-9fed-1307f2615ea2/resourceGroups/Default-AzureBatch-SouthCentralUS/providers/microsoft.media/mediaservices/sbbash
```

In the example above, the **scope** is the full resource path for the media services account. However, the **scope** could be at any level.

For example:
 
* At **subscription** level.
* At **resource group** level.
* At **resource** level (for example, Media account)

For more information, see [Create an Azure service principal with Azure CLI 2.0](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli)

Also, see [Manage Role-Based Access Control with the Azure command-line interface](../active-directory/role-based-access-control-manage-access-azure-cli.md). 

## Next steps

Get started with [uploading files to your account](media-services-portal-upload-files.md).