---
title: Azure resource policies | Microsoft Docs
description: Describes how to use Azure Resource Manager policies to ensure consistent resource properties are set during deployment. Policies can be applied at the subscription or resource groups.
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn

ms.assetid: abde0f73-c0fe-4e6d-a1ee-32a6fce52a2d
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/15/2017
ms.author: tomfitz

---
# Resource policy overview
Resource policies enable you to establish conventions for resources in your organization. By defining conventions, you can control costs and more easily manage your resources. For example, you can specify that only certain types of virtual machines are allowed, or you can require that all resources have a particular tag. Policies are inherited by all child resources. So, if a policy is applied to a resource group, it is applicable to all the resources in that resource group.

There are two concepts to understand about policies:

* policy definition - you describe when the policy is enforced and what action to take
* policy assignment - you apply the policy definition to a scope (subscription or resource group)

This topic focuses on policy definition. For information about policy assignment, see [Use Azure portal to assign and manage resource policies](resource-manager-policy-portal.md) or [Assign and manage policies through script](resource-manager-policy-create-assign.md).

Azure provides some built-in policy definitions that may reduce the number of policies you have to define. If a built-in policy definition works for your scenario, use that definition when assigning to a scope.

Policies are evaluated when creating and updating resources (PUT and PATCH operations).

> [!NOTE]
> Currently, policy does not evaluate resource types that do not support tags, kind, and location, such as the Microsoft.Resources/deployments resource type. This support will be added at a future time. To avoid backward compatibility issues, you should explicitly specify type when authoring policies. For example, a tag policy that does not specify types is applied for all types. In that case, a template deployment may fail if there is a nested resource that doesn't support tags, and the deployment resource type has been added to policy evaluation. 
> 
> 

## How is it different from RBAC?
There are a few key differences between policy and role-based access control (RBAC). RBAC focuses on **user** actions at different scopes. For example, you are added to the contributor role for a resource group at the desired scope, so you can make changes to that resource group. Policy focuses on **resource** properties during deployment. For example, through policies, you can control the types of resources that can be provisioned or restrict the locations in which the resources can be provisioned. Unlike RBAC, policy is a default allow and explicit deny system. 

To use policies, you must be authenticated through RBAC. Specifically, your account needs the:

* `Microsoft.Authorization/policydefinitions/write` permission to define a policy
* `Microsoft.Authorization/policyassignments/write` permission to assign a policy 

These permissions are not included in the **Contributor** role.

## Policy definition structure
You use JSON to create a policy definition. The policy definition contains elements for:

* parameters
* display name
* description
* policy rule
  * logical evaluation
  * effect

The following example shows a policy that limits where resources are deployed:

```json
{
  "properties": {
    "parameters": {
      "allowedLocations": {
        "type": "array",
        "metadata": {
          "description": "The list of locations that can be specified when deploying resources",
          "strongType": "location",
          "displayName": "Allowed locations"
        }
      }
    },
    "displayName": "Allowed locations",
    "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
    "policyRule": {
      "if": {
        "not": {
          "field": "location",
          "in": "[parameters('allowedLocations')]"
        }
      },
      "then": {
        "effect": "deny"
      }
    }
  }
}
```

## Parameters
Using parameters helps simplify your policy management by reducing the number of policy definitions. You define a policy for a resource property (such as limiting the locations where resources can be deployed), and include parameters in the definition. Then, you reuse that policy definition for different scenarios by passing in different values (such as specifying one set of locations for a subscription) when assigning the policy.

You declare parameters when you create policy definitions.

```json
"parameters": {
  "allowedLocations": {
    "type": "array",
    "metadata": {
      "description": "The list of allowed locations for resources.",
      "displayName": "Allowed locations"
    }
  }
}
```

The type of a parameter can be either string or array. The metadata property is used for tools like Azure portal to display user-friendly information. 

In the policy rule, you reference parameters with the following syntax: 

```json
{ 
    "field": "location",
    "in": "[parameters('allowedLocations')]"
}
```

## Display name and description

You use the **displayName** and **description** to identify the policy definition, and provide context for when it is used.

## Policy rule

The policy rule consists of **If** and **Then** blocks. In the **If** block, you define one or more conditions that specify when the policy is enforced. You can apply logical operators to these conditions to precisely define the scenario for a policy. In the **Then** block, you define the effect that happens when the **If** conditions are fulfilled.

```json
{
  "if": {
    <condition> | <logical operator>
  },
  "then": {
    "effect": "deny | audit | append"
  }
}
```

### Logical operators
The supported logical operators are:

* `"not": {condition  or operator}`
* `"allOf": [{condition or operator},{condition or operator}]`
* `"anyOf": [{condition or operator},{condition or operator}]`

The **not** syntax inverts the result of the condition. The **allOf** syntax (similar to the logical **And** operation) requires all conditions to be true. The **anyOf** syntax (similar to the logical **Or** operation) requires one or more conditions to be true.

You can nest logical operators. The following example shows a **not** operation that is nested within an **allOf** operation. 

```json
"if": {
  "allOf": [
    {
      "not": {
        "field": "tags",
        "containsKey": "application"
      }
    },
    {
      "field": "type",
      "equals": "Microsoft.Storage/storageAccounts"
    }
  ]
},
```

### Conditions
The condition evaluates whether a **field** meets certain criteria. The supported conditions are:

* `"equals": "value"`
* `"like": "value"`
* `"match": "value"`
* `"contains": "value"`
* `"in": ["value1","value2"]`
* `"containsKey": "keyName"`
* `"exists": "bool"`

When using the **like** condition, you can provide a wildcard (*) in the value.

When using the **match** condition, provide `#` to represent a digit, `?` for a letter, and any other character to represent that actual character. For examples, see [Set naming convention](#set-naming-convention).

### Fields
Conditions are formed by using fields. A field represents properties in the resource request payload that is used to describe the state of the resource.  

The following fields are supported:

* `name`
* `kind`
* `type`
* `location`
* `tags`
* `tags.*` 
* property aliases - for a list, see [Aliases](#aliases).

### Effect
Policy supports three types of effect - `deny`, `audit`, and `append`. 

* **Deny** generates an event in the audit log and fails the request
* **Audit** generates a warning event in audit log but does not fail the request
* **Append** adds the defined set of fields to the request 

For **append**, you must provide the following details:

```json
"effect": "append",
"details": [
  {
    "field": "field name",
    "value": "value of the field"
  }
]
```

The value can be either a string or a JSON format object. 

## Aliases

You use property aliases to access specific properties for a resource type. 

**Microsoft.Cache/Redis**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Cache/Redis/enableNonSslPort | Set whether the non-ssl Redis server port (6379) is enabled. |
| Microsoft.Cache/Redis/shardCount | Set the number of shards to be created on a Premium Cluster Cache.  |
| Microsoft.Cache/Redis/sku.capacity | Set the size of the Redis cache to deploy.  |
| Microsoft.Cache/Redis/sku.family | Set the SKU family to use. |
| Microsoft.Cache/Redis/sku.name | Set the type of Redis Cache to deploy. |

**Microsoft.Cdn/profiles**

| Alias | Description |
| ----- | ----------- |
| Microsoft.CDN/profiles/sku.name | Set the name of the pricing tier. |

**Microsoft.Compute/disks**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Compute/imageOffer | Set the offer of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imagePublisher | Set the publisher of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageSku | Set the SKU of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageVersion | Set the version of the platform image or marketplace image used to create the virtual machine. |


**Microsoft.Compute/virtualMachines**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Compute/imageOffer | Set the offer of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imagePublisher | Set the publisher of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageSku | Set the SKU of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageVersion | Set the version of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/licenseType | Set that the image or disk is licensed on-premises. This value is only used for images that contain the Windows Server operating system.  |
| Microsoft.Compute/virtualMachines/imageOffer | Set the offer of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/virtualMachines/imagePublisher | Set the publisher of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/virtualMachines/imageSku | Set the SKU of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/virtualMachines/imageVersion | Set the version of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/virtualMachines/osDisk.Uri | Set the vhd URI. |
| Microsoft.Compute/virtualMachines/sku.name | Set the size of the virtual machine. |

**Microsoft.Compute/virtualMachines/extensions**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Compute/virtualMachines/extensions/publisher | Set the name of the extension’s publisher. |
| Microsoft.Compute/virtualMachines/extensions/type | Set the type of extension. |
| Microsoft.Compute/virtualMachines/extensions/typeHandlerVersion | Set the version of the extension. |

**Microsoft.Compute/virtualMachineScaleSets**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Compute/imageOffer | Set the offer of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imagePublisher | Set the publisher of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageSku | Set the SKU of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageVersion | Set the version of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/licenseType | Set that the image or disk is licensed on-premises. This value is only used for images that contain the Windows Server operating system. |
| Microsoft.Compute/VirtualMachineScaleSets/computerNamePrefix | Set the computer name prefix for all  the virtual machines in the scale set. |
| Microsoft.Compute/VirtualMachineScaleSets/osdisk.imageUrl | Set the blob URI for user image. |
| Microsoft.Compute/VirtualMachineScaleSets/osdisk.vhdContainers | Set the container URLs that are used to store operating system disks for the scale set. |
| Microsoft.Compute/VirtualMachineScaleSets/sku.name | Set the size of virtual machines in a scale set. |
| Microsoft.Compute/VirtualMachineScaleSets/sku.tier | Set the tier of virtual machines in a scale set. |
  
**Microsoft.Network/applicationGateways**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Network/applicationGateways/sku.name | Set the size of the gateway. |

**Microsoft.Network/virtualNetworkGateways**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Network/virtualNetworkGateways/gatewayType | Set the type of this virtual network gateway. |
| Microsoft.Network/virtualNetworkGateways/sku.name | Set the gateway SKU name. |

**Microsoft.Sql/servers**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Sql/servers/version | Set the version of the server. |

**Microsoft.Sql/databases**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Sql/servers/databases/edition | Set the edition of the database. |
| Microsoft.Sql/servers/databases/elasticPoolName | Set the name of the elastic pool the database is in. |
| Microsoft.Sql/servers/databases/requestedServiceObjectiveId | Set the configured service level objective ID of the database. |
| Microsoft.Sql/servers/databases/requestedServiceObjectiveName | Set the name of the configured service level objective of the database.  |

**Microsoft.Sql/elasticpools**

| Alias | Description |
| ----- | ----------- |
| servers/elasticpools | Microsoft.Sql/servers/elasticPools/dtu | Set the total shared DTU for the database elastic pool. |
| servers/elasticpools | Microsoft.Sql/servers/elasticPools/edition | Set the edition of the elastic pool. |

**Microsoft.Storage/storageAccounts**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Storage/storageAccounts/accessTier | Set the access tier used for billing. |
| Microsoft.Storage/storageAccounts/accountType | Set the SKU name. |
| Microsoft.Storage/storageAccounts/enableBlobEncryption | Set whether the service encrypts the data as it is stored in the blob storage service. |
| Microsoft.Storage/storageAccounts/enableFileEncryption | Set whether the service encrypts the data as it is stored in the file storage service. |
| Microsoft.Storage/storageAccounts/sku.name | Set the SKU name. |
| Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly | Set to allow only https traffic to storage service. |


## Policy examples

The following topics contain policy examples:

* For examples of tag polices, see [Apply resource policies for tags](resource-manager-policy-tags.md).
* For examples of storage policies, see [Apply resource policies to storage accounts](resource-manager-policy-storage.md).
* For examples of virtual machine policies, see [Apply resource policies to Linux VMs](../virtual-machines/linux/policy.md?toc=%2fazure%2fazure-resource-manager%2ftoc.json) and [Apply resource policies to Windows VMs](../virtual-machines/windows/policy.md?toc=%2fazure%2fazure-resource-manager%2ftoc.json)

### Allowed resource locations
To specify which locations are allowed, see the example in the [Policy definition structure](#policy-definition-structure) section. To assign this policy definition, use the built-in policy with the resource ID `/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c`.

### Not allowed resource locations
To specify which locations are not allowed, use the following policy definition:

```json
{
  "properties": {
    "parameters": {
      "notAllowedLocations": {
        "type": "array",
        "metadata": {
          "description": "The list of locations that are not allowed when deploying resources",
          "strongType": "location",
          "displayName": "Not allowed locations"
        }
      }
    },
    "displayName": "Not allowed locations",
    "description": "This policy enables you to block locations that your organization can specify when deploying resources.",
    "policyRule": {
      "if": {
        "field": "location",
        "in": "[parameters('notAllowedLocations')]"
      },
      "then": {
        "effect": "deny"
      }
    }
  }
}
```

### Allowed resource types
The following example shows a policy that permits deployments for only the Microsoft.Resources, Microsoft.Compute, Microsoft.Storage, Microsoft.Network resource types. All others are denied:

```json
{
  "if": {
    "not": {
      "anyOf": [
        {
          "field": "type",
          "like": "Microsoft.Resources/*"
        },
        {
          "field": "type",
          "like": "Microsoft.Compute/*"
        },
        {
          "field": "type",
          "like": "Microsoft.Storage/*"
        },
        {
          "field": "type",
          "like": "Microsoft.Network/*"
        }
      ]
    }
  },
  "then": {
    "effect": "deny"
  }
}
```

### Set naming convention
The following example shows the use of wildcard, which is supported by the **like** condition. The condition states that if the name does match the mentioned pattern (namePrefix\*nameSuffix) then deny the request:

```json
{
  "if": {
    "not": {
      "field": "name",
      "like": "namePrefix*nameSuffix"
    }
  },
  "then": {
    "effect": "deny"
  }
}
```

To specify that resource names match a pattern, use the match condition. The following example requires names to start with `contoso` and contain six additional letters:

```json
{
  "if": {
    "not": {
      "field": "name",
      "match": "contoso??????"
    }
  },
  "then": {
    "effect": "deny"
  }
}
```

To require a date pattern of two digits, dash, three letters, dash, and four digits, use:

```json
{
  "if": {
    "field": "tags.date",
    "match": "##-???-####"
  },
  "then": {
    "effect": "deny"
  }
}
```

## Next steps
* After defining a policy rule, assign it to a scope. To assign policies through the portal, see [Use Azure portal to assign and manage resource policies](resource-manager-policy-portal.md). To assign policies through REST API, PowerShell or Azure CLI, see [Assign and manage policies through script](resource-manager-policy-create-assign.md).
* For guidance on how enterprises can use Resource Manager to effectively manage subscriptions, see [Azure enterprise scaffold - prescriptive subscription governance](resource-manager-subscription-governance.md).
* The policy schema is published at [http://schema.management.azure.com/schemas/2015-10-01-preview/policyDefinition.json](http://schema.management.azure.com/schemas/2015-10-01-preview/policyDefinition.json). 

