---
title: 注册和扫描 Azure Data Lake Storage (ADLS) Gen2
description: 本教程介绍了如何扫描 Azure Data Lake Storage Gen2。
author: shsandeep123
ms.author: sandeepshah
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/17/2020
ms.openlocfilehash: 0232682661387e635e6ce8bfb5fc6e03301d130c
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/19/2020
ms.locfileid: "97694809"
---
# <a name="register-and-scan-azure-data-lake-storage-gen2"></a>注册和扫描 Azure Data Lake Storage Gen2

本文概述了如何在 Azure 监控范围中注册 Azure Data Lake Storage Gen2 作为数据源，并设置扫描。

## <a name="supported-capabilities"></a>支持的功能

Azure Data Lake Storage Gen2 数据源支持以下功能：

- 用于捕获 Azure Data Lake Storage Gen2 中的元数据和分类的 **完全和增量扫描**

- ADF 复制/数据流活动的数据资产之间的 **沿袭**

## <a name="prerequisites"></a>先决条件

在注册数据源之前，请创建一个 Azure 监控范围帐户。 有关创建监控范围帐户的详细信息，请参阅 [快速入门：创建 Azure 监控范围帐户](create-catalog-portal.md)。

### <a name="setting-up-authentication-for-a-scan"></a>为扫描设置身份验证

Azure Data Lake Storage Gen2 支持以下身份验证方法：

- 托管标识
- 服务主体
- 帐户密钥

#### <a name="managed-identity-recommended"></a>建议 (托管标识) 

当你选择 " **托管标识**" 时，若要设置连接，必须首先向监控范围帐户授予扫描数据源的权限：

1. 导航到 ADLS Gen2 存储帐户。
1. 在左侧导航栏菜单中，选择“访问控制 (IAM)”。 
1. 选择“+ 添加”。
1. 将 **角色** 设置为 " **存储 Blob 数据读取器** "，然后在 " **选择** 输入框" 下输入 Azure 监控范围帐户名称。 然后，选择“保存”，将此角色分配给 Purview 帐户。

> [!Note]
> 有关更多详细信息，请参阅[使用 Azure Active Directory 授予对 blob 和队列的访问权限](https://docs.microsoft.com/azure/storage/common/storage-auth-aad)中的步骤

#### <a name="account-key"></a>帐户密钥

如果选择了 "身份验证方法"，则需要获取访问 **密钥，并** 将其存储在密钥保管库中：

1. 导航到 ADLS Gne2 存储帐户
1. 选择 **设置 > 访问密钥**
1. 复制 *密钥* 并将其保存到某个位置，以执行后续步骤
1. 导航到你的密钥保管库
1. 选择“设置”>“机密”
1. 选择 " **+ 生成/导入**"，并输入存储帐户中的 *密钥***名称** 和 **值**
1. 选择“创建”以完成
1. 如果密钥保管库尚未连接到 Purview，则需要[创建新的密钥保管库连接](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account)
1. 最后，使用密钥 [创建新凭据](manage-credentials.md#create-a-new-credential) 以设置扫描

#### <a name="service-principal"></a>服务主体

若要使用服务主体，可以使用现有的服务主体，也可以创建一个新的服务主体。 

> [!Note]
> 如果必须创建新的服务主体，请执行以下步骤：
> 1. 导航到 [Azure 门户](https://portal.azure.com)。
> 1. 从左侧菜单中选择“Azure Active Directory”。
> 1. 选择 **“应用注册”**。
> 1. 选择“+ 新建应用程序注册”。
> 1. 为应用程序输入名称（服务主体名称）。
> 1. 选择“仅此组织目录中的帐户”。
> 1. 对于重定向 URI，选择“Web”并输入所需的任何 URL；它无需是真实的 URL 或有效的 URL。
> 1. 然后选择“注册”。

需要获取服务主体的应用程序 ID 和机密：

1. 导航到 [Azure 门户](https://portal.azure.com)中的服务主体
1. 复制“概述”中的“应用程序(客户端) ID”和“证书和机密”中的“客户端机密”的值   。
1. 导航到你的密钥保管库
1. 选择“设置”>“机密”
1. 选择“+ 生成/导入”并输入所选的“名称”和“值”作为服务主体的“客户端机密”   
1. 选择“创建”以完成
1. 如果密钥保管库尚未连接到 Purview，则需要[创建新的密钥保管库连接](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account)
1. 最后，使用服务主体[创建新凭据](manage-credentials.md#create-a-new-credential)以设置扫描

##### <a name="granting-the-service-principal-access-to-your-adls-gen2-account"></a>授予服务主体对 ADLS gen2 帐户的访问权限

1. 导航到自己的存储帐户。
1. 在左侧导航栏菜单中，选择“访问控制 (IAM)”。 
1. 选择“+ 添加”。
1. 将 **角色** 设置为 " **存储 Blob 数据读取器** "，然后在 " **选择** 输入框" 下输入服务主体名称或对象 ID。 然后，选择 " **保存** "，将此角色分配给服务主体。
### <a name="firewall-settings"></a>防火墙设置

> [!NOTE]
> 如果为存储帐户启用了防火墙，则必须在设置扫描时使用 **托管标识** 身份验证方法。

1. 进入[Azure 门户](https://portal.azure.com)中的 ASLS Gen2 存储帐户
1. 导航到 " **设置" > 网络** 和
1. **从 "允许访问**" 下选择 **所选网络**
1. 在 "**例外**" 部分中，选择 "**允许受信任的 Microsoft 服务访问此存储帐户**" 并单击 "**保存**"

:::image type="content" source="./media/register-scan-adls-gen2/firewall-setting.png" alt-text="显示防火墙设置的屏幕截图":::

## <a name="register-azure-data-lake-storage-gen2-data-source"></a>注册 Azure Data Lake Storage Gen2 数据源

若要在数据目录中注册新的 ADLS Gen2 帐户，请执行以下操作：

1. 导航到你的 Purview 帐户
2. 在左侧导航区域中选择“源”
3. 选择“注册”
4. 在 " **注册源**" 中，选择 **Azure Data Lake Storage Gen2**
5. 选择“继续”

在 " **注册源" (Azure Data Lake Storage Gen2)** "屏幕上，执行以下操作：

1. 输入数据源将在目录中列出的名称。
2. 选择订阅以筛选存储帐户
3. 选择存储帐户
4. 选择一个集合或创建新集合（可选）
5. 选择“完成”以注册数据源。

:::image type="content" source="media/register-scan-adls-gen2/register-sources.png" alt-text="注册源选项" border="true":::

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>后续步骤

- [浏览 Azure Purview 数据目录](how-to-browse-catalog.md)
- [搜索 Azure Purview 数据目录](how-to-search-catalog.md)
