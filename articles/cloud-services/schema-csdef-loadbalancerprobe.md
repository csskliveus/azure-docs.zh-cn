---
title: Azure 云服务 (经典) LoadBalancerProbe 架构 |Microsoft Docs
description: Customer 定义的 LoadBalancerProbe 是角色实例中的终结点的运行状况探测。 它与服务定义文件中的 web 角色或辅助角色相结合。
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 3dca519f7fb4523ce9d9267f7629c1177cc5e3b6
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/23/2021
ms.locfileid: "98739779"
---
# <a name="azure-cloud-services-classic-definition-loadbalancerprobe-schema"></a>Azure 云服务 (经典) 定义 LoadBalancerProbe 架构

> [!IMPORTANT]
> [Azure 云服务 (扩展支持) ](../cloud-services-extended-support/overview.md) 是适用于 Azure 云服务产品的新的基于 azure 资源管理器的部署模型。进行此更改后，基于 Azure Service Manager 的部署模型运行的 Azure 云服务已重命名为云服务 (经典) ，所有新部署应使用 [云服务 (扩展支持) ](../cloud-services-extended-support/overview.md)。

负载均衡器探测是由客户定义的，用于探测 UDP 终结点和角色实例中的终结点的运行状况。 `LoadBalancerProbe` 不是一个独立的元素；它在服务定义文件中与 Web 角色或辅助角色结合。 `LoadBalancerProbe` 可由多个角色使用。

服务定义文件的默认扩展名为 .csdef。

## <a name="the-function-of-a-load-balancer-probe"></a>负载均衡器探测的功能
Azure 负载均衡器负责将传入流量路由到角色实例。 负载均衡器通过定期探测每个实例来确定哪些实例可以接收流量，从而确定该实例的运行状况。 负载均衡器每分钟可以多次探测每个实例。 可通过两个不同的选项向负载均衡器提供实例运行状况 - 默认负载均衡器探测或自定义负载均衡器探测，这可通过在 .csdef 文件中定义 LoadBalancerProbe 来实现。

仅当实例处于就绪状态时（就像实例未处于“繁忙”、“正在回收”或“正在停止”等状态时），默认负载均衡器探测才利用虚拟机内部的来宾代理进行侦听，并以“HTTP 200 正常”作为响应。 如果来宾代理无法使用“HTTP 200 正常”响应，则 Azure 负载均衡器将实例标记为无响应，并停止向该实例发送流量。 Azure 负载均衡器继续 ping 实例，并且如果来宾代理使用 HTTP 200 响应，Azure 负载均衡器会再次向该实例发送流量。 使用 Web 角色时，网站代码通常在不受 Azure 结构或来宾代理监视的 w3wp.exe 中运行。这意味着，系统不会向来宾代理报告 w3wp.exe 中的失败（例如， HTTP 500 响应），并且负载均衡器不会知道将该实例退出轮转。

自定义负载均衡器探测会取代默认来宾代理探测，并允许用户创建自己的自定义逻辑来确定角色实例的运行状况。 负载均衡器可定期（默认值为每 15 秒一次）探测终结点，并且如果在超时期限内（默认值为 31 秒）使用 TCP ACK 或 HTTP 200 进行了响应，则认为该实例处于轮转状态。 如果想要实现自己的逻辑（例如，在实例占用的 CPU 超过 90% 时返回非 200 状态）以便从负载均衡器轮转中删除实例，这种方案可能很有用。 对于使用 w3wp.exe 的 Web 角色，这还意味着可自动监视网站，因为网站代码中的错误会向负载均衡器探测返回非 200 状态。 如果未在 .csdef 文件中定义 LoadBalancerProbe，则会使用默认的负载均衡器行为（如上文所述）。

如果使用自定义负载均衡器探测，需要确保相关逻辑考虑了 RoleEnvironment.OnStop 方法。 如果使用默认负载均衡器探测，实例会将在调用 OnStop 之前退出轮转，但自定义负载均衡器探测可以继续在 OnStop 事件期间返回 200 正常。 如果使用 OnStop 事件清除缓存，请停止服务，或以其他方式进行可能影响服务的运行时行为的更改，然后需要确保自定义负载均衡器探测逻辑从轮转中删除实例。

## <a name="basic-service-definition-schema-for-a-load-balancer-probe"></a>负载均衡器探测的基本服务定义架构
 包含负载均衡器探测的服务定义文件的基本格式如下。

```xml
<ServiceDefinition …>
   <LoadBalancerProbes>
      <LoadBalancerProbe name="<load-balancer-probe-name>" protocol="[http|tcp]" path="<uri-for-checking-health-status-of-vm>" port="<port-number>" intervalInSeconds="<interval-in-seconds>" timeoutInSeconds="<timeout-in-seconds>"/>
   </LoadBalancerProbes>
</ServiceDefinition>
```

## <a name="schema-elements"></a>架构元素
服务定义文件的 `LoadBalancerProbes` 元素包含以下元素：

- [LoadBalancerProbes 元素](#LoadBalancerProbes)
- [LoadBalancerProbe 元素](#LoadBalancerProbe)

##  <a name="loadbalancerprobes-element"></a><a name="LoadBalancerProbes"></a> LoadBalancerProbes 元素
`LoadBalancerProbes` 元素描述负载均衡器探测的内容集合。 此元素是 [LoadBalancerProbe 元素](#LoadBalancerProbe)的父元素。 

##  <a name="loadbalancerprobe-element"></a><a name="LoadBalancerProbe"></a> LoadBalancerProbe 元素
`LoadBalancerProbe` 元素可定义模型的运行状况探测。 可定义多个负载均衡器探测。 

下表描述 `LoadBalancerProbe` 元素的属性：

|Attribute|类型|说明|
| ------------------- | -------- | -----------------|
| `name`              | `string` | 必需。 负载均衡器探测的名称。 该名称必须是唯一的。|
| `protocol`          | `string` | 必需。 指定终结点的协议。 可能的值包括 `http` 或 `tcp`。 如果已指定 `tcp`，则需要接收到的 ACK 才可完成进行探测。 如果已指定 `http`，则需要来自指定 URI 的 200 正常响应才可成功完成探测。|
| `path`              | `string` | 用于从 VM 请求运行状况的 URI。 如果 `protocol` 设置为 `http`，则需要 `path`。 否则，不允许使用该属性。<br /><br /> 没有默认值。|
| `port`              | `integer` | 可选。 用于传达探测的端口。 这对任何终结点都是可选的，因为相同的端口之后会用于探测。 还可为探测配置其他端口。 可能的值介于 1 和 65535（含）之间。<br /><br /> 默认值由终结点设置。|
| `intervalInSeconds` | `integer` | 可选。 探测终结点的运行状况的间隔（以秒为单位）。 通常，该间隔略小于所分配超时期限（以秒为单位）的一半，以便在退出轮转前执行两次完整的探测。<br /><br /> 默认值为 15，最小值为 5。|
| `timeoutInSeconds`  | `integer` | 可选。 应用于探测的超时期限（以秒为单位），在此期限内没有响应就会停止向终结点传送后续流量。 该值允许终结点以比 Azure 中通常所用的时间（默认值）更短或更长的时间退出轮转。<br /><br /> 默认值为 31，最小值为 11。|

## <a name="see-also"></a>另请参阅
[云服务（经典）定义架构](schema-csdef-file.md)