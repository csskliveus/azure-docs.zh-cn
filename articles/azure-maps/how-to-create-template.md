---
title: 使用 ARM 模板创建 Azure Maps 帐户 |Microsoft Azure 映射
description: 了解如何使用 Azure 资源管理器 (ARM) 模板创建 Azure Maps 帐户。
author: philmea
ms.author: philmea
ms.date: 10/20/2020
ms.topic: how-to
ms.service: azure-maps
ms.openlocfilehash: c715c0639e962f76f669515c1d2c826c8cf6cc9e
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2020
ms.locfileid: "92525057"
---
# <a name="create-your-azure-maps-account-using-an-arm-template"></a>使用 ARM 模板创建 Azure Maps 帐户

可以使用 Azure 资源管理器 (ARM) 模板来创建 Azure Maps 帐户。 获得帐户后，可以在你的网站或移动应用程序中实现 API。

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

如果你的环境满足先决条件，并且你熟悉如何使用 ARM 模板，请选择“部署到 Azure”按钮。 Azure 门户中会打开模板。

[![部署到 Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-maps-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

* 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速启动模板](https://azure.microsoft.com/resources/templates/101-maps-create/)。

:::code language="json" source="~/quickstart-templates/101-maps-create/azuredeploy.json":::

Azure Maps 帐户资源在此模板中定义：

* [**Microsoft. Maps/accounts**](/azure/templates/microsoft.maps/accounts)：创建 Azure Maps 帐户。

## <a name="deploy-the-template"></a>部署模板

1. 选择下图登录到 Azure 并打开一个模板。 模板创建 Azure Maps 帐户。

    [![部署到 Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-maps-create%2Fazuredeploy.json)

2. 选择或输入以下值。

    ![ARM 模板部署门户](./media/how-to-create-template/create-account-using-template-portal.png)

    除非指定，否则请使用默认值创建 Azure Maps 帐户。

    * 订阅：选择一个 Azure 订阅。
    * 资源组：选择“新建”，输入资源组的唯一名称，然后单击“确定”。 
    * 位置：选择一个位置。 例如 " **美国西部 2**"。
    * **帐户名称**：输入 Azure Maps 帐户的名称，该名称必须全局唯一。
    * **定价层**：选择适当的定价层，模板的默认值为 S0。

3. 选择“查看 + 创建”。 
4. 确认 "审阅" 页上的设置，然后单击 " **创建**"。 成功部署 Azure Maps 后，会收到一条通知：

    ![ARM 模板部署门户通知](./media/how-to-create-template/resource-manager-template-portal-deployment-notification.png)

Azure 门户用于部署模板。 还可以使用 Azure PowerShell、Azure CLI 和 REST API。 若要了解其他部署方法，请参阅[部署模板](../azure-resource-manager/templates/deploy-powershell.md)。

## <a name="review-deployed-resources"></a>查看已部署的资源

你可以使用 Azure 门户来检查 Azure Maps 帐户并查看你的密钥。 你还可以使用以下 Azure CLI 脚本来列出你的帐户密钥。

```azurecli-interactive
az maps account keys list --name MyMapsAccount --resource-group MyResourceGroup
```

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组，请删除该资源组，该资源组也会删除 Azure Maps 帐户。 使用 Azure CLI 删除资源组：

```azurecli-interactive
az group delete --name MyResourceGroup
```

## <a name="next-steps"></a>后续步骤

若要详细了解 Azure Maps 和 Azure 资源管理器，请继续阅读以下文章。

- 创建 Azure Maps [演示应用程序](quick-demo-map-app.md)
- 了解有关[ARM 模板](../azure-resource-manager/templates/overview.md)的详细信息