---
title: 使用设备将数据传输到 Azure 的选项 | Microsoft Docs
description: 了解如何在 Data Box Edge、Azure 文件同步和 StorSimple 8000 系列之间选择将本地数据传输到 Azure 的正确设备。
services: storsimple
author: alkohli
ms.service: storsimple
ms.topic: article
ms.date: 04/01/2019
ms.author: alkohli
ms.openlocfilehash: f2de6dbba2548f0c281df35edf2883d0c00af667
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/03/2020
ms.locfileid: "96571907"
---
# <a name="compare-storsimple-with-azure-file-sync-and-data-box-edge-data-transfer-options"></a>StorSimple 与 Azure 文件同步和 Data Box Edge 数据传输选项的比较 

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]
 
本文档概述了用于将本地数据传输到 Azure 的选项，比较： Data Box Edge 与 Azure 文件同步与 StorSimple 8000 系列。

- **[Data Box Edge](../databox-online/azure-stack-edge-overview.md)** - Data Box Edge 是一种本地网络设备，可将数据移入和移出 Azure，它还包含支持 AI 的 Edge 计算，用于在上传期间预处理数据。 Data Box Gateway 是该设备的虚拟版本，具有相同的数据传输功能。
- **[Azure 文件同步](../storage/files/storage-sync-files-deployment-guide.md)** - 使用 Azure 文件同步可将组织的文件共享集中在 Azure 文件中，同时不会失本地文件服务器的灵活性、性能和兼容性。 Azure 文件同步可将 Windows Server 转换为 Azure 文件共享的快速缓存。 在 2018 年早期已宣布推出 Azure 文件同步的正式版。
- **[StorSimple](./storsimple-overview.md)** – StorSimple 是与 Azure 存储紧密集成的混合设备，可帮助企业在单个解决方案中整合主要存储的存储基础结构、数据保护、存档和灾难恢复。 在[此处](https://support.microsoft.com/lifecycle/search?alpha=Azure%20StorSimple%208000%20Series)可以找到 StorSimple 的产品生命周期。

## <a name="comparison-summary"></a>比较摘要

|                           |StorSimple 8000   |Azure 文件同步   |Data Box Edge           |
|---------------------------|----------------------------------------|-------------------------------|-----------------------------------------|
|概述     |分层混合存储和存档|支持云分层和多站点同步的常规文件服务器存储。  |用于预处理数据并通过网络将其发送到 Azure 的存储解决方案。        |
|**方案**    |文件服务器、存档、备份目标 |文件服务器、存档（多站点）   |数据传输、数据预处理，包括机器学习推断、IoT、存档    |
|**Edge 计算** |不可用 |不可用 |支持使用 Azure IoT Edge 运行容器    |
|**外形规格**  |物理设备   |在 Windows Server 上安装的代理 |物理设备   |
|**硬件**     |Microsoft 作为服务的一部分提供的物理设备 | 由客户提供 |Microsoft 作为服务的一部分提供的物理设备  |
|**数据格式**  |自定义格式   |文件         |Blob 或文件    |
|**协议支持** |iSCSI          |SMB、NFS    | SMB 或 NFS      |
|**定价**      |[StorSimple](https://azure.microsoft.com/pricing/details/storsimple/) |[Azure 文件同步](https://azure.microsoft.com/pricing/details/storage/files/)  |[Data Box Edge](https://azure.microsoft.com/pricing/details/storage/databox/edge/)  |

## <a name="next-steps"></a>后续步骤

- 了解 [Azure Data Box Edge](../databox-online/azure-stack-edge-overview.md) 和 [Azure Data Box Gateway](../databox-online/data-box-gateway-overview.md)
- 了解 [Azure 文件同步](../storage/files/storage-sync-files-deployment-guide.md)
