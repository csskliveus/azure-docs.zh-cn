---
title: 教程：Azure Active Directory 与 xMatters OnDemand 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 xMatters OnDemand 之间配置单一登录。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/19/2020
ms.author: jeedes
ms.openlocfilehash: cbadf2e072cdd9bfdf64cb2b799355aada8ec4b0
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621143"
---
# <a name="tutorial-azure-active-directory-integration-with-xmatters-ondemand"></a>教程：Azure Active Directory 与 xMatters OnDemand 集成

本教程介绍如何将 xMatters OnDemand 与 Azure Active Directory (Azure AD) 集成。
将 xMatters OnDemand 与 Azure AD 集成可提供以下优势：

* 可在 Azure AD 中控制谁有权访问 xMatters OnDemand。
* 可让用户通过其 Azure AD 帐户自动登录到 xMatters OnDemand（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 xMatters OnDemand 的集成，需要以下项：

* 一个 Azure AD 订阅。 如果没有 Azure AD 环境，可以获取一个[免费帐户](https://azure.microsoft.com/free/)。
* 已启用 xMatters OnDemand 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* xMatters OnDemand 支持启用了 IDP 的 SSO

## <a name="adding-xmatters-ondemand-from-the-gallery"></a>从库添加 xMatters OnDemand

若要配置 xMatters OnDemand 与 Azure AD 的集成，需从库将 xMatters OnDemand 添加到托管 SaaS 应用列表。

1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 Azure 门户。
1. 在左侧导航窗格中，选择“Azure Active Directory”服务  。
1. 导航到“企业应用程序”，选择“所有应用程序” 。
1. 若要添加新的应用程序，请选择“新建应用程序”。
1. 在“从库中添加”部分的搜索框中，键入“xMatters OnDemand” 。
1. 从结果面板中选择“xMatters OnDemand”，然后添加该应用。 在该应用添加到租户时等待几秒钟。


## <a name="configure-and-test-azure-ad-sso-for-xmatters-ondemand"></a>配置并测试 xMatters OnDemand 的 Azure AD SSO

使用名为 B.Simon 的测试用户配置并测试 xMatters OnDemand 的 Azure AD SSO。 若要运行 SSO，需要在 Azure AD 用户与 xMatters OnDemand 相关用户之间建立链接关系。

若要配置并测试 xMatters OnDemand 的 Azure AD SSO，请执行以下步骤：

1. **[配置 Azure AD SSO](#configure-azure-ad-sso)** - 使用户能够使用此功能。
    1. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
    2. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
2. [配置 xMatters OnDemand SSO](#configure-xmatters-ondemand-sso) - 在应用程序端配置单一登录设置。
    1. [创建 xMatters OnDemand 测试用户](#create-xmatters-ondemand-test-user) - 在 xMatters OnDemand 中创建 Britta Simon 的对应用户，将其链接到用户的 Azure AD 表示形式。
3. **[测试 SSO](#test-sso)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-sso"></a>配置 Azure AD SSO

按照下列步骤在 Azure 门户中启用 Azure AD SSO。

1. 在 Azure 门户的“xMatters OnDemand”应用程序集成页上，找到“管理”部分，选择“单一登录”  。
1. 在“选择单一登录方法”页上选择“SAML” 。
1. 在“使用 SAML 设置单一登录”页上，单击“基本 SAML 配置”的编辑/笔形图标以编辑设置 。

   ![编辑基本 SAML 配置](common/edit-urls.png)

1. 在“基本 SAML 配置”部分，输入以下字段的值  ：

    a. 在“标识符”文本框中，使用以下模式之一键入 URL：

    | 标识符 |
    | ---------- |
    | `https://<companyname>.au1.xmatters.com.au/` |
    | `https://<companyname>.cs1.xmatters.com/` |
    | `https://<companyname>.xmatters.com/` |
    | `https://www.xmatters.com` |
    | `https://<companyname>.xmatters.com.au/` |

    b. 在“回复 URL”文本框中，使用以下模式之一键入 URL：

    | 回复 URL |
    | ---------- |
    |  `https://<companyname>.au1.xmatters.com.au` |
    | `https://<companyname>.xmatters.com/sp/<instancename>` |
    | `https://<companyname>.cs1.xmatters.com/sp/<instancename>` |
    | `https://<companyname>.au1.xmatters.com.au/<instancename>` |

    > [!NOTE]
    > 这些不是实际值。 请使用实际标识符和回复 URL 更新这些值。 请联系 [xMatters OnDemand 客户端支持团队](https://www.xmatters.com/company/contact-us/)获取这些值。 还可以参考 Azure 门户中的“基本 SAML 配置”  部分中显示的模式。

5. 在“使用 SAML 设置单一登录”  页上，在“SAML 签名证书”  部分中，单击“下载”  以根据要求从给定的选项下载 **证书(Base64)** 并将其保存在计算机上。

    ![证书下载链接](common/certificatebase64.png)

    > [!IMPORTANT]
    > 需将证书转发给 [xMatters OnDemand 支持团队](https://www.xmatters.com/company/contact-us/)。 xMatters 支持团队上传该证书后，用户才能最终完成单一登录配置。

6. 在“设置 xMatters OnDemand”部分，根据要求复制相应 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户

在本部分，我们将在 Azure 门户中创建名为 B.Simon 的测试用户。

1. 在 Azure 门户的左侧窗格中，依次选择“Azure Active Directory”、“用户”和“所有用户”  。
1. 选择屏幕顶部的“新建用户”  。
1. 在“用户”属性中执行以下步骤：
   1. 在“名称”字段中，输入 `B.Simon`。  
   1. 在“用户名”字段中输入 username@companydomain.extension。 例如，`B.Simon@contoso.com`。
   1. 选中“显示密码”复选框，然后记下“密码”框中显示的值。
   1. 单击“创建”。

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分将通过授予 B.Simon 访问 xMatters OnDemand 的权限，允许其使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”。 
1. 在应用程序列表中，选择“xMatters OnDemand”。
1. 在应用的概述页中，找到“管理”部分，选择“用户和组” 。
1. 选择“添加用户”，然后在“添加分配”对话框中选择“用户和组”。
1. 在“用户和组”对话框中，从“用户”列表中选择“B.Simon”，然后单击屏幕底部的“选择”按钮。
1. 如果你希望将某角色分配给用户，可以从“选择角色”下拉列表中选择该角色。 如果尚未为此应用设置任何角色，你将看到选择了“默认访问权限”角色。
1. 在“添加分配”对话框中，单击“分配”按钮。


## <a name="configure-xmatters-ondemand-sso"></a>配置 xMatters OnDemand SSO

1. 在其他 Web 浏览器窗口中，以管理员身份登录到 XMatters OnDemand 公司站点。

2. 单击“管理员”，然后单击“公司详细信息” 。

    ![“管理员”页面](./media/xmatters-ondemand-tutorial/admin.png "管理员")

3. 在“SAML 配置”  页上，执行以下步骤：

    ![“SAML 配置”部分](./media/xmatters-ondemand-tutorial/saml-configuration.png "SAML 配置")

    a. 选择“启用 SAML”  。

    b. 在“标识提供者 ID”文本框中，粘贴从 Azure 门户复制的“Azure AD 标识符”值。

    c. 在“单一登录 URL”文本框中，粘贴从 Azure 门户复制的“登录 URL”值。

    d. 在“注销 URL 重定向”文本框中，粘贴从 Azure 门户复制的“注销 URL”值 。

    e. 单击“选择文件”以上传从 Azure 门户下载的证书 (Base64) 。 

    f. 在“公司详细信息”页上，在顶部单击“保存更改”。

    ![公司详情](./media/xmatters-ondemand-tutorial/save-button.png "公司详情")

### <a name="create-xmatters-ondemand-test-user"></a>创建 xMatters OnDemand 测试用户

1. 登录到 XMatters OnDemand 租户。

2. 转到“用户”图标 > “用户”，然后选择“添加用户”  。

    ![用户](./media/xmatters-ondemand-tutorial/add-user.png "用户")

3. 在“添加用户”部分中，填写必填字段，然后单击“添加用户”按钮 。

    ![添加用户](./media/xmatters-ondemand-tutorial/add-user-2.png "添加用户")



### <a name="test-sso"></a>测试 SSO

在本部分，你将使用以下选项测试 Azure AD 单一登录配置。

* 在 Azure 门户中单击“测试此应用程序”，随后应会自动登录到为其设置了 SSO 的 xMatters OnDemand

* 你可使用 Microsoft 的“我的应用”。 在“我的应用”中单击“xMatters OnDemand”磁贴时，应会自动登录到为其设置了 SSO 的 xMatters OnDemand。 有关“我的应用”的详细信息，请参阅[“我的应用”简介](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)。

## <a name="next-steps"></a>后续步骤

配置 xMatters OnDemand 后，可以强制实施会话控制，以实时防止组织的敏感数据发生外泄和渗透。 会话控制从条件访问扩展而来。 [了解如何通过 Microsoft Cloud App Security 强制实施会话控制](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app)。
