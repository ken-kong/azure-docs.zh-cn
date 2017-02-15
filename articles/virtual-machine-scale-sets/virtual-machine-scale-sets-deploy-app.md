---
title: "在虚拟机规模集上部署应用 | Microsoft Docs"
description: "在虚拟机缩放集上部署应用"
services: virtual-machine-scale-sets
documentationcenter: 
author: gbowerman
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: f8892199-f2e2-4b82-988a-28ca8a7fd1eb
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/26/2016
ms.author: guybo
translationtype: Human Translation
ms.sourcegitcommit: 5919c477502767a32c535ace4ae4e9dffae4f44b
ms.openlocfilehash: 611099e8c28584e091f9dc6c07eebf7e397092ea


---
# <a name="deploy-an-app-on-virtual-machine-scale-sets"></a>在虚拟机缩放集上部署应用
在 VM 缩放集上运行的应用程序通常是以三种方法之一部署的：

* 部署时在平台映像中安装新软件。 在本文的语境中，平台映像是指从 Azure 应用商店获取的操作系统映像，例如 Ubuntu 16.04、Windows Server 2012 R2 等。

可以使用 [VM 扩展](../virtual-machines/virtual-machines-windows-extensions-features.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)在平台映像中安装新软件。 VM 扩展是部署 VM 时运行的软件。 可以使用自定义脚本扩展，在部署时运行所需的任何代码。 [此处](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-lapstack-autoscale)提供了包含以下两个 VM 扩展的示例 Azure Resource Manager 模板：用于安装 Apache 和 PHP 的“Linux 自定义脚本扩展”，用于发出 Azure 自动缩放所用性能数据的“诊断扩展”。

此方法的优点是在应用程序代码与 OS 之间提供某种程度的隔离，可以单独维护应用程序。 当然，这也意味着会出现更多的运动组件，如果脚本需要下载和配置的项很多，VM 部署时间也可能更长。

**如果在自定义脚本扩展命令中传递敏感信息（例如密码），请务必在自定义脚本扩展的 `protectedSettings` 属性（而不是 `settings` 属性）中指定 `commandToExecute`。**

* 创建在单个 VHD 中包含 OS 和应用程序的自定义 VM 映像。 此处的缩放集由一组 VM 构成，这些 VM 是从创建的映像复制的，必须对其进行维护。 这种方法不需要在部署 VM 时进行额外的配置。 但是，在 `2016-03-30` 版（和更低版本）的 VM 缩放集中，VM 的 OS 磁盘限制为一个存储帐户。 因此，一个缩放集中最多能包含 40 个 VM，而不像在平台映像中，每个缩放集限制为 100 个 VM。 有关详细信息，请参阅[规模集设计概述](virtual-machine-scale-sets-design-overview.md)。
* 可以部署一个平台或自定义映像（简单地说，就是一个容器主机），然后将应用程序安装为一个或多个可以使用 Orchestrator 或配置管理工具进行管理的容器。 这种方法的优势是可以从应用程序层抽象化云基础结构，并且可以独立维护应用程序。

## <a name="what-happens-when-a-vm-scale-set-scales-out"></a>扩展 VM 缩放集会发生什么情况？
通过增加容量将一个或多个 VM 添加到规模集时（无论是手动还是通过自动缩放），会自动安装应用程序。 例如，如果缩放集包含定义的扩展，则每次创建新 VM 时，这些扩展都会在新 VM 上运行。 如果缩放集基于自定义映像，所有新 VM 都是自定义源映像的副本。 如果缩放集 VM 是容器主机，则可以使用启动代码加载自定义脚本扩展中的容器，或者，扩展可以安装一个可向群集协调器（例如 Azure 容器服务）注册的代理。

## <a name="how-do-you-manage-application-updates-in-vm-scale-sets"></a>如何管理 VM 缩放集中的应用程序更新？
对于 VM 缩放集中的应用程序更新，有三个派生自上述三种应用程序部署方法的主要方法：

* 使用 VM 扩展进行更新。 每次创建新 VM、重置现有 VM 的映像或更新 VM 扩展时，都会执行针对 VM 缩放集定义的所有 VM 扩展。 如果需要更新应用程序，可行的方法是通过扩展直接更新应用程序 - 只需更新扩展定义即可。 为此，一个简单的方法是将 fileUris 更改为指向新软件。
* 不可变的自定义映像方法。 将应用程序（或应用组件）制作成 VM 映像时，可以专注于构建可靠的管道，将映像的构建、测试和部署自动化。 可对体系结构进行设计，帮助将分阶段的缩放集快速切换到生产环境。 [Azure Spinnaker 驱动程序的工作原理](https://github.com/spinnaker/deck/tree/master/app/scripts/modules/azure)很好地示范了这种方法 - [http://www.spinnaker.io/](http://www.spinnaker.io/)。

Packer 和 Terraform 也支持 Azure Resource Manager，因此也可以“代码方式”定义映像并在 Azure 中将其生成，然后在规模集中使用 VHD。 但是，这种做法会给应用商店映像造成问题，在这种情况下，扩展/自定义脚本变得更加重要，因为无法直接在应用商店中处理代码。

* 更新容器 将应用程序生命周期管理抽象化为高于云基础结构的级别（例如通过包装应用程序），将应用组件抽象化为容器，并通过容器协调器和配置管理器（例如 Chef/Puppet）管理这些容器。

然后，缩放集 VM 将变成容器的稳定基础，只需偶尔进行安全和 OS 相关的更新。 如前所述，Azure 容器服务就是采用此方法并围绕它构建服务的好例子。

## <a name="how-do-you-roll-out-an-os-update-across-update-domains"></a>如何跨更新域推出 OS 更新？
假设要在更新 OS 映像的同时让 VM 缩放集持续运行。 为此，一种做法是每次更新一个 VM 的 VM 映像。 可以使用 PowerShell 或 Azure CLI 实现此目的。 有单独的命令可在单个 VM 上更新 VM 规模集模型（其配置的定义方式），以及发出“手动升级”调用。

[此处](https://github.com/gbowerman/vmsstools)提供了一个示例 Python 脚本，它每次会自动更新一个更新域的 VM 规模集。 （注意：这种方法更多地充当一种概念认证，而不是可靠的、随时可用于生产的解决方案 - 可以在其中添加某些错误检查机制等等）。




<!--HONumber=Nov16_HO3-->

