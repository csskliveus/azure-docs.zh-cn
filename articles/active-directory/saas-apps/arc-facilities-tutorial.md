---
title: 教程：Azure Active Directory 单一登录 (SSO) 与 ARC Facilities 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 ARC Facilities 之间配置单一登录。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/08/2020
ms.author: jeedes
ms.openlocfilehash: 8cbbe3ada5d39e550daa25b6ac8a662f5c98f994
ms.sourcegitcommit: 67b44a02af0c8d615b35ec5e57a29d21419d7668
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/06/2021
ms.locfileid: "97914631"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-arc-facilities"></a>教程：Azure Active Directory 单一登录 (SSO) 与 ARC Facilities 集成

在本教程中，你将了解如何将 ARC Facilities 与 Azure Active Directory (Azure AD) 集成。 将 ARC Facilities 与 Azure AD 集成后，可以：

* 在 Azure AD 中控制谁有权访问 ARC Facilities。
* 让用户使用其 Azure AD 帐户自动登录到 ARC Facilities。
* 在一个中心位置（Azure 门户）管理帐户。


## <a name="prerequisites"></a>先决条件

若要开始操作，需备齐以下项目：

* 一个 Azure AD 订阅。 如果没有订阅，可以获取一个[免费帐户](https://azure.microsoft.com/free/)。
* 启用了 ARC Facilities 单一登录 (SSO) 的订阅。

## <a name="scenario-description"></a>方案描述

本教程在测试环境中配置并测试 Azure AD SSO。

* ARC Facilities 支持 IDP  发起的 SSO

* ARC Facilities 支持 **实时** 用户预配

> [!NOTE]
> 此应用程序的标识符是一个固定字符串值，因此只能在一个租户中配置一个实例。

## <a name="adding-arc-facilities-from-the-gallery"></a>从库中添加 ARC Facilities

若要配置 ARC Facilities 与 Azure AD 的集成，需要从库中将 ARC Facilities 添加到托管 SaaS 应用列表。

1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 Azure 门户。
1. 在左侧导航窗格中，选择“Azure Active Directory”服务  。
1. 导航到“企业应用程序”，选择“所有应用程序”   。
1. 若要添加新的应用程序，请选择“新建应用程序”  。
1. 在“从库中添加”部分的搜索框中，键入“ARC Facilities”   。
1. 从结果面板中选择“ARC Facilities”，然后添加该应用  。 在该应用添加到租户时等待几秒钟。

## <a name="configure-and-test-azure-ad-single-sign-on-for-arc-facilities"></a>配置和测试 ARC Facilities 的 Azure AD 单一登录

使用名为 **B.Simon** 的测试用户配置和测试 ARC Facilities 的 Azure AD SSO。 若要使 SSO 有效，需要在 Azure AD 用户与 ARC Facilities 相关用户之间建立关联。

若要为 ARC Facilities 配置和测试 Azure AD SSO，请完成以下构建基块：

1. **[配置 Azure AD SSO](#configure-azure-ad-sso)** - 使用户能够使用此功能。
    1. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 B. Simon 测试 Azure AD 单一登录。
    1. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 B. Simon 能够使用 Azure AD 单一登录。
1. **[配置 ARC Facilities SSO](#configure-arc-facilities-sso)** - 在应用程序端配置单一登录设置。
    1. **[创建 ARC Facilities 测试用户](#create-arc-facilities-test-user)** - 在 ARC Facilities 中创建 B.Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
1. **[测试 SSO](#test-sso)** - 验证配置是否正常工作。

## <a name="configure-azure-ad-sso"></a>配置 Azure AD SSO

按照下列步骤在 Azure 门户中启用 Azure AD SSO。

1. 在 Azure 门户的 ARC Facilities 应用程序集成页上，找到“管理”部分，并选择“单一登录”  。
1. 在“选择单一登录方法”页上选择“SAML” 。
1. 在“设置 SAML 单一登录”页面上，单击“基本 SAML 配置”旁边的铅笔图标以编辑设置 。

   ![编辑基本 SAML 配置](common/edit-urls.png)

1. 在基本 SAML 配置部分，应用程序进行了预配置，且已通过 Azure 预填充了必要的 URL。 用户需要单击“保存”  按钮来保存配置。

1. ARC Facilities 应用程序需要特定格式的 SAML 断言，这要求向 SAML 令牌属性配置添加自定义属性映射。 以下屏幕截图显示了默认属性的列表。 单击“编辑”图标以打开“用户属性”对话框  。

    ![显示“用户属性”对话框的屏幕截图，其中标出了“编辑”图标。](common/edit-attribute.png)

1. 除了上述属性，ARC Facilities 应用程序还要求在 SAML 响应中传递回更多的属性。 在“组声明(预览)”对话框中的“用户属性和声明”部分，执行以下步骤   ：

    a. 单击“声明中返回的组”旁边的 **笔**。 

    ![显示“用户属性和声明”部分的屏幕截图，其中笔位于“声明中返回的组”旁边。](./media/arc-facilities-tutorial/config01.png)

    ![显示“组声明”部分的屏幕截图，其中选中了“所有组”和“组 ID”，并标出了“保存”按钮。](./media/arc-facilities-tutorial/config02.png)

    b. 从单选列表中选择“所有组”。 

    c. 选择“组 ID”作为“源属性”  。 

    d. 单击“ **保存**”。

    > [!NOTE]
    > ARC Facilities 需要分配给应用程序的用户的角色。 请在 Azure AD 中设置这些角色，以便可为用户分配相应的角色。 若要了解如何在 Azure AD 中配置角色，请参阅[此文](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps#app-roles-ui)。

1. 在“使用 SAML 设置单一登录”页的“SAML 签名证书”部分中，找到“证书(Base64)”，选择“下载”以下载该证书并将其保存到计算机上   。

    ![证书下载链接](common/certificatebase64.png)

1. 在“设置 ARC Facilities”部分，根据要求复制相应的 URL。 

    ![复制配置 URL](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户

在本部分，我们将在 Azure 门户中创建名为 B.Simon 的测试用户。

1. 在 Azure 门户的左侧窗格中，依次选择“Azure Active Directory”、“用户”和“所有用户”  。
1. 选择屏幕顶部的“新建用户”。
1. 在“用户”属性中执行以下步骤：
   1. 在“名称”字段中，输入 `B.Simon`。  
   1. 在“用户名”字段中输入 username@companydomain.extension。 例如，`B.Simon@contoso.com` 。
   1. 选中“显示密码”复选框，然后记下“密码”框中显示的值。
   1. 单击“创建”。

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分，我们通过授予 B. Simon 访问 ARC Facilities 的权限，使其能够使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”。  
1. 在应用程序列表中，选择“ARC Facilities”  。
1. 在应用的概述页中，找到“管理”部分，选择“用户和组”   。
1. 选择“添加用户”，然后在“添加分配”对话框中选择“用户和组”。
1. 在“用户和组”对话框中，从“用户”列表中选择“B.Simon”，然后单击屏幕底部的“选择”按钮。   
1. 如果你希望将某角色分配给用户，可以从“选择角色”下拉列表中选择该角色。 如果尚未为此应用设置任何角色，你将看到选择了“默认访问权限”角色。
1. 在“添加分配”对话框中，单击“分配”按钮。  

## <a name="configure-arc-facilities-sso"></a>配置 ARC Facilities SSO

若要在 **ARC Facilities** 端配置单一登录，需要将下载的“证书(Base64)”  以及从 Azure 门户复制的相应 URL 发送给 [ARC Facilities 支持团队](mailto:support@arcfacilities.com)。 他们会对此进行设置，使两端的 SAML SSO 连接均正确设置。

### <a name="create-arc-facilities-test-user"></a>创建 ARC Facilities 测试用户

在本部分，我们将在 ARC Facilities 中创建一个名为 Britta Simon 的用户。 ARC Facilities 支持默认情况下启用的实时用户预配。 此部分不存在任何操作项。 如果 ARC Facilities 中尚不存在用户，则会在身份验证后创建一个新用户。

## <a name="test-sso"></a>测试 SSO 

在本部分，你将使用以下选项测试 Azure AD 单一登录配置。

* 在 Azure 门户中单击“测试此应用程序”后，应会自动登录到为其设置了 SSO 的 ARC Facilities

* 你可使用 Microsoft 的“我的应用”。 单击“我的应用”中的 ARC Facilities 磁贴时，应会自动登录到为其设置了 SSO 的 ARC Facilities。 有关“我的应用”的详细信息，请参阅[“我的应用”简介](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)。


## <a name="next-steps"></a>后续步骤

配置 ARC Facilities 后，可以强制实施会话控制，实时防止组织的敏感数据外泄和渗透。 会话控制从条件访问扩展而来。 [了解如何通过 Microsoft Cloud App Security 强制实施会话控制](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app)。