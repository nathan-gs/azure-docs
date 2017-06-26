---
title: Azure Stack POC deployment quickstart | Microsoft Docs
description: Learn how to deploy the Azure Stack Proof of Concept
services: azure-stack
documentationcenter: ''
author: ErikjeMS
manager: byronr
editor: ''

ms.assetid: 
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 05/10/2017
ms.author: erikje
ms.custom: mvc

---
# Azure Stack POC deployment quickstart

The [Azure Stack Proof of Concept (POC)](azure-stack-poc.md) is a testing and development environment that you can deploy to evaluate and demonstrate Azure Stack features and services. To get it up and running, you’ll need to prepare the environment hardware and run some scripts (this will take several hours). After that, you can sign in to the admin and tenant portals to manage Azure Stack and test offers. 

1. [**Plan your hardware, software, and network**](azure-stack-deploy.md). The computer that hosts the Azure Stack POC (the POC host) must meet hardware, software, and network requirements. You must also choose between using Azure Active Directory or Active Directory Federation Services. Be sure to comply with these prerequisites before starting your deployment so that the installation process runs smoothly. 

2. [**Download and extract the deployment package**](azure-stack-run-powershell-script.md#download-and-extract-microsoft-azure-stack-poc). You can download the deployment package to the POC host or to a another computer. The extracted deployment files take up 60 GB of free disk space, so using another computer can help reduce the hardware requirements for the POC host.

3. [**Prepare the POC host**](azure-stack-run-powershell-script.md#prepare-the-poc-host) by running the PrepareBootFromVHD.ps1 script. After this step, the POC host will boot to the Cloudbuilder.vhdx (a virtual hard drive that includes a bootable operating system and the Azure Stack install files).

4. [**To start the deployment, run the PowerShell deployment script**](azure-stack-run-powershell-script.md#run-the-powershell-deployment-script) on the POC host.

5. If your Azure Stack deployment uses Azure Active Directory, you must:

    - [Activate the administrator and tenant portals](azure-stack-run-powershell-script.md#activate-the-administrator-and-tenant-portals).
    - [Register Azure Stack with Azure](azure-stack-register.md) so that you can [download Azure marketplace items](azure-stack-download-azure-marketplace-item.md) to Azure Stack.

After completing these steps, you’ll have an Azure Stack POC environment with both administrator and tenant portals. Next, you can [connect and sign in](azure-stack-connect-azure-stack.md) to the portal. You can then start deploying resource providers, creating [offers](azure-stack-key-features.md#regions-services-plans-offers-and-subscriptions), and populating the Azure Stack [marketplace](azure-stack-marketplace.md).
