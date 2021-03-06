---
title: 从 Azure 门户将混合计算机连接到 Azure
description: 本文介绍如何使用 Azure 门户中的启用了 Azure Arc 的服务器安装代理并将计算机连接到 Azure。
ms.date: 11/05/2020
ms.topic: conceptual
ms.openlocfilehash: ca3c08acdef1b2a1f7c3774f5755967d472c93ed
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/05/2020
ms.locfileid: "93398022"
---
# <a name="connect-hybrid-machines-to-azure-from-the-azure-portal"></a>从 Azure 门户将混合计算机连接到 Azure

你可以通过手动执行一组步骤，为你的环境中的一台或多台 Windows 或 Linux 计算机启用启用了 Azure Arc 的服务器。 或者，可以运行我们提供的模板脚本来使用自动化方法。 此脚本可以自动下载和安装两个代理。

这种安装和配置代理的方法要求你在计算机上拥有管理员权限。 在 Linux 上，需使用 root 帐户；在 Windows 上，你需是“本地管理员组”的成员。

在开始之前，请务必查看[先决条件](agent-overview.md#prerequisites)，并验证你的订阅和资源是否符合要求。 有关支持的区域和其他相关注意事项的信息，请参阅 [支持的 Azure 区域](overview.md#supported-regions)。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="generate-the-installation-script-from-the-azure-portal"></a>从 Azure 门户生成安装脚本

Azure 门户中提供了用于自动下载和安装以及与 Azure Arc 建立连接的脚本。 若要完成该过程，请执行以下操作：

1. 在浏览器中转到 [Azure 门户](https://portal.azure.com)。

1. 在 " **服务器-Azure Arc** " 页上，选择左上角的 " **添加** "。

1. 在 " **选择方法** " 页上，选择 " **使用交互式脚本添加服务器** " 磁贴，然后选择 " **生成脚本** "。

1. 在“生成脚本”页上，选择你要在 Azure 中管理的计算机所在的订阅和资源组。 选择要将计算机元数据存储到的 Azure 位置。 此位置可以与资源组的位置相同或不同。

1. 在 " **先决条件** " 页上，查看信息，然后选择 " **下一步：资源详细信息** "。

1. 在 " **资源详细信息** " 页上，提供以下内容：

    1. 在 " **资源组** " 下拉列表中，选择要从中管理计算机的资源组。
    1. 在 " **区域** " 下拉列表中，选择用于存储服务器元数据的 Azure 区域。
    1. 在 " **操作系统** " 下拉列表中，选择要在其中运行脚本的操作系统。
    1. 如果计算机通过代理服务器进行通信以连接到 internet，请指定代理服务器 IP 地址或计算机将用于与代理服务器进行通信的名称和端口号。 按格式 `http://<proxyURL>:<proxyport>` 输入值。
    1. 在完成时选择“下一步:  标记”。

1. 在 " **标记** " 页上，查看建议的默认 **物理位置标记** 并输入一个值，或指定一个或多个 **自定义标记** 以支持你的标准。

1. 选择 **下一步：下载并运行脚本** 。

1. 在 " **下载并运行脚本** " 页上，查看摘要信息，然后选择 " **下载** "。 如果仍需进行更改，请选择“上一页”。

## <a name="install-and-validate-the-agent-on-windows"></a>在 Windows 上安装并验证代理

### <a name="install-manually"></a>手动安装

可以运行 Windows Installer 包 *AzureConnectedMachineAgent.msi* 来手动安装 Connected Machine 代理。 可以从 Microsoft 下载中心下载最新版本的 [Windows 代理 Windows Installer 包](https://aka.ms/AzureConnectedMachineAgent)。

>[!NOTE]
>* 若要安装或卸载代理，必须拥有“管理员”权限。
>* 必须先下载 Installer 包并将其复制到目标服务器上的某个文件夹，或者从共享网络文件夹下载。 如果在不指定任何选项的情况下运行该 Installer 包，它将启动一个安装向导，以交互方式指导用户安装代理。

如果计算机需要通过代理服务器与服务进行通信，则在安装代理后，需要运行以下步骤中描述的命令。 此命令设置代理服务器系统环境变量 `https_proxy` 。

如果不熟悉 Windows Installer 包的命令行选项，请查看 [Msiexec 标准命令行选项](/windows/win32/msi/standard-installer-command-line-options)和 [Msiexec 命令行选项](/windows/win32/msi/command-line-options)。

例如，若要查看帮助和快速参考选项，请运行包含 `/?` 参数的安装程序。

```dos
msiexec.exe /i AzureConnectedMachineAgent.msi /?
```

1. 若要无提示地安装代理并在存在的 `C:\Support\Logs` 文件夹中创建安装日志文件，请运行以下命令。

    ```dos
    msiexec.exe /i AzureConnectedMachineAgent.msi /qn /l*v "C:\Support\Logs\Azcmagentsetup.log"
    ```

    如果完成安装后代理无法启动，请检查日志以获取详细的错误信息。 日志目录为 *%ProgramData%\AzureConnectedMachineAgent\log* 。

2. 如果计算机需要通过代理服务器进行通信，若要设置代理服务器环境变量，请运行以下命令：

    ```powershell
    [Environment]::SetEnvironmentVariable("https_proxy", "http://{proxy-url}:{proxy-port}", "Machine")
    $env:https_proxy = [System.Environment]::GetEnvironmentVariable("https_proxy","Machine")
    # For the changes to take effect, the agent service needs to be restarted after the proxy environment variable is set.
    Restart-Service -Name himds
    ```

    >[!NOTE]
    >此预览版中的代理程序不支持设置代理身份验证。
    >

3. 安装代理后，需要通过运行以下命令将其配置为与 Azure Arc 服务通信：

    ```dos
    "%ProgramFiles%\AzureConnectedMachineAgent\azcmagent.exe" connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID"
    ```

### <a name="install-with-the-scripted-method"></a>使用脚本方法安装

1. 登录到服务器。

1. 打开权限提升的 PowerShell 命令提示符。

    >[!NOTE]
    >此脚本仅支持在64位版本的 Windows PowerShell 中运行。
    >

1. 切换到脚本所复制到的文件夹或共享，然后在服务器上运行 `./OnboardingScript.ps1` 脚本来执行复制的脚本。

如果完成安装后代理无法启动，请检查日志以获取详细的错误信息。 日志目录为 *%ProgramData%\AzureConnectedMachineAgent\log* 。

## <a name="install-and-validate-the-agent-on-linux"></a>在 Linux 上安装并验证代理

适用于 Linux 的 Connected Machine 代理以 Microsoft [包存储库](https://packages.microsoft.com/)中分发版的首选包格式（.RPM 或 .DEB）提供。 [shell 脚本捆绑包 `Install_linux_azcmagent.sh`](https://aka.ms/azcmagent) 执行以下操作：

* 将主机配置为从 packages.microsoft.com 下载代理包。

* 安装混合资源提供程序包。

（可选）可以通过包含 `--proxy "{proxy-url}:{proxy-port}"` 参数使用代理信息来配置代理。

该脚本还包含用于识别受支持和不受支持分发包的逻辑，并可验证执行安装所需的权限。

以下示例将下载并安装代理：

```bash
# Download the installation package.
wget https://aka.ms/azcmagent -O ~/Install_linux_azcmagent.sh

# Install the connected machine agent.
bash ~/Install_linux_azcmagent.sh
```

1. 若要下载并安装代理，请结合 `--proxy` 参数（用于将代理配置为通过代理服务器通信）运行以下命令：

    ```bash
    # Download the installation package.
    wget https://aka.ms/azcmagent -O ~/Install_linux_azcmagent.sh

    # Install the connected machine agent.
    bash ~/Install_linux_azcmagent.sh --proxy "{proxy-url}:{proxy-port}"
    ```

2. 安装代理后，需要通过运行以下命令将其配置为与 Azure Arc 服务通信：

    ```bash
    azcmagent connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --cloud "cloudName"
    if [ $? = 0 ]; then echo "\033[33mTo view your onboarded server(s), navigate to https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.HybridCompute%2Fmachines\033[m"; fi
    ```

### <a name="install-with-the-scripted-method"></a>使用脚本方法安装

1. 使用具有根访问权限的帐户登录到服务器。

1. 切换到脚本所复制到的文件夹或共享，然后在服务器上运行 `./OnboardingScript.sh` 脚本来执行复制的脚本。

如果完成安装后代理无法启动，请检查日志以获取详细的错误信息。 日志目录为 *var/opt/azcmagent/log* 。

## <a name="verify-the-connection-with-azure-arc"></a>验证是否与 Azure Arc 连接

安装代理并将其配置为连接到启用了 Azure Arc 的服务器后，请转到 Azure 门户，验证是否已成功连接服务器。 在 [Azure 门户](https://aka.ms/hybridmachineportal)中查看计算机。

![服务器连接成功](./media/onboard-portal/arc-for-servers-successful-onboard.png)

## <a name="next-steps"></a>后续步骤

* 有关疑难解答信息，请参阅 [连接计算机代理疑难解答指南](troubleshoot-agent-onboard.md)。

* 了解如何使用 [Azure Policy](../../governance/policy/overview.md) 管理计算机，例如，进行 VM [来宾配置](../../governance/policy/concepts/guest-configuration.md)，验证计算机是否向预期的 Log Analytics 工作区报告，使用[用于 VM 的 Azure Monitor](../../azure-monitor/insights/vminsights-enable-policy.md) 启用监视等。

* 详细了解 [Log Analytics 代理](../../azure-monitor/platform/log-analytics-agent.md)。 需要收集操作系统和工作负荷监视数据、使用自动化 runbook 或功能（如更新管理）管理该数据，或使用 [Azure 安全中心](../../security-center/security-center-introduction.md)之类的其他 azure 服务时，需要使用适用于 Windows 和 Linux 的 Log Analytics 代理。