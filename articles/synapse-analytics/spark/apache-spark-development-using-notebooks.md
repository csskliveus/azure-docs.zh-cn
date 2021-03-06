---
title: Synapse Studio 笔记本
description: 本文介绍如何创建和开发 Azure Synapse Studio 笔记本，以便进行数据准备和可视化。
services: synapse analytics
author: ruixinxu
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 10/19/2020
ms.author: ruxu
ms.reviewer: ''
ms.custom: devx-track-python
ms.openlocfilehash: 7bb0b9b4645be7e548fe116d9e9a2c92fae7dc96
ms.sourcegitcommit: aacbf77e4e40266e497b6073679642d97d110cda
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98120948"
---
# <a name="create-develop-and-maintain-synapse-studio-notebooks-in-azure-synapse-analytics"></a>在 Azure Synapse Analytics 中创建、开发和维护 Synapse Studio 笔记本

Synapse Studio 笔记本是一个 web 界面，用于创建包含实时代码、可视化效果和叙述性文本的文件。 笔记本是验证想法并使用快速试验从数据中获取见解的好地方。 笔记本还广泛用于数据准备、数据可视化、机器学习和其他大数据方案。

借助 Azure Synapse Studio 笔记本，可以：

* 无需设置即可开始工作。
* 利用内置企业安全功能确保数据安全。
* 针对 Spark 和 SQL，分析跨原始格式（CSV、txt、JSON 等）、已处理的文件格式（parquet、Delta Lake、ORC 等）以及 SQL 表格数据文件的数据。
* 利用增强的创作功能和内置的数据可视化功能提高工作效率。

本文介绍如何在 Azure Synapse Studio 中使用笔记本。

## <a name="preview-of-the-new-notebook-experience"></a>全新笔记本体验预览
Synapse 团队将新的笔记本组件引入 Synapse Studio，为 Microsoft 客户提供一致的笔记本体验，并最大限度地提高可发现性、生产力、共享和协作。 全新的笔记本体验已准备就绪，可供预览。 检查笔记本工具栏中的 " **预览功能** " 按钮，将其打开。 下表捕获了现有笔记本 (之间的功能比较，并将其称为 "经典笔记本" ) 新的预览。  

|Feature|传统笔记本|预览笔记本|
|--|--|--|
|% 运行| 不支持 | &#9745;|
|% 历史记录| 不支持 |&#9745;
|负载百分比| 不支持 |&#9745;|
|%% html| 不支持 |&#9745;|
|拖放以移动单元| 不支持 |&#9745;|
|永久性显示 ( # A1 输出|&#9745;| 不可用 |
|全部取消| &#9745;| 不可用|
|运行上述所有单元格|&#9745;| 不可用 |
|运行下面的所有单元格|&#9745;| 不可用 |
|通过工具栏按钮设置文本单元格的格式|&#9745;| 不可用 |
|撤消单元格操作| &#9745;| 不可用 |


## <a name="create-a-notebook"></a>创建笔记本

可通过两种方法创建笔记本。 可以从“对象资源管理器”创建新笔记本或将现有笔记本导入到 Azure Synapse 工作区。 Azure Synapse Studio 笔记本可识别标准 Jupyter Notebook IPYNB 文件。

![创建导入笔记本](./media/apache-spark-development-using-notebooks/synapse-create-import-notebook-2.png)

## <a name="develop-notebooks"></a>开发笔记本

笔记本由单元格组成，单元格是单独的代码块或可单独运行或作为组运行的文本块。

### <a name="add-a-cell"></a>添加单元格

有多种方法可向笔记本添加新单元格。

# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

1. 展开左上方“+ 单元格”按钮，然后选择“添加代码单元格”或“添加文本单元格”  。

    ![add-cell-with-cell-button](./media/apache-spark-development-using-notebooks/synapse-add-cell-1.png)

2. 将鼠标悬停在两个单元格之间的空白地方，然后选择“添加代码”或“添加文本” 。

    ![add-cell-between-space](./media/apache-spark-development-using-notebooks/synapse-add-cell-2.png)

3. 使用[命令模式下的快捷键](#shortcut-keys-under-command-mode)。 按 A 在当前单元格上方插入单元格。 按 B 在当前单元格下方插入单元格。


# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

1. 展开左上方的 " **+ 单元** " 按钮，然后选择 " **代码单元** " 或 " **Markdown 单元**"。
    ![添加-azure-笔记本-单元按钮](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-add-cell-1.png)
2. 选择单元开头的加号，然后选择 " **代码单元** " 或 " **Markdown 单元**"。

    ![添加-azure-笔记本-跨空间](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-add-cell-2.png)

3. [在命令模式下使用 Aznb 快捷键](#shortcut-keys-under-command-mode)。 按 A 在当前单元格上方插入单元格。 按 B 在当前单元格下方插入单元格。

---

### <a name="set-a-primary-language"></a>设置主要语言

Azure Synapse Studio 笔记本支持四种 Apache Spark 语言：

* pySpark (Python)
* Spark (Scala)
* SparkSQL
* .NET for Apache Spark (C#)

可以从顶部命令栏中的下拉列表为新添加的单元格设置主要语言。

   ![default-synapse-language](./media/apache-spark-development-using-notebooks/synapse-default-language.png)

### <a name="use-multiple-languages"></a>使用多种语言

通过在单元格开始位置指定正确的语言 magic 命令，可以在一个笔记本中使用多种语言。 下表列出了用于切换单元格语言的 magic 命令。

|magic 命令 |语言 | 说明 |  
|---|------|-----|
|%%pyspark| Python | 针对 Spark 上下文执行 Python 查询。  |
|%%spark| Scala | 针对 Spark 上下文执行 Scala 查询。  |  
|%%sql| SparkSQL | 针对 Spark 上下文执行 SparkSQL 查询。  |
|%%csharp | .NET for Spark C# | 针对 Spark 上下文执行 .NET for Spark C# 查询。 |

下图是一个示例，说明如何在 Spark(Scala) 笔记本中使用 %%pyspark magic 命令编写 PySpark 查询，或使用 %%sql magic 命令编写 SparkSQL 查询  。 请注意，笔记本的主要语言设置为 pySpark。

   ![Synapse spark 幻命令](./media/apache-spark-development-using-notebooks/synapse-spark-magics.png)

### <a name="use-temp-tables-to-reference-data-across-languages"></a>使用临时表跨语言引用数据

不能直接在 Synapse Studio 笔记本中跨不同语言引用数据或变量。 在 Spark 中，可以跨语言引用临时表。 下面是一个示例，说明如何使用 Spark 临时表作为解决方法，在 `PySpark` 和 `SparkSQL` 中读取 `Scala` 数据帧。

1. 在单元格1中，使用 Scala 从 SQL 池连接器读取数据帧，并创建一个临时表。

   ```scala
   %%scala
   val scalaDataFrame = spark.read.sqlanalytics("mySQLPoolDatabase.dbo.mySQLPoolTable")
   scalaDataFrame.createOrReplaceTempView( "mydataframetable" )
   ```

2. 在单元格 2 中，使用 Spark SQL 查询数据。
   
   ```sql
   %%sql
   SELECT * FROM mydataframetable
   ```

3. 在单元格 3 中，使用 PySpark 中的数据。

   ```python
   %%pyspark
   myNewPythonDataFrame = spark.sql("SELECT * FROM mydataframetable")
   ```

### <a name="ide-style-intellisense"></a>IDE 样式 IntelliSense

Azure Synapse Studio 笔记本集成了 Monaco 编辑器，将 IDE 样式的 IntelliSense 引入到了单元格编辑器中。 语法突出显示、错误标记和自动代码完成有助于您编写代码并更快地确定问题。

对于不同的语言，IntelliSense 功能处于不同的成熟度级别。 使用下表来查看受支持的内容。

|Languages| 语法突出显示 | 语法错误标记  | 语法代码补全 | 变量代码补全| 系统函数代码补全| 用户函数代码补全| 智能缩进 | 代码折叠|
|--|--|--|--|--|--|--|--|--|
|PySpark (Python)|是|是|是|是|是|是|是|是|
|Spark (Scala)|是|是|是|是|-|-|-|是|
|SparkSQL|是|是|-|-|-|-|-|-|
|.NET for Spark (C#)|是|-|-|-|-|-|-|-|

### <a name="format-text-cell-with-toolbar-buttons"></a>通过工具栏按钮设置文本单元格的格式

# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

可以使用文本单元格工具栏中的格式按钮执行常见的 markdown 操作。 它包括将文本设为粗体、将文本设为斜体、插入代码片段、插入未排序列表、插入已排序列表以及插入 URL 中的图像。

  ![Synapse 文本单元工具栏](./media/apache-spark-development-using-notebooks/synapse-text-cell-toolbar.png)

# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

"格式" 按钮工具栏尚不可用于预览笔记本体验。 

---

### <a name="undo-cell-operations"></a>撤消单元格操作

# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

选择 " **撤消** " 按钮或按 **Ctrl + Z** 撤销最近的单元格操作。 现在可以撤销最近的 20 个历史单元格操作。 

   ![Synapse 撤消单元](./media/apache-spark-development-using-notebooks/synapse-undo-cells.png)
# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

撤消单元操作尚不可用于预览笔记本体验。 

---

### <a name="move-a-cell"></a>移动单元格

# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

选择省略号 (…) 以访问最右侧的其他单元格操作菜单。 然后选择“上移单元格”或“下移单元格”移动当前单元格 。 

还可使用[命令模式下的快捷键](#shortcut-keys-under-command-mode)。 按 Ctrl+Alt+↑，上移当前单元格。 按 Ctrl+Alt+↓，下移当前单元格。

   ![move-a-cell](./media/apache-spark-development-using-notebooks/synapse-move-cells.png)

# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

单击单元格的左侧，然后将其拖到所需的位置。 
    ![Synapse 移动单元](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-drag-drop-cell.gif)

---

### <a name="delete-a-cell"></a>删除单元格

# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

要删除单元格，请选择省略号 (…) 以访问最右侧的其他单元格操作菜单，然后选择“删除单元格”。 

还可使用[命令模式下的快捷键](#shortcut-keys-under-command-mode)。 按 D,D，删除当前单元格。
  
   ![delete-a-cell](./media/apache-spark-development-using-notebooks/synapse-delete-cell.png)

# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

若要删除某个单元，请选择该单元格右侧的 "删除" 按钮。 

还可使用[命令模式下的快捷键](#shortcut-keys-under-command-mode)。 按 **Shift + D** 删除当前单元格。 

   ![azure-笔记本-删除单元](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-delete-cell.png)

---

### <a name="collapse-a-cell-input"></a>折叠单元格输入

# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

选择当前单元格底部的箭头按钮以将其折叠。 若要展开它，请在折叠单元格时选择箭头按钮。

   ![collapse-cell-input](./media/apache-spark-development-using-notebooks/synapse-collapse-cell-input.gif)

# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

在单元工具栏上选择 " **更多命令** " 和 " ( ..." ) ，并 **输入** "折叠当前单元格的输入"。 若要展开该单元格，请选择隐藏单元格时 **隐藏的输入** 。

   ![azure-笔记本-折叠-单元-输入](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-collapse-cell-input.gif)

---

### <a name="collapse-a-cell-output"></a>折叠单元格输出

# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

选择当前单元格输出的左上角的 " **折叠输出** " 按钮，以将其折叠。 若要展开它，请选择 "在单元格输出处于折叠状态时 **显示单元格输出** "。

   ![collapse-cell-output](./media/apache-spark-development-using-notebooks/synapse-collapse-cell-output.gif)

# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

在单元工具栏上选择 " **更多命令** " "省略号 ( ..." ) ，并选择 " **输出** " 以折叠当前单元格的输出。 若要展开它，请在隐藏单元格的输出时选择相同的按钮。

   ![azure-笔记本折叠单元-输出](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-collapse-cell-output.gif)


---

## <a name="run-notebooks"></a>运行笔记本

你可以逐个或同时在笔记本中运行代码单元格。 每个单元格的状态和进度都在笔记本中表示。

### <a name="run-a-cell"></a>运行单元格

可以通过多种方法在单元格中运行代码。

1. 将鼠标悬停在要运行的单元格上，并选择“运行单元格”按钮，或按 Ctrl+Enter 。

   ![run-cell-1](./media/apache-spark-development-using-notebooks/synapse-run-cell.png)
  
2. 使用[命令模式下的快捷键](#shortcut-keys-under-command-mode)。 按 Shift+Enter 运行当前单元格并选择下面的单元格。 按 Alt+Enter 运行当前单元格并在下面插入一个新单元格。

---

### <a name="run-all-cells"></a>运行所有单元格
选择 " **全部运行** " 按钮以按顺序运行当前笔记本中的所有单元格。

   ![run-all-cells](./media/apache-spark-development-using-notebooks/synapse-run-all.png)


# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

### <a name="run-all-cells-above-or-below"></a>运行上方或下方的所有单元格

要访问最右侧的其他单元格操作菜单，请选择省略号 (…)。然后选择“运行上方的单元格”，按顺序运行当前单元格上方的所有单元格。 选择“运行下方的单元格”，按顺序运行当前单元格下方的所有单元格。

   ![run-cells-above-or-below](./media/apache-spark-development-using-notebooks/synapse-run-cells-above-or-below.png)


### <a name="cancel-all-running-cells"></a>取消所有正在运行的单元
选择 " **全部取消** " 按钮可取消正在运行的单元格或在队列中等待的单元格。 
   ![全部取消单元](./media/apache-spark-development-using-notebooks/synapse-cancel-all.png) 

# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

取消所有正在运行的单元格对于预览笔记本体验尚未提供。 

---



### <a name="reference-notebook"></a>参考笔记本

# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

不支持。

# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

你可以使用 ```%run <notebook path>``` 魔棒命令在当前笔记本的上下文中引用另一个笔记本。 参考笔记本中定义的所有变量都在当前笔记本中可用。 ```%run``` 幻命令支持嵌套调用，但不支持递归调用。 如果语句深度大于五，则将收到异常。 ```%run``` 命令目前仅支持将笔记本路径作为参数传递。 

---


### <a name="cell-status-indicator"></a>单元格状态指示器

逐步单元格执行状态显示在单元格下方，有助于你查看其当前进度。 单元格运行完成后，将显示具有总持续时间和结束时间的执行摘要，并将其保留在此处供将来参考。

![cell-status](./media/apache-spark-development-using-notebooks/synapse-cell-status.png)

### <a name="spark-progress-indicator"></a>Spark 进度指示器

Azure Synapse Studio 笔记本仅基于 Spark。 代码单元远程在无服务器 Apache Spark 池上执行。 Spark 作业进度指示器提供有实时进度栏，可帮助你了解作业执行状态。
每个作业或阶段的任务数有助于识别 spark 作业的并行级别。 你还可以通过选择作业 (或阶段) 名称上的链接，深入了解特定作业 (或阶段) 的 Spark UI。


![spark-progress-indicator](./media/apache-spark-development-using-notebooks/synapse-spark-progress-indicator.png)

### <a name="spark-session-config"></a>Spark 会话配置

可以在“配置会话”中为当前 Spark 会话指定超时持续时间、数量和执行器的大小。 重启 Spark 会话，以使配置更改生效。 缓存的所有笔记本变量都将被清除。

[![会话管理](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-spark-session-management.png)](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-spark-session-management.png#lightbox)

#### <a name="spark-session-config-magic-command"></a>Spark 会话配置幻命令
还可以通过神奇命令 **%% configure** 指定 spark 会话设置。 Spark 会话需要重新启动才能使设置生效。 建议你在笔记本开头运行 **%% 配置** 。 下面是一个示例， https://github.com/cloudera/livy#request-body 有关有效参数的完整列表，请参阅 

```
%%configure -f
{
    to config the session.
    "driverMemory":"2g",
    "driverCores":3,
    "executorMemory":"2g",
    "executorCores":2,
    "jars":["myjar1.jar","myjar.jar"],
    "conf":{
        "spark.driver.maxResultSize":"10g"
    }
}
```


## <a name="bring-data-to-a-notebook"></a>将数据引入笔记本

如以下代码示例所示，可以从 Azure Blob 存储、Azure Data Lake Store Gen 2 和 SQL 池加载数据。

### <a name="read-a-csv-from-azure-data-lake-store-gen2-as-a-spark-dataframe"></a>从 Azure Data Lake Store Gen2 读取 CSV 作为 Spark 数据帧

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
account_name = "Your account name"
container_name = "Your container name"
relative_path = "Your path"
adls_path = 'abfss://%s@%s.dfs.core.windows.net/%s' % (container_name, account_name, relative_path)

df1 = spark.read.option('header', 'true') \
                .option('delimiter', ',') \
                .csv(adls_path + '/Testfile.csv')

```

#### <a name="read-a-csv-from-azure-blob-storage-as-a-spark-dataframe"></a>从 Azure Blob 存储读取 CSV 作为 Spark 数据帧

```python

from pyspark.sql import SparkSession

# Azure storage access info
blob_account_name = 'Your account name' # replace with your blob name
blob_container_name = 'Your container name' # replace with your container name
blob_relative_path = 'Your path' # replace with your relative folder path
linked_service_name = 'Your linked service name' # replace with your linked service name

blob_sas_token = mssparkutils.credentials.getConnectionStringOrCreds(linked_service_name)

# Allow SPARK to access from Blob remotely

wasb_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)

spark.conf.set('fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name), blob_sas_token)
print('Remote blob path: ' + wasb_path)

df = spark.read.option("header", "true") \
            .option("delimiter","|") \
            .schema(schema) \
            .csv(wasbs_path)
```

### <a name="read-data-from-the-primary-storage-account"></a>从主存储帐户读取数据

你可以直接访问主存储帐户中的数据。 无需提供密钥。 在数据资源管理器中，右键单击某个文件，然后选择“新建笔记本”以查看自动生成了数据提取器的新笔记本。

![data-to-cell](./media/apache-spark-development-using-notebooks/synapse-data-to-cell.png)

## <a name="save-notebooks"></a>保存笔记本

可以在工作区中保存单个笔记本或所有笔记本。

1. 要保存对单个笔记本所做的更改，请在笔记本命令栏上选择“发布”按钮。

   ![publish-notebook](./media/apache-spark-development-using-notebooks/synapse-publish-notebook.png)

2. 要保存工作区中的所有笔记本，请在工作区命令栏上选择“发布所有”按钮。 

   ![publish-all](./media/apache-spark-development-using-notebooks/synapse-publish-all.png)

在笔记本属性中，可以配置保存时是否包括单元格输出。

   ![notebook-properties](./media/apache-spark-development-using-notebooks/synapse-notebook-properties.png)

## <a name="magic-commands"></a>magic 命令
可以在 Azure Synapse Studio 笔记本中使用熟悉的 Jupyter 幻命令。 查看以下列表作为当前可用的幻命令。 告诉我们 [GitHub 上的用例](https://github.com/MicrosoftDocs/azure-docs/issues/new) ，以便我们可以继续构建更复杂的命令来满足你的需求。

# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

可用行 magic：[%lsmagic](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-lsmagic)、[%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time)、[%timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit)

可用单元 magic： [%% time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time)， [%% timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit)， [%% capture](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-capture)， [%% writefile](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-writefile)， [%% sql](#use-multiple-languages)， [%% pyspark](#use-multiple-languages)， [%% spark](#use-multiple-languages)， [%% csharp](#use-multiple-languages)，[%% configure](#spark-session-config-magic-command)



# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

可用行 magic： [% lsmagic](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-lsmagic)， [% time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time)， [% timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit)， [% history](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-history)， [% run](#reference-notebook)， [% load](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-load)

可用单元 magic： [%% time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time)， [%% timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit)， [%% capture](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-capture)， [%% writefile](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-writefile)， [%% sql](#use-multiple-languages)， [%% pyspark](#use-multiple-languages)， [%% spark](#use-multiple-languages)， [%% csharp](#use-multiple-languages)， [%% html](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-html)， [%% 配置](#spark-session-config-magic-command)

--- 

## <a name="integrate-a-notebook"></a>集成笔记本

### <a name="add-a-notebook-to-a-pipeline"></a>向管道添加笔记本

选择右上角的 " **添加到管道** " 按钮，将笔记本添加到现有管道或创建新管道。

![将笔记本添加到管道](./media/apache-spark-development-using-notebooks/add-to-pipeline.png)

### <a name="designate-a-parameters-cell"></a>指定参数单元

# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

若要参数化笔记本，请选择省略号 ( ... ) 以访问最右侧的其他单元格操作菜单。 然后选择 " **切换参数单元** "，将该单元格指定为参数单元。

![切换-参数](./media/apache-spark-development-using-notebooks/toggle-parameter-cell.png)

# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

若要参数化笔记本，请选择省略号 ( ... ) ，以访问单元工具栏中的 **更多命令** 。 然后选择 " **切换参数单元** "，将该单元格指定为参数单元。

![azure-笔记本-切换-参数](./media/apache-spark-development-using-notebooks/azure-notebook-toggle-parameter-cell.png)

---

Azure 数据工厂将查找参数单元，并将此单元格视为执行时传入的参数的默认值。 执行引擎将使用输入参数在参数单元下面添加一个新的单元格，以覆盖默认值。 如果未指定参数单元，则插入的单元格将插入笔记本的顶部。


### <a name="assign-parameters-values-from-a-pipeline"></a>从管道分配参数值

创建带参数的笔记本后，可以使用 Azure Synapse 笔记本活动通过管道执行此操作。 将活动添加到管道画布后，你将能够在 "**设置**" 选项卡上的 "**基本参数**" 部分下设置参数值。 

![分配参数](./media/apache-spark-development-using-notebooks/assign-parameter.png)

分配参数值时，可以使用 [管道表达式语言](../../data-factory/control-flow-expression-language-functions.md) 或 [系统变量](../../data-factory/control-flow-system-variables.md)。



## <a name="shortcut-keys"></a>快捷键

与 Jupyter Notebooks 类似，Azure Synapse Studio 笔记本具有模式用户界面。 键盘根据笔记本单元格的具体模式执行不同的任务。 Synapse Studio 笔记本支持给定代码单元格的以下两种模式：命令模式和编辑模式。

1. 当没有文本光标提示你键入时，单元格处于命令模式。 当单元格处于命令模式时，可以将笔记本作为一个整体进行编辑，但不能键入单独的单元格。 通过按 `ESC` 或使用鼠标在单元格的编辑器区域外选择来进入命令模式。

   ![command-mode](./media/apache-spark-development-using-notebooks/synapse-command-mode-2.png)

2. 编辑模式由文本光标指示，提示你在编辑器区域中键入内容。 当单元格处于编辑模式时，您可以在单元格中键入。 按下 `Enter` 或使用鼠标在单元格的编辑器区域中进行选择，以进入编辑模式。
   
   ![edit-mode](./media/apache-spark-development-using-notebooks/synapse-edit-mode-2.png)

### <a name="shortcut-keys-under-command-mode"></a>命令模式下的快捷键

# <a name="classical-notebook"></a>[传统笔记本](#tab/classical)

使用以下击键快捷方式，你可以更轻松地在 Azure Synapse 笔记本中导航并运行代码。

| 操作 |Synapse Studio 笔记本快捷方式  |
|--|--|
|运行当前单元格并选择下方的单元格 | Shift+Enter |
|运行当前单元格并在下方插入 | Alt+Enter |
|选择上方的单元格| 向上 |
|选择下方的单元格| 向下 |
|在上方插入单元格| A |
|在下方插入单元格| B |
|展开上方的所选单元格| Shift+Up |
|展开下方的所选单元格| Shift+Down|
|上移单元格| Ctrl+Alt+↑ |
|下移单元格| Ctrl+Alt+↓ |
|删除所选单元格| D, D |
|切换到编辑模式| Enter |

# <a name="preview-notebook"></a>[预览笔记本](#tab/preview)

| 操作 |Synapse Studio 笔记本快捷方式  |
|--|--|
|运行当前单元格并选择下方的单元格 | Shift+Enter |
|运行当前单元格并在下方插入 | Alt+Enter |
|运行当前单元格| Ctrl+Enter |
|选择上方的单元格| 向上 |
|选择下方的单元格| 向下 |
|选择上一个单元格| K |
|选择下一个单元格| J |
|在上方插入单元格| A |
|在下方插入单元格| B |
|删除所选单元格| Shift + D |
|切换到编辑模式| Enter |

---

### <a name="shortcut-keys-under-edit-mode"></a>编辑模式下的快捷键


使用以下击键快捷方式，可以在编辑模式下更轻松地在 Azure Synapse 笔记本中导航并运行代码。

| 操作 |Synapse Studio 笔记本快捷方式  |
|--|--|
|上移光标 | 向上 |
|下移光标|向下|
|撤消|Ctrl+Z|
|重做|Ctrl+Y|
|注释/撤销注释|Ctrl+/|
|删除插入提示之前的字|Ctrl+Backspace|
|删除插入提示之后的字|Ctrl+Delete|
|转到单元格开头|Ctrl+Home|
|转到单元格末尾 |Ctrl+End|
|左移一个字|Ctrl+Left|
|右移一个字|Ctrl+Right|
|全选|Ctrl+A|
|缩进| Ctrl +]|
|取消缩进|Ctrl+[|
|切换到命令模式| Esc |

---

## <a name="next-steps"></a>后续步骤
- [查看 Synapse 示例笔记本](https://github.com/Azure-Samples/Synapse/tree/master/Notebooks)
- [快速入门：使用 Web 工具在 Azure Synapse Analytics 中创建 Apache Spark 池](../quickstart-apache-spark-notebook.md)
- [Azure Synapse Analytics 中的 Apache Spark 是什么](apache-spark-overview.md)
- [将 .NET for Apache Spark 与 Azure Synapse Analytics 配合使用](spark-dotnet.md)
- [.NET for Apache Spark 文档](/dotnet/spark?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Azure Synapse Analytics](../index.yml)