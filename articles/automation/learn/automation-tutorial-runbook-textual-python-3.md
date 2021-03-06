---
title: 在 Azure 自动化中创建 Python 3 runbook（预览版）
description: 本文指导如何创建、测试和发布简单的 Python 3 runbook（预览版）。
services: automation
ms.subservice: process-automation
ms.date: 12/22/2020
ms.topic: tutorial
ms.openlocfilehash: cf028722c8c7174aac42f9485e31a599d7713ba3
ms.sourcegitcommit: f7084d3d80c4bc8e69b9eb05dfd30e8e195994d8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/22/2020
ms.locfileid: "97734056"
---
# <a name="tutorial-create-a-python-3-runbook-preview"></a>教程：创建 Python 3 runbook（预览版）

本教程介绍了如何在 Azure 自动化中创建 [Python 3 runbook](../automation-runbook-types.md#python-runbooks)（预览版）。 在 Python 2 和 3 下编译 Python runbook。 可以在 Azure 门户中使用文本编辑器直接编辑 Runbook 的代码。

> [!div class="checklist"]
> * 创建简单的 Python Runbook
> * 测试并发布 Runbook
> * 运行 Runbook 作业并跟踪其状态
> * 更新 Runbook 以使用 Runbook 参数启动 Azure 虚拟机

## <a name="prerequisites"></a>先决条件

要完成本教程，需要以下各项：

- Azure 订阅。 如果还没有帐户，可以[激活 MSDN 订户权益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或注册[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

- [自动化帐户](../automation-security-overview.md) ，用来保存 Runbook 以及向 Azure 资源进行身份验证。 此帐户必须有权启动和停止虚拟机。

- Azure 虚拟机。 在本教程中，你将启动和停止此计算机，因此它不应为生产 VM。

- 可以使用 Linux 或 Windows 混合 Runbook 辅助角色来运行 Python 3 runbook。 请按照说明安装  [Linux](../automation-linux-hrw-install.md)  或  [Windows](../automation-windows-hrw-install.md)  混合 Runbook 辅助角色。 Linux 混合辅助角色没有特定的先决条件。 但是，若要使用适用于 Python 3 runbook 的 Windows 混合辅助角色，请基于是否仅支持 Python 3 或是否 Python 2 和 3 共存的情况配置托管混合辅助角色的计算机。

   * 如果只安装了 Python 2 或 Python 3，则需要将 python.exe 所在文件夹的路径追加到 PATH 环境变量上。 例如，如果 python.exe 位于安装路径 `C:\Python` 中，则需要将该路径追加到 PATH 环境变量中。

   * 如果同时安装了 Python 2 和 Python 3，你想要运行两种类型的 runbook，则需要配置以下环境变量：

     * Python 2 - 创建名为 `PYTHON_2_PATH` 的新环境变量，并指定安装文件夹。 例如，如果安装文件夹为 `C:\Python27`，则需要将此路径添加到该变量中。
     
     * Python 3 - 创建名为 `PYTHON_3_PATH` 的新环境变量，并指定安装文件夹。 例如，如果安装文件夹为 `C:\Python3`，则需要将此路径添加到该变量中。

## <a name="create-a-new-runbook"></a>创建新的 Runbook

首先创建一个输出文本 Hello World 的简单 Runbook。

1. 在 Azure 门户中，打开自动化帐户。

    通过自动化帐户页面可快速查看此帐户中的资源。 应该已拥有某些资产。 大多数此类资产都是自动包括在新的自动化帐户中的模块。 还应具有在 [“先决条件”](#prerequisites)中提到的凭证资产。

2. 在“流程自动化”下选择“Runbook”，以打开 Runbook 的列表。 

3. 选择“添加 Runbook”，创建一个新的 Runbook。

4. 将该 Runbook 命名为 MyFirstRunbook-Python。

5. 选择“Python 3”作为“Runbook 类型” 。

6. 选择“创建”以创建 runbook 并打开文本编辑器 。

## <a name="add-code-to-the-runbook"></a>将代码添加到 Runbook

现在添加一个简单的命令来输出文本 `Hello World`。

```python
print("Hello World!")
```

选择“保存”以保存 runbook。

## <a name="test-the-runbook"></a>测试 Runbook

在发布 Runbook 使其可在生产中使用之前，需对其进行测试以确保其能正常工作。 测试 runbook 时，可以运行其“草稿”版本并以交互方式查看其输出。

1. 选择“测试窗格”以打开“测试”窗格 。

2. 选择“启动”以启动测试 。 这应该是唯一的已启用选项。

3. 将创建一个 [Runbook 作业](../automation-runbook-execution.md) 并显示其状态。
   作业状态一开始为“排队”，表示正在等待云中的 runbook 辅助角色变为可用状态。 在某个辅助角色认领此作业后，作业状态将更改为“正在启动”，然后当 runbook 开始实际运行时，此状态将变为“正在运行” 。

4. Runbook 作业完成后，会显示其输出。 在此示例中，应会看到 `Hello World`。

5. 关闭“测试”窗格以返回到画布。

## <a name="publish-and-start-the-runbook"></a>发布和启动 Runbook

创建的 Runbook 仍处于草稿模式。 需要首先发布此 Runbook 才能在生产中运行它。 当发布 Runbook 时，可以用草稿版本覆盖现有的已发布版本。 在此示例中，因此才创建了 Runbook，因此还没有已发布版本。

1. 选择“发布”以发布该 Runbook，并在出现提示时选择“是” 。

2. 如果在“Runbook”页中向左滚动来查看该 Runbook，应该会看到“创作状态”为“已发布”    。

3. 向右滚动查看“MyFirstRunbook-Python3”窗格。

   可以通过顶部的选项来启动 Runbook、查看 Runbook，或计划在将来的某个时间启动它。

4. 选择“启动”，在“启动 Runbook”窗格打开后选择“确定”  。

5. 此时会为你创建的 runbook 作业打开一个“作业”窗格。 可以关闭此窗格，但我们将使其保持打开状态，以便查看作业的进度。

6. 作业状态显示在“作业摘要”中并且与测试该 Runbook 时看到的状态相匹配。

7. 在 runbook 状态显示为“已完成”后，选择“输出” 。 此时会打开“输出”窗格，可以在其中看到 `Hello World`。

8. 关闭“输出”窗格。

9. 选择“所有日志”以打开 runbook 作业的“流”窗格 。 在“输出”流中应当只会看到 `Hello World`。 但是，此窗格可以显示 runbook 作业的其他流，例如“详细”流和“错误”（如果 runbook 向其中写入内容） 。

10. 关闭“流”窗格和“作业”窗格以返回到“MyFirstRunbook-Python3”窗格  。

11. 选择“作业”以打开此 runbook 的“作业”页 。 此页列出此 Runbook 创建的所有作业。 由于只运行该作业一次，应该只会看到一个列出的作业。

12. 可以选择此作业，打开在启动 runbook 时查看过的“作业”窗格。 你可以随时返回到此窗格，并查看为特定 Runbook 创建的任何作业的详细信息。

## <a name="add-authentication-to-manage-azure-resources"></a>添加身份验证来管理 Azure 资源

已经测试并发布 Runbook，但到目前为止它不执行任何有用的操作。 需要让其管理 Azure 资源。
因此，脚本必须使用你的自动化帐户的凭据进行身份验证。

> [!NOTE]
> 必须使用服务主体功能来创建自动化帐户，以便使用运行方式证书。
> 如果你的自动化帐户不是使用服务主体创建的，则可根据[使用用于 Python 的 Azure 管理库进行身份验证](/azure/python/python-sdk-azure-authenticate)中的说明进行身份验证。

1. 通过选择“MyFirstRunbook-Python3”窗格上的“编辑”打开文本编辑器 。

2. 添加以下代码以对 Azure 进行身份验证：

   ```python
   import os
   from azure.mgmt.compute import ComputeManagementClient
   import azure.mgmt.resource 
   import automationassets 
   
   def get_automation_runas_credential(runas_connection): 
   from OpenSSL import crypto 
   import binascii 
   from msrestazure import azure_active_directory 
   import adal 
    
   # Get the Azure Automation RunAs service principal certificate 
   cert = automationassets.get_automation_certificate("AzureRunAsCertificate") 
   pks12_cert = crypto.load_pkcs12(cert) 
   pem_pkey = crypto.dump_privatekey(crypto.FILETYPE_PEM,pks12_cert.get_privatekey()) 
    
   # Get run as connection information for the Azure Automation service principal 
   application_id = runas_connection["ApplicationId"] 
   thumbprint = runas_connection["CertificateThumbprint"] 
   tenant_id = runas_connection["TenantId"] 
    
   # Authenticate with service principal certificate 
   resource ="https://management.core.windows.net/" 
   authority_url = ("https://login.microsoftonline.com/"+tenant_id) 
   context = adal.AuthenticationContext(authority_url) 
   return azure_active_directory.AdalAuthentication( 
   lambda: context.acquire_token_with_client_certificate( 
           resource, 
           application_id, 
           pem_pkey, 
           thumbprint) 
   ) 
    
   # Authenticate to Azure using the Azure Automation RunAs service principal 
   runas_connection = automationassets.get_automation_connection("AzureRunAsConnection") 
   azure_credential = get_automation_runas_credential(runas_connection) 
   ```

## <a name="add-code-to-create-python-compute-client-and-start-the-vm"></a>添加代码以创建 Python Compute 客户端并启动 VM

若要使用 Azure VM，请创建 [Python 的 Azure Compute 客户端](/python/api/azure-mgmt-compute/azure.mgmt.compute.computemanagementclient)实例。

使用此计算客户端启动 VM。 将以下代码添加到 Runbook：

```python
# Initialize the compute management client with the RunAs credential and specify the subscription to work against.
compute_client = ComputeManagementClient(
    azure_credential,
    str(runas_connection["SubscriptionId"])
)


print('\nStart VM')
async_vm_start = compute_client.virtual_machines.start(
    "MyResourceGroup", "TestVM")
async_vm_start.wait()
```

其中 `MyResourceGroup` 是包含 VM 的资源组的名称，`TestVM` 是要启动的 VM 的名称。

再次测试并运行 Runbook，以便看到它启动 VM。

## <a name="use-input-parameters"></a>使用输入参数

对于资源组名称和 VM 名称，Runbook 当前使用硬编码的值。 现在我们来添加用于从输入参数获取这些值的代码。

使用 `sys.argv` 变量获取参数值。 将以下代码添加到 Runbook，使其紧随其他 `import` 语句之后：

```python
import sys

resource_group_name = str(sys.argv[1])
vm_name = str(sys.argv[2])
```

它将导入 `sys` 模块，并创建两个变量以用于保存资源组名称和 VM 名称。 请注意，参数列表的元素 `sys.argv[0]` 是脚本的名称，不是用户的输入。

现在可以修改 Runbook 的最后两行，以使用输入参数值，而不使用硬编码的值：

```python
async_vm_start = compute_client.virtual_machines.start(
    resource_group_name, vm_name)
async_vm_start.wait()
```

启动 Python runbook 时（无论是从“测试”窗格中启动，还是将其作为已发布的 runbook 启动），可以在“启动 Runbook”页的“参数”下输入参数的值  。

开始在第一个框中输入值时，会出现第二个框，依此类推，这样就可以根据需要输入任意数量的参数值。

这些值可在 `sys.argv` 数组中供脚本使用，就像在你刚才添加的代码中那样。

输入资源组的名称作为第一个参数的值，输入 VM 的名称作为第二个参数的值。

![输入参数值](../media/automation-tutorial-runbook-textual-python/runbook-python-params.png)

选择“确定”启动 runbook。 Runbook 运行，并启动指定的 VM。

## <a name="error-handling-in-python"></a>Python 中的错误处理

还可以使用以下约定从 Python Runbook 中检索各种流，包括“警告”、“错误”和“调试”流。

```python
print("Hello World output")
print("ERROR: - Hello world error")
print("WARNING: - Hello world warning")
print("DEBUG: - Hello world debug")
print("VERBOSE: - Hello world verbose")
```

以下示例显示了在 `try...except` 块中使用的此约定。

```python
try:
    raise Exception('one', 'two')
except Exception as detail:
    print ('ERROR: Handling run-time error:', detail)
```

## <a name="next-steps"></a>后续步骤

- 若要了解有关 Runbook 类型、其优点和限制的详细信息，请参阅 [Azure 自动化 Runbook 类型](../automation-runbook-types.md)。

- 若要了解如何使用 Python 进行 Azure 开发，请参阅[适用于 Python 开发人员的 Azure](/azure/python/)。

- 若要查看 Python 3 runbook 示例，请参阅 [GitHub 上的 Azure 自动化](https://github.com/azureautomation/runbooks/tree/master/Utility/Python)。
