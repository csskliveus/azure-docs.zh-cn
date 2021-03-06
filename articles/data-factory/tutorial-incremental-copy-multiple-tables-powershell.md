---
title: 使用 PowerShell 以增量方式复制多个表
description: 在本教程中，你将创建一个带管道的 Azure 数据工厂，该管道将增量数据从 SQL Server 数据库中的多个表加载到 Azure SQL 数据库。
services: data-factory
ms.author: yexu
author: dearandyxu
manager: anandsub
ms.reviewer: douglasl, maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 06/10/2020
ms.openlocfilehash: 61a4119947b1412d3e874458e06748fd40a381b4
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510254"
---
# <a name="incrementally-load-data-from-multiple-tables-in-sql-server-to-azure-sql-database-using-powershell"></a>使用 PowerShell 以递增方式将数据从 SQL Server 中的多个表加载到 Azure SQL 数据库

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

在本教程中，你将创建一个带管道的 Azure 数据工厂，该管道将增量数据从 SQL Server 数据库中的多个表加载到 Azure SQL 数据库。    

在本教程中执行以下步骤：

> [!div class="checklist"]
> * 准备源和目标数据存储。
> * 创建数据工厂。
> * 创建自我托管的集成运行时。
> * 安装 Integration Runtime。 
> * 创建链接服务。 
> * 创建源、接收器和水印数据集。
> * 创建、运行和监视管道。
> * 查看结果。
> * 在源表中添加或更新数据。
> * 重新运行和监视管道。
> * 查看最终结果。

## <a name="overview"></a>概述
下面是创建此解决方案所要执行的重要步骤： 

1. **选择水印列**。

    为源数据存储中的每个表选择一个列，你可以在其中标识每次运行的新记录或更新记录。 通常，在创建或更新行时，此选定列中的数据（例如 last_modify_time 或 ID）会不断递增。 此列中的最大值用作水印。

2. **准备用于存储水印值的数据存储**。

    本教程在 SQL 数据库中存储水印值。

3. **创建包含以下活动的管道**：
    
    a. 创建一个 ForEach 活动，循环访问一个列表，其中的源表名称是作为参数传递到管道的。 对于每个源表，它会调用以下活动，为该表执行增量加载。

    b. 创建两个 Lookup 活动。 使用第一个 Lookup 活动检索上一个水印值。 使用第二个 Lookup 活动检索新的水印值。 这些水印值会传递到 Copy 活动。

    c. 创建 Copy 活动，用于复制源数据存储中其水印列值大于旧水印值但小于新水印值的行。 然后，该活动将源数据存储中的增量数据作为新文件复制到 Azure Blob 存储。

    d. 创建 StoredProcedure 活动，用于更新下一次运行的管道的水印值。 

    下面是高级解决方案示意图： 

    ![以增量方式加载数据](media/tutorial-incremental-copy-multiple-tables-powershell/high-level-solution-diagram.png)


如果没有 Azure 订阅，请在开始之前创建一个[免费](https://azure.microsoft.com/free/)帐户。

## <a name="prerequisites"></a>先决条件

* **SQL Server**。 在本教程中，请将 SQL Server 数据库用作源数据存储。 
* **Azure SQL 数据库**。 使用 Azure SQL 数据库中的数据库作为接收器数据存储。 如果没有 SQL 数据库，请参阅[在 Azure SQL 数据库中创建数据库](../azure-sql/database/single-database-create-quickstart.md)了解创建步骤。 

### <a name="create-source-tables-in-your-sql-server-database"></a>在 SQL Server 数据库中创建源表

1. 打开 [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) 或 [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio)，然后连接到 SQL Server 数据库。

2. 在 **服务器资源管理器 (SSMS)** 或“连接”窗格 (Azure Data Studio) 中，右键单击数据库，然后选择“新建查询”。

3. 对数据库运行以下 SQL 命令，以便创建名为 `customer_table` 和 `project_table` 的表：

    ```sql
    create table customer_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );
    
    create table project_table
    (
        Project varchar(255),
        Creationtime datetime
    );
        
    INSERT INTO customer_table
    (PersonID, Name, LastModifytime)
    VALUES
    (1, 'John','9/1/2017 12:56:00 AM'),
    (2, 'Mike','9/2/2017 5:23:00 AM'),
    (3, 'Alice','9/3/2017 2:36:00 AM'),
    (4, 'Andy','9/4/2017 3:21:00 AM'),
    (5, 'Anny','9/5/2017 8:06:00 AM');
    
    INSERT INTO project_table
    (Project, Creationtime)
    VALUES
    ('project1','1/1/2015 0:00:00 AM'),
    ('project2','2/2/2016 1:23:00 AM'),
    ('project3','3/4/2017 5:16:00 AM');
    ```

### <a name="create-destination-tables-in-your-azure-sql-database"></a>在 Azure SQL 数据库中创建目标表

1. 打开 [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) 或 [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio)，然后连接到 SQL Server 数据库。

2. 在 **服务器资源管理器 (SSMS)** 或“连接”窗格 (Azure Data Studio) 中，右键单击数据库，然后选择“新建查询”。

3. 对数据库运行以下 SQL 命令，以便创建名为 `customer_table` 和 `project_table` 的表：  

    ```sql
    create table customer_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );
    
    create table project_table
    (
        Project varchar(255),
        Creationtime datetime
    );
    ```

### <a name="create-another-table-in-azure-sql-database-to-store-the-high-watermark-value"></a>在 Azure SQL 数据库中再创建一个表，用于存储高水印值

1. 针对数据库运行以下 SQL 命令，创建一个名为 `watermarktable` 的表来存储水印值： 
    
    ```sql
    create table watermarktable
    (
    
        TableName varchar(255),
        WatermarkValue datetime,
    );
    ```
2. 将两个源表的初始水印值插入水印表中。

    ```sql

    INSERT INTO watermarktable
    VALUES 
    ('customer_table','1/1/2010 12:00:00 AM'),
    ('project_table','1/1/2010 12:00:00 AM');
    
    ```

### <a name="create-a-stored-procedure-in-the-azure-sql-database"></a>在 Azure SQL 数据库中创建存储过程 

运行以下命令，在数据库中创建存储过程。 此存储过程在每次管道运行后更新水印值。 

```sql
CREATE PROCEDURE usp_write_watermark @LastModifiedtime datetime, @TableName varchar(50)
AS

BEGIN

UPDATE watermarktable
SET [WatermarkValue] = @LastModifiedtime 
WHERE [TableName] = @TableName

END

```

### <a name="create-data-types-and-additional-stored-procedures-in-azure-sql-database"></a>在 Azure SQL 数据库中创建数据类型和其他存储过程

运行以下查询，在数据库中创建两个存储过程和两个数据类型。 以便将源表中的数据合并到目标表中。 

为了方便入门，我们直接使用这些存储过程通过表变量来传入增量数据，然后将其合并到目标存储中。 请注意，不能将大量的增量行（超过 100 行）存储在表变量中。  

如果确实需要将大量增量行合并到目标存储中，则建议你先使用复制活动将所有增量数据复制到目标存储的某个临时“暂存”表中，然后在不使用表变量的情况下生成你自己的存储过程，以便将它们从“暂存”表合并到“最终”表中。 


```sql
CREATE TYPE DataTypeforCustomerTable AS TABLE(
    PersonID int,
    Name varchar(255),
    LastModifytime datetime
);

GO

CREATE PROCEDURE usp_upsert_customer_table @customer_table DataTypeforCustomerTable READONLY
AS

BEGIN
  MERGE customer_table AS target
  USING @customer_table AS source
  ON (target.PersonID = source.PersonID)
  WHEN MATCHED THEN
      UPDATE SET Name = source.Name,LastModifytime = source.LastModifytime
  WHEN NOT MATCHED THEN
      INSERT (PersonID, Name, LastModifytime)
      VALUES (source.PersonID, source.Name, source.LastModifytime);
END

GO

CREATE TYPE DataTypeforProjectTable AS TABLE(
    Project varchar(255),
    Creationtime datetime
);

GO

CREATE PROCEDURE usp_upsert_project_table @project_table DataTypeforProjectTable READONLY
AS

BEGIN
  MERGE project_table AS target
  USING @project_table AS source
  ON (target.Project = source.Project)
  WHEN MATCHED THEN
      UPDATE SET Creationtime = source.Creationtime
  WHEN NOT MATCHED THEN
      INSERT (Project, Creationtime)
      VALUES (source.Project, source.Creationtime);
END
```

### <a name="azure-powershell"></a>Azure PowerShell

按[安装和配置 Azure PowerShell](/powershell/azure/azurerm/install-azurerm-ps) 中的说明安装最新的 Azure PowerShell 模块。

## <a name="create-a-data-factory"></a>创建数据工厂

1. 为资源组名称定义一个变量，稍后会在 PowerShell 命令中使用该变量。 将以下命令文本复制到 PowerShell，在双引号中指定 [Azure 资源组](../azure-resource-manager/management/overview.md)的名称，然后运行命令。 示例为 `"adfrg"`。

    ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    如果该资源组已存在，请勿覆盖它。 为 `$resourceGroupName` 变量分配另一个值，然后再次运行命令

2. 定义一个用于数据工厂位置的变量。 

    ```powershell
    $location = "East US"
    ```
3. 若要创建 Azure 资源组，请运行以下命令： 

    ```powershell
    New-AzResourceGroup $resourceGroupName $location
    ``` 
    如果该资源组已存在，请勿覆盖它。 为 `$resourceGroupName` 变量分配另一个值，然后再次运行命令

4. 定义一个用于数据工厂名称的变量。 

    > [!IMPORTANT]
    >  更新数据工厂名称，使之全局唯一。 例如，ADFIncMultiCopyTutorialFactorySP1127。 

    ```powershell
    $dataFactoryName = "ADFIncMultiCopyTutorialFactory";
    ```
5. 要创建数据工厂，请运行以下 **Set-AzDataFactoryV2** cmdlet： 
    
    ```powershell
    Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location $location -Name $dataFactoryName 
    ```

请注意以下几点：

* 数据工厂的名称必须全局唯一。 如果收到以下错误，请更改名称并重试：

    ```powershell
    Set-AzDataFactoryV2 : HTTP Status Code: Conflict
    Error Code: DataFactoryNameInUse
    Error Message: The specified resource name 'ADFIncMultiCopyTutorialFactory' is already in use. Resource names must be globally unique.
    ```

* 若要创建数据工厂实例，用于登录到 Azure 的用户帐户必须属于参与者或所有者角色，或者是 Azure 订阅的管理员。

* 若要查看目前提供数据工厂的 Azure 区域的列表，请在以下页面上选择感兴趣的区域，然后展开“分析”以找到“数据工厂”：[可用产品(按区域)](https://azure.microsoft.com/global-infrastructure/services/)。 数据工厂使用的数据存储（Azure 存储、SQL 数据库、SQL 托管实例等）和计算资源（Azure HDInsight 等）可位于其他区域中。

[!INCLUDE [data-factory-create-install-integration-runtime](../../includes/data-factory-create-install-integration-runtime.md)]

## <a name="create-linked-services"></a>创建链接服务

可在数据工厂中创建链接服务，将数据存储和计算服务链接到数据工厂。 在本部分中，你将创建 SQL Server 数据库和 Azure SQL 数据库中数据库的链接服务。 

### <a name="create-the-sql-server-linked-service"></a>创建 SQL Server 链接服务

在此步骤中，请将 SQL Server 数据库链接到数据工厂。

1. 使用以下内容在 C:\ADFTutorials\IncCopyMultiTableTutorial 文件夹中创建一个名为 **SqlServerLinkedService.json** 的 JSON 文件（如果本地文件夹尚不存在，请创建它们）。 根据连接到 SQL Server 时所使用的身份验证选择适当的节。  

    > [!IMPORTANT]
    > 根据连接到 SQL Server 时所使用的身份验证选择适当的节。

    如果使用 SQL 身份验证，请复制以下 JSON 定义：

    ```json
    {  
        "name":"SqlServerLinkedService",
        "properties":{  
            "annotations":[  
    
            ],
            "type":"SqlServer",
            "typeProperties":{  
                "connectionString":"integrated security=False;data source=<servername>;initial catalog=<database name>;user id=<username>;Password=<password>"
            },
            "connectVia":{  
                "referenceName":"<integration runtime name>",
                "type":"IntegrationRuntimeReference"
            }
        }
    } 
   ```    
    如果使用 Windows 身份验证，请复制以下 JSON 定义：

    ```json
    {  
        "name":"SqlServerLinkedService",
        "properties":{  
            "annotations":[  
    
            ],
            "type":"SqlServer",
            "typeProperties":{  
                "connectionString":"integrated security=True;data source=<servername>;initial catalog=<database name>",
                "userName":"<username> or <domain>\\<username>",
                "password":{  
                    "type":"SecureString",
                    "value":"<password>"
                }
            },
            "connectVia":{  
                "referenceName":"<integration runtime name>",
                "type":"IntegrationRuntimeReference"
            }
        }
    }
    ```
    > [!IMPORTANT]
    > - 根据连接到 SQL Server 时所使用的身份验证选择适当的节。
    > - 将 &lt;integration runtime name> 替换为 Integration Runtime 的名称。
    > - 保存文件之前，请将 &lt;servername>、&lt;databasename>、&lt;username> 和 &lt;password> 替换为 SQL Server 数据库的值。
    > - 如需在用户帐户或服务器名称中使用斜杠字符 (`\`)，请使用转义字符 (`\`)。 示例为 `mydomain\\myuser`。

2. 在 PowerShell 中，运行以下 cmdlet 切换到 C:\ADFTutorials\IncCopyMultiTableTutorial 文件夹。

    ```powershell
    Set-Location 'C:\ADFTutorials\IncCopyMultiTableTutorial'
    ```

3. 运行 **Set-AzDataFactoryV2LinkedService** cmdlet 以创建链接服务 AzureStorageLinkedService。 在以下示例中，传递 *ResourceGroupName* 和 *DataFactoryName* 参数的值： 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SqlServerLinkedService" -File ".\SqlServerLinkedService.json"
    ```

    下面是示例输出：

    ```console
    LinkedServiceName : SqlServerLinkedService
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.SqlServerLinkedService
    ```

### <a name="create-the-sql-database-linked-service"></a>创建 SQL 数据库链接服务

1. 使用以下内容，在 C:\ADFTutorials\IncCopyMultiTableTutorial 文件夹中创建名为 **AzureSQLDatabaseLinkedService.json** 的 JSON 文件。 （如果文件夹 ADF 不存在，请创建。）保存文件之前，请将 &lt;servername&gt;、&lt;database name&gt;、&lt;user name&gt; 和 &lt;password&gt; 替换为 SQL Server 数据库名称、数据库名称、用户名和密码。 

    ```json
    {  
        "name":"AzureSQLDatabaseLinkedService",
        "properties":{  
            "annotations":[  
    
            ],
            "type":"AzureSqlDatabase",
            "typeProperties":{  
                "connectionString":"integrated security=False;encrypt=True;connection timeout=30;data source=<servername>.database.windows.net;initial catalog=<database name>;user id=<user name>;Password=<password>;"
            }
        }
    }
    ```
2. 在 PowerShell 中运行 **Set-AzDataFactoryV2LinkedService** cmdlet，以便创建链接服务 AzureSQLDatabaseLinkedService。 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSQLDatabaseLinkedService" -File ".\AzureSQLDatabaseLinkedService.json"
    ```

    下面是示例输出：

    ```console
    LinkedServiceName : AzureSQLDatabaseLinkedService
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDatabaseLinkedService
    ```

## <a name="create-datasets"></a>创建数据集

在此步骤中，请创建多个数据集，分别表示数据源、数据目标以及用于存储水印的位置。

### <a name="create-a-source-dataset"></a>创建源数据集

1. 在同一文件夹中，创建包含以下内容的名为 **SourceDataset.json** 的 JSON 文件： 

    ```json
    {  
        "name":"SourceDataset",
        "properties":{  
            "linkedServiceName":{  
                "referenceName":"SqlServerLinkedService",
                "type":"LinkedServiceReference"
            },
            "annotations":[  
    
            ],
            "type":"SqlServerTable",
            "schema":[  
    
            ]
        }
    }
   
    ```

    管道中的 Copy 活动使用 SQL 查询来加载数据，而不是加载整个表。

2. 运行 **Set-AzDataFactoryV2Dataset** cmdlet 以创建数据集 SourceDataset。
    
    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SourceDataset" -File ".\SourceDataset.json"
    ```

    下面是该 cmdlet 的示例输出：
    
    ```json
    DatasetName       : SourceDataset
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.SqlServerTableDataset
    ```

### <a name="create-a-sink-dataset"></a>创建接收器数据集

1. 在同一文件夹中，创建包含以下内容的名为 **SinkDataset.json** 的 JSON 文件。 tableName 元素由管道在运行时动态设置。 管道中的 ForEach 活动循环访问一个包含表名的列表，每一次迭代都将表名传递到此数据集。 

    ```json
    {  
        "name":"SinkDataset",
        "properties":{  
            "linkedServiceName":{  
                "referenceName":"AzureSQLDatabaseLinkedService",
                "type":"LinkedServiceReference"
            },
            "parameters":{  
                "SinkTableName":{  
                    "type":"String"
                }
            },
            "annotations":[  
    
            ],
            "type":"AzureSqlTable",
            "typeProperties":{  
                "tableName":{  
                    "value":"@dataset().SinkTableName",
                    "type":"Expression"
                }
            }
        }
    }
    ```

2. 运行 **Set-AzDataFactoryV2Dataset** cmdlet 以创建数据集 SinkDataset。
    
    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SinkDataset" -File ".\SinkDataset.json"
    ```

    下面是该 cmdlet 的示例输出：
    
    ```json
    DatasetName       : SinkDataset
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

### <a name="create-a-dataset-for-a-watermark"></a>为水印创建数据集

在此步骤中，创建用于存储高水印值的数据集。 

1. 在同一文件夹中，创建包含以下内容的名为 **WatermarkDataset.json** 的 JSON 文件： 

    ```json
    {
        "name": " WatermarkDataset ",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": "watermarktable"
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }    
    ```
2. 运行 **Set-AzDataFactoryV2Dataset** cmdlet 以创建数据集 WatermarkDataset。
    
    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "WatermarkDataset" -File ".\WatermarkDataset.json"
    ```

    下面是该 cmdlet 的示例输出：
    
    ```json
    DatasetName       : WatermarkDataset
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset    
    ```

## <a name="create-a-pipeline"></a>创建管道

此管道使用表名列表作为参数。 **ForEach 活动** 循环访问包含表名的列表，并执行以下操作： 

1. 通过 **Lookup 活动** 检索旧的水印值（初始值或上次迭代中使用的值）。

2. 通过 **Lookup 活动** 检索新的水印值（源表中水印列的最大值）。

3. 通过 **Copy 活动** 将这两个水印值之间的数据从源数据库复制到目标数据库。

4. 通过 **StoredProcedure 活动** 更新旧水印值，以便在下一迭代的第一步使用该值。 

### <a name="create-the-pipeline"></a>创建管道

1. 在同一文件夹中，创建包含以下内容的名为 **IncrementalCopyPipeline.json** 的 JSON 文件： 

    ```json
    {  
        "name":"IncrementalCopyPipeline",
        "properties":{  
            "activities":[  
                {  
                    "name":"IterateSQLTables",
                    "type":"ForEach",
                    "dependsOn":[  
    
                    ],
                    "userProperties":[  
    
                    ],
                    "typeProperties":{  
                        "items":{  
                            "value":"@pipeline().parameters.tableList",
                            "type":"Expression"
                        },
                        "isSequential":false,
                        "activities":[  
                            {  
                                "name":"LookupOldWaterMarkActivity",
                                "type":"Lookup",
                                "dependsOn":[  
    
                                ],
                                "policy":{  
                                    "timeout":"7.00:00:00",
                                    "retry":0,
                                    "retryIntervalInSeconds":30,
                                    "secureOutput":false,
                                    "secureInput":false
                                },
                                "userProperties":[  
    
                                ],
                                "typeProperties":{  
                                    "source":{  
                                        "type":"AzureSqlSource",
                                        "sqlReaderQuery":{  
                                            "value":"select * from watermarktable where TableName  =  '@{item().TABLE_NAME}'",
                                            "type":"Expression"
                                        }
                                    },
                                    "dataset":{  
                                        "referenceName":"WatermarkDataset",
                                        "type":"DatasetReference"
                                    }
                                }
                            },
                            {  
                                "name":"LookupNewWaterMarkActivity",
                                "type":"Lookup",
                                "dependsOn":[  
    
                                ],
                                "policy":{  
                                    "timeout":"7.00:00:00",
                                    "retry":0,
                                    "retryIntervalInSeconds":30,
                                    "secureOutput":false,
                                    "secureInput":false
                                },
                                "userProperties":[  
    
                                ],
                                "typeProperties":{  
                                    "source":{  
                                        "type":"SqlServerSource",
                                        "sqlReaderQuery":{  
                                            "value":"select MAX(@{item().WaterMark_Column}) as NewWatermarkvalue from @{item().TABLE_NAME}",
                                            "type":"Expression"
                                        }
                                    },
                                    "dataset":{  
                                        "referenceName":"SourceDataset",
                                        "type":"DatasetReference"
                                    },
                                    "firstRowOnly":true
                                }
                            },
                            {  
                                "name":"IncrementalCopyActivity",
                                "type":"Copy",
                                "dependsOn":[  
                                    {  
                                        "activity":"LookupOldWaterMarkActivity",
                                        "dependencyConditions":[  
                                            "Succeeded"
                                        ]
                                    },
                                    {  
                                        "activity":"LookupNewWaterMarkActivity",
                                        "dependencyConditions":[  
                                            "Succeeded"
                                        ]
                                    }
                                ],
                                "policy":{  
                                    "timeout":"7.00:00:00",
                                    "retry":0,
                                    "retryIntervalInSeconds":30,
                                    "secureOutput":false,
                                    "secureInput":false
                                },
                                "userProperties":[  
    
                                ],
                                "typeProperties":{  
                                    "source":{  
                                        "type":"SqlServerSource",
                                        "sqlReaderQuery":{  
                                            "value":"select * from @{item().TABLE_NAME} where @{item().WaterMark_Column} > '@{activity('LookupOldWaterMarkActivity').output.firstRow.WatermarkValue}' and @{item().WaterMark_Column} <= '@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}'",
                                            "type":"Expression"
                                        }
                                    },
                                    "sink":{  
                                        "type":"AzureSqlSink",
                                        "sqlWriterStoredProcedureName":{  
                                            "value":"@{item().StoredProcedureNameForMergeOperation}",
                                            "type":"Expression"
                                        },
                                        "sqlWriterTableType":{  
                                            "value":"@{item().TableType}",
                                            "type":"Expression"
                                        },
                                        "storedProcedureTableTypeParameterName":{  
                                            "value":"@{item().TABLE_NAME}",
                                            "type":"Expression"
                                        },
                                        "disableMetricsCollection":false
                                    },
                                    "enableStaging":false
                                },
                                "inputs":[  
                                    {  
                                        "referenceName":"SourceDataset",
                                        "type":"DatasetReference"
                                    }
                                ],
                                "outputs":[  
                                    {  
                                        "referenceName":"SinkDataset",
                                        "type":"DatasetReference",
                                        "parameters":{  
                                            "SinkTableName":{  
                                                "value":"@{item().TABLE_NAME}",
                                                "type":"Expression"
                                            }
                                        }
                                    }
                                ]
                            },
                            {  
                                "name":"StoredProceduretoWriteWatermarkActivity",
                                "type":"SqlServerStoredProcedure",
                                "dependsOn":[  
                                    {  
                                        "activity":"IncrementalCopyActivity",
                                        "dependencyConditions":[  
                                            "Succeeded"
                                        ]
                                    }
                                ],
                                "policy":{  
                                    "timeout":"7.00:00:00",
                                    "retry":0,
                                    "retryIntervalInSeconds":30,
                                    "secureOutput":false,
                                    "secureInput":false
                                },
                                "userProperties":[  
    
                                ],
                                "typeProperties":{  
                                    "storedProcedureName":"[dbo].[usp_write_watermark]",
                                    "storedProcedureParameters":{  
                                        "LastModifiedtime":{  
                                            "value":{  
                                                "value":"@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}",
                                                "type":"Expression"
                                            },
                                            "type":"DateTime"
                                        },
                                        "TableName":{  
                                            "value":{  
                                                "value":"@{activity('LookupOldWaterMarkActivity').output.firstRow.TableName}",
                                                "type":"Expression"
                                            },
                                            "type":"String"
                                        }
                                    }
                                },
                                "linkedServiceName":{  
                                    "referenceName":"AzureSQLDatabaseLinkedService",
                                    "type":"LinkedServiceReference"
                                }
                            }
                        ]
                    }
                }
            ],
            "parameters":{  
                "tableList":{  
                    "type":"array"
                }
            },
            "annotations":[  
    
            ]
        }
    }
    ```
2. 运行 **Set-AzDataFactoryV2Pipeline** cmdlet 以创建管道 IncrementalCopyPipeline。
    
   ```powershell
   Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "IncrementalCopyPipeline" -File ".\IncrementalCopyPipeline.json"
   ``` 

   下面是示例输出： 

   ```console
    PipelineName      : IncrementalCopyPipeline
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Activities        : {IterateSQLTables}
    Parameters        : {[tableList, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
   ```
 
## <a name="run-the-pipeline"></a>运行管道

1. 在同一文件夹中创建包含以下内容的名为 **Parameters.json** 的参数文件：

    ```json
    {
        "tableList": 
        [
            {
                "TABLE_NAME": "customer_table",
                "WaterMark_Column": "LastModifytime",
                "TableType": "DataTypeforCustomerTable",
                "StoredProcedureNameForMergeOperation": "usp_upsert_customer_table"
            },
            {
                "TABLE_NAME": "project_table",
                "WaterMark_Column": "Creationtime",
                "TableType": "DataTypeforProjectTable",
                "StoredProcedureNameForMergeOperation": "usp_upsert_project_table"
            }
        ]
    }
    ```
2. 使用 **Invoke-AzDataFactoryV2Pipeline** cmdlet 运行管道 IncrementalCopyPipeline。 将占位符替换为自己的资源组和数据工厂名称。

    ```powershell
    $RunId = Invoke-AzDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroup $resourceGroupName -dataFactoryName $dataFactoryName -ParameterFile ".\Parameters.json"        
    ``` 

## <a name="monitor-the-pipeline"></a>监视管道

1. 登录 [Azure 门户](https://portal.azure.com)。

2. 选择“所有服务”，使用关键字“数据工厂”进行搜索，然后选择“数据工厂”。 

3. 在数据工厂列表中搜索你的数据工厂，然后选择它来打开“数据工厂”页。 

4. 在“数据工厂”页上，选择“创作和监视”以在单独的选项卡中启动 Azure 数据工厂。

5. 在“开始使用”页上，选择左侧的“监视”。 
![屏幕截图显示了 Azure 数据工厂的“开始使用”页。](media/doc-common-process/get-started-page-monitor-button.png)    

6. 可以看到所有管道运行及其状态。 请注意，在以下示例中，管道运行的状态为“成功”。 选择“参数”列中的链接即可查看传递至管道的参数。 如果出现错误，请查看“错误”列中的链接。

    ![屏幕截图显示了数据工厂的管道运行，包括你的管道。](media/tutorial-incremental-copy-multiple-tables-powershell/monitor-pipeline-runs-4.png)    
7. 在“操作”列中选择链接时，会看到管道的所有活动运行。 

8. 若要回到“管道运行”视图，请选择“所有管道运行” 。 

## <a name="review-the-results"></a>查看结果

在 SQL Server Management Studio 中对目标 SQL 数据库运行以下查询，验证数据是否已从源表复制到目标表： 

**查询** 
```sql
select * from customer_table
```

**输出**
```
===========================================
PersonID    Name    LastModifytime
===========================================
1           John    2017-09-01 00:56:00.000
2           Mike    2017-09-02 05:23:00.000
3           Alice   2017-09-03 02:36:00.000
4           Andy    2017-09-04 03:21:00.000
5           Anny    2017-09-05 08:06:00.000
```

**查询**

```sql
select * from project_table
```

**输出**

```
===================================
Project     Creationtime
===================================
project1    2015-01-01 00:00:00.000
project2    2016-02-02 01:23:00.000
project3    2017-03-04 05:16:00.000
```

**查询**

```sql
select * from watermarktable
```

**输出**

```
======================================
TableName       WatermarkValue
======================================
customer_table  2017-09-05 08:06:00.000
project_table   2017-03-04 05:16:00.000
```

请注意，已更新这两个表的水印值。 

## <a name="add-more-data-to-the-source-tables"></a>向源表中添加更多数据

对源 SQL Server 数据库运行以下查询，在 customer_table 中更新现有行。 将新行插入到 project_table 中。 

```sql
UPDATE customer_table
SET [LastModifytime] = '2017-09-08T00:00:00Z', [name]='NewName' where [PersonID] = 3

INSERT INTO project_table
(Project, Creationtime)
VALUES
('NewProject','10/1/2017 0:00:00 AM');
``` 

## <a name="rerun-the-pipeline"></a>重新运行管道

1. 现在，通过执行以下 PowerShell 命令重新运行管道：

    ```powershell
    $RunId = Invoke-AzDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroup $resourceGroupname -dataFactoryName $dataFactoryName -ParameterFile ".\Parameters.json"
    ```
2. 按照[监视管道](#monitor-the-pipeline)部分的说明监视管道运行。 当管道状态为“正在进行”时，可以在“操作”下看到另一操作链接，用于取消管道运行。  

3. 选择“刷新”对列表进行刷新，直到管道运行成功。 

4. 也可选择“操作”下的“查看活动运行”链接，查看与此管道运行相关联的所有活动运行。  

## <a name="review-the-final-results"></a>查看最终结果

在 SQL Server Management Studio 中对目标数据库运行以下查询，验证更新的/全新的数据是否已从源表复制到目标表。 

**查询** 
```sql
select * from customer_table
```

**输出**
```
===========================================
PersonID    Name    LastModifytime
===========================================
1           John    2017-09-01 00:56:00.000
2           Mike    2017-09-02 05:23:00.000
3           NewName 2017-09-08 00:00:00.000
4           Andy    2017-09-04 03:21:00.000
5           Anny    2017-09-05 08:06:00.000
```

请注意 **PersonID** 为 3 时对应的 **Name** 和 **LastModifytime** 的新值。 

**查询**

```sql
select * from project_table
```

**输出**

```
===================================
Project     Creationtime
===================================
project1    2015-01-01 00:00:00.000
project2    2016-02-02 01:23:00.000
project3    2017-03-04 05:16:00.000
NewProject  2017-10-01 00:00:00.000
```

请注意，已将 **NewProject** 条目添加到 project_table。 

**查询**

```sql
select * from watermarktable
```

**输出**

```
======================================
TableName       WatermarkValue
======================================
customer_table  2017-09-08 00:00:00.000
project_table   2017-10-01 00:00:00.000
```

请注意，已更新这两个表的水印值。

## <a name="next-steps"></a>后续步骤
已在本教程中执行了以下步骤： 

> [!div class="checklist"]
> * 准备源和目标数据存储。
> * 创建数据工厂。
> * 创建自承载 Integration Runtime (IR)。
> * 安装 Integration Runtime。
> * 创建链接服务。 
> * 创建源、接收器和水印数据集。
> * 创建、运行和监视管道。
> * 查看结果。
> * 在源表中添加或更新数据。
> * 重新运行和监视管道。
> * 查看最终结果。

请转到下一篇教程，了解如何在 Azure 上使用 Spark 群集转换数据：

> [!div class="nextstepaction"]
>[使用更改跟踪技术，以增量方式将 Azure SQL 数据库中的数据加载到 Azure Blob 存储](tutorial-incremental-copy-change-tracking-feature-powershell.md)
