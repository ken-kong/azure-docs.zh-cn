---
title: "使用 Azure PowerShell 管理虚拟机规模集 | Microsoft Docs"
description: "管理虚拟机规模集常用的 Azure PowerShell cmdlets，如管理如何启动和停止实例，或更改此规模集容量。"
services: virtual-machine-scale-sets
documentationcenter: 
author: iainfoulds
manager: jeconnoc
editor: 
tags: azure-resource-manager
ms.assetid: d35fa77a-de96-4ccd-a332-eb181d1f4273
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/13/2017
ms.author: iainfou
ms.openlocfilehash: 5b5f3eb05f0d6c10f7efe8af1b93b2cb4fc585c5
ms.sourcegitcommit: 901a3ad293669093e3964ed3e717227946f0af96
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/21/2017
---
# <a name="manage-a-virtual-machine-scale-set-with-azure-powershell"></a>使用 Azure PowerShell 管理虚拟机规模集
在虚拟机规模集的整个生命周期内，可能需要运行一个或多个管理任务。 此外，可能还需要创建自动执行各种生命周期任务的脚本。 本文详细介绍了执行这些任务常用的一些 Azure PowerShell cmdlet。

若要完成这些管理任务，需要最新的 Azure PowerShell 模块。 有关安装和使用最新版本的详细信息，请参阅 [Azure PowerShell 入门](/powershell/azure/get-started-azureps)。 如果需要创建虚拟机规模集，可以[在 Azure 门户中创建规模集](virtual-machine-scale-sets-create-portal.md)。


## <a name="view-information-about-a-scale-set"></a>查看有关规模集的信息
若要查看有关规模集的全部信息，请使用 [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss)。 以下示例将获取有关 myResourceGroup 资源组中 myScaleSet 规模集的信息。 按如下所示输入自己的名称：

```powershell
Get-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```


## <a name="view-vms-in-a-scale-set"></a>查看规模集中的 VM
要在规模集中查看 VM 实例的列表，请使用 [Get-AzureRmVmssVM](/powershell/module/azurerm.compute/get-azurermvmssvm)。 以下示例将列出 myScaleSet 规模集和 myResourceGroup 资源组中的所有 VM 实例。 为这些名称提供自己的值：

```powershell
Get-AzureRmVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

要查看与特定 VM 实例有关的其他信息，请将 `-InstanceId` 参数添加到 [Get-AzureRmVmssVM](/powershell/module/azurerm.compute/get-azurermvmssvm)，并指定要查看的实例。 以下示例将查看与 myScaleSet 规模集和 myResourceGroup 资源组中 VM 实例“0”有关的信息。 按如下所示输入自己的名称：

```powershell
Get-AzureRmVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```


## <a name="change-the-capacity-of-a-scale-set"></a>更改规模集的容量
以上命令显示了与规模集和 VM 实例相关的信息。 要增加或减少规模集中的实例数，可以更改其容量。 规模集会自动创建或删除所需数量的 VM，然后配置 VM 以接收应用程序流量。

首先，使用 [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss) 创建的规模集对象，然后为 `sku.capacity` 指定新的值。 要应用容量更改，请使用 [Update-AzureRmVmss](/powershell/module/azurerm.compute/update-azurermvmss)。 以下示例更新 myResourceGroup 资源组中的 myScaleSet，将其容量更新为能容纳 5 个实例。 请按照如下所示，提供值：

```powershell
# Get current scale set
$vmss = Get-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"

# Set and update the capacity of your scale set
$vmss.sku.capacity = 5
Update-AzureRmVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet" -VirtualMachineScaleSet $vmss 
```

更新规模集容量需要花费数分钟。 如果减少规模集的容量，将首先删除实例 ID 最大的 VM。


## <a name="stop-and-start-vms-in-a-scale-set"></a>停止和启动规模集中的 VM
要停止规模集中的一个或多个 VM，请使用 [Stop-AzureRmVmss](/powershell/module/azurerm.compute/stop-azurermvmss)。 通过 `-InstanceId` 参数，可指定要停止的一个或多个 VM。 若不指定实例 ID，则停止规模集中的所有 VM。 要停止多个 VM，请用逗号分隔每个实例 ID。

以下示例将停止 myScaleSet 规模集和 myResourceGroup 资源组中的实例“0”。 请按照如下所示，提供值：

```powershell
Stop-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```

默认情况下，将取消分配已停止的 VM，这些 VM 不会产生计算费用。 若要在停止 VM 后保持预配状态，请将 `-StayProvisioned` 参数添加到上面的命令中。 保持预配状态的已停止 VM 会产生常规计算费用。


### <a name="start-vms-in-a-scale-set"></a>启动规模集中的 VM
要在规模集中启动一个或多个 VM，请使用 [Start-AzureRmVmss](/powershell/module/azurerm.compute/start-azurermvmss)。 通过 `-InstanceId` 参数，可指定要启动的一个或多个 VM。 若不指定实例 ID，则启动规模集中的所有 VM。 要启动多个 VM，请用逗号分隔每个实例 ID。

以下示例将启动 myScaleSet 规模集和 myResourceGroup 资源组中的实例“0”。 请按照如下所示，提供值：

```powershell
Start-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```


## <a name="restart-vms-in-a-scale-set"></a>重启规模集中的 VM
若要重启规模集中的一个或多个 VM，请使用 [Retart-AzureRmVmss](/powershell/module/azurerm.compute/restart-azurermvmss)。 通过 `-InstanceId` 参数，可指定要重启的一个或多个 VM。 若不指定实例 ID，则重启规模集中的所有 VM。 要重启多个 VM，请用逗号分隔每个实例 ID。

以下示例将重启 myScaleSet 规模集和 myResourceGroup 资源组中的实例“0”。 请按照如下所示，提供值：

```powershell
Restart-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```


## <a name="remove-vms-from-a-scale-set"></a>删除规模集中的 VM
要删除规模集中的一个或多个 VM，请使用 [Remove-AzureRmVmss](/powershell/module/azurerm.compute/remove-azurermvmss)。 通过 `-InstanceId` 参数，可指定要删除的一个或多个 VM。 若不指定实例 ID，规模集中所有 VM 都将删除。 要删除多个 VM，请用逗号分隔每个实例 ID。

以下示例将删除 myScaleSet 规模集和 myResourceGroup 资源组中的实例“0”。 请按照如下所示，提供值：

```powershell
Remove-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```


## <a name="next-steps"></a>后续步骤
规模集的其他常见任务包括如何[部署应用程序](virtual-machine-scale-sets-deploy-app.md)和[升级 VM 实例](virtual-machine-scale-sets-upgrade-scale-set.md)。 也可使用 Azure PowerShell 来[配置自动缩放规则](virtual-machine-scale-sets-autoscale-overview.md)。
