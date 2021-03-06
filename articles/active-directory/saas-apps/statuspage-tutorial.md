---
title: 教程：Azure Active Directory 单一登录 (SSO) 与 StatusPage 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 StatusPage 之间配置单一登录。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/18/2020
ms.author: jeedes
ms.openlocfilehash: 40cb3c6e326cbacfbfb9879cfa92b5780d86fe99
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/22/2020
ms.locfileid: "97724198"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-statuspage"></a>教程：Azure Active Directory 单一登录 (SSO) 与 StatusPage 的集成

本教程介绍如何将 StatusPage 与 Azure Active Directory (Azure AD) 集成。
将 StatusPage 与 Azure AD 集成提供以下优势：

* 可以在 Azure AD 中控制谁有权访问 StatusPage。
* 可让用户使用其 Azure AD 帐户自动登录到 StatusPage（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 StatusPage 的集成，需备齐以下项目：

* 一个 Azure AD 订阅。 如果没有 Azure AD 环境，可以获取一个[免费帐户](https://azure.microsoft.com/free/)
* 已启用 StatusPage 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* StatusPage 支持 **IDP** 发起的 SSO

## <a name="adding-statuspage-from-the-gallery"></a>从库中添加 StatusPage

要配置 StatusPage 与 Azure AD 的集成，需要从库中将 StatusPage 添加到托管 SaaS 应用列表。

1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 Azure 门户。
1. 在左侧导航窗格中，选择“Azure Active Directory”服务  。
1. 导航到“企业应用程序”，选择“所有应用程序” 。
1. 若要添加新的应用程序，请选择“新建应用程序”。
1. 在“从库中添加”部分的搜索框中，键入 StatusPage 。
1. 在结果面板中选择 StatusPage，然后添加该应用。 在该应用添加到租户时等待几秒钟。

## <a name="configure-and-test-azure-ad-sso-for-statuspage"></a>配置并测试 StatusPage 的 Azure AD SSO

在本部分，我们基于名为 **Britta Simon** 的测试用户来配置并测试 StatusPage 的 Azure AD 单一登录。
若要正常使用单一登录，需要在 Azure AD 用户与 StatusPage 相关用户之间建立链接关系。

若要配置并测试 StatusPage 的 Azure AD SSO，请执行以下步骤：

1. **[配置 Azure AD SSO](#configure-azure-ad-sso)** - 使用户能够使用此功能。
    1. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
    1. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
1. **[配置 StatusPage SSO](#configure-statuspage-sso)** - 在应用程序端配置单一登录设置。
    1. **[创建 StatusPage 测试用户](#create-statuspage-test-user)** - 在 StatusPage 中创建 Britta Simon 的对应用户，并将其关联到其在 Azure AD 中的表示形式。
6. **[测试 SSO](#test-sso)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-sso"></a>配置 Azure AD SSO

按照下列步骤在 Azure 门户中启用 Azure AD SSO。

1. 在 Azure 门户的“AskYourTeam”应用程序集成页上，找到“管理”部分并选择“单一登录”  。
1. 在“选择单一登录方法”页上选择“SAML” 。
1. 在“使用 SAML 设置单一登录”页上，单击“基本 SAML 配置”的编辑/笔形图标以编辑设置 。

   ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“设置 SAML 单一登录”页上，执行以下步骤：

    a. 在“标识符”文本框中，使用以下模式之一键入 URL：

    | 标识符 |
    |--------------|
    | `https://<subdomain>.statuspagestaging.com/` |
    | `https://<subdomain>.statuspage.io/` |
    |

    b. 在“回复 URL”文本框中，使用以下模式键入 URL： 

     | 回复 URL |
    |--------------|
    | `https://<subdomain>.statuspagestaging.com/sso/saml/consume` |
    | `https://<subdomain>.statuspage.io/sso/saml/consume` |
    |

    > [!NOTE]
    > 通过 [SupportTeam@statuspage.io](mailto:SupportTeam@statuspage.io) 联系 StatusPage 支持团队，请求配置单一登录所需的元数据。 
    >
    > a. 从元数据复制“颁发者”值，并将其粘贴到“标识符”  文本框中。
    >
    > b. 从元数据复制回复 URL，并将其粘贴到“回复 URL”  文本框中。

5. 在“使用 SAML 设置单一登录”  页上，在“SAML 签名证书”  部分中，单击“下载”  以根据要求从给定的选项下载 **证书(Base64)** 并将其保存在计算机上。

    ![证书下载链接](common/certificatebase64.png)

6. 在“设置 StatusPage”部分，根据要求复制相应的 URL。 

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

在本部分中，通过授予 Britta Simon 访问 StatusPage 的权限，允许她使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”、“StatusPage”。   

2. 在应用程序列表中，选择“StatusPage”  。

3. 在左侧菜单中，选择“用户和组”  。

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

5. 在“用户和组”  对话框中，选择“用户”列表中的 Britta Simon  ，然后单击屏幕底部的“选择”  按钮。

6. 如果你希望将某角色分配给用户，可以从“选择角色”下拉列表中选择该角色。 如果尚未为此应用设置任何角色，你将看到选择了“默认访问权限”角色。

7. 在“添加分配”对话框中，单击“分配”按钮。  

## <a name="configure-statuspage-sso"></a>配置 StatusPage SSO

1. 若要在 StatusPage 中自动执行配置，需要通过单击“安装扩展”来安装“我的应用安全登录”浏览器扩展 。

    ![我的应用扩展](common/install-myappssecure-extension.png)

2. 将扩展添加到浏览器后，单击“安装 StatusPage”会将你定向到 StatusPage 应用程序。 在此处提供管理员凭据以登录到 StatusPage。 浏览器扩展会自动配置该应用程序，并自动执行步骤 3-6。

    ![设置配置](common/setup-sso.png)

3. 若要手动设置 StatusPage，请在另一个 Web 浏览器窗口中，以管理员身份登录到 StatusPage 公司站点。

1. 在主工具栏中，单击“管理帐户”  。

    ![显示从 StatusPage 公司站点选择了“管理帐户”的屏幕截图。](./media/statuspage-tutorial/tutorial_statuspage_06.png)

1. 单击“单一登录”  选项卡。

    ![显示“单一登录”选项卡的屏幕截图。](./media/statuspage-tutorial/tutorial_statuspage_07.png)

1. 在“SSO 设置”页上，执行以下步骤：

    ![显示“SSO 设置”页的屏幕截图，可在其中输入所述值。](./media/statuspage-tutorial/tutorial_statuspage_08.png)

    ![显示“保存配置”按钮的屏幕截图。](./media/statuspage-tutorial/tutorial_statuspage_09.png)

    a. 在“SSO 目标 URL”文本框中，粘贴从 Azure 门户复制的“登录 URL”值   。

    b. 在记事本中打开下载的证书，复制其内容，并将其粘贴到“证书”文本框中。 

    c. 单击“保存配置”  。

### <a name="create-statuspage-test-user"></a>创建 StatusPage 测试用户

本部分的目的是在 StatusPage 中创建名为“Britta Simon”的用户。

StatusPage 支持实时预配。 已在[配置 Azure AD 单一登录](#configure-azure-ad-sso)中启用此功能。

**若要在 StatusPage 中创建名为“Britta Simon”的用户，请执行以下步骤：**

1. 以管理员身份登录 StatusPage 公司站点。

1. 在顶部菜单中，单击“管理帐户”  。

    ![显示从 StatusPage 公司站点选择了“管理帐户”的屏幕截图。](./media/statuspage-tutorial/tutorial_statuspage_06.png)

1. 单击“团队成员”选项卡  。
  
    ![显示“团队成员”选项卡的屏幕截图。](./media/statuspage-tutorial/tutorial_statuspage_10.png) 

1. 单击“添加团队成员”  。
  
    ![显示“添加团队成员”按钮的屏幕截图。](./media/statuspage-tutorial/tutorial_statuspage_11.png) 

1. 在相关文本框中键入要预配的有效用户的“电子邮件地址”、“名字”和“姓氏”    。 

    ![显示“添加用户”对话框的屏幕截图，可在其中输入所述值。](./media/statuspage-tutorial/tutorial_statuspage_12.png) 

1. 选择“客户端管理员”作为 **角色**。

1. 单击“创建帐户”  。

### <a name="test-sso"></a>测试 SSO

在本部分，你将使用以下选项测试 Azure AD 单一登录配置。

* 在 Azure 门户中单击“测试此应用程序”后，应会自动登录到为其设置了 SSO 的 StatusPage

* 你可使用 Microsoft 的“我的应用”。 单击“我的应用”中的 StatusPage 磁贴时，你应当会自动登录到为其设置了 SSO 的 StatusPage。 有关“我的应用”的详细信息，请参阅[“我的应用”简介](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)。

## <a name="next-steps"></a>后续步骤

配置 StatusPage 后，可以强制实施会话控制，实时防止组织的敏感数据外泄和渗透。 会话控制从条件访问扩展而来。 [了解如何通过 Microsoft Cloud App Security 强制实施会话控制](/cloud-app-security/proxy-deployment-any-app)。
