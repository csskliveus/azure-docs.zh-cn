---
title: 注册新设备-Azure IoT Edge |Microsoft Docs
description: 在 IoT 中心注册单个 IoT Edge 设备，以便使用对称密钥或 x.509 证书进行手动预配
author: kgremban
manager: philmea
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 10/06/2020
ms.author: kgremban
ms.openlocfilehash: 97faf5eff7187bbabe23bbcab60514eef4acc063
ms.sourcegitcommit: 484f510bbb093e9cfca694b56622b5860ca317f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/21/2021
ms.locfileid: "98633844"
---
# <a name="register-an-iot-edge-device-in-iot-hub"></a>在 IoT 中心注册 IoT Edge 设备

本文提供在 IoT 中心注册新 IoT Edge 设备的步骤。

连接到 IoT 中心的每个设备都有一个设备 ID，用于跟踪云到设备或设备到云的通信。 可以使用设备连接信息来配置设备，这些信息包括 IoT 中心主机名、设备 ID 以及设备用于向 IoT 中心进行身份验证的信息。

本文中的步骤逐步讲解称为 "手动设置" 的过程，在此过程中，会将单个设备连接到其 IoT 中心。 对于手动预配，可以通过两种方式来验证 IoT Edge 设备：

* **对称密钥**：在 IoT 中心创建新的设备标识时，该服务会创建两个密钥。 将其中一个密钥置于设备上，并在进行身份验证时将该密钥提供给 IoT 中心。

  此身份验证方法更容易上手，但不够安全。

* **X.509 自签名**：创建两个 X.509 标识证书并将其置于设备上。 在 IoT 中心创建新的设备标识时，需要提供两个证书的指纹。 当设备向 IoT 中心进行身份验证时，它将提供一个证书，IoT 中心将验证证书是否与其指纹相匹配。

  此身份验证方法更安全，建议用于生产场景。

本文介绍这两种身份验证方法。

如果有多个设备要设置，并且不想手动设置每个设备，请使用以下文章之一来了解 IoT Edge 如何与 IoT 中心设备预配服务一起使用：

* [使用 x.509 证书创建和预配 IoT Edge 设备](how-to-auto-provision-x509-certs.md)
* [使用 TPM 创建和预配 IoT Edge 设备](how-to-auto-provision-simulated-device-linux.md)
* [使用对称密钥创建和预配 IoT Edge 设备](how-to-auto-provision-symmetric-keys.md)

## <a name="prerequisites"></a>先决条件

# <a name="portal"></a>[Portal](#tab/azure-portal)

Azure 订阅中的免费或标准 [IoT 中心](../iot-hub/iot-hub-create-through-portal.md)。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Azure 订阅中的免费或标准 [IoT 中心](../iot-hub/iot-hub-create-through-portal.md)
* [Visual Studio Code](https://code.visualstudio.com/)
* 适用于 Visual Studio Code 的 [Azure IoT 工具](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

* Azure 订阅中的免费或标准 [IoT 中心](../iot-hub/iot-hub-create-using-cli.md)。
* 环境中的 [Azure CLI](/cli/azure/install-azure-cli)。 Azure CLI 版本必须至少是 2.0.70 或更新版本。 请使用 `az --version` 验证版本。 此版本支持 az 扩展命令，并引入了 Knack 命令框架。

---

## <a name="option-1-register-with-symmetric-keys"></a>选项1：注册对称密钥

可以根据自己的喜好使用若干工具在 IoT 中心注册新的 IoT Edge 设备并检索其连接字符串。

# <a name="portal"></a>[Portal](#tab/azure-portal)

在 Azure 门户的 IoT 中心，IoT Edge 设备与未启用边缘的 IoT 设备分开创建和管理。

1. 登录 [Azure 门户](https://portal.azure.com)，导航到 IoT 中心。

1. 在左侧窗格中，从菜单中选择“IoT Edge”，然后选择“添加 IoT Edge 设备” 。

   ![从 Azure 门户添加 IoT Edge 设备](./media/how-to-register-device/portal-add-iot-edge-device.png)

1. 在“创建设备”页面上，提供以下信息：

   * 创建描述性设备 ID。
   * 选择“对称密钥”作为身份验证类型。
   * 使用默认设置自动生成身份验证密钥并将新设备连接到中心。

1. 选择“保存”。 

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

### <a name="sign-in-to-access-your-iot-hub"></a>登录以访问 IoT 中心

可使用适用于 Visual Studio Code 的 Azure IoT 扩展来执行与 IoT 中心相关的操作。 要使这些操作正常工作，需要登录到 Azure 帐户，并选择中心。

1. 在 Visual Studio Code 中打开“资源管理器”视图。
1. 在资源管理器底部，展开“Azure IoT 中心”部分  。

   ![展开“Azure IoT 中心设备”部分](./media/how-to-register-device/azure-iot-hub-devices.png)

1. 单击“Azure IoT 中心”部分标题中的“...”   。 如果没有看到省略号，请单击标题或将鼠标指针悬停在标题上。
1. 选择“选择 IoT 中心”  。
1. 如果尚未登录到 Azure 帐户，请按照相关提示执行此操作。
1. 选择 Azure 订阅。
1. 选择 IoT 中心。

### <a name="register-a-new-device-with-visual-studio-code"></a>使用 Visual Studio Code 注册新设备

1. 在 Visual Studio Code 资源管理器中，展开 " **Azure IoT 中心** " 部分。
1. 单击“Azure IoT 中心”部分标题中的“...”   。 如果没有看到省略号，请单击标题或将鼠标指针悬停在标题上。
1. 选择“创建 IoT Edge 设备”  。
1. 在打开的文本框中提供设备 ID。

在输出屏幕中，可以看到命令的结果。 其中显示有设备信息，包括所提供的“deviceId”以及可用于将物理设备连接到 IoT 中心的“connectionString”   。

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

使用 [az iot hub device-identity create](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-create) 命令在 IoT 中心创建新设备标识。 例如：

   ```azurecli
   az iot hub device-identity create --device-id [device id] --hub-name [hub name] --edge-enabled
   ```

此命令包括三个参数：

* `--device-id` 或 `-d` ：提供 IoT 中心内唯一的描述性名称。
* `--hub-name` 或 `-n`：提供 IoT 中心的名称。
* `--edge-enabled` 或 `--ee`：声明该设备是 IoT Edge 设备。

   ![az iot hub device-identity create output](./media/how-to-register-device/create-edge-device-cli.png)

---

现在，已在 IoT 中心内注册了设备，接下来要检索用于完成 IoT Edge 运行时安装和设置的连接字符串。 按照本文后面的步骤 [查看已注册的设备并检索连接字符串](#view-registered-devices-and-retrieve-connection-strings)。

## <a name="option-2-register-with-x509-certificates"></a>选项2：注册到 x.509 证书

手动设置 x.509 证书需要 IoT Edge 版本1.0.10 或更高版本。

对于 x.509 证书身份验证，每个设备的身份验证信息以从设备标识证书获取的 *指纹* 形式提供。 在注册设备时，会向 IoT 中心提供这些指纹，以便服务在连接时能够识别设备。

### <a name="create-certificates-and-thumbprints"></a>创建证书和指纹

使用 x.509 证书预配 IoT Edge 设备时，将使用所谓的 *设备标识证书*。 此证书仅用于预配 IoT Edge 设备，并使用 Azure IoT 中心对设备进行身份验证。 它是不对其他证书进行签名的叶证书。 设备标识证书独立于证书颁发机构 (CA) 证书，IoT Edge 设备将其提供给模块或下游设备进行验证。 有关如何在 IoT Edge 设备中使用 CA 证书的详细信息，请参阅 [了解 Azure IoT Edge 使用证书的方式](iot-edge-certs.md)。

若要使用 X.509 进行手动预配，需要以下文件：

* 两个设备标识证书，它们在 .cer 或 pem 格式中具有匹配的私钥证书。

  其中一组证书/密钥文件提供给 IoT Edge 运行时。 创建设备标识证书时，请将证书公用名 (CN) 设置为你希望设备在 IoT 中心中具有的设备 ID。

* 从两个设备标识证书获取的指纹。

  SHA-1 哈希的指纹值为 40 个十六进制字符，SHA-256 哈希的指纹值为 64 个十六进制字符。 在注册设备时，两个指纹都将提供给 IoT 中心。

如果没有可用的证书，可以[创建演示证书以测试 IoT Edge 设备功能](how-to-create-test-certificates.md)。 按照该文章中的说明设置证书创建脚本、创建根 CA 证书，然后创建两个 IoT Edge 设备标识证书。

从证书中检索指纹的一种方法是使用以下 openssl 命令：

```cmd
openssl x509 -in <certificate filename>.pem -text -fingerprint
```

### <a name="register-a-new-device"></a>注册新设备

你可以使用多个工具在 IoT 中心注册新的 IoT Edge 设备，并上传其证书指纹。

# <a name="portal"></a>[Portal](#tab/azure-portal)

在 Azure 门户的 IoT 中心，IoT Edge 设备与未启用边缘的 IoT 设备分开创建和管理。

1. 登录 [Azure 门户](https://portal.azure.com)，导航到 IoT 中心。

1. 在左侧窗格中，从菜单中选择“IoT Edge”，然后选择“添加 IoT Edge 设备” 。

   ![从 Azure 门户添加 IoT Edge 设备](./media/how-to-register-device/portal-add-iot-edge-device.png)

1. 在“创建设备”页面上，提供以下信息：

   * 创建描述性设备 ID。 记下此设备 ID，因为在下一部分会用到它。
   * 选择“X.509 自签名”作为身份验证类型。
   * 提供主要和辅助标识证书指纹。 SHA-1 哈希的指纹值为 40 个十六进制字符，SHA-256 哈希的指纹值为 64 个十六进制字符。

1. 选择“保存”。 

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

目前，适用于 Visual Studio Code 的 Azure IoT 扩展不支持通过 x.509 证书进行设备注册。

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

使用 [az iot hub device-identity create](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-create) 命令在 IoT 中心创建新设备标识。 例如：

   ```azurecli
   az iot hub device-identity create --device-id [device id] --hub-name [hub name] --edge-enabled --auth-method x509_thumbprint --primary-thumbprint [SHA thumbprint] --secondary-thumbprint [SHA thumbprint]
   ```

此命令包含多个参数：

* `--device-id` 或 `-d`：提供对 IoT 中心唯一的描述性名称。 记下此设备 ID，因为在下一部分会用到它。
* `hub-name` 或 `-n`：提供 IoT 中心的名称。
* `--edge-enabled` 或 `--ee`：声明该设备是 IoT Edge 设备。
* `--auth-method` 或 `--am`：声明设备将要使用的授权类型。 在本例中，我们将使用 X.509 证书指纹。
* `--primary-thumbprint` 或 `--ptp`：提供一个 X.509 证书指纹用作主密钥。
* `--secondary-thumbprint` 或 `--stp`：提供一个 X.509 证书指纹用作辅助密钥。

---

现在，已在 IoT 中心内注册了设备，接下来可以在设备上安装和设置 IoT Edge 运行时。 IoT Edge 使用 x.509 证书进行身份验证的设备不使用连接字符串，因此你可以继续下一步：

* [安装或卸载适用于 Linux 的 Azure IoT Edge](how-to-install-iot-edge.md)
* [安装或卸载 Windows Azure IoT Edge](how-to-install-iot-edge-windows-on-windows.md)

## <a name="view-registered-devices-and-retrieve-connection-strings"></a>查看已注册的设备并检索连接字符串

使用对称密钥身份验证的设备需要其连接字符串才能完成 IoT Edge 运行时的安装和设置。

使用 x.509 证书身份验证的设备不需要连接字符串。 相反，这些设备需要其 IoT 中心名称、其设备名称及其证书文件才能完成 IoT Edge 运行时的安装和设置。

# <a name="portal"></a>[Portal](#tab/azure-portal)

所有连接到 IoT 中心并已启用 Edge 的设备都列在 **IoT Edge** 页上。

![使用 Azure 门户查看 IoT 中心内的所有 IoT Edge 设备](./media/how-to-register-device/portal-view-devices.png)

如果已准备好设置设备，则需要连接字符串，该字符串使用物理设备在 IoT 中心内的标识链接该设备。

使用对称密钥进行身份验证的设备具有其连接字符串，可以在门户中进行复制。

1. 在门户的 **IoT Edge** 页中，单击 IoT Edge 设备列表中的设备 ID。
2. 复制“主连接字符串”或“辅助连接字符串”的值   。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

### <a name="view-iot-edge-devices-with-visual-studio-code"></a>使用 Visual Studio Code 查看 IoT Edge 设备

Visual Studio Code 资源管理器的“Azure IoT 中心”部分中列出了连接到 IoT 中心的所有设备  。 IoT Edge 设备和非 Edge 设备可通过不同的图标加以区别。事实上， **$edgeAgent** 和 **$edgeHub** 模块会部署到每个 IoT Edge 设备。

![使用 VS Code 查看 IoT 中心内的所有 IoT Edge 设备](./media/how-to-register-device/view-devices.png)

### <a name="retrieve-the-connection-string-with-visual-studio-code"></a>通过 Visual Studio Code 检索连接字符串

如果已准备好设置设备，则需要连接字符串，该字符串使用物理设备在 IoT 中心内的标识链接该设备。

1. 右键单击“Azure IoT 中心”部分中的设备 ID  。
1. 选择“复制设备连接字符串”  。

   连接字符串会复制到剪贴板。

还可从右键菜单中选择“获取设备信息”，在输出窗口中查看包括连接字符串在内的所有设备信息  。

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

### <a name="view-iot-edge-devices-with-the-azure-cli"></a>通过 Azure CLI 查看 IoT Edge 设备

使用 [az iot hub device-identity list](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-list) 命令在 IoT 中心查看所有设备。 例如：

   ```azurecli
   az iot hub device-identity list --hub-name [hub name]
   ```

注册为 IoT Edge 设备的任何设备的 **capabilities.iotEdge** 属性都会设置为 **true**。

### <a name="retrieve-the-connection-string-with-the-azure-cli"></a>通过 Azure CLI 检索连接字符串

如果已准备好设置设备，则需要连接字符串，该字符串使用物理设备在 IoT 中心内的标识链接该设备。 使用 [az iot hub device-identity connection-string show](/cli/azure/ext/azure-iot/iot/hub/device-identity/connection-string#ext_azure_iot_az_iot_hub_device_identity_connection_string_show) 命令返回单个设备的连接字符串：

   ```azurecli
   az iot hub device-identity connection-string show --device-id [device id] --hub-name [hub name]
   ```

>[!TIP]
>此 `connection-string show` 命令是在 Azure IoT 扩展的0.9.8 版本中引入的，它替换了不推荐使用的 `show-connection-string` 命令。 如果在运行此命令时遇到错误，请确保将扩展版本更新为0.9.8 或更高版本。 有关详细信息和最新更新，请参阅 [Azure CLI Microsoft Azure IoT 扩展](https://github.com/Azure/azure-iot-cli-extension)。

`device-id` 参数的值区分大小写。

复制要在设备上使用的连接字符串时，请不要在连接字符串的两侧加上引号。

---

## <a name="next-steps"></a>后续步骤

现在，已在 IoT 中心内注册了设备，接下来可以在设备上安装和设置 IoT Edge 运行时。

* [安装或卸载适用于 Linux 的 Azure IoT Edge](how-to-install-iot-edge.md)
* [安装或卸载 Windows Azure IoT Edge](how-to-install-iot-edge-windows-on-windows.md)