---
title: 创建弹性访问控制管理策略-Azure AD
description: 本文档为组织提供了策略方面的指导，组织可采用这些策略来提供复原能力，以降低意外中断期间的锁定风险
services: active-directory
author: martincoetzer
manager: daveba
tags: azuread
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.workload: identity
ms.date: 06/08/2020
ms.author: martinco
ms.collection: M365-identity-device-management
ms.openlocfilehash: d7e4d0c41990fcc23dd19b5682997f6381bfdb20
ms.sourcegitcommit: 2aa52d30e7b733616d6d92633436e499fbe8b069
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/06/2021
ms.locfileid: "97937087"
---
# <a name="create-a-resilient-access-control-management-strategy-with-azure-active-directory"></a>使用 Azure Active Directory 创建可复原的访问控制管理策略

>[!NOTE]
> 本文档中包含的信息代表 Microsoft Corporation 在发布之日对所讨论问题的当前观点。 由于 Microsoft 必须响应不断变化的市场条件，因此不应将其解释为 Microsoft 作出的承诺，并且 Microsoft 无法保证在发布日期之后提供的任何信息的准确性。

如果组织依靠单一访问控制（如多重身份验证 (MFA) 或单一网络位置）来保护其 IT 系统，那么当该单一访问控制变得不可用或配置错误时，组织很容易因无法访问其应用和资源而受到影响。 例如，自然灾害可能导致大部分电信基础设施或公司网络无法使用。 这种中断可能会导致最终用户和管理员无法登录。

本文档为组织提供了策略方面的指导，组织可采用这些策略来提供复原能力，以降低意外中断期间的锁定风险，具体方案如下：

 1. 组织可以通过实施缓解策略或应急计划，**在中断之前** 提高其复原能力，以降低锁定风险。
 2. 组织可以通过制定缓解策略和应急计划，**在中断期间** 继续访问他们选择的应用和资源。
 3. 组织应确保 **在中断之后** 以及在回滚其实施的任何应急计划之前保留日志等信息。
 4. 尚未实施预防策略或备选计划的组织可能能够实施 **紧急选项** 来应对中断。

## <a name="key-guidance"></a>重要指导

本文档有四个要点：

* 使用紧急访问帐户避免管理员锁定。
* 使用条件访问而不是按用户 MFA 实现 MFA。
* 通过使用多个条件性访问控制来缓解用户锁定。
* 通过为每个用户预配多种身份验证方法或等效项来缓解用户锁定风险。

## <a name="before-a-disruption"></a>中断前

缓解实际中断风险必须是组织处理可能出现的访问控制问题的重中之重。 缓解措施包括规划实际事件和实施策略，以确保访问控制和操作在中断期间不受影响。

### <a name="why-do-you-need-resilient-access-control"></a>为什么需要可复原的访问控制？

 标识是访问应用和资源的用户的控制平面。 标识系统可控制哪些用户以及用户在哪些条件（例如访问控制或身份验证要求）下有权访问应用程序。 如果由于无法预料的情况，无法在一个或多个身份验证或访问控制要求下对用户进行身份验证，则组织可能会遇到以下一个或两个问题：

* **管理员锁定：** 管理员无法管理租户或服务。
* **用户锁定：** 用户无法访问应用或资源。

### <a name="administrator-lockout-contingency"></a>管理员锁定应急计划

若要解锁管理员对租户的访问权限，应创建紧急访问帐户。 这些紧急访问帐户（也称为 *不受限* 帐户）允许在正常的特权帐户访问过程不可用时通过访问 Azure AD 配置来对其进行管理。 应遵循[紧急访问帐户建议]( ../users-groups-roles/directory-emergency-access.md)创建至少两个紧急访问帐户。

### <a name="mitigating-user-lockout"></a>缓解用户锁定风险

 若要降低用户锁定的风险，请使用具有多个控件的条件性访问策略，使用户能够选择如何访问应用和资源。 通过为用户提供多种选择，例如，使用 MFA 登录 **或** 从受管理设备登录 **或** 从公司网络登录，用户可以在其中一种访问控制不可用时，使用其他选项来继续工作。

#### <a name="microsoft-recommendations"></a>Microsoft 的建议

在组织现有的条件性访问策略中合并以下访问控制：

1. 为每个用户预配依赖于不同信道的多种身份验证方法，例如基于 Internet 的 Microsoft Authenticator 应用、在设备上生成的 OATH 令牌和手机短信。 下面的 PowerShell 脚本将帮助你提前标识，你的用户应该注册哪些其他方法： [AZURE AD MFA 身份验证方法分析的脚本](/samples/azure-samples/azure-mfa-authentication-method-analysis/azure-mfa-authentication-method-analysis/)。
2. 在 Windows 10 设备上部署 Windows Hello for Business，以便直接通过设备登录满足 MFA 要求。
3. 通过 [Azure AD 混合联接](../devices/overview.md)或 [Microsoft Intune 受管理设备](/intune/planning-guide)使用受信任的设备。 受信任的设备将改善用户体验，因为受信任的设备本身就能满足策略的强身份验证要求，无需向用户发出 MFA 质询。 而在注册新设备以及从不受信任的设备访问应用或资源时，需使用 MFA。
4. 当用户或登录面临风险时，使用“Azure AD 标识保护”基于风险的策略来阻止访问，而不是使用固定的 MFA 策略。
5. 如果要使用 Azure AD MFA NPS 扩展来保护 VPN 访问权限，请考虑将 VPN 解决方案作为 [SAML 应用](../manage-apps/view-applications-portal.md) 进行联合，并按建议确定应用类别。 

>[!NOTE]
> 基于风险的策略需要 [Azure AD Premium P2](https://azure.microsoft.com/pricing/details/active-directory/) 许可证。

以下示例介绍了在为用户提供可复原的访问控制来访问其应用和资源时所必须创建的策略。 在此示例中，你需要一个安全组 **AppUsers**（包含你要授予其访问权限的目标用户），一个名为 **CoreAdmins** 的组（包含核心管理员），以及一个名为 **EmergencyAccess** 的组（包含紧急访问帐户）。
此示例策略集将在 **AppUsers** 中的选定用户从受信任的设备进行连接时向其授予对选定应用的访问权限，或提供强身份验证（例如 MFA）。 它将紧急帐户和核心管理员排除在外。

**CA 缓解策略集：**

* 策略1：阻止访问目标组以外的人员
  * 用户和组：包含所有用户。 排除 AppUsers、CoreAdmins 和 EmergencyAccess
  * 云应用：包括所有应用
  * 条件： (None) 
  * 授予控制权限：阻止
* 策略2：向需要 MFA 或受信任设备的 AppUsers 授予访问权限。
  * 用户和组：包括 AppUsers。 排除 CoreAdmins 和 EmergencyAccess
  * 云应用：包括所有应用
  * 条件： (None) 
  * 授权控制：授予访问权限，需要多重身份验证，要求设备符合要求的设备。 对于多个控件：需要一个选定的控件。

### <a name="contingencies-for-user-lockout"></a>用户锁定应急计划

或者，组织也可以创建应急策略。 若要创建应急策略，就必须在业务连续性、运营成本、财务成本和安全风险之间定义权衡标准。 例如，可以仅针对一部分用户、一部分应用、一部分客户端或一部分位置激活应急策略。 在未实施缓解方法的中断期间，应急策略将为管理员和最终用户提供对应用和资源的访问权限。 在未使用时，Microsoft 建议在 [仅报告模式下](../conditional-access/howto-conditional-access-insights-reporting.md) 启用应变策略，以便管理员可以监视策略需要打开时的潜在影响。

 了解你在中断期间可能面临的风险有助于降低风险，是规划过程中的关键环节。 若要创建应急计划，请首先确定组织的以下业务要求：

1. 提前确定你的任务关键型应用程序：你必须为其提供访问权限的应用是什么，甚至具有更低的风险/安全状态？ 为这些应用生成一个列表，确保你的其他利益干系人（业务部门、安全部门、法律部门、领导层）都同意：即使所有访问控制都失效，这些应用仍必须继续运行。 你最终可能会对应用进行以下分类：
   * **类别 1 任务关键应用**：不可用时间不能超过几分钟，例如直接影响组织收入的应用。
   * **类别 2 重要应用**：需要在几小时内访问其业务。
   * **类别 3 低优先级应用**：可以承受几天的中断。
2. 对于类别 1 和 2 的应用，Microsoft 建议你预先规划要允许的访问级别类型：
   * 是允许完全访问还是会话受限（例如限制下载）？
   * 是否允许访问应用的一部分，而不是整个应用？
   * 是否允许信息工作者访问而阻止管理员访问，直到访问控制恢复为止？
3. 对于这些应用，Microsoft 还建议你对慎重开放哪些访问途径以及关闭哪些访问途径做好规划：
   * 是否仅允许浏览器访问，而阻止可以保存脱机数据的富客户端访问？
   * 是否仅允许公司网络内部的用户访问，而阻止外部用户访问？
   * 在中断期间，是否仅允许从某些国家或地区进行访问？
   * 如果没有替代访问控制，你希望应急策略（尤其是对于任务关键应用）失败还是成功？

#### <a name="microsoft-recommendations"></a>Microsoft 的建议

"应急条件访问策略" 是一种用于省略 Azure AD MFA、第三方 MFA、基于风险或基于设备的控制的 **备份策略** 。 为了最大限度地减少启用应急策略时出现的意外中断，策略应在不使用时保持为仅报告模式。 管理员可以使用条件性访问见解工作簿来监视其应急策略的潜在影响。 当你的组织决定激活你的应变计划时，管理员可以启用策略并禁用常规的基于控制的策略。

>[!IMPORTANT]
> 启用应急计划时，禁用对用户强制执行安全措施的策略（即使只是暂时的）会对你的安全状况造成威胁。

* 如果一种凭据类型或访问控制机制的中断就会影响对应用的访问，则配置一组回退策略。 对于需要第三方 MFA 提供程序的活动策略，在需要将域加入作为控制的仅报告状态中配置策略。
* 遵循[密码指南](https://aka.ms/passwordguidance)白皮书中的做法，降低不良参与者在不需要 MFA 时猜测密码的风险。
* 部署 [Azure AD 自助密码重置 (SSPR)](./tutorial-enable-sspr.md) 和 [Azure AD 密码保护](./howto-password-ban-bad-on-premises-deploy.md)，以确保用户不使用你选择禁用的常用密码和术语。
* 如果未达到某个身份验证级别，则使用可限制应用内访问权限的策略，而不是简单地回退到完全访问权限。 例如：
  * 配置一个将受限会话声明发送到 Exchange 和 SharePoint 的备用策略。
  * 如果组织使用 Microsoft Cloud App Security，请考虑回退到使用 MCAS 的策略，MCAS 允许只读访问但不允许上传。
* 请将策略命名，以确保在中断期间能够轻松找到它们。 在策略名称中包含以下元素：
  * 策略的标签编号。
  * 要显示的文本。此策略仅用于紧急情况。 例如： **在紧急情况下启用**
  * 策略应用到的中断。 例如： **在 MFA 中断期间**
  * 一个序号，用于指示策略的激活顺序。
  * 策略应用到的应用。
  * 策略应用到的控制措施。
  * 策略要求满足的条件。
  
应变策略的此命名标准如下： 

```
EMnnn - ENABLE IN EMERGENCY: [Disruption][i/n] - [Apps] - [Controls] [Conditions]
```

下面的示例： **一个用于还原对关键任务的协作应用程序的访问权限的应急条件性访问策略示例**，是典型的公司应急措施。 在这种情况下，组织通常要求对所有 Exchange Online 和 SharePoint Online 访问进行 MFA，而在这种情况下，客户端的 MFA 提供程序发生中断， (是 Azure AD MFA、本地 MFA 提供程序还是第三方 MFA) 。 此策略仅在特定目标用户从受信任的公司网络访问应用时才允许其从受信任的 Windows 设备访问这些应用，从而缓解此中断带来的影响。 它还将紧急帐户和核心管理员排除在这些限制之外。 然后，目标用户将会获取 Exchange Online 和 SharePoint Online 的访问权限，而其他用户仍由于服务中断而无法访问应用。 此示例需要一个名为 **CorpNetwork** 的网络位置和一个包含目标用户的安全组 **ContingencyAccess**，一个包含核心管理员的名为 **CoreAdmins** 的组，以及一个包含紧急访问帐户的名为 **EmergencyAccess** 的组。 该应急策略需要四个策略来提供所需的访问权限。 

**用于还原对关键任务的协作应用程序的访问权限的应急条件性访问策略示例：**

* 策略1：需要适用于 Exchange 和 SharePoint 的已加入域的设备
  * 名称： EM001-在紧急情况下启用： MFA 中断 [1/4]-Exchange SharePoint-需要混合 Azure AD 联接
  * 用户和组：包括 ContingencyAccess。 排除 CoreAdmins 和 EmergencyAccess
  * 云应用： Exchange Online 和 SharePoint Online
  * 条件：任意
  * 授权控制：需要加入域
  * 状态：仅报告
* 策略2：阻止 Windows 以外的平台
  * 名称： EM002-在紧急情况下启用： MFA 中断 [2/4]-Exchange SharePoint-阻止访问，Windows 除外
  * 用户和组：包含所有用户。 排除 CoreAdmins 和 EmergencyAccess
  * 云应用： Exchange Online 和 SharePoint Online
  * 条件：设备平台包括所有平台，排除 Windows
  * 授予控制权限：阻止
  * 状态：仅报告
* 策略3：阻止除 CorpNetwork 以外的网络
  * 名称： EM003-在紧急情况下启用： MFA 中断 [3/4]-Exchange SharePoint-阻止访问，企业网络除外
  * 用户和组：包含所有用户。 排除 CoreAdmins 和 EmergencyAccess
  * 云应用： Exchange Online 和 SharePoint Online
  * 条件：位置包含任何位置，排除 CorpNetwork
  * 授予控制权限：阻止
  * 状态：仅报告
* 策略4：显式阻止 EAS
  * 名称： EM004-在紧急情况下启用： MFA 中断 [4/4]-Exchange-阻止所有用户使用 EAS
  * 用户和组：包括所有用户
  * 云应用：包括 Exchange Online
  * 条件：客户端应用： Exchange Active Sync
  * 授予控制权限：阻止
  * 状态：仅报告

激活顺序：

1. 从现有 MFA 策略中排除 ContingencyAccess、CoreAdmins 和 EmergencyAccess。 确保 ContingencyAccess 中的用户能够访问 SharePoint Online 和 Exchange Online。
2. 启用策略1：验证不在排除组中的已加入域的设备上的用户是否能够访问 Exchange Online 和 SharePoint Online。 确保排除组中的用户能够从任意设备访问 SharePoint Online 和 Exchange。
3. 启用策略2：验证不在排除组中的用户是否无法从其移动设备访问 SharePoint Online 和 Exchange Online。 确保排除组中的用户能够从任意设备 (Windows/iOS/Android) 访问 SharePoint 和 Exchange。
4. 启用策略3：验证不在排除组中的用户是否无法访问 SharePoint 和 Exchange 企业网络（即使已加入域的计算机）。 确保排除组中的用户能够从任意网络访问 SharePoint 和 Exchange。
5. 启用策略4：验证所有用户不能从移动设备上的本机邮件应用程序中获取 Exchange Online。
6. 禁用 SharePoint Online 和 Exchange Online 的现有 MFA 策略。

在下一个示例中， **示例 B-应急条件访问策略，以允许对 Salesforce 进行移动访问**。 在此场景中，客户通常要求使用移动设备访问 Salesforce（配置为使用 Azure AD 进行单一登录）的销售员工只能使用合规的设备进行访问。 此示例中的中断是指评估设备符合性时出现问题，在销售团队需要访问 Salesforce 以完成交易的敏感时间发生中断。 这些应急策略将授权关键用户从移动设备访问 Salesforce，以便他们可以继续完成交易而不会中断业务。 在此示例中，**SalesforceContingency** 包含需要保留访问权限的所有销售员工，**SalesAdmins** 包含必要的 Salesforce 管理员。

**示例 B-应急条件访问策略：**

* 策略1：阻止不在 SalesContingency 团队中的每个人
  * 名称： EM001-在紧急情况下启用：设备合规性中断 [1/2]-Salesforce-阻止所有用户（SalesforceContingency 除外）
  * 用户和组：包含所有用户。 排除 SalesAdmins 和 SalesforceContingency
  * 云应用： Salesforce。
  * 条件：无
  * 授予控制权限：阻止
  * 状态：仅报告
* 策略2：从移动 (以外的任何平台阻止销售团队，以减少攻击面) 
  * 名称： EM002-在紧急情况下启用：设备一致性中断 [2/2]-Salesforce-阻止除 iOS 和 Android 之外的所有平台
  * 用户和组：包括 SalesforceContingency。 排除 SalesAdmins
  * 云应用： Salesforce
  * 条件：设备平台包括所有平台，排除 iOS 和 Android
  * 授予控制权限：阻止
  * 状态：仅报告

激活顺序：

1. 从 Salesforce 的现有设备符合性策略中排除 SalesAdmins 和 SalesforceContingency。 确保 SalesforceContingency 组中的用户能够访问 Salesforce。
2. 启用策略1：验证 SalesContingency 之外的用户无法访问 Salesforce。 确保 SalesAdmins 和 SalesforceContingency 中的用户能够访问 Salesforce。
3. 启用策略2：验证 SalesContingency 组中的用户是否无法从其 Windows/Mac 便携式计算机访问 Salesforce，但仍可从其移动设备进行访问。 确保 SalesAdmin 仍可以从任意设备访问 Salesforce。
4. 禁用 Salesforce 的现有设备符合性策略。

### <a name="contingencies-for-user-lockout-from-on-prem-resources-nps-extension"></a>用户从本地资源中锁定 (NPS 扩展的紧急情况) 

如果要使用 Azure AD MFA NPS 扩展来保护 VPN 访问权限，请考虑将 VPN 解决方案作为 [SAML 应用](../manage-apps/view-applications-portal.md) 进行联合，并按建议确定应用类别。 

如果已部署 Azure AD MFA NPS 扩展，以使用 MFA 保护本地资源（例如 VPN 和远程桌面网关），则在紧急情况下，你应该提前考虑禁用 MFA。

在这种情况下，你可以禁用 NPS 扩展，因此，NPS 服务器仅验证主要身份验证，而不会对用户强制执行 MFA。

禁用 NPS 扩展： 
-   将 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\AuthSrv\Parameters 注册表项作为备份导出。 
-   删除 "AuthorizationDLLs" 和 "ExtensionDLLs" 的注册表值，而不是参数键。 
-    (IAS) 服务重新启动网络策略服务，使更改生效 
-   确定 VPN 的主要身份验证是否成功。

一旦该服务已恢复，并且你已准备好在用户上强制执行 MFA，请启用 NPS 扩展： 
-   从备份导入注册表项 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\AuthSrv\Parameters 
-    (IAS) 服务重新启动网络策略服务，使更改生效 
-   确定主要身份验证以及 VPN 的辅助身份验证是否成功。
-   查看 NPS 服务器和 VPN 日志，确定在紧急时段内登录的用户。

### <a name="deploy-password-hash-sync-even-if-you-are-federated-or-use-pass-through-authentication"></a>即使你是联合用户或使用直通身份验证，也可以部署密码哈希同步

如果满足以下条件，也可能发生用户锁定：

- 组织使用包含直通身份验证或联合身份验证的混合标识解决方案。
- 本地标识系统（如 Active Directory、AD FS 或某个从属组件）不可用。 
 
若要提高复原能力，你的组织应[启用密码哈希同步](../hybrid/choose-ad-authn.md)，因为它能让你在本地标识系统关闭的情况下[切换为使用密码哈希同步](../hybrid/plan-connect-user-signin.md)。

#### <a name="microsoft-recommendations"></a>Microsoft 的建议
 无论你的组织使用联合身份验证还是直通身份验证，都可以使用 Azure AD Connect 向导启用密码哈希同步。

>[!IMPORTANT]
> 无需将用户从联合身份验证转换为托管身份验证，也能使用密码哈希同步。

## <a name="during-a-disruption"></a>中断期间

如果选择实施缓解计划，你将能够自动解决单一访问控制中断问题。 但是，如果选择创建应急计划，则可以在访问控制中断期间激活应急策略：

1. 启用应急策略，授权目标用户从特定网络访问特定应用。
2. 禁用基于控制的常规策略。

### <a name="microsoft-recommendations"></a>Microsoft 的建议

根据中断期间所使用的缓解计划或应急计划，组织可以只通过密码就授予访问权限。 没有保护措施是一个相当大的安全风险，必须进行仔细权衡。 组织必须：

1. 作为变更控制策略的一部分，记录每项变更和之前的状态，以便在访问控制完全正常运行后立即回滚你实施的所有应急计划。
2. 假设恶意参与者在你禁用 MFA 时试图通过密码喷射或网络钓鱼攻击来获取密码。 此外，不良参与者可能已经拥有以前未授权访问任何资源，但可能在此时段试图访问这些资源的密码。 对于高级管理人员等关键用户，可以在为其禁用 MFA 之前重置密码，从而在一定程度上缓解此风险。
3. 存档所有登录活动，以确定在 MFA 被禁用期间，哪些人访问了哪些内容。
4. 会审此窗口期间[报告的所有风险检测](../reports-monitoring/concept-sign-ins.md)。

## <a name="after-a-disruption"></a>中断后

还原导致中断的服务后，请撤消作为已激活的应变计划的一部分所做的更改。 

1. 启用常规策略
2. 禁用你的应变策略返回到仅报告模式。 
3. 回滚在中断期间所做的并进行了记录的任何其他更改。
4. 如果使用了紧急访问帐户，请记住重新生成凭据并以物理方式保护新凭据的详细信息，以作为紧急访问帐户过程的一部分。
5. 继续会审发生可疑活动中断后 [报告的所有风险检测](../reports-monitoring/concept-sign-ins.md) 。
6. 撤销[使用 PowerShell](/powershell/module/azuread/revoke-azureaduserallrefreshtoken) 针对一组用户颁发的所有刷新令牌。 撤销所有刷新令牌对于在中断期间使用的特权帐户非常重要，这样做将迫使它们重新进行身份验证并满足已还原策略的控制要求。

## <a name="emergency-options"></a>紧急选项

 在紧急情况下，如果你的组织以前未实施缓解或应变计划，请在 " [用户锁定的紧急](#contingencies-for-user-lockout) 情况" 部分中执行建议，前提是他们已使用条件性访问策略来强制执行 MFA。
如果你的组织使用的是每用户 MFA 旧策略，则可以考虑以下替代方法：

1. 如果有公司网络出站 IP 地址，则可以将它们添加为受信任的 IP，以便仅对公司网络启用身份验证。
   1. 如果没有出站 IP 地址清单，或需要在公司网络内外均启用访问，则可通过指定 0.0.0.0/1 和 128.0.0.0/1 将整个 IPv4 地址空间添加为可信 IP。

>[!IMPORTANT]
 > 如果你放宽了受信任的 IP 地址以取消阻止访问，则与 IP 地址关联的风险检测 (例如，不会生成无法进行的旅行或不熟悉的位置) 。

>[!NOTE]
 > 为 Azure AD MFA 配置 [受信任的 ip](./howto-mfa-mfasettings.md) 仅适用于 [Azure AD Premium 许可证](./concept-mfa-licensing.md)。

## <a name="learn-more"></a>了解详细信息

* [Azure AD 身份验证文档](./howto-mfaserver-iis.md)
* [在 Azure AD 中管理紧急访问管理账户](../roles/security-emergency-access.md)
* [在 Azure Active Directory 中配置命名位置](../reports-monitoring/quickstart-configure-named-locations.md)
  * [Set-MsolDomainFederationSettings](/powershell/module/msonline/set-msoldomainfederationsettings)
* [如何配置联接到混合 Azure Active Directory 的设备](../devices/hybrid-azuread-join-plan.md)
* [Windows Hello for Business 部署指南](/windows/security/identity-protection/hello-for-business/hello-deployment-guide)
  * [密码指南 - Microsoft Research](https://research.microsoft.com/pubs/265143/microsoft_password_guidance.pdf)
* [Azure Active Directory 条件访问中的条件是什么？](../conditional-access/concept-conditional-access-conditions.md)
* [Azure Active Directory 条件访问中的访问控制是什么？](../conditional-access/controls.md)
* [什么是条件访问仅限报告模式？](../conditional-access/concept-conditional-access-report-only.md)
