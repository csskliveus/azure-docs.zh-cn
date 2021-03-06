---
title: 适用于 SAP 应用程序的 Azure 邻近性组 |Microsoft Docs
description: 介绍 Azure 邻近布局组的 SAP 部署方案
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/29/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1e6aaf1b37073bf93e0aca8237161bf11af3a872
ms.sourcegitcommit: 42922af070f7edf3639a79b1a60565d90bb801c0
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/31/2020
ms.locfileid: "97827217"
---
# <a name="azure-proximity-placement-groups-for-optimal-network-latency-with-sap-applications"></a>适用于 SAP 应用程序的最佳网络延迟的 Azure 邻近性放置组
基于 SAP NetWeaver 或 SAP S/4HANA 体系结构的 SAP 应用程序对于 SAP 应用程序层和 SAP 数据库层之间的网络延迟很敏感。 此敏感度是指在应用程序层中运行的大多数业务逻辑的结果。 因为 SAP 应用程序层运行业务逻辑，所以它将以较高的频率向数据库层发出查询，每秒的速率为上千或数万。 在大多数情况下，这些查询的性质很简单。 它们通常可以在500微秒内或更短的时间内在数据库层上运行。

网络上花费的时间将此类查询从应用程序层发送到数据库层并接收结果集，这对运行业务流程所需的时间有重大影响。 网络延迟的这一敏感度是你可能希望在 SAP 部署项目中实现某些最大网络延迟的原因。 请参阅 [SAP 说明 #1100926-常见问题解答：网络性能](https://launchpad.support.sap.com/#/notes/1100926/E) 以获得有关如何对网络延迟分类的准则。

在许多 Azure 区域中，数据中心的数量越来越多。 同时，客户（尤其是对于高端 SAP 系统）使用 M 或 Mv2 系列的更多特别 VM Sku 或 HANA 大型实例。 Azure 虚拟机的所有数据中心都不会始终提供这些 Azure 虚拟机类型。 这些事实可以为优化 SAP 应用程序层和 SAP DBMS 层之间的网络延迟创造机会。

为了使你能够优化网络延迟，Azure 提供了 [邻近的放置组](../../linux/co-location.md)。 邻近位置组可用于强制将不同 VM 类型分组到单个 Azure 数据中心，以尽可能优化这些不同 VM 类型之间的网络延迟。 在将第一个 VM 部署到此类邻近布局组的过程中，VM 会绑定到特定的数据中心。 这种情况下，构造的使用也会引入一些限制：

- 你不能假定所有 Azure VM 类型在所有 Azure 数据中心都可用。 因此，可以限制一个邻近位置组内不同 VM 类型的组合。 出现这些限制的原因是，运行特定 VM 类型所需的主机硬件可能不存在于放置组所部署到的数据中心
- 当调整位于某个邻近布局组内的 Vm 的各个部分时，你无法自动假设，在所有情况下，新的 VM 类型都与作为邻近位置组一部分的其他 Vm 在同一数据中心内可用
- Azure add-on 硬件可能会将邻近组的某些 Vm 强制转换到另一个 Azure 数据中心。 有关涵盖这种情况的详细信息，请阅读文档 [归置资源以提高延迟](../../linux/co-location.md#planned-maintenance-and-proximity-placement-groups)  

> [!IMPORTANT]
> 由于潜在限制，应使用邻近位置组：
>
> - 仅在必要时
> - 仅限单个 SAP 系统的粒度，不适用于整个系统布局或完整的 SAP 环境
> - 通过一种方法，可将不同的 VM 类型和邻近位置组中的 Vm 数量保存到最小

假设你通过指定可用性区域并选择相同的可用性区域来部署 Vm，则这些 Vm 之间的网络延迟应该足以操作 SAP NetWeaver 和 S/4HANA 系统，并满足性能和吞吐量。 这一假设是独立于一个数据中心还是多个数据中心来构建某个特定区域，这种假设。 在区域性部署中使用邻近位置组的唯一原因是要将 Azure 可用性集部署的 Vm 与区域性部署的 Vm 一起分配。


## <a name="what-are-proximity-placement-groups"></a>什么是邻近性放置组？ 
Azure 邻近度布局组是一种逻辑构造。 定义邻近的位置组时，它将绑定到 Azure 区域和 Azure 资源组。 部署 Vm 时，将引用邻近的放置组：

- 数据中心内部署的第一个 Azure VM。 你可以将第一台虚拟机看作是基于 Azure 分配算法部署在数据中心中的 "作用域 VM"，这些算法最终与特定可用性区域的用户定义组合在一起。
- 部署了引用邻近位置组的所有后续 Vm，以将所有后续部署的 Azure Vm 放在第一个虚拟机所在的数据中心内。

> [!NOTE]
> 如果没有部署可在放置第一个 VM 的数据中心中运行特定 VM 类型的主机硬件，则所请求的 VM 类型的部署将失败。 你将收到失败消息。

单个 [Azure 资源组](../../../azure-resource-manager/management/manage-resources-portal.md) 可以有多个分配给它的邻近位置组。 但邻近的放置组只能分配给一个 Azure 资源组。


## <a name="proximity-placement-groups-with-sap-systems-that-use-only-azure-vms"></a>仅使用 Azure Vm 的 SAP 系统的邻近性放置组
Azure 上的大多数 SAP NetWeaver 和 S/4HANA 系统部署不使用 [HANA 大型实例](./hana-overview-architecture.md)。 对于不使用 HANA 大型实例的部署，必须在 SAP 应用程序层和 DBMS 层之间提供最佳性能。 为此，请仅为系统定义 Azure 邻近性放置组。

在大多数客户部署中，客户会为 SAP 系统生成单个 [Azure 资源组](../../../azure-resource-manager/management/manage-resources-portal.md) 。 在这种情况下，之间存在一对一的关系，例如，生产 ERP 系统资源组及其邻近位置组。 在其他情况下，客户会水平组织其资源组，并在单个资源组中收集所有生产系统。 在这种情况下，在生产 SAP 系统的资源组和生产 SAP ERP 的多个邻近性放置组之间建立了一对多关系，SAP BW 等。

避免将多个 SAP 生产系统或非生产系统捆绑到单个邻近位置组中。 当少量 SAP 系统或 SAP 系统以及某些应用程序需要较低的延迟网络通信时，可以考虑将这些系统移到一个邻近的放置组。 避免系统的捆绑包，因为更多的系统会在邻近的放置组中进行分组，所以可能性越高：

- 需要的 VM 类型不能在邻近位置组的作用域内的特定数据中心内运行。
- 由于在一段时间内将软件添加到邻近的放置组，因此，在需要更多的情况下，可能无法再使用非主流 Vm （如 M 系列 Vm）的资源。

如下所述，理想配置如下所示：

![仅包含 Azure Vm 的邻近度放置组](./media/sap-proximity-placement-scenarios/ppg-for-all-azure-vms.png)

在这种情况下，单个 SAP 系统每个资源组中分组，每个资源组一个邻近的放置组。 无论你使用 HANA 横向扩展配置还是 DBMS 扩展配置，都没有任何依赖关系。

## <a name="proximity-placement-groups-and-hana-large-instances"></a>邻近位置组和 HANA 大型实例
如果某些 SAP 系统依赖于应用程序层的 [HANA 大型实例](./hana-overview-architecture.md) ，则当你使用在 [修订版4行或 stamp](./hana-network-architecture.md#networking-architecture-for-hana-large-instance)中部署的 hana 大型实例单元时，你可以在 hana 大型实例单元和 Azure vm 之间显著改进网络延迟。 其中一项改进是，在部署时，HANA 大型实例单元使用邻近位置组进行部署。 你可以使用该邻近布局组来部署你的应用程序层虚拟机。 因此，这些 Vm 将部署在托管 HANA 大型实例单元的同一数据中心内。

若要确定 HANA 大型实例单元是否部署在修订版本4的 stamp 或行中，请 [通过 Azure 门户查看 AZURE HANA 大型实例](./hana-li-portal.md#look-at-attributes-of-single-hli-unit)的文章。 在 HANA 大型实例单元的特性概述中，还可以确定邻近位置组的名称，因为它是在部署 HANA 大型实例单元时创建的。 "属性概述" 中显示的名称是应该将应用程序层 Vm 部署到的邻近感应位置组的名称。

与仅使用 Azure 虚拟机的 SAP 系统相比，在使用 HANA 大型实例时，在决定要使用的 [Azure 资源组](../../../azure-resource-manager/management/manage-resources-portal.md) 数量上，灵活性更小。 [Hana 大型实例租户](./hana-know-terms.md)的所有 Hana 大型实例单元按[本文](./hana-li-portal.md#display-of-hana-large-instance-units-in-the-azure-portal)中的说明分组。 除非你将部署到不同的租户，例如生产系统和非生产系统或其他系统，否则，所有 HANA 大型实例单元都将部署在一个 HANA 大型实例租户中。 此租户与资源组具有一对一关系。 但将为每个单位定义单独的邻近位置组。

因此，单个租户的 Azure 资源组和邻近位置组之间的关系如下所示：

![邻近位置组和 HANA 大型实例](./media/sap-proximity-placement-scenarios/ppg-for-hana-large-instance-units.png)

## <a name="example-of-deployment-with-proximity-placement-groups"></a>具有邻近位置组的部署示例
下面是一些 PowerShell 命令，可用于在 Azure 邻近位置组中部署 Vm。

第一步，在登录到 [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/)后，将检查是否在要用于部署的 Azure 订阅中：

<pre><code>
Get-AzureRmContext
</code></pre>

如果需要更改为其他订阅，可以通过运行以下命令来执行此操作：

<pre><code>
Set-AzureRmContext -Subscription "my PPG test subscription"
</code></pre>

通过运行以下命令创建新的 Azure 资源组：

<pre><code>
New-AzResourceGroup -Name "myfirstppgexercise" -Location "westus2"
</code></pre>

通过运行以下命令创建新的邻近位置组：

<pre><code>
New-AzProximityPlacementGroup -ResourceGroupName "myfirstppgexercise" -Name "letsgetclose" -Location "westus2"
</code></pre>

使用如下命令将第一个 VM 部署到邻近位置组：

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppganchorvm" -Location "westus2" -OpenPorts 80,3389 -ProximityPlacementGroup "letsgetclose" -Size "Standard_DS11_v2"
</code></pre>

上述命令将部署基于 Windows 的 VM。 此 VM 部署成功后，将在 Azure 区域中定义邻近感应位置组的数据中心作用域。 引用邻近位置组的所有后续 VM 部署（如前面的命令中所示）将部署在同一 Azure 数据中心，前提是 VM 类型可以托管在该数据中心内的硬件上，并且该 VM 类型的容量可用。

## <a name="combine-availability-sets-and-availability-zones-with-proximity-placement-groups"></a>将可用性集和可用性区域与邻近组相结合
使用可用性区域 SAP 系统部署的一个缺点是，不能在特定区域中使用可用性集来部署 SAP 应用程序层。 你希望 SAP 应用程序层部署在 DBMS 层所在的同一区域中。 不支持在部署单个 VM 时引用可用性区域和可用性集。 因此，以前已强制通过引用区域来部署应用程序层。 你无法确保应用程序层 Vm 分散到不同的更新域和故障域中。

通过使用邻近位置组，可以绕过此限制。 下面是部署顺序：

- 创建邻近位置组。
- 通过引用可用性区域，部署定位点 VM，通常是 DBMS 服务器。
- 创建引用 Azure 邻近性组的可用性集。  (参见本文后面的命令。 ) 
- 通过引用可用性集和邻近度放置组来部署应用程序层 Vm。

在部署 VM 时，你将引用可用性区域和邻近位置组，而不是按上一部分所示部署第一个 VM：

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppganchorvm" -Location "westus2" -OpenPorts 80,3389 -Zone "1" -ProximityPlacementGroup "letsgetclose" -Size "Standard_E16_v3"
</code></pre>

成功部署此虚拟机将在一个可用性区域中托管 SAP 系统的数据库实例。 邻近位置组的作用域固定到表示所定义可用性区域的数据中心之一。

假设你以与 DBMS Vm 相同的方式部署中心服务 Vm，同时引用同一区域或区域和相同的邻近位置组。 在下一步中，需要创建要用于 SAP 系统的应用程序层的可用性集。

定义和创建邻近位置组。 用于创建可用性集的命令需要对邻近位置组 ID 的其他引用， (不是名称) 。 可以通过使用以下命令获取邻近组的 ID：

<pre><code>
Get-AzProximityPlacementGroup -ResourceGroupName "myfirstppgexercise" -Name "letsgetclose"
</code></pre>

创建可用性集时，如果使用托管磁盘 (默认值，则需要考虑其他参数，除非另外指定) 和邻近位置组：

<pre><code>
New-AzAvailabilitySet -ResourceGroupName "myfirstppgexercise" -Name "myppgavset" -Location "westus2" -ProximityPlacementGroupId "/subscriptions/my very long ppg id string" -sku "aligned" -PlatformUpdateDomainCount 3 -PlatformFaultDomainCount 2 
</code></pre>

理想情况下，应使用三个容错域。 但受支持的容错域的数量可能会因区域而异。 在这种情况下，特定区域可能有两个容错域的最大数量。 若要部署应用程序层 Vm，需要添加对可用性集名称和邻近组名称的引用，如下所示：

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppgavsetappvm" -Location "westus2" -OpenPorts 80,3389 -AvailabilitySetName "myppgavset" -ProximityPlacementGroup "letsgetclose" -Size "Standard_DS11_v2"
</code></pre>

此部署的结果是：
- SAP 系统的 DBMS 层和中心服务位于特定的可用性区域或可用性区域中。
- 一种 SAP 应用程序层，位于同一 Azure 数据中心内作为 DBMS VM 或 Vm 的可用性集。

> [!NOTE]
> 由于你将一个 DBMS VM 部署到一个区域，而第二个 DBMS VM 在另一个区域中，以创建高可用性配置，因此需要为每个区域提供不同的邻近位置组。 对于你使用的任何可用性集，情况也是如此。

## <a name="move-an-existing-system-into-proximity-placement-groups"></a>将现有系统移动到邻近的放置组
如果已部署 SAP 系统，则可能需要优化某些关键系统的网络延迟，并在同一数据中心内找到应用程序层和 DBMS 层。 若要将完整的 Azure 可用性集的 Vm 移动到已确定作用域的现有邻近位置组，需要关闭可用性集的所有 Vm，并通过 Azure 门户、PowerShell 或 CLI 将可用性集分配给现有的邻近性放置组。 如果要将不属于可用性集的 VM 移动到现有的邻近位置组，只需关闭 VM，并将其分配给现有的邻近布局组。 


## <a name="next-steps"></a>后续步骤
查看文档：

- [Azure 上的 SAP 工作负荷：规划和部署清单](./sap-deployment-checklist.md)
- [预览：使用 Azure CLI 将 Vm 部署到邻近位置组](../../linux/proximity-placement-groups.md)
- [预览：使用 PowerShell 将 Vm 部署到邻近位置组](../../windows/proximity-placement-groups.md)
- [适用于 SAP 工作负荷的 Azure 虚拟机 DBMS 部署的注意事项](./dbms_guide_general.md)