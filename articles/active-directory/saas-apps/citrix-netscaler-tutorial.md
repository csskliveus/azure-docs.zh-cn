---
title: 教程：Azure Active Directory 单一登录与 Citrix ADC（基于 Kerberos 的身份验证）集成 | Microsoft Docs
description: 了解如何使用基于 Kerberos 的身份验证在 Azure Active Directory 和 Citrix ADC 之间配置单一登录 (SSO)。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/15/2020
ms.author: jeedes
ms.openlocfilehash: e5f1f16ba9a2f276d41373eaf4c819522f2d9e2b
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/16/2020
ms.locfileid: "97607569"
---
# <a name="tutorial-azure-active-directory-single-sign-on-integration-with-citrix-adc-kerberos-based-authentication"></a>教程：Azure Active Directory 单一登录与 Citrix ADC（基于 Kerberos 的身份验证）集成

本教程介绍如何将 Citrix ADC 与 Azure Active Directory (Azure AD) 集成。 将 Citrix ADC 与 Azure AD 集成后，可以：

* 在 Azure AD 中控制谁有权访问 Citrix ADC。
* 让用户使用其 Azure AD 帐户自动登录到 Citrix ADC。
* 在一个中心位置（Azure 门户）管理帐户。

## <a name="prerequisites"></a>先决条件

若要开始操作，需备齐以下项目：

* 一个 Azure AD 订阅。 如果没有订阅，可以获取一个[免费帐户](https://azure.microsoft.com/free/)。
* 已启用 Citrix ADC 单一登录 (SSO) 的订阅。

## <a name="scenario-description"></a>方案描述

本教程在测试环境中配置并测试 Azure AD SSO。 本教程包括以下方案：

* 用于 Citrix ADC 的 SP 发起的 SSO

* 用于 Citrix ADC 的实时用户预配

* [Citrix ADC 的基于 Kerberos 的身份验证](#publish-the-web-server)

* [Citrix ADC 的基于标头的身份验证](header-citrix-netscaler-tutorial.md#publish-the-web-server)


## <a name="add-citrix-adc-from-the-gallery"></a>从库中添加 Citrix ADC

若要将 Citrix ADC 与 Azure AD 集成，请先从库中将 Citrix ADC 添加到托管 SaaS 应用的列表：

1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 Azure 门户。

1. 在左侧菜单中，选择“Azure Active Directory”  。

1. 转到“企业应用程序”，并选择“所有应用程序”。  

1. 若要添加新的应用程序，请选择“新建应用程序”。

1. 在“从库中添加”部分的搜索框中，输入“Citrix ADC” 。

1. 在结果中选择“Citrix ADC”，然后添加该应用。 在该应用添加到租户时等待几秒钟。

## <a name="configure-and-test-azure-ad-sso-for-citrix-adc"></a>配置并测试 Citrix ADC 的 Azure AD SSO

使用名为 B.Simon 的测试用户配置并测试 Citrix ADC 的 Azure AD SSO。 若要正常使用 SSO，需要在 Azure AD 用户与 Citrix ADC 中的相关用户之间建立关联关系。

若要配置并测试 Citrix ADC 的 Azure AD SSO，请执行以下步骤：

1. [配置 Azure AD SSO](#configure-azure-ad-sso) - 使用户能够使用此功能。

    1. [创建 Azure AD 测试用户](#create-an-azure-ad-test-user) - 使用 B.Simon 用户测试 Azure AD SSO。

    1. [分配 Azure AD 测试用户](#assign-the-azure-ad-test-user) - 使 B.Simon 能够使用 Azure AD SSO。

1. [配置 Citrix ADC SSO](#configure-citrix-adc-sso) - 在应用程序端配置 SSO 设置。

    * [创建 Citrix ADC 测试用户](#create-a-citrix-adc-test-user) - 在 Citrix ADC 中创建 B.Simon 的对应用户，并将其关联到该用户的 Azure AD 表示形式。

1. [测试 SSO](#test-sso) - 验证配置是否正常工作。

## <a name="configure-azure-ad-sso"></a>配置 Azure AD SSO

若要通过 Azure 门户启用 Azure AD SSO，请完成以下步骤：

1. 在 Azure 门户的“Citrix ADC”应用程序集成页的“管理”下，选择“单一登录”  。

1. 在“选择单一登录方法”窗格中选择“SAML”。  

1. 在“使用 SAML 设置单一登录”窗格中，选择“基本 SAML 配置”对应的笔形“编辑”图标以编辑设置    。

   ![编辑基本 SAML 配置](common/edit-urls.png)

1. 在“基本 SAML 配置”部分，若要在 IDP 发起的模式下配置应用程序，请执行以下步骤：  

    1. 在“标识符”  文本框中，输入采用以下模式的 URL：`https://<Your FQDN>`

    1. 在“回复 URL”  文本框中，输入采用以下模式的 URL：`http(s)://<Your FQDN>.of.vserver/cgi/samlauth`

1. 若要在 SP 发起的模式下配置应用程序，请选择“设置其他 URL”并完成以下步骤：  

    * 在“登录 URL”  文本框中，输入具有以下模式的 URL：`https://<Your FQDN>/CitrixAuthService/AuthService.asmx`

    > [!NOTE]
    > * 本部分使用的 URL 不是实际值。 请使用“标识符”、“回复 URL”和“登录 URL”的实际值更新这些值。 请联系 [Citrix ADC 客户端支持团队](https://www.citrix.com/contact/technical-support.html)来获取这些值。 还可以参考 Azure 门户中的“基本 SAML 配置”  部分中显示的模式。
    > * 若要设置 SSO，必须可以从公共网站访问 URL。 必须在 Citrix ADC 端启用防火墙或其他安全设置，才能让 Azure AD 在所配置的 URL 上发布令牌。

1. 在“使用 SAML 设置单一登录”窗格的“SAML 签名证书”部分，对于“应用联合元数据 URL”，请复制 URL 并将其保存在记事本中    。

    ![证书下载链接](common/certificatebase64.png)

1. 在“设置 Citrix ADC”部分，根据要求复制相关 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户

在本部分，你将在 Azure 门户中创建名为 B.Simon 的测试用户。

1. 在 Azure 门户的左菜单上，依次选择“Azure Active Directory”、“用户”、“所有用户”。   

1. 选择窗格顶部的“新建用户”。 

1. 在“用户”属性中，完成以下步骤： 

   1. 对于“名称”  ，请输入 `B.Simon`。  

   1. 对于“用户名”  ，请输入 _username@companydomain.extension_ 。 例如，`B.Simon@contoso.com` 。

   1. 选中“显示密码”复选框，然后记下或复制“密码”中显示的值。  

   1. 选择“创建”  。

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分中，通过授予用户 B.Simon 访问 Citrix ADC 的权限，允许其使用 Azure SSO。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”。 

1. 在应用程序列表中，选择“Citrix ADC”。

1. 在应用概览的“管理”下，选择“用户和组”   。
1. 选择“添加用户”。 然后，在“添加分配”对话框中选择“用户和组”。  
1. 在“用户和组”对话框的“用户”列表中，选择“B.Simon”。    选择“选择”。 
1. 如果你希望将某角色分配给用户，可以从“选择角色”下拉列表中选择该角色。 如果尚未为此应用设置任何角色，你将看到选择了“默认访问权限”角色。
1. 在“添加分配”对话框中选择“分配”。

## <a name="configure-citrix-adc-sso"></a>配置 Citrix ADC SSO

根据你要配置的身份验证类型选择相应的步骤链接：

- [为基于 Kerberos 的身份验证配置 Citrix ADC SSO](#publish-the-web-server)

- [为基于标头的身份验证配置 Citrix ADC SSO](header-citrix-netscaler-tutorial.md#publish-the-web-server)

### <a name="publish-the-web-server"></a>发布 Web 服务器 

若要创建虚拟服务器：

1. 选择“流量管理” > “负载均衡” > “服务”。   
    
1. 选择 **添加** 。

    ![Citrix ADC 配置 -“服务”窗格](./media/citrix-netscaler-tutorial/web01.png)

1. 为运行应用程序的 Web 服务器设置以下值：

   * **服务名称**
   * 服务器 IP/现有服务器 
   * 协议 
   * 端口 

### <a name="configure-the-load-balancer"></a>配置负载均衡器

若要配置负载均衡器：

1. 转到“流量管理” > “负载均衡” > “虚拟服务器”。   

1. 选择 **添加** 。

1. 如以下屏幕截图所述设置以下值：

    * **名称**
    * 协议 
    * **IP 地址**
    * 端口 

1. 选择“确定”  。

    ![Citrix ADC 配置 -“基本设置”窗格](./media/citrix-netscaler-tutorial/load01.png)

### <a name="bind-the-virtual-server"></a>绑定虚拟服务器

若要将负载均衡器与虚拟服务器绑定：

1. 在“服务和服务组”窗格中，选择“无负载均衡虚拟服务器服务绑定”。  

   ![Citrix ADC 配置 -“负载均衡虚拟服务器服务绑定”窗格](./media/citrix-netscaler-tutorial/bind01.png)

1. 如以下屏幕截图所示验证设置，然后选择“关闭”。 

   ![Citrix ADC 配置 - 验证虚拟服务器服务绑定](./media/citrix-netscaler-tutorial/bind02.png)

### <a name="bind-the-certificate"></a>绑定证书

若要将此服务发布为 TLS，请绑定服务器证书，然后测试应用程序：

1. 在“证书”下，选择“无服务器证书”。  

   ![Citrix ADC 配置 -“服务器证书”窗格](./media/citrix-netscaler-tutorial/bind03.png)

1. 如以下屏幕截图所示验证设置，然后选择“关闭”。 

   ![Citrix ADC 配置 - 验证证书](./media/citrix-netscaler-tutorial/bind04.png)

## <a name="citrix-adc-saml-profile"></a>Citrix ADC SAML 配置文件

若要配置 Citrix ADC SAML 配置文件，请完成以下部分。

### <a name="create-an-authentication-policy"></a>创建身份验证策略

若要创建身份验证策略：

1. 转到“安全性” > “AAA – 应用程序流量” > “策略” > “身份验证” > “身份验证策略”。     

1. 选择 **添加** 。

1. 在“创建身份验证策略”窗格中，输入或选择以下值： 

    * **Name**：输入身份验证策略的名称。
    * **操作**：输入 **SAML**，然后选择“添加”。 
    * **表达式**：输入 **true**。     
    
    ![Citrix ADC 配置 -“创建身份验证策略”窗格](./media/citrix-netscaler-tutorial/policy01.png)

1. 选择“创建”  。

### <a name="create-an-authentication-saml-server"></a>创建身份验证 SAML 服务器

若要创建身份验证 SAML 服务器，请转到“创建身份验证 SAML 服务器”窗格，然后完成以下步骤： 

1. 对于“名称”，请输入身份验证 SAML 服务器的名称。 

1. 在“导出 SAML 元数据”下： 

   1. 选中“导入元数据”复选框。 

   1. 输入前面在 Azure SAML UI 中复制的联合元数据 URL。
    
1. 对于“颁发者名称”，请输入相关的 URL。 

1. 选择“创建”  。

![Citrix ADC 配置 -“创建身份验证 SAML 服务器”窗格](./media/citrix-netscaler-tutorial/server01.png)

### <a name="create-an-authentication-virtual-server"></a>创建身份验证虚拟服务器

若要创建身份验证虚拟服务器：

1.  转到“安全性” > “AAA - 应用程序流量” > “策略” > “身份验证” > “身份验证虚拟服务器”。     

1.  选择“添加”，然后完成以下步骤： 

    1. 对于“名称”，请输入身份验证虚拟服务器的名称。 

    1. 选中“不可寻址”复选框。 

    1. 对于“协议”，请选择“SSL”。  

    1. 选择“确定”  。
    
1. 选择“继续”。 

### <a name="configure-the-authentication-virtual-server-to-use-azure-ad"></a>配置身份验证虚拟服务器以使用 Azure AD

修改身份验证虚拟服务器的两个部分：

1.  在“高级身份验证策略”窗格中，选择“无身份验证策略”。  

    ![Citrix ADC 配置 -“高级身份验证策略”窗格](./media/citrix-netscaler-tutorial/virtual01.png)

1. 在“策略绑定”窗格中选择身份验证策略，然后选择“绑定”。  

    ![Citrix ADC 配置 -“策略绑定”窗格](./media/citrix-netscaler-tutorial/virtual02.png)

1. 在“基于表单的虚拟服务器”窗格中，选择“无负载均衡虚拟服务器”。  

    ![Citrix ADC 配置 -“基于表单的虚拟服务器”窗格](./media/citrix-netscaler-tutorial/virtual03.png)

1. 对于“身份验证 FQDN”，请输入完全限定的域名 (FQDN)（必需）。 

1. 选择要使用 Azure AD 身份验证保护的负载均衡虚拟服务器。

1. 选择“绑定”。 

    ![Citrix ADC 配置 -“负载均衡虚拟服务器绑定”窗格](./media/citrix-netscaler-tutorial/virtual04.png)

    > [!NOTE]
    > 确保在“身份验证虚拟服务器配置”窗格中选择“完成”。  

1. 若要验证更改，请在浏览器中转到应用程序 URL。 应会看到租户登录页，而不是之前所看到的“未经身份验证的访问”消息。

    ![Citrix ADC 配置 - Web 浏览器中的登录页](./media/citrix-netscaler-tutorial/virtual05.png)

## <a name="configure-citrix-adc-sso-for-kerberos-based-authentication"></a>为基于 Kerberos 的身份验证配置 Citrix ADC SSO

### <a name="create-a-kerberos-delegation-account-for-citrix-adc"></a>为 Citrix ADC 创建 Kerberos 委托帐户

1. 创建用户帐户（在此示例中，我们使用 _AppDelegation_）。

    ![Citrix ADC 配置 -“属性”窗格](./media/citrix-netscaler-tutorial/kerberos01.png)

1. 为此帐户设置一个主机 SPN。 

    示例： `setspn -S HOST/AppDelegation.IDENTT.WORK identt\appdelegation`
    
    在本示例中：

    * `IDENTT.WORK` 是域 FQDN。
    * `identt` 是域 NetBIOS 名称。
    * `appdelegation` 是委托用户帐户名称。

1. 配置 Web 服务器的委托，如以下屏幕截图所示：
 
    ![Citrix ADC 配置 -“属性”窗格下的“委托”](./media/citrix-netscaler-tutorial/kerberos02.png)

    > [!NOTE]
    > 在屏幕截图示例中，运行 Windows 集成身份验证 (WIA) 站点的内部 Web 服务器名称是 _CWEB2_。

### <a name="citrix-adc-aaa-kcd-kerberos-delegation-accounts"></a>Citrix ADC AAA KCD（Kerberos 委托帐户）

若要配置 Citrix ADC AAA KCD 帐户，请执行以下操作：

1.  转到“Citrix 网关” > “AAA KCD (Kerberos 约束委托)帐户”。  

1.  选择“添加”  ，然后输入或选择以下值：

    * **Name**：输入 KCD 帐户的名称。

    * **领域**：以大写形式输入域和扩展。

    * **服务 SPN**：`http/<host/fqdn>@<DOMAIN.COM>`。
    
        > [!NOTE]
        > `@DOMAIN.COM` 是必需的，必须大写。 示例：`http/cweb2@IDENTT.WORK`。

    * **委托的用户**：输入委托的用户名。

    * 选中“委托用户的密码”复选框，输入并确认密码。 

1. 选择“确定”  。
 
    ![Citrix ADC 配置 -“配置 KCD 帐户”窗格](./media/citrix-netscaler-tutorial/kerberos03.png)

### <a name="citrix-traffic-policy-and-traffic-profile"></a>Citrix 流量策略和流量配置文件

若要 Citrix 流量策略和流量配置文件，请执行以下操作：

1.  转到“安全性  >  AAA - 应用程序流量  >  策略  >  流量策略、配置文件和窗体 SSO 配置文件流量策略”。    

1.  选择“流量配置文件”。 

1.  选择 **添加** 。

1.  若要配置流量配置文件，请输入或选择以下值。

    * **Name**：输入流量配置文件的名称。

    * **单一登录**：选择“启用”。 

    * **KCD 帐户**：选择在上一部分创建的 KCD 帐户。

1. 选择“确定”  。

    ![Citrix ADC 配置 -“配置流量配置文件”窗格](./media/citrix-netscaler-tutorial/kerberos04.png)
 
1.  选择“流量策略”。 

1.  选择 **添加** 。

1.  若要配置流量策略，请输入或选择以下值：

    * **Name**：输入流量策略的名称。

    * **配置文件**：选择在上一部分创建的流量配置文件。

    * **表达式**：输入 **true**。

1. 选择“确定”  。

    ![Citrix ADC 配置 -“配置流量策略”窗格](./media/citrix-netscaler-tutorial/kerberos05.png)

### <a name="bind-a-traffic-policy-to-a-virtual-server-in-citrix"></a>将流量策略绑定到 Citrix 中的虚拟服务器

使用 GUI 将流量策略绑定到虚拟服务器：

1. 转到“流量管理” > “负载均衡” > “虚拟服务器”。   

1. 在虚拟服务器列表中，选择要绑定重写策略的虚拟服务器，然后选择“打开”  。

1. 在“负载均衡虚拟服务器”窗格中的“高级设置”下，选择“策略”。    为 NetScaler 实例配置的所有策略将显示在列表中。
 
    ![Citrix ADC 配置 -“负载均衡虚拟服务器”窗格](./media/citrix-netscaler-tutorial/kerberos06.png)

    ![Citrix ADC 配置 -“策略”对话框](./media/citrix-netscaler-tutorial/kerberos07.png)

1.  选中要绑定到此虚拟服务器的策略名称旁边的复选框。
 
    ![Citrix ADC 配置 -“负载均衡虚拟服务器流量策略绑定”窗格](./media/citrix-netscaler-tutorial/kerberos09.png)

1. 在“选择类型”对话框中： 

    1. 对于“选择策略”，请选择“流量”。  

    1. 对于“选择类型”，请选择“请求”。  

    ![Citrix ADC 配置 -“选择类型”窗格](./media/citrix-netscaler-tutorial/kerberos08.png)

1. 在绑定策略后，选择“完成”。 
 
    ![Citrix ADC 配置 -“策略”窗格](./media/citrix-netscaler-tutorial/kerberos10.png)

1. 使用 WIA 网站测试绑定。

    ![Citrix ADC 配置 - Web 浏览器中的测试页](./media/citrix-netscaler-tutorial/kerberos11.png)    

### <a name="create-a-citrix-adc-test-user"></a>创建 Citrix ADC 测试用户

本部分将在 Citrix ADC 中创建一个名为 B.Simon 的用户。 Citrix ADC 支持默认启用的实时用户预配。 在本部分无需采取任何措施。 如果 Citrix ADC 中尚不存在用户，则会在身份验证后创建一个新用户。

> [!NOTE]
> 如果需要手动创建用户，请联系 [Citrix ADC 客户端支持团队](https://www.citrix.com/contact/technical-support.html)。

## <a name="test-sso"></a>测试 SSO 

在本部分，你将使用以下选项测试 Azure AD 单一登录配置。 

* 在 Azure 门户中单击“测试此应用程序”。 这会重定向到 Citrix ADC 登录 URL，可从那里启动登录流。 

* 直接转到 Citrix ADC 登录 URL，并从那里启动登录流。

* 你可使用 Microsoft 的“我的应用”。 单击“我的应用”中的 Citrix ADC 磁贴时，会重定向到 Citrix ADC 登录 URL。 有关“我的应用”的详细信息，请参阅[“我的应用”简介](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)。


## <a name="next-steps"></a>后续步骤

配置 Citrix ADC 后，就可以强制实施会话控制，从而实时保护组织的敏感数据免于外泄和渗透。 会话控制从条件访问扩展而来。 [了解如何通过 Microsoft Cloud App Security 强制实施会话控制](/cloud-app-security/proxy-deployment-any-app)。