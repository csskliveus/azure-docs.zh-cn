---
title: 使用 Azure NetApp 文件管理快照 | Microsoft Docs
description: 介绍如何使用 Azure NetApp 文件创建、管理和使用快照。
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/18/2020
ms.author: b-juche
ms.openlocfilehash: 35fce3723e92a3a7c68aaa62b28b756432182a8c
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97629657"
---
# <a name="manage-snapshots-by-using-azure-netapp-files"></a>使用 Azure NetApp 文件管理快照

Azure NetApp 文件支持创建按需快照，并使用快照策略来计划自动创建快照。 还可以将快照还原到新卷，使用客户端还原单个文件，或者使用快照还原现有的卷。

> [!NOTE] 
> 有关跨区域复制中快照管理的注意事项，请参阅 [使用跨区域复制的要求和注意事项](cross-region-replication-requirements-considerations.md)。

## <a name="create-an-on-demand-snapshot-for-a-volume"></a>为卷创建按需快照

可以按需创建卷快照。 

1.  前往要为其创建快照的卷。 单击 " **快照**"。

    ![导航到快照](../media/azure-netapp-files/azure-netapp-files-navigate-to-snapshots.png)

2.  单击“+ 添加快照”，为卷创建按需快照。

    ![添加快照](../media/azure-netapp-files/azure-netapp-files-add-snapshot.png)

3.  在“新建快照”窗口中，为要创建的新快照提供名称。   

    ![新建快照](../media/azure-netapp-files/azure-netapp-files-new-snapshot.png)

4. 单击“确定”  。 

## <a name="manage-snapshot-policies"></a>管理快照策略

你可以计划使用快照策略自动执行卷快照。 你还可以根据需要修改快照策略，或者删除不再需要的快照策略。  

### <a name="register-the-feature"></a>注册功能

**快照策略** 功能目前处于预览阶段。 如果你是首次使用此功能，则需要先注册该功能。 

1. 注册功能： 

    ```azurepowershell-interactive
    Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFSnapshotPolicy
    ```

2. 检查功能注册的状态： 

    > [!NOTE]
    > 在将 `Registering` 更改为之前，RegistrationState 的状态可能最长为60分钟 `Registered` 。 等到状态 **注册** 后再继续。

    ```azurepowershell-interactive
    Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFSnapshotPolicy
    ```
你还可以使用 [Azure CLI 命令](/cli/azure/feature?preserve-view=true&view=azure-cli-latest) `az feature register` 并 `az feature show` 注册功能并显示注册状态。 

### <a name="create-a-snapshot-policy"></a>创建快照策略 

快照策略允许你按每小时、每天、每周或每月周期来指定快照创建频率。 还需要为卷指定要保留的最大快照数。  

1.  从 "NetApp 帐户" 视图中，单击 " **快照策略**"。

    ![快照策略导航](../media/azure-netapp-files/snapshot-policy-navigation.png)

2.  在 "快照策略" 窗口中，将 "策略状态" 设置为 " **已启用**"。 

3.  单击 **每小时**、 **每日**、 **每周** 或 **每月** 选项卡，创建每小时、每日、每周或每月快照策略。 指定 **要保留的快照数**。  

    请参阅 [Azure NetApp 文件的资源限制](azure-netapp-files-resource-limits.md) ，了解卷允许的最大快照数。 

    以下示例显示每小时的快照策略配置。 

    ![每小时快照策略](../media/azure-netapp-files/snapshot-policy-hourly.png)

    以下示例显示了每日快照策略配置。

    ![每日快照策略](../media/azure-netapp-files/snapshot-policy-daily.png)

    以下示例显示了每周快照策略配置。

    ![每周快照策略](../media/azure-netapp-files/snapshot-policy-weekly.png)

    以下示例显示了每月快照策略配置。

    ![每月快照策略](../media/azure-netapp-files/snapshot-policy-monthly.png) 

4.  单击“ **保存**”。  

如果需要创建其他快照策略，请重复步骤3。
创建的策略将显示在 "快照策略" 页中。

如果希望卷使用快照策略，需要 [将策略应用到卷](azure-netapp-files-manage-snapshots.md#apply-a-snapshot-policy-to-a-volume)。 

### <a name="apply-a-snapshot-policy-to-a-volume"></a>将快照策略应用到卷

如果希望卷使用创建的快照策略，需要将策略应用到卷。 

不能将快照策略应用于跨区域复制中的目标卷。  

1.  请在 " **卷** " 页上，右键单击要向其应用快照策略的卷，然后选择 " **编辑**"。

    ![卷右键单击菜单](../media/azure-netapp-files/volume-right-cick-menu.png) 

2.  在 "编辑" 窗口中的 " **快照策略**" 下，选择要用于卷的策略。  单击 **"确定"** 应用该策略。  

    ![快照策略编辑](../media/azure-netapp-files/snapshot-policy-edit.png) 

### <a name="modify-a-snapshot-policy"></a>修改快照策略 

您可以修改现有的快照策略，以更改策略状态、快照频率 (每小时、每天、每周或每月) 或要保留的快照数。  
 
1.  从 "NetApp 帐户" 视图中，单击 " **快照策略**"。

2.  右键单击要修改的快照策略，然后选择 " **编辑**"。

    ![快照策略右键单击菜单](../media/azure-netapp-files/snapshot-policy-right-click-menu.png) 

3.  在出现的 "快照策略" 窗口中进行更改，然后单击 " **保存**"。 

### <a name="delete-a-snapshot-policy"></a>删除快照策略 

你可以删除不再想要保留的快照策略。   

1.  从 "NetApp 帐户" 视图中，单击 " **快照策略**"。

2.  右键单击要修改的快照策略，然后选择 " **删除**"。

    ![快照策略右键单击菜单](../media/azure-netapp-files/snapshot-policy-right-click-menu.png) 

3.  单击 **"是"** 以确认要删除该快照策略。   

    ![快照策略删除确认](../media/azure-netapp-files/snapshot-policy-delete-confirm.png) 

## <a name="edit-the-hide-snapshot-path-option"></a>编辑 "隐藏快照路径" 选项
"隐藏快照路径" 选项控制卷的快照路径是否可见。 在创建 [NFS](azure-netapp-files-create-volumes.md#create-an-nfs-volume) 或 [SMB](azure-netapp-files-create-volumes-smb.md#add-an-smb-volume) 卷的过程中，可以选择指定是否隐藏快照路径。 然后，可以根据需要编辑 "隐藏快照路径" 选项。  

> [!NOTE]
> 对于跨区域复制中的 [目标卷](cross-region-replication-create-peering.md#create-the-data-replication-volume-the-destination-volume) ，默认情况下启用 "隐藏快照路径" 选项，并且不能修改此设置。 

1. 若要查看卷的 "隐藏快照路径" 选项设置，请选择该卷。 " **隐藏快照路径** " 字段显示是否启用了选项。   
    ![描述 "隐藏快照路径" 字段的屏幕截图。](../media/azure-netapp-files/hide-snapshot-path-field.png) 
2. 若要编辑 "隐藏快照路径" 选项，请在 "卷" 页上单击 " **编辑** "，并根据需要修改 " **隐藏快照路径** " 选项。   
    ![描述 "编辑卷快照" 选项的屏幕截图。](../media/azure-netapp-files/volume-edit-snapshot-options.png) 

## <a name="restore-a-snapshot-to-a-new-volume"></a>将快照还原到新卷

目前，只能将快照还原到新卷。 
1. 从 "卷" 边栏选项卡中选择 " **快照** " 以显示快照列表。 
2. 右键单击要还原的快照，并从菜单选项中选择 " **还原到新卷** "。  

    ![将快照还原到新卷](../media/azure-netapp-files/azure-netapp-files-snapshot-restore-to-new-volume.png)

3. 在 "创建卷" 窗口中，为新卷提供以下信息：  
    * **名称**   
        指定要创建的卷的名称。  
        
        名称在资源组中必须唯一。 它的长度必须至少为三个字符。  它可以使用任何字母数字字符。

    * **配额**  
        指定要分配给卷的逻辑存储量。  

    ![还原到新卷](../media/azure-netapp-files/snapshot-restore-new-volume.png) 

4. 单击 " **查看 + 创建**"。  单击“创建”。   
    新卷使用的协议与快照使用的协议相同。   
    快照还原到的新卷将显示在“卷”边栏选项卡中。

## <a name="restore-a-file-from-a-snapshot-using-a-client"></a>使用客户端从快照还原文件

如果不想将 [整个快照还原到卷](#restore-a-snapshot-to-a-new-volume)，则可以选择使用已装入卷的客户端从快照还原文件。  

装载的卷包含 NFS 客户端中名为 (的快照目录  `.snapshot`) 或 `~snapshot` 客户端可以访问的 SMB 客户端) 中的 (。 快照目录包含对应于卷快照的子目录。 每个子目录都包含快照文件。 如果意外删除或覆盖了某个文件，则可以通过将该文件从快照子目录复制到读写目录，将该文件还原到父读写目录。 

如果看不到快照目录，则可能是由于当前启用了 "隐藏快照路径" 选项而隐藏。 可以 [编辑 "隐藏快照路径" 选项](#edit-the-hide-snapshot-path-option) 以禁用它。  

### <a name="restore-a-file-by-using-a-linux-nfs-client"></a>使用 Linux NFS 客户端还原文件 

1. 使用 `ls` Linux 命令列出要从目录还原的文件 `.snapshot` 。 

    例如：

    `$ ls my.txt`   
    `ls: my.txt: No such file or directory`   

    `$ ls .snapshot`   
    `daily.2020-05-14_0013/              hourly.2020-05-15_1106/`   
    `daily.2020-05-15_0012/              hourly.2020-05-15_1206/`   
    `hourly.2020-05-15_1006/             hourly.2020-05-15_1306/`   

    `$ ls .snapshot/hourly.2020-05-15_1306/my.txt`   
    `my.txt`

2. 使用 `cp` 命令将文件复制到父目录。  

    例如： 

    `$ cp .snapshot/hourly.2020-05-15_1306/my.txt .`   

    `$ ls my.txt`   
    `my.txt`   

### <a name="restore-a-file-by-using-a-windows-client"></a>使用 Windows 客户端还原文件 

1. 如果 `~snapshot` 卷的目录处于隐藏状态，则在要显示的父目录中 [显示隐藏项](https://support.microsoft.com/help/4028316/windows-view-hidden-files-and-folders-in-windows-10) `~snapshot` 。

    ![显示隐藏项目](../media/azure-netapp-files/snapshot-show-hidden.png) 

2. 导航到中的子目录 `~snapshot` ，找到要还原的文件。  右键单击该文件。 选择“复制”。  

    ![复制要还原的文件](../media/azure-netapp-files/snapshot-copy-file-restore.png) 

3. 返回到父目录。 右键单击父目录，并选择 `Paste` 将该文件粘贴到该目录。

    ![粘贴要还原的文件](../media/azure-netapp-files/snapshot-paste-file-restore.png) 

4. 你还可以右键单击父目录，选择 " **属性**"，单击 " **以前的版本** " 选项卡以查看快照列表，然后选择 " **还原** " 以还原文件。  

    ![属性以前的版本](../media/azure-netapp-files/snapshot-properties-previous-version.png) 

## <a name="revert-a-volume-using-snapshot-revert"></a>使用快照还原还原卷

快照还原功能使你能够快速将卷还原到创建特定快照时的状态。 在大多数情况下，还原卷比将单个文件从快照还原到活动文件系统要快得多。 与将快照还原到新卷相比，此方法也更节省空间。 

可以在卷的 "快照" 菜单中找到 "恢复卷" 选项。 选择重新确定的快照后，Azure NetApp 文件会将该卷还原到所选快照创建时所包含的数据和时间戳。 

> [!IMPORTANT]
> 在所选快照创建后拍摄的活动文件系统数据和快照将丢失。 快照还原操作会将目标卷中的 *所有* 数据替换为所选快照中的数据。 选择快照时，应注意快照内容和创建日期。 不能撤消快照还原操作。

1. 中转到卷的 " **快照** " 菜单。  右键单击要用于还原操作的快照。 选择 " **恢复卷**"。 

    ![描述快照右键单击菜单的屏幕截图](../media/azure-netapp-files/snapshot-right-click-menu.png) 

2. 在 "将卷还原到快照" 窗口中，键入卷的名称，然后单击 " **还原**"。   

    卷现在会还原到所选快照的时间点。

    ![将卷还原到快照窗口的屏幕截图](../media/azure-netapp-files/snapshot-revert-volume.png) 

## <a name="delete-snapshots"></a>删除快照  

您可以删除不再需要保留的快照。 

1. 中转到卷的 " **快照** " 菜单。 右键单击要删除的快照。 选择“删除” 。

    ![描述快照右键单击菜单的屏幕截图](../media/azure-netapp-files/snapshot-right-click-menu.png) 

2. 在 "删除快照" 窗口中，通过单击 **"是"** 确认是否要删除快照。 

    ![确认删除快照的屏幕截图](../media/azure-netapp-files/snapshot-confirm-delete.png)  

## <a name="next-steps"></a>后续步骤

* [快照策略问题故障排除](troubleshoot-snapshot-policies.md)
* [Azure NetApp 文件的资源限制](azure-netapp-files-resource-limits.md)
* [Azure NetApp 文件快照101视频](https://www.youtube.com/watch?v=uxbTXhtXCkw&feature=youtu.be)
* [什么是 Azure 应用程序一致性快照工具](azacsnap-introduction.md)
