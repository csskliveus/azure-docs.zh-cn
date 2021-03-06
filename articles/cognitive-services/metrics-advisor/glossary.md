---
title: 指标顾问术语表
titleSuffix: Azure Cognitive Services
description: 指标 Advisor 服务的关键理念和概念
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: mbullwin
ms.openlocfilehash: 6c461983053a145dfda58b9e3d26b39db0c339e5
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/28/2020
ms.locfileid: "92893414"
---
# <a name="metrics-advisor-glossary-of-common-vocabulary-and-concepts"></a>常见词汇和概念的指标顾问术语表

本文档介绍了在指标顾问中使用的技术术语。 使用本文了解使用服务时可能会遇到的常见概念和对象。

## <a name="data-feed"></a>数据馈送

> [!NOTE]
> 多个度量值可以共享同一个数据源，甚至相同的数据馈送。

数据馈送是从数据源（例如 Cosmos DB 或 SQL server）引入的度量值。 数据馈送包含以下行：
* 时间戳
* 零个或多个维度
* 一个或多个度量值。 

## <a name="metric"></a>指标

指标是用于监视和评估特定业务流程的状态的可计量度量值。 它可以是划分为维度的多个时序值的组合。 例如， *web 运行状况* 指标可能包含 *用户计数* 和 *en-us 市场* 的维度。

## <a name="dimension"></a>维度

维度是一个或多个分类值。 这些值的组合标识特定的单变量时序，例如：国家/地区、语言、租户等。

## <a name="multi-dimensional-metric"></a>多维指标

什么是多维度指标？ 让我们使用两个示例。 

**业务收入**

假设你的业务收入有数据。 时序数据可能如下所示：

| 时间戳 | 类别 | 市场 | 收入 |
| ----------|----------|--------|----- |
| 2020-6-1 | 食物 | US | 1000 |
| 2020-6-1 | Apparel | US | 2000 |
| 2020-6-2 | 食物 | 英国 | 800 | 
| ...      | ...  |... | ... |

在此示例中，" *类别* " 和 " *市场* " 是维度。 *收入* 是 (KPI) 的关键绩效指标，可将其切分到不同的类别和/或市场，还可以进行聚合。 例如，所有市场的 *食物* 收入。
 
**复杂应用程序的错误计数**

假设你有一个应用程序中记录的错误数的数据。 时序数据可能如下所示：

| 时间戳 | 应用程序组件 | 区域 | 错误计数 |
| ----------|----------|--------|----- |
| 2020-6-1 | 员工数据库 | 欧盟西部 | 9000 |
| 2020-6-1 | 消息队列 | 美国东部 | 1000 |
| 2020-6-2 | 消息队列 | 美国东部 | 8000| 
| ...      | ...           | ...     |  ...|

在此示例中， *应用程序组件* 和 *区域* 是维度。 *错误计数* 是可切分到不同类别和/或市场的 KPI，也可以进行聚合。 例如，所有区域中的 *消息队列* 的错误计数。

## <a name="measure"></a>测量

度量值是基本或特定于单位的术语以及指标的可计量值。

## <a name="time-series"></a>时序

时序是一系列 (的数据点，或按时间顺序列出或绘制) 图表。 最常见的情况是，时间序列是在连续的、均匀间距的时间点上创建的序列。 这是一系列离散数据。

在指标顾问中，特定维度组合上的一个指标值称为一个序列。

## <a name="granularity"></a>粒度

粒度指示将在数据源中生成数据点的频率。 例如，每天，每小时。

## <a name="start-time"></a>开始时间

开始时间是希望指标 Advisor 开始从数据源引入数据的时间。 数据源必须具有指定开始时间的数据。

## <a name="confidence-boundaries"></a>置信度边界

> [!NOTE]
> 置信边界并不是唯一用于查找异常的度量值。 此边界之外的数据点可能会被检测模型标记为正常。 

在指标顾问中，置信度边界表示所使用算法的敏感度，并用于筛选出过于敏感的异常。 在 web 门户上，置信度边界显示为透明的蓝色带区。 带区中的所有点都被视为普通点。

指标顾问提供的工具可用于调整使用的算法的敏感性。 有关详细信息，请参阅 [如何：配置度量值和微调检测配置](how-tos/configure-metrics.md) 。

![置信度边界](media/confidence-bounds.png)


## <a name="hook"></a>自已

利用指标顾问，您可以创建并订阅实时警报。 这些警报 [使用挂钩](how-tos/alerts.md)通过 internet 发送。

## <a name="anomaly-incident"></a>异常事件

检测配置应用到指标后，只要其中的任何序列出现异常，就会生成事件。 在大型数据集中，这可能是一项艰巨的工作，因此，指标顾问将某个指标内的异常系列分组到一个事件中。 该服务还将评估严重性，并提供用于 [诊断事件](how-tos/diagnose-incident.md)的工具。

### <a name="incident-tree"></a>事件树

在指标审查程序中，您可以对指标应用异常情况检测，然后，指标顾问会自动监视所有维度组合的所有时间序列。 每当检测到任何异常时，指标顾问会将异常聚合到事件。
事件发生后，指标顾问将提供一个事件树，其中包含一个产生异常的层次结构，并识别出影响最大的事件树。 每个事件都具有根本原因异常，这是树的顶级节点。

### <a name="anomaly-grouping"></a>异常分组

指标顾问提供了查找具有类似模式的相关时序的功能。 它还可以更深入地了解其他维度的影响，并将异常关联起来。

### <a name="time-series-comparison"></a>时序比较

您可以选择多个时间序列来比较单个可视化效果中的趋势。 这为查看和比较相关的系列提供了一种清晰和深刻的方式。

## <a name="detection-configuration"></a>检测配置

>[!Note]
>检测配置仅适用于单个度量值。

在指标顾问 web 门户中，查看指标时，将在左侧面板上列出检测配置 (如敏感度、自动暂停和方向) 。 可对参数进行优化，并将其应用于此指标内的所有序列。

每个时序都需要检测配置，并确定时序中的某个点是否为异常。 首次载入数据时，指标顾问将为整个指标设置默认配置。 

您还可以通过对一组序列或特定的序列应用优化参数来进一步优化配置。 对于时序，只会应用一个配置：
* 应用于特定序列的配置将覆盖组的配置
* 组的配置将覆盖应用于整个指标的配置。

指标顾问提供多种 [检测方法](how-tos/configure-metrics.md#anomaly-detection-methods)，您可以使用逻辑运算符对它们进行组合。

### <a name="smart-detection"></a>智能检测

使用多机器学习算法的异常情况检测。

**敏感度** ：用于调整异常情况检测容错的数值。 直观地而言，值越大，时间序列周围的上边界和下边界越小。

### <a name="hard-threshold"></a>硬阈值

上限或下限范围内的值是异常。

**最小值** ：下限

**Max** ：上限

### <a name="change-threshold"></a>更改阈值

使用之前的点值确定这一点是否是异常的。

**更改百分比** ：与前一个点相比，如果更改的百分比超过此参数，则当前点为异常情况。

**更改点** ：要回顾多少点数。

### <a name="common-parameters"></a>通用参数

**方向** ：仅当偏差在 *向上* 、 *向下* 或 *同时* 出现时，点才是异常情况。

**异常之前** 无效：只有在以前的指定百分比也是异常情况时，数据点才是异常情况。

## <a name="alert-settings"></a>警报设置

警报设置确定哪些异常会触发警报。 可以用不同的设置设置多个警报。 例如，你可以为具有较低业务影响的异常创建警报，并为更重要的警报创建另一个警报。

你还可以跨指标创建警报。 例如，仅当两个指定的指标有异常时，才会触发警报。

### <a name="alert-scope"></a>警报范围

警报范围指警报适用的作用域。 下面有四个选择：

**所有序列的异常** ：对于指标中的所有序列中的异常，将触发警报。

**序列组中的异常** ：仅针对序列组的特定维度中的异常触发警报。 指定的维数应小于总数维度。

**常用系列中的异常** ：仅针对添加为收藏夹的异常触发警报。 可以为每个检测到的配置选择一组序列作为收藏夹。

**所有系列的前 n 个异常中的异常** ：仅针对前 n 个系列中的异常触发警报。 您可以设置参数以指定要考虑的时间戳的数量，以及必须在多长时间内发送警报。

### <a name="severity"></a>严重性

严重性是指指标顾问用来描述事件严重性的评分，其中包括 *高* 、 *中* 和 *低* 。

目前，指标顾问使用以下因素来度量警报严重性：
1. 度量值的值的比例和数量。
1. 异常的置信度。
1. 你最喜欢的设置也会影响严重性。

### <a name="auto-snooze"></a>自动暂停

某些异常是暂时性问题，特别是对于小型粒度指标。 您可以为特定数量的时间点 *暂停* 警报。 如果在指定的点数内发现异常，则不会触发警报。 可以在 "指标" 级别或 "序列" 级别设置 "自动暂停" 的行为。

可以在 "指标" 级别或 "序列" 级别设置 "暂停" 的行为。

## <a name="data-feed-settings"></a>数据馈送设置

### <a name="ingestion-time-offset"></a>引入时间偏移

默认情况下，根据粒度 (例如 *每日* ) ，数据是引入的。 通过使用正整数，可以按指定的值延迟数据引入。 使用负数，可以按指定的值提升引入量。

### <a name="max-ingestion-per-minute"></a>每分钟最多引入数

如果数据源支持有限并发，请设置此参数。 否则，请保留默认设置。

### <a name="stop-retry-after"></a>停止重试

如果数据引入失败，指标顾问将在一段时间后自动重试。 周期的开始时间是第一次数据引入的时间。 重试周期的长度根据粒度定义。 如果使用 () 的默认值 `-1` ，则将根据粒度确定重试周期：

| 粒度       | 停止重试           |
| :------------ | :--------------- |
| 每日，自定义 ( # B0 = 1 天) 、每周、每月、每年     | 7 天          |
| 每小时，自定义 ( # A0 1 天)        | 72小时 |

### <a name="min-retry-interval"></a>最小重试间隔

可以指定重试从源中提取数据时的最小间隔。 如果使用 () 的默认值 `-1` ，则将根据粒度确定重试间隔：

| 粒度       | 最小重试间隔           |
| :------------ | :--------------- |
| 每日，自定义 ( # B0 = 1 天) ，每周，每月     | 30 分钟          |
| 每小时，自定义 ( # A0 1 天)       | 10 分钟 |
| 每年 | 1 天          |

### <a name="grace-period"></a>宽限期

> [!Note]
> 宽限期从常规引入时间开始，加上指定的引入时间偏移量。
    
宽限期是指指标顾问将继续从数据源提取数据的一段时间，但不会激发任何警报。 如果在宽限期后未引入任何数据，将触发 " *数据馈送不可用* " 警报。

### <a name="snooze-alerts-in"></a>暂停警报

如果将此选项设置为零，则每个 " *不可* 用" 时间戳都将触发警报。 如果设置为非零的值，则在未提取任何数据的情况下，将暂停指定的 " *未使用* 的警报数"。

## <a name="data-feed-permissions"></a>数据馈送权限

有两个角色可用于管理数据馈送权限： *管理员* 和 *查看器* 。 

* *管理员* 对数据馈送和其中的指标进行完全控制。 它们可以激活、暂停、删除数据馈送并更新源和配置。 *管理员* 通常是度量值的所有者。

* *查看器* 可以查看数据馈送或指标，但无法进行更改。 

## <a name="next-steps"></a>后续步骤
- [指标顾问概述](overview.md)
- [使用 Web 门户](quickstarts/web-portal.md)