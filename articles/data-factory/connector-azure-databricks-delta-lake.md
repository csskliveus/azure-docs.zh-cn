---
title: 将数据复制到 Azure Databricks Delta Lake 以及从中复制数据
description: 了解如何通过在 Azure 数据工厂管道中使用复制活动，向/从 Azure Databricks Delta Lake 复制数据。
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/24/2020
ms.openlocfilehash: c7e8f96e7917173aaec308b8ae5218684a722483
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507466"
---
# <a name="copy-data-to-and-from-azure-databricks-delta-lake-by-using-azure-data-factory"></a>使用 Azure 数据工厂向/从Azure Databricks Delta Lake 复制数据

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

本文概述了如何使用 Azure 数据工厂中的复制活动向/从 Azure Databricks Delta Lake 复制数据。 本文是在总体概述复制活动的 [Azure 数据工厂中的复制活动](copy-activity-overview.md)的基础上编写的。

## <a name="supported-capabilities"></a>支持的功能

以下活动支持此 Azure Databricks Delta Lake 连接器：

- 带有[支持的源或接收器矩阵](copy-activity-overview.md)表的[复制活动](copy-activity-overview.md)
- [Lookup 活动](control-flow-lookup-activity.md)

通常，Azure 数据工厂支持使用具有以下功能的 Delta Lake 来满足你的各种需求。

- 复制活动支持使用 Azure Databricks Delta Lake 连接器将数据从任何受支持的源数据存储复制到 Azure Databricks Delta Lake 表，以及从 Delta Lake 表复制到任何受支持的接收器数据存储。 它利用 Databricks 群集执行数据移动，详见[“先决条件”部分](#prerequisites)。
- [映射数据流](concepts-data-flow-overview.md) 支持将 Azure 存储上的通用 [增量格式](format-delta.md) 作为源和接收器来读取和写入无代码 ETL 的增量文件，并在托管 Azure Integration Runtime 上运行。
- [Databricks 活动](transform-data-databricks-notebook.md)支持在 Delta Lake 之上协调以代码为中心的 ETL 或机器学习工作负荷。

## <a name="prerequisites"></a>先决条件

若要使用此 Azure Databricks Delta Lake 连接器，你需要在 Azure Databricks 中设置群集。

- 为了将数据复制到 Delta Lake，复制活动会调用 Azure Databricks 群集来从 Azure 存储中读取数据。该存储可以是原始源，也可以是数据工厂通过内置的暂存复制首先将源数据写入到其中的暂存区域。 从 [Delta Lake 作为源](#delta-lake-as-source)中了解详细信息。
- 类似地，为了从 Delta Lake 复制数据，复制活动会调用 Azure Databricks 群集来将数据写入到 Azure 存储。该存储可以是你的原始接收器，也可以是数据工厂通过内置的暂存复制继续从中将数据写入到最终接收器的暂存区域。 从 [Delta Lake 作为接收器](#delta-lake-as-sink)中了解详细信息。

Databricks 群集需要有权访问 Azure Blob 或 Azure Data Lake Storage Gen2 帐户、用于源/接收器/暂存的存储容器/文件系统，以及要在其中写入 Delta Lake 表的容器/文件系统。

- 若要使用 **Azure Data Lake Storage Gen2**，你可以在 Databricks 群集上的 Apache Spark 配置中配置 **服务主体** 或 **存储帐户访问密钥**。 按照 [直接使用服务主体进行访问](/azure/databricks/data/data-sources/azure/azure-datalake-gen2#--access-directly-with-service-principal-and-oauth-20) 或 [直接使用存储帐户访问密钥](/azure/databricks/data/data-sources/azure/azure-datalake-gen2#--access-directly-using-the-storage-account-access-key)访问中的步骤进行操作。

- 若要使用 **Azure Blob 存储**，你可以在 Databricks 群集上的 Apache Spark 配置中配置 **存储帐户访问密钥** 或 **SAS 令牌**。 按照 [使用 RDD API 访问 Azure Blob 存储](/azure/databricks/data/data-sources/azure/azure-storage#access-azure-blob-storage-using-the-rdd-api)中的步骤操作。

在复制活动执行期间，如果你配置的群集已终止，则数据工厂会自动启动它。 如果你使用数据工厂创作 UI 来创作管道，则对于数据预览之类的操作，你需要有一个实时群集，数据工厂不会代表你启动群集。

#### <a name="specify-the-cluster-configuration"></a>指定群集配置

1. 在“群集模式”下拉列表中，选择“标准” 。

2. 在“Databricks Runtime 版本”下拉列表中，选择一个 Databricks 运行时版本。

3. 将以下属性添加到 [Spark 配置](/azure/databricks/clusters/configure#spark-config)来打开[自动优化](/azure/databricks/delta/optimizations/auto-optimize)：

   ```
   spark.databricks.delta.optimizeWrite.enabled true
   spark.databricks.delta.autoCompact.enabled true
   ```

4. 根据集成和缩放需求配置群集。

有关群集配置的详细信息，请参阅[配置群集](/azure/databricks/clusters/configure)。

## <a name="get-started"></a>入门

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

以下部分详细介绍了用来定义特定于 Azure Databricks Delta Lake 连接器的数据工厂实体的属性。

## <a name="linked-service-properties"></a>链接服务属性

Azure Databricks Delta Lake 链接服务支持以下属性。

| Property    | 说明                                                  | 必须 |
| :---------- | :----------------------------------------------------------- | :------- |
| type        | type 属性必须设置为 **AzureDatabricksDeltaLake**。 | 是      |
| 域      | 指定 Azure Databricks 工作区 URL，例如 `https://adb-xxxxxxxxx.xx.azuredatabricks.net`。 |          |
| clusterId   | 指定现有群集的群集 ID。 该群集应该是已创建的交互式群集。 <br>可以通过 Databricks 工作区 ->“群集”->“交互式群集名称”->“配置”->“标记”找到交互式群集的群集 ID。 [了解详细信息](/azure/databricks/clusters/configure#cluster-tags)。 |          |
| accessToken | 数据工厂通过 Azure Databricks 进行身份验证时，必须使用访问令牌。 需从 Databricks 工作区生成访问令牌。 可在 [此处](/azure/databricks/dev-tools/api/latest/authentication#generate-token)找到更详细的查找访问令牌的步骤。 |          |
| connectVia  | 用于连接到数据存储的 [Integration Runtime](concepts-integration-runtime.md)。 可使用 Azure Integration Runtime 或自承载集成运行时（如果数据存储位于专用网络）。 如果未指定，则使用默认 Azure Integration Runtime。 | 否       |

**示例：**

```json
{
    "name": "AzureDatabricksDeltaLakeLinkedService",
    "properties": {
        "type": "AzureDatabricksDeltaLake",
        "typeProperties": {
            "domain": "https://adb-xxxxxxxxx.xx.azuredatabricks.net",
            "clusterId": "<cluster id>",
            "accessToken": {
                "type": "SecureString", 
                "value": "<access token>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>数据集属性

有关可用于定义数据集的各部分和属性的完整列表，请参阅[数据集](concepts-datasets-linked-services.md)一文。 

Azure Databricks Delta Lake 数据集支持以下属性。

| Property  | 说明                                                  | 必须                    |
| :-------- | :----------------------------------------------------------- | :-------------------------- |
| type      | 数据集的 type 属性必须设置为 **AzureDatabricksDeltaLakeDataset**。 | 是                         |
| database | 数据库的名称。 |对于源为“否”，对于接收器为“是”  |
| 表 | 增量表的名称。 |对于源为“否”，对于接收器为“是”  |

**示例：**

```json
{
    "name": "AzureDatabricksDeltaLakeDataset",
    "properties": {
        "type": "AzureDatabricksDeltaLakeDataset",
        "typeProperties": {
            "database": "<database name>",
            "table": "<delta table name>"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "linkedServiceName": {
            "referenceName": "<name of linked service>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供了 Azure Databricks Delta Lake 源和接收器支持的属性的列表。

### <a name="delta-lake-as-source"></a>Delta Lake 作为源

为了从 Azure Databricks Delta Lake 复制数据，复制活动的 **source** 节需要支持以下属性。

| Property                     | 说明                                                  | 必须 |
| :--------------------------- | :----------------------------------------------------------- | :------- |
| type                         | 复制活动源的 type 属性必须设置为 **AzureDatabricksDeltaLakeSource**。 | 是      |
| query          | 指定用于读取数据的 SQL 查询。 对于“按时间顺序查看”控制，请遵循以下模式：<br>- `SELECT * FROM events TIMESTAMP AS OF timestamp_expression`<br>- `SELECT * FROM events VERSION AS OF version` | 否       |
| exportSettings | 用于从增量表检索数据的高级设置。 | 否       |
| ***在 `exportSettings` 下：** _ |  |  |
| type | 导出命令的类型，设置为 _*AzureDatabricksDeltaLakeExportCommand**。 | 是 |
| dateFormat | 将日期类型格式化为具有日期格式的字符串。 自定义日期格式遵循[日期/时间模式](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html)中的格式。 如果未指定，则它使用默认值 `yyyy-MM-dd`。 | 否 |
| timestampFormat | 将时间戳类型格式化为具有时间戳格式的字符串。 自定义日期格式遵循[日期/时间模式](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html)中的格式。 如果未指定，则它使用默认值 `yyyy-MM-dd'T'HH:mm:ss[.SSS][XXX]`。 | 否 |

#### <a name="direct-copy-from-delta-lake"></a>从 Delta Lake 进行的直接复制

如果接收器数据存储和格式符合此部分所述条件，则可使用复制活动将数据从 Azure Databricks Delta 表直接复制到接收器。 数据工厂将检查设置，如果不符合以下条件，复制活动运行将会失败：

- 接收器链接服务是 [Azure Blob 存储](connector-azure-blob-storage.md)或 [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md)。 应当在 Azure Databricks 群集配置中预先配置帐户凭据，请从[先决条件](#prerequisites)中了解详细信息。

- 接收器数据格式为“Parquet”、“带分隔符的文本”或“Avro”，具有以下配置，并且指向文件夹而非文件。   

    - 对于“Parquet”格式，压缩编解码器为“none”、“snappy”或或“gzip”。
    - 对于“带分隔符的文本”格式：
        - `rowDelimiter` 是任意单个字符。
        - `compression` 可以是“none”、“bzip2”、“gzip”。  
        - 不支持 `encodingName` UTF-7。
    - 对于“Avro”格式，压缩编解码器为“none”、“deflate”或“snappy”。   

- 在复制活动源中，`additionalColumns` 未指定。
- 如果将数据复制为带分隔符的文本，则在复制活动接收器中，`fileExtension` 需要是“.csv”。
- 在复制活动映射中，未启用类型转换。

**示例：**

```json
"activities":[
    {
        "name": "CopyFromDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delta lake input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AzureDatabricksDeltaLakeSource",
                "sqlReaderQuery": "SELECT * FROM events TIMESTAMP AS OF timestamp_expression"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="staged-copy-from-delta-lake"></a>从 Delta Lake 进行的暂存复制

如上一部分所述，如果接收器数据存储或格式与直接复制条件不匹配，请通过临时的 Azure 存储实例启用内置的暂存复制。 暂存复制功能也能提供更高的吞吐量。 数据工厂将数据从 Azure Databricks Delta Lake 导出到临时存储，然后将数据复制到接收器，最后从临时存储中清除临时数据。 若要详细了解如何通过暂存方式复制数据，请参阅[暂存复制](copy-activity-performance-features.md#staged-copy)。

若要使用此功能，请创建 [Azure Blob 存储链接服务](connector-azure-blob-storage.md#linked-service-properties)或 [Azure Data Lake Storage Gen2 链接服务](connector-azure-data-lake-storage.md#linked-service-properties)，该服务引用作为临时暂存帐户的存储帐户。 然后，在复制活动中指定 `enableStaging` 和 `stagingSettings` 属性。

>[!NOTE]
>应当在 Azure Databricks 群集配置中预先配置暂存存储帐户凭据，请从[先决条件](#prerequisites)中了解详细信息。

**示例：**

```json
"activities":[
    {
        "name": "CopyFromDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delta lake input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AzureDatabricksDeltaLakeSource",
                "sqlReaderQuery": "SELECT * FROM events TIMESTAMP AS OF timestamp_expression"
            },
            "sink": {
                "type": "<sink type>"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingStorage",
                    "type": "LinkedServiceReference"
                },
                "path": "mystagingpath"
            }
        }
    }
]
```

### <a name="delta-lake-as-sink"></a>Delta Lake 作为接收器

为了将数据复制到 Azure Databricks Delta Lake，复制活动的 **sink** 节需要支持以下属性。

| Property      | 说明                                                  | 必须 |
| :------------ | :----------------------------------------------------------- | :------- |
| type          | 复制活动接收器的 type 属性，设置为 **AzureDatabricksDeltaLakeSink**。 | 是      |
| preCopyScript | 指定一个 SQL 查询。每次运行时，复制活动在将数据写入到 Databricks 增量表之前都会运行该查询。 你可以使用此属性来清除预加载的数据，或添加一个 truncate table 或 Vacuum语句。 | 否       |
| importSettings | 用于将数据写入增量表的高级设置。 | 否 |
| **_在 `importSettings` 下：_* _ |                                                              |  |
| type | 导入命令的类型，设置为 _*AzureDatabricksDeltaLakeImportCommand**。 | 是 |
| dateFormat | 将字符串格式化为具有日期格式的日期类型。 自定义日期格式遵循[日期/时间模式](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html)中的格式。 如果未指定，则它使用默认值 `yyyy-MM-dd`。 | 否 |
| timestampFormat | 将字符串格式化为具有时间戳格式的时间戳类型。 自定义日期格式遵循[日期/时间模式](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html)中的格式。 如果未指定，则它使用默认值 `yyyy-MM-dd'T'HH:mm:ss[.SSS][XXX]`。 | 否 |

#### <a name="direct-copy-to-delta-lake"></a>到 Delta Lake 的直接复制

如果源数据存储和格式符合此部分所述条件，则可使用复制活动将数据从源直接复制到 Azure Databricks Delta Lake。 Azure 数据工厂将检查设置，如果不符合以下条件，复制活动运行将会失败：

- 源链接服务是 [Azure Blob 存储](connector-azure-blob-storage.md)或 [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md)。 应当在 Azure Databricks 群集配置中预先配置帐户凭据，请从[先决条件](#prerequisites)中了解详细信息。

- 源数据格式为“Parquet”、“带分隔符的文本”或“Avro”，具有以下配置，并且指向文件夹而非文件。   

    - 对于“Parquet”格式，压缩编解码器为“none”、“snappy”或或“gzip”。
    - 对于“带分隔符的文本”格式：
        - `rowDelimiter` 为默认值或任意单个字符。
        - `compression` 可以是“none”、“bzip2”、“gzip”。  
        - 不支持 `encodingName` UTF-7。
    - 对于“Avro”格式，压缩编解码器为“none”、“deflate”或“snappy”。   

- 在复制活动源中： 

    - `wildcardFileName` 仅包含通配符 `*` 而未包含 `?`，未指定 `wildcardFolderName`。
    - 未指定 `prefix`、`modifiedDateTimeStart`、`modifiedDateTimeEnd` 和 `enablePartitionDiscovery`。
    - 未指定 `additionalColumns`。

- 在复制活动映射中，未启用类型转换。

**示例：**

```json
"activities":[
    {
        "name": "CopyToDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Delta lake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDatabricksDeltaLakeSink"
            }
        }
    }
]
```

#### <a name="staged-copy-to-delta-lake"></a>到 Delta Lake 的暂存复制

如上一部分所述，如果源数据存储或格式与直接复制条件不匹配，请通过临时的 Azure 存储实例启用内置的暂存复制。 暂存复制功能也能提供更高的吞吐量。 数据工厂会自动转换数据以满足暂存存储中的数据格式要求，然后再从其中将数据加载到 Delta Lake。 最后，它会从存储中清理临时数据。 若要详细了解如何通过暂存方式复制数据，请参阅[暂存复制](copy-activity-performance-features.md#staged-copy)。

若要使用此功能，请创建 [Azure Blob 存储链接服务](connector-azure-blob-storage.md#linked-service-properties)或 [Azure Data Lake Storage Gen2 链接服务](connector-azure-data-lake-storage.md#linked-service-properties)，该服务引用作为临时暂存帐户的存储帐户。 然后，在复制活动中指定 `enableStaging` 和 `stagingSettings` 属性。

>[!NOTE]
>应当在 Azure Databricks 群集配置中预先配置暂存存储帐户凭据，请从[先决条件](#prerequisites)中了解详细信息。

**示例：**

```json
"activities":[
    {
        "name": "CopyToDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Delta lake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDatabricksDeltaLakeSink"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingBlob",
                    "type": "LinkedServiceReference"
                },
                "path": "mystagingpath"
            }
        }
    }
]
```

## <a name="monitoring"></a>监视

Azure 数据工厂提供与其他连接器相同的[复制活动监视体验](copy-activity-monitoring.md)。 此外，由于在 Azure Databricks 群集上运行从/到 delta lake 的加载数据，你可以进一步 [查看详细的群集日志](/azure/databricks/clusters/clusters-manage#--view-cluster-logs) 并 [监视性能](/azure/databricks/clusters/clusters-manage#--monitor-performance)。

## <a name="lookup-activity-properties"></a>查找活动属性

有关属性的详细信息，请参阅[查找活动](control-flow-lookup-activity.md)。

## <a name="next-steps"></a>后续步骤

有关数据工厂中复制活动支持用作源和接收器的数据存储的列表，请参阅[支持的数据存储和格式](copy-activity-overview.md#supported-data-stores-and-formats)。