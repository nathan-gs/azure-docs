---
title: Use PowerShell to create AAD app and configure it to access Azure Media Services API | Microsoft Docs
description: This topic shows how to use PowerShell to create AAD app and configure it to access Azure Media Services API.
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

# Use PowerShell to create AAD app and configure it to access Azure Media Services API

This topic shows how to use a PowerShell script to create Azure Active Directory (AAD) application and service principal to access Azure Media Services resources.  

## Prerequisites

- An Azure account. For details, see [Azure Free Trial](https://azure.microsoft.com/pricing/free-trial/). 
- A Media Services account. For more information, see [Create an Azure Media Services account using the Azure portal](media-services-portal-create-account.md)
- Azure PowerShell version 0.8.8 or later. For more information, see [How to use Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview).
- Azure Resource Manager cmdlets.  

## Create an AAD app with PowerShell  

```powershell
Login-AzureRmAccount
Import-Module AzureRM.Resources
Set-AzureRmContext -SubscriptionId $SubscriptionId
$ServicePrincipal = New-AzureRMADServicePrincipal -DisplayName $ApplicationDisplayName -Password $Password

Get-AzureRmADServicePrincipal -ObjectId $ServicePrincipal.Id 
$NewRole = $null
$Scope = "/subscriptions/your subscription id/resourceGroups/userresourcegroup/providers/microsoft.media/mediaservices/your media account"

$Retries = 0;While ($NewRole -eq $null -and $Retries -le 6)
{
	# Sleep here for a few seconds to allow the service principal application to become active (should only take a couple of seconds normally)
    Sleep 15
    New-AzureRMRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $ServicePrincipal.ApplicationId -Scope $Scope | Write-Verbose -ErrorAction SilentlyContinue
    $NewRole = Get-AzureRMRoleAssignment -ServicePrincipalName $ServicePrincipal.ApplicationId -ErrorAction SilentlyContinue
    $Retries++;
}
```

For more information, see the following articles:

- [Use Azure PowerShell to create a service principal to access resources](../azure-resource-manager/resource-group-authenticate-service-principal.md).
- [Manage Role-Based Access Control with Azure PowerShell](../active-directory/role-based-access-control-manage-access-powershell.md).
- [How to manually configure daemon apps with certificates](https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential/blob/master/Manual-Configuration-Steps.md#add-the-certificate-as-a-key-for-the-todolistdaemonwithcert-application-in-azure-ad)

## Next steps

Get started with [uploading files to your account](media-services-portal-upload-files.md).