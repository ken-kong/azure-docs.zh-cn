---
title: "适用于 SAP 的 Microsoft Azure 认证 | Microsoft Docs"
description: "Azure 平台上 SAP 的当前配置和认证的更新列表。"
services: virtual-machines-linux
documentationcenter: 
author: RicksterCDN
manager: timlt
editor: 
tags: azure-resource-manager
keywords: 
ms.assetid: 
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 10/12/2017
ms.author: rclaus
ms.custom: 
ms.openlocfilehash: 865fa54c908481b3f4c211f12293538c617b6129
ms.sourcegitcommit: a036a565bca3e47187eefcaf3cc54e3b5af5b369
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/17/2017
---
# <a name="sap-certifications-and-configurations-running-on-microsoft-azure"></a>在 Microsoft Azure 上运行的 SAP 认证和配置

SAP 和 Microsoft 具有悠久的合作历史，建立了强大的合作伙伴关系，使其客户可以互惠互利。 Microsoft 会不断更新其平台并向 SAP 提交新认证详细信息，确保 Microsoft Azure 是运行 SAP 工作负荷的最佳平台。 下表概述了我们支持的配置以及不断增加的认证的列表。 

## <a name="sap-hana-certifications"></a>SAP HANA 认证
引用：

- [SAP 说明 2316233 - Microsoft Azure 上的 SAP HANA（大型实例）](https://launchpad.support.sap.com/#/notes/2316233)，其中涵盖了有关 SAP HANA 支持的 HANA 大型实例。
- [SAP HANA 认证的 IaaS 平台](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Amazon%20Web%20Services%2CMicrosoft%20Azure)，提供对本机 Azure VM 的 SAP HANA 支持。

| SAP 产品 | 支持的 OS | Azure 产品 |
| --- | --- | --- |
| SAP HANA Developer Edition（包括由 SQLODBC、ODBO（仅限 Windows）、ODBC 和 JDBC 驱动程序组成的 HANA Client 软件）、HANA Studio 和 HANA 数据库 | Red Hat Enterprise Linux、SUSE Linux Enterprise | D 系列 VM 系列 |
| HANA 上的 Business One | SUSE Linux Enterprise | DS14_v2 |
| SAP S/4 HANA |Red Hat Enterprise Linux、SUSE Linux Enterprise | 受控的 GS5 可用性、Azure 上的 SAP HANA（大型实例） |
| Suite on HANA、OLTP | Red Hat Enterprise Linux、SUSE Linux Enterprise | 用于非生产方案的单节点部署的 GS5、Azure 上的 SAP HANA（大型实例） |
| 适用于 BW 和 OLAP 的 HANA 企业版 | Red Hat Enterprise Linux、SUSE Linux Enterprise | 用于单节点部署的 GS5、Azure 上的 SAP HANA（大型实例） |
| SAP BW/4 HANA | Red Hat Enterprise Linux、SUSE Linux Enterprise | 用于单节点部署的 GS5、Azure 上的 SAP HANA（大型实例） |

## <a name="sap-netweaver-certifications"></a>SAP NetWeaver 认证
在 Microsoft 和 SAP 的全面支持下，Microsoft Azure 已针对以下 SAP 产品进行认证。
引用：

- [1928533 - Azure 上的 SAP 应用程序：支持的产品和 Azure VM 类型](https://launchpad.support.sap.com/#/notes/1928533)，适用于所有基于 SAP NetWeaver 的应用程序，包括 SAP TREX、SAP LiveCache 和 SAP 内容服务器。 以及所有数据库，但 SAP HANA 除外。


| SAP 产品 | 来宾 OS | RDBMS | 虚拟机类型 |
| --- | --- | --- | --- |
| SAP 业务套件软件 |Windows、SUSE Linux Enterprise、Red Hat Enterprise Linux、Oracle Linux |SQL Server、Oracle（仅限 Windows 和 Oracle Linux）、DB2、SAP ASE |A5 到 A11、D11 到 D14、DS11 到 DS14、DS11_v2 到 DS15_v2、GS1 到 GS5、M 系列 |
| 多合一 SAP 业务软件 |Windows、SUSE Linux Enterprise、Red Hat Enterprise Linux |SQL Server、Oracle（仅限 Windows 和 Oracle Linux）、DB2、SAP ASE |A5 到 A11、D11 到 D14、DS11 到 DS14、DS11_v2 到 DS15_v2、GS1 到 GS5、M 系列 |
| SAP BusinessObjects BI |Windows |不适用 |A5 到 A11、D11 到 D14、DS11 到 DS14、DS11_v2 到 DS15_v2、GS1 到 GS5、M 系列 |
| SAP NetWeaver |Windows、SUSE Linux Enterprise、Red Hat Enterprise Linux |SQL Server、Oracle（仅限 Windows 和 Oracle Linux）、DB2、SAP ASE |A5 到 A11、D11 到 D14、DS11 到 DS14、DS11_v2 到 DS15_v2、GS1 到 GS5、M 系列 |

## <a name="other-sap-workload-supported-on-azure"></a>在 Azure 上支持的其他 SAP 工作负荷

| SAP 产品 | 来宾 OS | RDBMS | 虚拟机类型 |
| --- | --- | --- | --- |
| SQL Server 上的 SAP Business On | Windows  | SQL Server | 所有 NetWeaver 认证的 VM 类型 |
| SAP BPC 10.01 MS SP08 | Windows | | 所有 NetWeaver 认证的 VM 类型<br /> SAP 说明 #2451795 |
| SAP 业务对象 BI 平台 | Windows | | SAP 说明 #2145537 |
| SAP 数据服务 4.2 | | | SAP 说明 #2288344 |
| SAP Hybris Commerce Platform 5.x 和 6.x | Windows | SQL Server、Oracle | 所有 NetWeaver 认证的 VM 类型<br /> [Hybris Wiki](https://wiki.hybris.com/display/SUP/Using+the+hybris+Platform+with+the+Cloud) |
