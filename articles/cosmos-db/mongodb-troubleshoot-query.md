---
title: 排查使用适用于 MongoDB 的 Azure Cosmos DB API 时的查询问题
description: 了解如何识别、诊断和排查 Azure Cosmos DB 的 API 的 MongoDB 查询问题。
author: timsander1
ms.service: cosmos-db
ms.topic: troubleshooting
ms.subservice: cosmosdb-mongo
ms.date: 10/12/2020
ms.author: tisande
ms.reviewer: sngun
ms.openlocfilehash: 88ef081c75a64b5cb7517ba6994834b3a64a0e6f
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340883"
---
# <a name="troubleshoot-query-issues-when-using-the-azure-cosmos-db-api-for-mongodb"></a>排查使用适用于 MongoDB 的 Azure Cosmos DB API 时的查询问题
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

本文逐步说明排查 Azure Cosmos DB 中的查询问题的一般建议方法。 虽然不应将本文中所述的步骤视为针对潜在查询问题的完全防御方法，但我们在其中包含了最常见的性能提示。 在 Azure Cosmos DB 的 API for MongoDB 中，你应使用本文作为排查速度缓慢或开销较高的查询的起点。 如果使用 Azure Cosmos DB 核心 (SQL) API，请参阅 [SQL API 查询故障排除指南](troubleshoot-query-performance.md) 一文。

Azure Cosmos DB 中的查询优化广泛分类，如下所示：

- 可降低查询请求单位 (RU) 费用的优化
- 仅降低延迟的优化

如果减少查询的 RU 费用，通常还会降低延迟。

本文提供可使用 [nutrition 数据集](https://github.com/CosmosDB/labs/blob/master/dotnet/setup/NutritionData.json)重新创建的示例。

> [!NOTE] 
> 本文假设使用的是版本3.6 的 Azure Cosmos DB "s API for MongoDB。 在版本3.2 中执行质量较差的某些查询在版本3.6 中进行了重大改进。 通过填写 [支持请求](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)来升级到版本3.6。

## <a name="use-explain-command-to-get-metrics"></a>使用 $explain 命令获取指标

在 Azure Cosmos DB 中优化查询时，第一步应始终获取查询 [的 RU 费用](find-request-unit-charge-mongodb.md) 。 作为一种粗略的指导原则，你应该了解如何使用大于50 

除了获取 RU 费用以外，还应使用 `$explain` 命令获取查询和索引使用情况指标。 下面是运行查询并使用 `$explain` 命令显示查询和索引使用情况指标的示例：

**$explain 命令：**

```
db.coll.find({foodGroup: "Baby Foods"}).explain({"executionStatistics": true })
```

**输出：**

```json
{
    "stages" : [ 
        {
            "stage" : "$query",
            "timeInclusiveMS" : 905.2888,
            "timeExclusiveMS" : 905.2888,
            "in" : 362,
            "out" : 362,
            "details" : {
                "database" : "db-test",
                "collection" : "collection-test",
                "query" : {
                    "foodGroup" : {
                        "$eq" : "Baby Foods"
                    }
                },
                "pathsIndexed" : [],
                "pathsNotIndexed" : [ 
                    "foodGroup"
                ],
                "shardInformation" : [ 
                    {
                        "activityId" : "e68e6bdd-5e89-4ec5-b053-3dbbc2428140",
                        "shardKeyRangeId" : "0",
                        "durationMS" : 788.5867,
                        "preemptions" : 1,
                        "outputDocumentCount" : 362,
                        "retrievedDocumentCount" : 8618
                    }
                ],
                "queryMetrics" : {
                    "retrievedDocumentCount" : 8618,
                    "retrievedDocumentSizeBytes" : 104963042,
                    "outputDocumentCount" : 362,
                    "outputDocumentSizeBytes" : 2553535,
                    "indexHitRatio" : 0.0016802042237178,
                    "totalQueryExecutionTimeMS" : 777.72,
                    "queryPreparationTimes" : {
                        "queryCompilationTimeMS" : 0.19,
                        "logicalPlanBuildTimeMS" : 0.14,
                        "physicalPlanBuildTimeMS" : 0.09,
                        "queryOptimizationTimeMS" : 0.03
                    },
                    "indexLookupTimeMS" : 0,
                    "documentLoadTimeMS" : 687.22,
                    "vmExecutionTimeMS" : 774.09,
                    "runtimeExecutionTimes" : {
                        "queryEngineExecutionTimeMS" : 37.45,
                        "systemFunctionExecutionTimeMS" : 10.82,
                        "userDefinedFunctionExecutionTimeMS" : 0
                    },
                    "documentWriteTimeMS" : 49.42
                }
            }
        }
    ],
    "estimatedDelayFromRateLimitingInMilliseconds" : 0.0,
    "continuation" : {
        "hasMore" : false
    },
    "ok" : 1.0
}
```

`$explain`命令输出时间较长，并且包含有关查询执行的详细信息。 但一般情况下，在优化查询性能时，应重点关注几个部分：

| 指标 | 描述 | 
| ------ | ----------- |
| `timeInclusiveMS` | 后端查询延迟 |
| `pathsIndexed` | 显示查询使用的索引 | 
| `pathsNotIndexed` | 显示查询可以使用的索引（如果可用） | 
| `shardInformation` | 特定[物理分区](./partitioning-overview.md#physical-partitions)的查询性能摘要 | 
| `retrievedDocumentCount` | 查询引擎加载的文档数 | 
| `outputDocumentCount` | 查询结果中返回的文档数 | 
| `estimatedDelayFromRateLimitingInMilliseconds` | 由于速率限制而导致的额外查询延迟 | 

获取查询度量值后，请将与查询的进行比较 `retrievedDocumentCount` `outputDocumentCount` 。 使用这种比较可以确定要在本文中查看的相关部分。 `retrievedDocumentCount`是查询引擎需要加载的文档数。 `outputDocumentCount`是查询结果所需的文档数。 如果 `retrievedDocumentCount`  明显大于，则表明 `outputDocumentCount` 查询中至少有一个部分无法使用索引，并且需要执行扫描。

请参阅以下部分，了解适用于你的方案的相关查询优化。

### <a name="querys-ru-charge-is-too-high"></a>查询的 RU 费用过高

#### <a name="retrieved-document-count-is-significantly-higher-than-output-document-count"></a>“已检索文档计数”明显大于“输出文档计数”

- [包括必要的索引。](#include-necessary-indexes)

- [了解哪些聚合运算使用索引。](#understand-which-aggregation-operations-use-the-index)

#### <a name="retrieved-document-count-is-approximately-equal-to-output-document-count"></a>“已检索文档计数”约等于“输出文档计数”

- [最大限度地减少跨分区查询。](#minimize-cross-partition-queries)

### <a name="querys-ru-charge-is-acceptable-but-latency-is-still-too-high"></a>查询的 RU 费用可接受，但延迟仍然过高

- [提高邻近度。](#improve-proximity)

- [增加预配的吞吐量。](#increase-provisioned-throughput)

## <a name="queries-where-retrieved-document-count-exceeds-output-document-count"></a>检索到的文档计数超出输出文档计数的查询

 `retrievedDocumentCount`是查询引擎加载所需的文档数。 `outputDocumentCount`是查询返回的文档数。 如果 `retrievedDocumentCount` 明显大于，则表明 `outputDocumentCount` 查询中至少有一个部分无法使用索引，并且需要执行扫描。

下面是一个不完全由索引提供服务的扫描查询示例：

**$explain 命令：**

```
db.coll.find(
  {
    $and : [
            { "foodGroup" : "Cereal Grains and Pasta"}, 
            { "description" : "Oat bran, cooked"}
        ]
  }
).explain({"executionStatistics": true })
```

**输出：**

```json
{
    "stages" : [ 
        {
            "stage" : "$query",
            "timeInclusiveMS" : 436.5716,
            "timeExclusiveMS" : 436.5716,
            "in" : 1,
            "out" : 1,
            "details" : {
                "database" : "db-test",
                "collection" : "indexing-test",
                "query" : {
                    "$and" : [ 
                        {
                            "foodGroup" : {
                                "$eq" : "Cereal Grains and Pasta"
                            }
                        }, 
                        {
                            "description" : {
                                "$eq" : "Oat bran, cooked"
                            }
                        }
                    ]
                },
                "pathsIndexed" : [],
                "pathsNotIndexed" : [ 
                    "foodGroup", 
                    "description"
                ],
                "shardInformation" : [ 
                    {
                        "activityId" : "13a5977e-a10a-4329-b68e-87e4f0081cac",
                        "shardKeyRangeId" : "0",
                        "durationMS" : 435.4867,
                        "preemptions" : 1,
                        "outputDocumentCount" : 1,
                        "retrievedDocumentCount" : 8618
                    }
                ],
                "queryMetrics" : {
                    "retrievedDocumentCount" : 8618,
                    "retrievedDocumentSizeBytes" : 104963042,
                    "outputDocumentCount" : 1,
                    "outputDocumentSizeBytes" : 6064,
                    "indexHitRatio" : 0.0,
                    "totalQueryExecutionTimeMS" : 433.64,
                    "queryPreparationTimes" : {
                        "queryCompilationTimeMS" : 0.12,
                        "logicalPlanBuildTimeMS" : 0.09,
                        "physicalPlanBuildTimeMS" : 0.1,
                        "queryOptimizationTimeMS" : 0.02
                    },
                    "indexLookupTimeMS" : 0,
                    "documentLoadTimeMS" : 387.44,
                    "vmExecutionTimeMS" : 432.93,
                    "runtimeExecutionTimes" : {
                        "queryEngineExecutionTimeMS" : 45.36,
                        "systemFunctionExecutionTimeMS" : 16.86,
                        "userDefinedFunctionExecutionTimeMS" : 0
                    },
                    "documentWriteTimeMS" : 0.13
                }
            }
        }
    ],
    "estimatedDelayFromRateLimitingInMilliseconds" : 0.0,
    "continuation" : {
        "hasMore" : false
    },
    "ok" : 1.0
}
```

`retrievedDocumentCount` (8618) 明显高于 `outputDocumentCount` (1) ，表示此查询需要进行文档扫描。 

### <a name="include-necessary-indexes"></a>包括必要的索引

你应检查 `pathsNotIndexed` 阵列并添加这些索引。 在此示例中， `foodGroup` `description` 应为路径和创建索引。

```json
"pathsNotIndexed" : [ 
                    "foodGroup", 
                    "description"
                ]
```

Azure Cosmos DB 的 MongoDB API 中的索引最佳做法不同于 MongoDB。 在 MongoDB Azure Cosmos DB 的 API 中，复合索引仅用于需要以多个属性有效排序的查询。 如果对多个属性使用筛选器进行查询，则应为这些属性中的每一个创建单个字段索引。 查询谓词可以使用多个单字段索引。

[通配符索引](mongodb-indexing.md#wildcard-indexes) 可以简化索引。 与在 MongoDB 中不同，通配符索引可以在查询谓词中支持多个字段。 如果使用一个通配符索引，而不是为每个属性创建单独的索引，查询性能不会有差异。 为所有属性添加通配符索引是优化所有查询的最简单方法。

你可以随时添加新索引，而不会影响写入或读取可用性。 你可以[跟踪索引转换进度](./how-to-manage-indexing-policy.md#dotnet-sdk)。

### <a name="understand-which-aggregation-operations-use-the-index"></a>了解哪些聚合运算使用索引

大多数情况下，Azure Cosmos DB 的 MongoDB API 中的聚合操作将部分使用索引。 通常，查询引擎将首先应用相等和范围筛选器并使用索引。 应用这些筛选器后，查询引擎可以评估其他筛选器，并根据需要加载其余文档以计算聚合。 

下面是一个示例：

```
db.coll.aggregate( [
   { $match: { foodGroup: 'Fruits and Fruit Juices' } },
   {
     $group: {
        _id: "$foodGroup",
        total: { $max: "$version" }
     }
   }
] )
```

在这种情况下，索引可以优化该 `$match` 阶段。 为添加索引 `foodGroup` 可以显著提高查询性能。 与 MongoDB 一样，应 `$match` 尽可能地在聚合管道中放置，以最大程度地利用索引。

在 Azure Cosmos DB 的 MongoDB API 中，索引不用于实际聚合，在本例中为 `$max` 。 添加索引 `version` 不会提高查询性能。

## <a name="queries-where-retrieved-document-count-is-equal-to-output-document-count"></a>检索到的文档计数等于输出文档计数的查询

如果 `retrievedDocumentCount` 约等于 `outputDocumentCount` ，则查询引擎不必扫描很多不必要的文档。

### <a name="minimize-cross-partition-queries"></a>最大限度地减少跨分区查询

当请求单位和数据存储需求增加时，Azure Cosmos DB 将使用[分区](partitioning-overview.md)来扩展单个容器。 每个物理分区具有不同的独立索引。 如果查询具有匹配容器分区键的相等性筛选器，则你只需检查相关分区的索引。 这种优化可以减少查询所需的 RU 总数。 [详细了解分区内查询和跨分区查询之间的差异](how-to-query-container.md)。

如果有大量预配的 RU（超过 30,000）或存储了大量数据（约超过 100 GB），那么你可能需要有一个足够大的容器，以查看查询 RU 费用的显著降低。 

可以通过检查 `shardInformation` 阵列了解每个单独物理分区的查询指标。 唯一值的数目 `shardKeyRangeId` 是需要执行查询的物理分区数。 在此示例中，查询在四个物理分区上执行。 务必要了解，执行与索引利用率完全无关。 换句话说，跨分区查询仍可使用索引。

```json
  "shardInformation" : [ 
                    {
                        "activityId" : "42f670a8-a201-4c58-8023-363ac18d9e18",
                        "shardKeyRangeId" : "5",
                        "durationMS" : 24.3859,
                        "preemptions" : 1,
                        "outputDocumentCount" : 463,
                        "retrievedDocumentCount" : 463
                    }, 
                    {
                        "activityId" : "a8bf762a-37b9-4c07-8ed4-ae49961373c0",
                        "shardKeyRangeId" : "2",
                        "durationMS" : 35.8328,
                        "preemptions" : 1,
                        "outputDocumentCount" : 905,
                        "retrievedDocumentCount" : 905
                    }, 
                    {
                        "activityId" : "3754e36b-4258-49a6-8d4d-010555628395",
                        "shardKeyRangeId" : "1",
                        "durationMS" : 67.3969,
                        "preemptions" : 1,
                        "outputDocumentCount" : 1479,
                        "retrievedDocumentCount" : 1479
                    }, 
                    {
                        "activityId" : "a69a44ee-db97-4fe9-b489-3791f3d52878",
                        "shardKeyRangeId" : "0",
                        "durationMS" : 185.1523,
                        "preemptions" : 1,
                        "outputDocumentCount" : 867,
                        "retrievedDocumentCount" : 867
                    }
                ]
```

## <a name="optimizations-that-reduce-query-latency"></a>可降低查询延迟的优化

在许多情况下，当查询延迟仍然过高时，RU 费用是可接受的。 以下部分概述了降低查询延迟的提示。 如果在同一数据集上多次运行相同的查询，则每次都将具有相同的 RU 费用。 但是，每次执行查询时，查询延迟可能各不相同。

### <a name="improve-proximity"></a>提高邻近度

在非 Azure Cosmos DB 帐户的区域中运行的查询，比在同一区域中运行的查询的延迟更高。 例如，如果在台式机上运行代码，则延迟比从 Azure Cosmos DB 所在的同一 Azure 区域中的某个虚拟机上运行查询要高出几十或几百毫秒（或更高）。 可以轻松[在 Azure Cosmos DB 中全局分发数据](tutorial-global-distribution-mongodb.md)，以确保将数据置于更靠近应用的位置。

### <a name="increase-provisioned-throughput"></a>增加预配的吞吐量

在 Azure Cosmos DB 中，预配的吞吐量以请求单位 (RU) 计量。 假设某个查询消耗 5 RU 吞吐量。 如果预配 1000 RU，则每秒可以运行该查询 200 次。 如果在没有足够的可用吞吐量的情况下尝试运行查询，Azure Cosmos DB 将会限制请求数。 Azure Cosmos DB 的 MongoDB API 在等待一小段时间后将自动重试此查询。 受限制的请求需要花费更长的时间，因此增加预配的吞吐量可以改进查询延迟。

`estimatedDelayFromRateLimitingInMilliseconds`如果增加吞吐量，则值可提供潜在的延迟优势。

## <a name="next-steps"></a>后续步骤

* [ (SQL API) 的查询性能故障排除 ](troubleshoot-query-performance.md)
* [管理 Azure Cosmos DB 的用于 MongoDB 的 API 中的索引编制](mongodb-indexing.md)