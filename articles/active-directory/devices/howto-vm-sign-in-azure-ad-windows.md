---
title: 使用 Azure Active Directory（预览版）登录到 Azure 中的 Windows 虚拟机
description: 使用 Azure AD 登录到运行 Windows 的 Azure VM
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 07/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.custom: references_regions, devx-track-azurecli
ms.collection: M365-identity-device-management
ms.openlocfilehash: 22a4bdc92ea2a91425c1070a5837c672307de665
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/22/2021
ms.locfileid: "98683770"
---
# <a name="sign-in-to-windows-virtual-machine-in-azure-using-azure-active-directory-authentication-preview"></a>使用 Azure Active Directory 身份验证（预览版）登录到 Azure 中的 Windows 虚拟机

组织现在可以对运行 Windows Server 2019 Datacenter Edition 或 Windows 10 1809 及更高版本的 Azure 虚拟机 (VM) 使用 Azure Active Directory (AD) 身份验证 。 使用 Azure AD 对 VM 进行身份验证提供了一种方法来集中控制和实施策略。 Azure 基于角色的访问控制 (Azure RBAC) 和 Azure AD 条件访问等工具使你能够控制可访问 VM 的人员。 本文介绍如何创建和配置 Windows Server 2019 VM，以便使用 Azure AD 身份验证。

> [!NOTE]
> 适用于 Azure Windows VM 的 Azure AD 登录是 Azure Active Directory 的公共预览功能。 有关预览版的详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

使用 Azure AD 身份验证登录到 Azure 中的 Windows VM 有许多好处，其中包括：

- 利用你通常所使用的联合或托管 Azure AD 凭据。
- 不再需要管理本地管理员帐户。
- 借助 Azure RBAC，你可以根据需要授予对 VM 的适当访问权限，并在不再需要时将其删除。
- 在允许访问 VM 之前，Azure AD 条件访问可以强制执行其他要求，例如： 
   - 多重身份验证
   - 登录风险检查
- 自动化和缩放 Azure Windows VM 的 Azure AD 联接，这些 VM 是 VDI 部署的一部分。

> [!NOTE]
> 启用此功能后，Azure 中的 Windows 虚拟机将建立与 Azure AD 的联接。 不能将其与其他域建立联接，如本地 AD 或 Azure AD DS。 如果需要执行此操作，则需要通过卸载扩展来断开 VM 与 Azure AD 租户的连接。

## <a name="requirements"></a>要求

### <a name="supported-azure-regions-and-windows-distributions"></a>受支持的 Azure 区域和 Windows 发行版

下面是目前（此功能的预览期间）受支持的 Windows 发行版：

- Windows Server 2019 Datacenter
- Windows 10 1809 和更高版本

> [!IMPORTANT]
> 仅允许从 Windows 10 电脑远程连接到已建立 Azure AD 联接的 VM，而这些电脑需已向 VM 的同一目录注册 Azure AD（从 Windows 10 20H1 开始）、建立 Azure AD 联接或建立混合 Azure AD联接。 

下面是目前（此功能的预览期间）受支持的 Azure 区域：

- 所有 Azure 全局区域

> [!IMPORTANT]
> 若要使用此预览版功能，请只部署受支持的 Windows 发行版，并且只在受支持的 Azure 区域部署。 Azure 政府版或主权云中当前不支持此功能。

### <a name="network-requirements"></a>网络要求

若要为 Azure 中的 Windows VM 启用 Azure AD 身份验证，你需要确保 VM 网络配置允许通过 TCP 端口 443 对以下终结点进行出站访问：

- https： \/ /enterpriseregistration.windows.net
- https:\//login.microsoftonline.com
- https： \/ /device.login.microsoftonline.com
- https： \/ /pas.windows.net

## <a name="enabling-azure-ad-login-in-for-windows-vm-in-azure"></a>为 Azure 中的 Windows VM 启用 Azure AD 登录

若要为 Azure 中的 Windows VM 使用 Azure AD 登录，需要先为 Windows VM 启用 Azure AD 登录选项，然后需要为有权登录到 VM 的用户配置 Azure 角色分配。
可以通过多种方式为 Windows VM 启用 Azure AD 登录：

- 创建 Windows VM 时使用 Azure 门户体验
- 在创建 Windows VM **或现有 WINDOWS vm** 时使用 Azure Cloud Shell 体验

### <a name="using-azure-portal-create-vm-experience-to-enable-azure-ad-login"></a>使用 Azure 门户创建 VM 体验以启用 Azure AD 登录

可以为 Windows Server 2019 Datacenter 或Windows 10 1809 及更高版本的 VM 映像启用 Azure AD 登录。 

若要使用 Azure AD 登录在 Azure 中创建 Windows Server 2019 Datacenter VM，请执行以下操作： 

1. 使用有权创建 VM 的帐户登录到 [Azure 门户](https://portal.azure.com)，然后选择“+ 创建资源”。
1. 在“搜索市场”搜索栏中键入“Windows Server”。
   1. 单击“Windows Server”，然后从“选择软件计划”下拉列表中选择“Windows Server 2019 Datacenter” 。
   1. 单击“创建”。
1. 在“管理”选项卡上，在“Azure Active Directory”部分下，通过将“关闭”改为“开启”来启用“使用 AAD 凭据登录(预览)”选项 。
1. 确保“标识”部分下的“系统分配的托管标识”设置为“启用” 。 使用 Azure AD 凭据启用登录后，此操作将自动执行。
1. 完成创建虚拟机的其余体验。 在此预览期间，必须为 VM 创建管理员用户名和密码。

![使用 Azure AD 凭据登录来创建 VM](./media/howto-vm-sign-in-azure-ad-windows/azure-portal-login-with-azure-ad.png)

> [!NOTE]
> 为使用 Azure AD 凭据登录到 VM，首先需要配置 VM 的角色分配，如下面的其中一部分所述。

### <a name="using-the-azure-cloud-shell-experience-to-enable-azure-ad-login"></a>使用 Azure Cloud Shell 体验启用 Azure AD 登录

Azure Cloud Shell 是免费的交互式 shell，可以使用它运行本文中的操作步骤。 Cloud Shell 中预安装并配置了常用的 Azure 工具以供你的帐户使用。 只需要选择“复制”按钮来复制代码，将其粘贴到 Cloud Shell 中，然后按 Enter 键来运行它。 可通过多种方式打开 Cloud Shell：

选择代码块右上角的“试用”。
在浏览器中打开 Cloud Shell。
选择 [Azure 门户](https://portal.azure.com)右上角菜单上的“Cloud Shell”按钮。

如果选择在本地安装并使用 CLI，本文要求运行 Azure CLI 版本2.0.31 或更高版本。 若要查找版本，请运行 az --version。 如果需要进行安装或升级，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli) 一文。

1. 使用 [az group create](/cli/azure/group#az-group-create) 创建资源组。 
1. 使用受支持区域中受支持的发行版通过 [az vm create](/cli/azure/vm#az-vm-create) 创建 VM。 
1. 安装 Azure AD 登录 VM 扩展。 

下面的示例将一个名为 myVM 的 VM 部署到 default-machinelearning-southcentralus 区域中名为 myResourceGroup 的资源组中。 在以下示例中，可以根据需要提供自己的资源组和 VM 名称。

```AzureCLI
az group create --name myResourceGroup --location southcentralus

az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image Win2019Datacenter \
    --assign-identity \
    --admin-username azureuser \
    --admin-password yourpassword
```

> [!NOTE]
> 在安装 Azure AD 登录 VM 扩展之前，需要在虚拟机上启用系统分配的托管标识。

创建 VM 和支持资源需要几分钟时间。

最后，安装 Azure AD 登录 VM 扩展，以便为 Windows VM 启用 Azure AD 登录。 VM 扩展是小型应用程序，可在 Azure 虚拟机上提供部署后配置和自动化任务。 请使用 [az vm extension](/cli/azure/vm/extension#az-vm-extension-set) set 在 myResourceGroup 资源组中名为 myVM 的 VM 上安装 AADLoginForWindows 扩展：

> [!NOTE]
> 可以在现有 Windows Server 2019 或 Windows 10 1809 及更高版本的 VM 上安装 AADLoginForWindows 扩展，以使其能够进行 Azure AD 身份验证。 AZ CLI 的示例如下所示。

```AzureCLI
az vm extension set \
    --publisher Microsoft.Azure.ActiveDirectory \
    --name AADLoginForWindows \
    --resource-group myResourceGroup \
    --vm-name myVM
```

在 VM 上安装扩展后，将显示 `Succeeded` 的 `provisioningState`。

## <a name="configure-role-assignments-for-the-vm"></a>为 VM 配置角色分配

现在已创建 VM，你需要配置 Azure RBAC 策略来确定可登录到 VM 的人员。 使用两个 Azure 角色来授权 VM 登录：

- **虚拟机管理员登录**：分配了此角色的用户可以使用管理员权限登录到 Azure 虚拟机。
- **虚拟机用户登录名**：分配了此角色的用户可以使用常规用户权限登录到 Azure 虚拟机。

> [!NOTE]
> 若要允许用户通过 RDP 登录到 VM，必须分配“虚拟机管理员登录名”或“虚拟机用户登录名”角色。 分配了 VM“所有者”或“参与者”角色的 Azure 用户不会自动获得通过 RDP 登录到 VM 的权限。 这是为了在控制虚拟机的用户群与可访问虚拟机的用户群之间形成经过审核的分隔。

可以通过多种方式配置 VM 的角色分配：

- 使用 Azure AD 门户体验
- 使用 Azure Cloud Shell 体验

> [!NOTE]
> “虚拟机管理员登录”和“虚拟机用户登录”角色使用 dataActions，因此无法在管理组范围内进行分配。 目前只能在订阅、资源组或资源范围中分配这些角色。

### <a name="using-azure-ad-portal-experience"></a>使用 Azure AD 门户体验

若要为启用 Azure AD 的 Windows Server 2019 Datacenter VM 配置角色分配，请执行以下操作：

1. 导航到特定的虚拟机概述页面
1. 从菜单选项中选择“访问控制(IAM)”
1. 选择“添加”、“添加角色分配”以打开“添加角色分配”窗格 。
1. 在“角色”下拉列表中，选择一个角色，例如“虚拟机管理员登录”或“虚拟机用户登录”  。
1. 在“选择”字段中，选择用户、组、服务主体或托管标识。 如果没有在列表中看到安全主体，则可在“选择”框中键入相应内容，以便在目录中搜索显示名称、电子邮件地址和对象标识符。
1. 选择“保存”以分配该角色。

片刻之后，会在所选范围内为安全主体分配角色。

![为要访问 VM 的用户分配角色](./media/howto-vm-sign-in-azure-ad-windows/azure-portal-access-control-assign-role.png)

### <a name="using-the-azure-cloud-shell-experience"></a>使用 Azure Cloud Shell 体验

以下示例使用 [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) 为当前的 Azure 用户分配登录到 VM 所需的“虚拟机管理员登录名”角色。 活动 Azure 帐户的用户名是使用 [az account show](/cli/azure/account#az-account-show) 获得的，而 scope 则设置为在前面的步骤中使用 [az vm show](/cli/azure/vm#az-vm-show) 创建的 VM。 也可在资源组或订阅级别设置范围，这种情况下会应用正常的 Azure RBAC 继承权限。 有关详细信息，请参阅 [使用 Azure Active Directory 身份验证登录到 Azure 中的 Linux 虚拟机](../../virtual-machines/linux/login-using-aad.md)。

```   AzureCLI
$username=$(az account show --query user.name --output tsv)
$vm=$(az vm show --resource-group myResourceGroup --name myVM --query id -o tsv)

az role assignment create \
    --role "Virtual Machine Administrator Login" \
    --assignee $username \
    --scope $vm
```

> [!NOTE]
> 如果 AAD 域和登录用户名域不匹配，则必须使用 `--assignee-object-id` 指定用户帐户的对象 ID，而不是仅指定 `--assignee` 的用户名。 可以使用 [az ad user list](/cli/azure/ad/user#az-ad-user-list) 获取用户帐户的对象 ID。

有关如何使用 Azure RBAC 管理对 Azure 订阅资源的访问的详细信息，请参阅以下文章：

- [使用 Azure CLI 添加或删除 Azure 角色分配](../../role-based-access-control/role-assignments-cli.md)
- [使用 Azure 门户添加或删除 Azure 角色分配](../../role-based-access-control/role-assignments-portal.md)
- [使用 Azure PowerShell 添加或删除 Azure 角色分配](../../role-based-access-control/role-assignments-powershell.md)。

## <a name="using-conditional-access"></a>使用条件访问

在授权访问 Azure 中已启用 Azure AD 登录的 Windows VM 之前，可以强制执行条件访问策略，如多重身份验证或用户登录风险检查。 若要应用条件访问策略，必须从云应用或操作分配选项中选择“Azure Windows VM 登录”应用，然后使用登录风险作为条件和/或要求多重身份验证作为授权访问控制。 

> [!NOTE]
> 如果使用“需要多重身份验证”作为请求访问“Azure Windows VM 登录”应用的授权访问控制，则必须提供多重身份验证声明，作为向 Azure 中目标 Windows VM 发起 RDP 会话的客户端的一部分。 为在 Windows 10 客户端上实现这一点，唯一的方法是向 RDP 客户端使用 Windows Hello 企业版 PIN 或生物特征身份验证。 在 Windows 10 版本 1809 中，已向 RDP 客户端添加对生物特征身份验证的支持。 使用 Windows Hello 企业版身份验证的远程桌面仅适用于使用证书信任模型的部署，而当前不适用于密钥信任模型。

> [!WARNING]
> VM 登录不支持每用户启用/强制执行的 Azure AD 多重身份验证。

## <a name="log-in-using-azure-ad-credentials-to-a-windows-vm"></a>使用 Azure AD 凭据登录到 Windows VM

> [!IMPORTANT]
> 仅允许从 Windows 10 电脑远程连接到已建立 Azure AD 联接的 VM，而这些电脑需已向 VM 的同一目录注册 Azure AD（要求的最低版本为 20H1）、建立 Azure AD 联接或建立混合 Azure AD联接。 此外，对于使用 Azure AD 凭证的 RDP，用户必须属于两个 Azure 角色之一，即虚拟机管理员登录或虚拟机用户登录。 如果使用已注册 Azure AD 的 Windows 10 电脑，则必须以 AzureAD\UPN 格式（例如 AzureAD\john@contoso.com）输入凭据。 此时，无法通过将 Azure Active Directory 身份验证与 AADLoginForWindows 扩展配合使用来使用Azure Bastion 进行登录；仅支持直接 RDP。

若要使用 Azure AD 登录到 Windows Server 2019 虚拟机，请执行以下操作： 

1. 导航到已启用 Azure AD 登录的虚拟机的概述页面。
1. 选择“连接”，打开“连接到虚拟机”边栏选项卡。
1. 选择“下载 RDP 文件”。
1. 选择“打开”，启动远程桌面连接客户端。
1. 选择“连接”，启动“Windows 登录”对话框。
1. 使用 Azure AD 凭据进行登录。

现在，你已使用分配的角色权限（例如“VM 用户”或“VM 管理员”）登录到 Windows Server 2019 Azure 虚拟机。 

> [!NOTE]
> 可以将 .RDP 文件保存在本地计算机上，以便将来启动与虚拟机的远程桌面连接，而无需导航到 Azure 门户中的虚拟机概述页面以及使用连接选项。

## <a name="troubleshoot"></a>疑难解答

### <a name="troubleshoot-deployment-issues"></a>排查部署问题

AADLoginForWindows 扩展必须成功安装，VM 才能完成 Azure AD 联接过程。 如果 VM 扩展无法正确安装，请执行以下步骤。

1. 使用本地管理员帐户通过 RDP 登录 VM，并检查位于  
   
   C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.ActiveDirectory.AADLoginForWindows\0.3.1.0 下的 CommandExecuti'n.log。 

   > [!NOTE]
   > 如果扩展在初始失败后重启，则含有部署错误的日志将保存为 CommandExecution_YYYYMMDDHHMMSSSSS.log。 "
1. 在 VM 上打开 PowerShell 命令提示符，并针对在 Azure 主机返回上运行的实例元数据服务 (IMDS) 终结点验证这些查询：

   | 要运行的命令 | 预期输出 |
   | --- | --- |
   | `curl -H @{"Metadata"="true"} "http://169.254.169.254/metadata/instance?api-version=2017-08-01"` | 有关 Azure VM 的正确信息 |
   | `curl -H @{"Metadata"="true"} "http://169.254.169.254/metadata/identity/info?api-version=2018-02-01"` | 与 Azure 订阅关联的有效租户 ID |
   | `curl -H @{"Metadata"="true"} "http://169.254.169.254/metadata/identity/oauth2/token?resource=urn:ms-drs:enterpriseregistration.windows.net&api-version=2018-02-01"` | Azure Active Directory 为分配给此 VM 的托管标识颁发的有效访问令牌 |

   > [!NOTE]
   > 可以使用 [http://calebb.net/](http://calebb.net/) 之类的工具对访问令牌进行解码。 验证访问令牌中的“appid”是否与分配给 VM 的托管标识匹配。

1. 确保可以使用命令行从 VM 访问所需的终结点：
   
   - 卷 https： \/ /login.microsoftonline.com/-
   - 卷 https： \/ /login.microsoftonline.com/ `<TenantID>` /-D –

   > [!NOTE]
   > 将 `<TenantID>` 替换为与 Azure 订阅关联的 Azure AD 租户 ID。

   - 卷 https： \/ /enterpriseregistration.windows.net/-
   - 卷 https： \/ /device.login.microsoftonline.com/-
   - 卷 https： \/ /pas.windows.net/-

1. 可以通过运行 `dsregcmd /status` 查看设备状态。 目标是让设备状态显示为 `AzureAdJoined : YES`。

   > [!NOTE]
   > Azure AD 联接活动捕获在事件查看器中的用户设备注册\管理日志下。

如果 AADLoginForWindows 扩展失败并出现某些错误代码，则可以执行以下步骤：

#### <a name="issue-1-aadloginforwindows-extension-fails-to-install-with-terminal-error-code-1007-and-exit-code--2145648574"></a>问题 1：AADLoginForWindows 扩展安装失败，终端错误代码为“1007”，退出代码为：-2145648574。

此退出代码转换为 DSREG_E_MSI_TENANTID_UNAVAILABLE，因为该扩展无法查询 Azure AD 租户信息。

1. 验证 Azure VM 是否可以从实例元数据服务检索租户 ID。

   - 以本地管理员身份通过 RDP 登录 VM，并通过从 VM 上的提升命令行运行以下命令来验证终结点是否返回有效的租户 ID：
      
      - curl -H Metadata:true http://169.254.169.254/metadata/identity/info?api-version=2018-02-01

1. VM 管理员尝试安装 AADLoginForWindows 扩展，但系统分配的托管标识尚未先启用 VM。 导航到 VM 的“标识”边栏选项卡。 在“系统分配”选项卡中，验证状态是否切换为“启用”。

#### <a name="issue-2-aadloginforwindows-extension-fails-to-install-with-exit-code--2145648607"></a>问题 2：AADLoginForWindows 扩展安装失败，退出代码为：-2145648607

此退出代码转换为 DSREG_AUTOJOIN_DISC_FAILED，因为扩展无法到达 `https://enterpriseregistration.windows.net` 终结点。

1. 验证是否可以使用命令行从 VM 访问所需的终结点：

   - 卷 https： \/ /login.microsoftonline.com/-
   - 卷 https： \/ /login.microsoftonline.com/ `<TenantID>` /-D –
   
   > [!NOTE]
   > 将 `<TenantID>` 替换为与 Azure 订阅关联的 Azure AD 租户 ID。 如果需要查找租户 ID，可以将鼠标悬停在帐户名上以获取目录/租户 ID，或在 Azure 门户中选择“Azure Active directory”>“属性”>“目录 ID”。

   - 卷 https： \/ /enterpriseregistration.windows.net/-
   - 卷 https： \/ /device.login.microsoftonline.com/-
   - 卷 https： \/ /pas.windows.net/-

1. 如果任何命令失败，并且出现“无法解析主机 `<URL>`”，请尝试运行此命令，确定 VM 正在使用的 DNS 服务器。
   
   `nslookup <URL>`

   > [!NOTE] 
   > 替换 `<URL>` 为终结点所使用的完全限定的域名，例如 "login.microsoftonline.com"。

1. 接下来，查看指定公共 DNS 服务器是否允许命令成功：

   `nslookup <URL> 208.67.222.222`

1. 如有必要，请更改分配给 Azure VM 所属的网络安全组的 DNS 服务器。

#### <a name="issue-3-aadloginforwindows-extension-fails-to-install-with-exit-code-51"></a>问题 3：AADLoginForWindows 扩展安装失败，退出代码为：51

退出代码 51 转换为“VM 的操作系统不支持此扩展”。

在公共预览版中，AADLoginForWindows 扩展仅适用于在 Windows Server 2019 或 Windows 10（版本 1809 或更高版本）上安装。 确保 Windows 版本受支持。 如果不支持 Windows 版本，请卸载 VM 扩展。

### <a name="troubleshoot-sign-in-issues"></a>排查登录问题

尝试使用 Azure AD 凭据进行 RDP 时的一些常见错误包括未分配 Azure 角色、客户端未经授权或需要 2FA 登录方法。 请根据以下信息来纠正这些问题。

可以通过运行 `dsregcmd /status` 查看设备和 SSO 状态。 目标是让设备状态显示为 `AzureAdJoined : YES`，并让 `SSO State` 显示为 `AzureAdPrt : YES`。

此外，使用 Azure AD 帐户的 RDP 登录捕获在事件查看器中的 AAD\Operational 事件日志下。

#### <a name="azure-role-not-assigned"></a>未分配 Azure 角色

如果在启动到 VM 的远程桌面连接时看到以下错误消息： 

- 你的帐户配置为阻止你使用此设备。 有关详细信息，请与系统管理员联系

![你的帐户配置为阻止你使用此设备。](./media/howto-vm-sign-in-azure-ad-windows/rbac-role-not-assigned.png)

验证是否已为 VM [配置 AZURE RBAC 策略](../../virtual-machines/linux/login-using-aad.md) ，该策略向用户授予 "虚拟机管理员登录名" 或 "虚拟机用户登录" 角色：

> [!NOTE]
> 如果在 Azure 角色分配方面遇到问题，请参阅[排查 Azure RBAC 的问题](../../role-based-access-control/troubleshooting.md#azure-role-assignments-limit)。
 
#### <a name="unauthorized-client"></a>未经授权的客户端

如果在启动到 VM 的远程桌面连接时看到以下错误消息： 

- 你的凭据无效

![你的凭据无效](./media/howto-vm-sign-in-azure-ad-windows/your-credentials-did-not-work.png)

请确认你要用于发起远程桌面连接的 Windows 10 电脑是否已在 VM 联接的同一个 Azure AD 目录建立 Azure AD 联接，或建立混合 Azure AD 联接。 如需详细了解设备标识，请参阅[什么是设备标识](./overview.md)一文。

> [!NOTE]
> Windows 10 版本 20H1 添加了对已注册 Azure AD 电脑的支持来启动到 VM 的 RDP 连接。 在使用已注册 Azure AD（而不是已建立 Azure AD 联接或已建立混合 Azure AD 联接）的电脑作为 RDP 客户端来启动与 VM 的连接时，必须采用 AzureAD\UPn 格式（例如 AzureAD\john@contoso.com）输入凭据。

此外，请验证在 Azure AD 联接建立完成后是否尚未卸载 AADLoginForWindows 扩展。
 
#### <a name="mfa-sign-in-method-required"></a>需要 MFA 登录方法

如果在启动到 VM 的远程桌面连接时看到以下错误消息： 

- 你尝试使用的登录方法不受允许。 请尝试其他登录方法或与系统管理员联系。

![你尝试使用的登录方法不受允许。](./media/howto-vm-sign-in-azure-ad-windows/mfa-sign-in-method-required.png)

如果配置了一种条件访问策略，该访问策略在访问资源之前需要多重身份验证 (MFA)，则需要确保启动到 VM 的远程桌面连接的 Windows 10 电脑使用强身份验证方法（如 Windows Hello）登录。 如果不对远程桌面连接使用强身份验证方法，则会看到上述错误。

如果尚未部署 Windows Hello 企业版，并且目前还没有此选项，则可以通过配置条件访问策略排除 MFA 要求，也就是要在该策略中将“Azure Windows VM 登录”应用从需要 MFA 的云应用列表中排除。 若要了解有关 Windows Hello 企业版的详细信息，请参阅 [Windows Hello 企业版概述](/windows/security/identity-protection/hello-for-business/hello-identity-verification)。

> [!NOTE]
> Windows 10 的多种版本都支持使用 RDP 的 Windows Hello 企业版 PIN 身份验证，但 Windows 10 1809 版本中新增了对使用 RDP 的生物特征身份验证的支持。 在 RDP 期间使用 Windows Hello 企业版身份验证仅适用于使用证书信任模型的部署，而当前不适用于密钥信任模型。
 
## <a name="preview-feedback"></a>预览功能反馈

在 [Azure AD 反馈论坛](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032)上分享此预览功能的反馈或报告使用时存在的问题。

## <a name="next-steps"></a>后续步骤

有关 Azure Active Directory 的详细信息，请参阅[什么是 Azure Active Directory](../fundamentals/active-directory-whatis.md)