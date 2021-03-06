---
title: 授权 Microsoft&reg; 平滑流式处理客户端移植工具包
description: 了解如何为 Microsoft&reg; 平滑流式处理客户端移植工具包授权。
services: media-services
documentationcenter: ''
author: xpouyat
manager: femila
editor: ''
ms.assetid: e3b488e7-8428-4c10-a072-eb3af46c82ad
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/21/2019
ms.author: xpouyat
ms.openlocfilehash: eb88d2cf96cc8557f2b8c729554106530cf6f956
ms.sourcegitcommit: 484f510bbb093e9cfca694b56622b5860ca317f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/21/2021
ms.locfileid: "98629617"
---
# <a name="licensing-microsoftreg-smooth-streaming-client-porting-kit"></a>授权 Microsoft&reg; 平滑流式处理客户端移植工具包

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]
 
## <a name="overview"></a>概述
Microsoft 平滑流式处理客户端移植工具包（简称 **SSPK**）是经过优化的平滑流式处理客户端实现，可帮助嵌入式设备制造商、有线和移动运营商、内容服务提供商、手持设备制造商、独立软件供应商 (ISV) 和解决方案提供商创建产品和服务，用于流式传输平滑流式处理格式的自适应内容。 SSPK 是平滑流式处理客户端的与设备和平台无关的实现，许可接受方可将它移植到任何设备和平台。 

下面是一个高级体系结构。IIS 平滑流式处理移植工具包框架是 Microsoft 提供的平滑流式处理客户端实现，包含用于播放平滑流式处理内容的所有核心逻辑。 然后，特定设备或平台的合作伙伴可以通过实现相应的接口来移植该内容。 

![SSPK](./media/media-services-sspk/sspk-arch.png)

## <a name="description"></a>说明
经过授权的 SSPK 可以提供优异的商业价值。 SSPK 许可证为行业提供：

* 采用 C++ 的平滑流式处理移植工具包源代码 
  * 实现平滑流式处理客户端功能
  * 添加格式解析、启发、缓冲逻辑，等等
* 播放器应用程序 API 
  * 可与媒体播放器应用程序交互的编程接口
* 平台抽象层 (PAL) 接口 
  * 可与操作系统（线程、套接字）交互的编程接口
* 硬件抽象层 (HAL) 接口 
  * 可与硬件 A/V 解码器（解码、绘制）交互的编程接口
* 数字权限管理 (DRM) 接口 
  * 可通过 DRM 抽象层 (DAL) 处理 DRM 的编程接口
  * Microsoft PlayReady 移植工具包是单独发售的，但可通过此接口集成。 有关 Microsoft PlayReady 设备许可的详细信息，请单击 [此处](https://www.microsoft.com/playready/licensing/device_technology.mspx#pddipdl)。
* 实现示例 
  * 适用于 Linux 的 PAL 实现示例
  * 适用于 GStreamer 的 HAL 实现示例

## <a name="licensing-options"></a>许可选项
Microsoft 平滑流式处理客户端移植工具包根据两份不同的许可协议提供给受证人：一份用于开发平滑流式处理客户端中期产品，另一份用于将平滑流式处理客户端最终产品分发给最终用户。

* 需要使用源代码移植工具包开发中期产品的芯片组制造商、系统集成商或独立软件供应商 (ISV)，应该执行 Microsoft 平滑流式处理客户端移植工具包 **中期产品许可证**。
* 需要拥有对最终用户分发平滑流式处理客户端最终产品的权限的设备制造商或 ISV 应该执行 Microsoft 平滑流式处理客户端移植工具包 **最终产品许可证**。

### <a name="microsoft-smooth-streaming-client-porting-kit-interim-product-license"></a>Microsoft 平滑流式处理客户端移植工具包中期产品许可证
Microsoft 根据此许可证提供平滑流式处理客户端移植工具包和所需的知识产权，使客户能够开发平滑流式处理客户端中期产品并分发给其他可分发平滑流式处理客户端最终产品的平滑流式处理客户端移植工具包设备受证者。

#### <a name="fee-structure"></a>费用结构
支付 50,000 美元的一次性许可费即可访问平滑流式处理客户端移植工具包。 

### <a name="microsoft-smooth-streaming-client-porting-kit-final-product-license"></a>Microsoft 平滑流式处理客户端移植工具包最终产品许可证
Microsoft 根据此许可证提供全部所需的知识产权，以便从其他平滑流式处理客户端移植工具包受证者接收平滑流式处理客户端中期产品，并将公司自有品牌的平滑流式处理客户端最终产品分发给最终用户。

#### <a name="fee-structure"></a>费用结构
平滑流式处理客户端最终产品根据如下所述的特许权使用费模式提供：

* 交付的每个设备实现支付 0.10 美元
* 每年的特许权使用费上限为 50,000 美元
* 每年的前 10,000 个设备实现无需支付特许权使用费 

## <a name="licensing-procedure-and-sspk-access"></a>许可过程和 SSPK 访问权限
有关所有许可查询的信息，请发送电子邮件到 [sspkinfo@microsoft.com](mailto:sspkinfo@microsoft.com)。

已注册的中期受证者可以访问 SSPK 分发门户。

中期和最终 SSPK 受证者可以将技术问题提交到 [smoothpk@microsoft.com](mailto:smoothpk@microsoft.com)。

## <a name="microsoft-smooth-streaming-client-interim-product-agreement-licensees"></a>Microsoft 平滑流式处理客户端中期产品协议受证人

* Adroit Business Solutions, Inc
* Advanced Digital Broadcast SA
* AirTies Kablosuz Iletism Sanayive Dis Ticaret A.S.
* Albis Technologies Ltd.
* Alticast Corporation
* Amazon Digital Services, Inc.
* Arion Technology, Inc.
* AVC Multimedia Software Co., Ltd.
* Cavium, Inc.
* EchoStar Purchasing Corporation
* Enseo, Inc.
* Fluendo S.A.
* Guangzhou Shikun Electronics., Ltd.
* HANDAN BroadInfoCom Co., Ltd.
* Infomir GMBH
* Irdeto USA Inc.
* Liberty Global Services BV
* MediaTek Inc.
* MStar Co, Ltd
* Nintendo Co., Ltd.
* OpenTV, Inc.
* Saffron Digital Limited
* Sichuan Changhong Electric Co., Ltd
* SoftAtHome
* Sony Corporation
* Tatung Technology Inc.
* Top Victory Investments, Ltd.
* Vestel Elektronik Sanayi ve Ticaret A.S.
* VisualOn, Inc.
* ZTE Corporation

## <a name="microsoft-smooth-streaming-client-final-product-agreement-licensees"></a>Microsoft 平滑流式处理客户端最终产品协议受证人
* Advanced Digital Broadcast SA
* AirTies Kablosuz Iletism Sanayive Dis Ticaret A.S.
* AmTRAN Technology Co., Ltd 
* Arcadyan Technology Corporation
* Arcelik A.S
* Compal Electronics, Inc.
* EXPRESS LUCK TECHNOLOGY LIMITED
* Fluendo S.A.
* FUNAI ELECTRIC CO., LTD
* Hisense International Co., Ltd. 
* HKC Corporation Limited
* Hong Kong Konka Ltd
* Innolux Corporation
* Innopia Technologies, Inc
* Jinpin 电气公司有限公司. Zhuhai
* K-Tronics (Suzhou) Technology Co., Ltd. 
* Kaonmedia Co., Ltd.
* KDDI Corporation
* Mega Fame Electronics Co. Limited
* MIRC Electronics Limited
* MOKA INTERNATIONAL LIMITED
* Nintendo Co., Ltd.
* ONEPLUS ELECTRONICS (SHENZHEN) CO., LTD.
* Panasonic Corporation
* Qingdao Haier Optronics Co., Ltd.
* Shenzhen ATEKO PHOTO Electricity Co.,Ltd.
* Shenzhen Chuangwei-RGB Electronics Co.,Ltd.
* Shenzhen Jiuzhou Electric Co., Ltd
* Shenzhen KTC Technology Co., Ltd. 
* Shenzhen Maxmade Technology Co., Ltd
* Shenzhen MTC Co., Ltd
* Shenzhen Skyworth Digital Technology Co., Ltd
* Sichuan Changhong Electric Co., Ltd.
* Skardin Industrial Corp.
* Sky Deutschland Fernsehen GmbH & Co. KG
* SMARDTV GLOBAL SAS
* SoftAtHome
* Sony Corporation
* Technicolor Delivery Technologies, SAS
* Top Victory Investments, Ltd.
* UMC Poland sp. z .o.o.
* Vizio, Inc.
* ZTE Corporation

## <a name="media-services-learning-paths"></a>媒体服务学习路径
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供反馈
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

