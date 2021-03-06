---
title: "在生产环境中部署模型 - Azure 机器学习 | Microsoft Docs"
description: "如何将模型部署到生产环境使其在进行业务决策方面能够发挥积极作用。"
documentationcenter: 
author: bradsev
manager: cgronlun
editor: cgronlun
ms.assetid: 
ms.service: machine-learning
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/30/2017
ms.author: bradsev;
ms.openlocfilehash: 122c6db2a915dd2a933e261b5b735e7214407d66
ms.sourcegitcommit: 95500c068100d9c9415e8368bdffb1f1fd53714e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="deploy-models-in-production"></a>在生产环境中部署模型

生产部署可让模型在企业中发挥积极作用。 所部署模型提供的预测可用于业务决策。

## <a name="production-platforms"></a>生产平台
可通过多种方法和平台将模型投入生产。 下面是几个选项：


- [Azure 机器学习中的模型部署](../preview/model-management-overview.md)
- [SQL-Server 中的模型部署](https://docs.microsoft.com/sql/advanced-analytics/tutorials/sqldev-py6-operationalize-the-model)
- [Microsoft Machine Learning Server](https://docs.microsoft.com/sql/advanced-analytics/r/r-server-standalone)


>[!NOTE]
>在部署之前，必须确保模型的延迟评分够低，使模型可在生产环境中使用。
>


>[!NOTE]
>对于使用 Azure 机器学习工作室的部署，请参阅[部署 Azure 机器学习 Web 服务](../studio/publish-a-machine-learning-web-service.md)。
>

## <a name="ab-testing"></a>A/B 测试
如果在生产环境中部署了多个模型，执行 [A/B 测试](https://en.wikipedia.org/wiki/A/B_testing)来比较模型的性能可能很有用。 

 
## <a name="next-steps"></a>后续步骤

我们还提供了相应的演练，用于演示**具体方案**的操作过程的所有步骤。 [示例演练](walkthroughs.md)一文列出了相关步骤并以缩略图说明的形式提供了链接。 这些演练演示如何将云、本地工具和服务合并到工作流或管道中，以创建智能应用程序。 
 


