---
title: "Azure 搜索中的索引 | Microsoft Docs"
description: "了解 Azure 搜索中的索引概念以及如何使用索引。"
services: search
documentationcenter: 
author: ashmaka
ms.assetid: a395e166-bf2e-4fca-8bfc-116a46c5f7b1
ms.service: search
ms.devlang: na
ms.workload: search
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.date: 11/08/2017
ms.author: ashmaka
ms.openlocfilehash: 87f1121594d8577b5dacac4026aa7d86b2921d10
ms.sourcegitcommit: 68aec76e471d677fd9a6333dc60ed098d1072cfc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/18/2017
---
# <a name="indexes-in-azure-search"></a>Azure 搜索中的索引
> [!div class="op_single_selector"]
> * [概述](search-what-is-an-index.md)
> * [门户](search-create-index-portal.md)
> * [.NET](search-create-index-dotnet.md)
> * [REST](search-create-index-rest-api.md)
> 
> 

在 Azure 搜索中，索引是 Azure 搜索服务使用的文档及其他结构的永久存储。 文档是索引中的一个可搜索数据单元。 例如，电子商务零售商可能有所销售每件商品的文档，新闻机构可能有每篇报道的文档。 将这些概念对应到更为熟悉的数据库等效对象：*索引*在概念上类似于*表*，*文档*大致相当于表中的*行*。

在 Azure 搜索中添加/上传文档以及提交搜索查询时，提交的是搜索服务中特定索引的请求。

## <a name="field-types-and-attributes-in-an-azure-search-index"></a>Azure 搜索索引中的字段类型和属性
定义架构时，必须在索引中指定每个字段的名称、类型和属性。 字段类型的作用是对该字段中存储的数据进行分类。 对各个字段设置属性的目的是指定字段的使用方式。 下表枚举了可以指定的类型和属性。

### <a name="field-types"></a>字段类型
| 类型 | 说明 |
| --- | --- |
| *Edm.String* |全文搜索可以选择性地标记化（断词、词干提取等）的文本。 |
| *Collection(Edm.String)* |全文搜索可以选择性标记化的字符串列表。 理论上，集合中的项目数没有上限，但集合的有效负载大小上限为 16 MB。 |
| *Edm.Boolean* |包含 true/false 值。 |
| *Edm.Int32* |32 位整数值。 |
| *Edm.Int64* |64 位整数值。 |
| *Edm.Double* |双精度数字数据。 |
| *Edm.DateTimeOffset* |以 OData V4 格式表示的日期时间值（例如 `yyyy-MM-ddTHH:mm:ss.fffZ` 或 `yyyy-MM-ddTHH:mm:ss.fff[+/-]HH:mm`。 |
| *Edm.GeographyPoint* |表示地球上的地理位置的点。 |

若要深入了解 Azure 搜索支持的数据类型，请参阅[此处](https://docs.microsoft.com/rest/api/searchservice/Supported-data-types)。

### <a name="field-attributes"></a>字段属性
| 属性 | 说明 |
| --- | --- |
| *Key* |为每个文档提供唯一 ID 用于查找文档的字符串。 每个索引必须有一个 key。 只有一个字段可以是 key，并且此字段类型必须设置为 Edm.String。 |
| *Retrievable* |指定是否可以在搜索结果中返回字段。 |
| *Filterable* |允许在筛选查询中使用字段。 |
| *Sortable* |允许查询使用此字段对搜索结果排序。 |
| *Facetable* |允许在 [分面导航](search-faceted-navigation.md) 结构中使用字段进行用户自主筛选。 通常，包含重复值的字段更适合分面导航，这些重复值可用于将多个文档（例如，同属一个品牌或服务类别的多个文档）组合在一起。 |
| *Searchable* |将字段标记为可全文搜索。 |

若要深入了解 Azure 搜索的索引属性，请参阅[此处](https://docs.microsoft.com/rest/api/searchservice/Create-Index)。

## <a name="guidance-for-defining-an-index-schema"></a>索引架构定义指南
设计索引时，请循序渐进完成规划阶段，认真思考每个决策。 设计索引时，必须牢记搜索用户体验和业务需求，因为必须为每个字段分配 [适当的属性](https://docs.microsoft.com/rest/api/searchservice/Create-Index)。 部署索引后再进行更改会重新生成并重新加载数据。

如果数据存储要求随时间的推移而改变，可以通过添加或删除分区增加或减少容量。 有关详细信息，请参阅 [Azure 中管理搜索服务](search-manage.md)或[服务限制](search-limits-quotas-capacity.md)。

