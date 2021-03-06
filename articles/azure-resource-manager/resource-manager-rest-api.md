---
title: Resource Manager REST API| Microsoft Docs
description: "概述 Resource Manager REST API 身份验证和用例"
services: azure-resource-manager
documentationcenter: na
author: navalev
manager: timlt
editor: 
ms.assetid: e8d7a1d2-1e82-4212-8288-8697341408c5
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/13/2017
ms.author: navale;tomfitz;
ms.openlocfilehash: 2f7ba23775545637de865f9ef63680ae22c62164
ms.sourcegitcommit: fbba5027fa76674b64294f47baef85b669de04b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/26/2018
---
# <a name="resource-manager-rest-apis"></a>Resource Manager REST API
> [!div class="op_single_selector"]
> * [Azure PowerShell](powershell-azure-resource-manager.md)
> * [Azure CLI](xplat-cli-azure-resource-manager.md)
> * [Portal](resource-group-portal.md) 
> * [REST API](resource-manager-rest-api.md)
> 
> 

在每次调用 Azure 资源管理器、每次部署模板以及每次配置存储帐户时，都会对 Azure 资源管理器的 RESTful API 进行一次或多次调用。 本主题专门介绍这些 API 以及如何在完全不使用任何 SDK 的情况下调用它们。 如果想要完全控制对 Azure 的请求，或者偏好语言的 SDK 无法使用或不支持所需的操作，则此方法很有用。

本文不逐一介绍 Azure 中公开的每个 API，而以某些操作为例，说明如何连接到这些操作。 了解基础知识后，可阅读 [Azure 资源管理器 REST API 参考](https://docs.microsoft.com/rest/api/resources/)，查找有关如何使用其余 API 的详细信息。

## <a name="authentication"></a>身份验证
Azure Active Directory (AD) 负责处理 Resource Manager 的身份验证。 若要连接到任何 API，首先需要使用 Azure AD 进行身份验证，以接收可传递给每个请求的身份验证令牌。 由于我们要介绍对 REST API 的单纯直接调用，因此我们假设你不想通过提示输入用户名和密码进行身份验证。 我们还假定你未使用双因素身份验证机制。 因此，我们将创建所谓的 Azure AD 应用程序和一个用于登录的服务主体。 但请记住，Azure AD 支持多个身份验证过程，而这些过程全都可用于检索后续 API 请求所需的身份验证令牌。
有关分步说明，请参阅[创建 Azure AD 应用程序和服务主体](resource-group-create-service-principal-portal.md)。

### <a name="generating-an-access-token"></a>生成访问令牌
通过向外调用位于 login.microsoftonline.com 的 Azure AD 来对 Azure AD 进行身份验证。若要进行身份验证，需要提供以下信息：

* Azure AD 租户 ID（用于登录的 Azure AD 名称，通常与公司同名，但不一定总是如此）
* 应用程序 ID（在创建 Azure AD 应用程序期间获取）
* 密码（在创建 Azure AD 应用程序时选择）

在下面的 HTTP 请求中，确保将“Azure AD Tenant ID”、“Application ID”和“Password”替换为正确的值。

**常规 HTTP 请求：**

```HTTP
POST /<Azure AD Tenant ID>/oauth2/token?api-version=1.0 HTTP/1.1 HTTP/1.1
Host: login.microsoftonline.com
Cache-Control: no-cache
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&resource=https%3A%2F%2Fmanagement.core.windows.net%2F&client_id=<Application ID>&client_secret=<Password>
```

... 将（如果身份验证成功）导致类似于下面的响应：

```json
{
  "token_type": "Bearer",
  "expires_in": "3600",
  "expires_on": "1448199959",
  "not_before": "1448196059",
  "resource": "https://management.core.windows.net/",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhb...86U3JI_0InPUk_lZqWvKiEWsayA"
}
```
（为了方便阅读，上述响应中的 access_token 已缩短）

**使用 Bash 生成访问令牌：**

```console
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&resource=https://management.core.windows.net/&client_id=<application id>&client_secret=<password you selected for authentication>" https://login.microsoftonline.com/<Azure AD Tenant ID>/oauth2/token?api-version=1.0
```

**使用 PowerShell 生成访问令牌：**

```powershell
Invoke-RestMethod -Uri https://login.microsoftonline.com/<Azure AD Tenant ID>/oauth2/token?api-version=1.0 -Method Post
 -Body @{"grant_type" = "client_credentials"; "resource" = "https://management.core.windows.net/"; "client_id" = "<application id>"; "client_secret" = "<password you selected for authentication>" }
```

响应包含访问令牌和令牌有效期限的相关信息，以及可以将该令牌用于哪项资源的相关信息。
对于对 Resource Manager API 的所有请求，必须传入在前面的 HTTP 调用中收到的访问令牌。 可以使用值“Bearer YOUR_ACCESS_TOKEN”传递它作为名为“Authorization”的标头值。 请注意“Bearer”与访问令牌之间有空格。

从上面的 HTTP 结果可以看到，令牌在特定时间段内保持有效状态，应在这段期间缓存并重复使用该同一令牌。 即使可以在每次调用 API 时对 Azure AD 进行身份验证，但这样做的效率很低。

## <a name="calling-resource-manager-rest-apis"></a>调用 Resource Manager REST API
本主题仅使用几个 API 来说明 REST 操作的基本用法。 有关所有操作的信息，请参阅 [Azure 资源管理器 REST API](https://docs.microsoft.com/rest/api/resources/)。

### <a name="list-all-subscriptions"></a>列出所有订阅
可以执行的最简单操作之一是列出可以访问的可用订阅。 在以下请求中，可以看到如何传入访问令牌作为标头：

（将 YOUR_ACCESS_TOKEN 替换为实际访问令牌。）

```HTTP
GET /subscriptions?api-version=2015-01-01 HTTP/1.1
Host: management.azure.com
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

...因此，将收到此服务主体可以访问的订阅列表

（为了方便阅读，订阅 ID 已缩短）

```json
{
  "value": [
    {
      "id": "/subscriptions/3a8555...555995",
      "subscriptionId": "3a8555...555995",
      "displayName": "Azure Subscription",
      "state": "Enabled",
      "subscriptionPolicies": {
        "locationPlacementId": "Internal_2014-09-01",
        "quotaId": "Internal_2014-09-01"
      }
    }
  ]
}
```

### <a name="list-all-resource-groups-in-a-specific-subscription"></a>列出特定订阅中的所有资源组
适用于 Resource Manager API 的所有资源都放在资源组中。 可以使用以下 HTTP GET 请求在 Resource Manager 中查询订阅中的现有资源组。 请注意此次如何将订阅 ID 作为 URL 的一部分传入。

（将 YOUR_ACCESS_TOKEN 和 SUBSCRIPTION_ID 替换为实际访问令牌和订阅 ID）

```HTTP
GET /subscriptions/SUBSCRIPTION_ID/resourcegroups?api-version=2015-01-01 HTTP/1.1
Host: management.azure.com
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

收到的响应取决于是否已定义任何资源组，如果已定义，有多少个。

（为了方便阅读，订阅 ID 已缩短）

```json
{
    "value": [
        {
            "id": "/subscriptions/3a8555...555995/resourceGroups/myfirstresourcegroup",
            "name": "myfirstresourcegroup",
            "location": "eastus",
            "properties": {
                "provisioningState": "Succeeded"
            }
        },
        {
            "id": "/subscriptions/3a8555...555995/resourceGroups/mysecondresourcegroup",
            "name": "mysecondresourcegroup",
            "location": "northeurope",
            "tags": {
                "tagname1": "My first tag"
            },
            "properties": {
                "provisioningState": "Succeeded"
            }
        }
    ]
}
```

### <a name="create-a-resource-group"></a>创建资源组
到目前为止，我们一直只在 Resource Manager API 中查询信息。 是创建一些资源的时候了，让我们从最简单的开始，创建一个资源组。 以下 HTTP 请求会在所选区域/位置中创建一个资源组，并向其添加标记。

（将 YOUR_ACCESS_TOKEN、SUBSCRIPTION_ID、RESOURCE_GROUP_NAME 替换为实际的访问令牌、订阅 ID 和要创建的资源组名称）

```HTTP
PUT /subscriptions/SUBSCRIPTION_ID/resourcegroups/RESOURCE_GROUP_NAME?api-version=2015-01-01 HTTP/1.1
Host: management.azure.com
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json

{
  "location": "northeurope",
  "tags": {
    "tagname1": "test-tag"
  }
}
```

如果成功，将收到类似于下面的响应：

```json
{
  "id": "/subscriptions/3a8555...555995/resourceGroups/RESOURCE_GROUP_NAME",
  "name": "RESOURCE_GROUP_NAME",
  "location": "northeurope",
  "tags": {
    "tagname1": "test-tag"
  },
  "properties": {
    "provisioningState": "Succeeded"
  }
}
```

已在 Azure 中成功创建资源组。 祝贺你！

### <a name="deploy-resources-to-a-resource-group-using-a-resource-manager-template"></a>使用 Resource Manager 模板将资源部署到资源组
有了 Resource Manager，就可以使用模板部署资源。 模板可定义多个资源及其依赖项。 在本部分中，我们假设你熟悉资源管理器模板，并仅演示如何进行 API 调用以开始部署。 有关构造模板的详细信息，请参阅[创作 Azure 资源管理器模板](resource-group-authoring-templates.md)。

模板的部署与调用其他 API 的方式并没有太大差别。 一个重要方面是模板的部署可能需要很长时间。 API 调用只返回部署的状态，并由作为开发人员的你来进行查询，以了解部署何时完成。 有关详细信息，请参阅[跟踪异步 Azure 操作](resource-manager-async-operations.md)。

在本示例中，我们将使用 [GitHub](https://github.com/Azure/azure-quickstart-templates) 上提供的公开模板。 我们使用的模板会将 Linux VM 部署到美国西部区域。 尽管本示例使用公共存储库（如 GitHub）中提供的模板，但你也可以改为传递整个模板作为请求的一部分。 请注意，我们会在请求中提供在已部署的模板中使用的参数值。

（将 SUBSCRIPTION_ID、RESOURCE_GROUP_NAME、DEPLOYMENT_NAME、YOUR_ACCESS_TOKEN、GLOBALY_UNIQUE_STORAGE_ACCOUNT_NAME、ADMIN_USER_NAME、ADMIN_PASSWORD 和 DNS_NAME_FOR_PUBLIC_IP 替换为适用于请求的值）

```HTTP
PUT /subscriptions/SUBSCRIPTION_ID/resourcegroups/RESOURCE_GROUP_NAME/providers/microsoft.resources/deployments/DEPLOYMENT_NAME?api-version=2015-01-01 HTTP/1.1
Host: management.azure.com
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json

{
  "properties": {
    "templateLink": {
      "uri": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-simple-linux-vm/azuredeploy.json",
      "contentVersion": "1.0.0.0",
    },
    "mode": "Incremental",
    "parameters": {
        "newStorageAccountName": {
          "value": "GLOBALY_UNIQUE_STORAGE_ACCOUNT_NAME"
        },
        "adminUsername": {
          "value": "ADMIN_USER_NAME"
        },
        "adminPassword": {
          "value": "ADMIN_PASSWORD"
        },
        "dnsNameForPublicIP": {
          "value": "DNS_NAME_FOR_PUBLIC_IP"
        },
        "ubuntuOSVersion": {
          "value": "15.04"
        }
    }
  }
}
```

为了提高本文档的可读性，此处省略了此请求的较长 JSON 响应。 该响应包含创建的样板化部署的相关信息。

## <a name="next-steps"></a>后续步骤

- 若要了解如何处理异步 REST 操作，请参阅[跟踪异步 Azure 操作](resource-manager-async-operations.md)。
