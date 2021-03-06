---
title: "教程：Azure Active Directory 与 Slack 集成 | Microsoft Docs"
description: "了解如何在 Azure Active Directory 和 Slack 之间配置单一登录。"
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.assetid: ffc5e73f-6c38-4bbb-876a-a7dd269d4e1c
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/28/2018
ms.author: jeedes
ms.openlocfilehash: cd0cecde7f98e73911e7dec734cffeeee6f09a72
ms.sourcegitcommit: 782d5955e1bec50a17d9366a8e2bf583559dca9e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/02/2018
---
# <a name="tutorial-azure-active-directory-integration-with-slack"></a>教程：Azure Active Directory 与 Slack 集成

在本教程中，了解如何将 Slack 与 Azure Active Directory (Azure AD) 集成。

将 Slack 与 Azure AD 集成提供以下优势：

- 可在 Azure AD 中控制谁有权访问 Slack
- 可以让用户使用其 Azure AD 帐户自动登录到 Slack（单一登录）
- 可以在一个中心位置（即 Azure 门户）管理帐户

如需了解有关 SaaS 应用与 Azure AD 集成的详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](active-directory-appssoaccess-whatis.md)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 Slack 的集成，需要具有以下项：

- Azure AD 订阅
- 已启用 Slack 单一登录的订阅

> [!NOTE]
> 为了测试本教程中的步骤，我们不建议使用生产环境。

测试本教程中的步骤应遵循以下建议：

- 除非必要，请勿使用生产环境。
- 如果没有 Azure AD 试用环境，可以在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。

## <a name="scenario-description"></a>方案描述
在本教程中，将在测试环境中测试 Azure AD 单一登录。 本教程中概述的方案包括两个主要构建基块：

1. 从库中添加 Slack
2. 配置和测试 Azure AD 单一登录

## <a name="adding-slack-from-the-gallery"></a>从库中添加 Slack
要配置 Slack 与 Azure AD 的集成，需要从库中将 Slack 添加到托管 SaaS 应用列表。

**若要从库中添加 Slack，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)**的左侧导航面板中，单击“Azure Active Directory”图标。 

    ![Active Directory][1]

2. 导航到“企业应用程序”。 然后转到“所有应用程序”。

    ![应用程序][2]
    
3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![应用程序][3]

4. 在搜索框中，键入“Slack”。

    ![创建 Azure AD 测试用户](./media/active-directory-saas-slack-tutorial/tutorial_slack_search.png)

5. 在结果窗格中，选择“Slack”，并单击“添加”按钮添加该应用程序。

    ![创建 Azure AD 测试用户](./media/active-directory-saas-slack-tutorial/tutorial_slack_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录
在本部分中，将基于名为“Britta Simon”的测试用户配置和测试 Slack 的 Azure AD 单一登录。

若要运行单一登录，Azure AD 需要知道与 Azure AD 用户相对应的 Slack 用户。 换句话说，需要建立 Azure AD 用户与 Slack 中相关用户之间的链接关系。

通过将 Slack 中“用户名”的值指定为 Azure AD 中“用户名”的值来建立此关联。

若要配置并测试 Slack 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configuring-azure-ad-single-sign-on)** - 让用户使用此功能。
2. **[创建 Azure AD 测试用户](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
3. **[创建 Slack 测试用户](#creating-a-slack-test-user)** - 在 Slack 中创建 Britta Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
4. **[分配 Azure AD 测试用户](#assigning-the-azure-ad-test-user)** - 让 Britta Simon 使用 Azure AD 单一登录。
5. **[测试单一登录](#testing-single-sign-on)** - 验证配置是否正常工作。

### <a name="configuring-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

本部分中，将在 Azure 门户中启用 Azure AD 单一登录并在 Slack 应用程序中配置单一登录。

**若要配置 Slack 的 Azure AD 单一登录，请执行以下步骤：**

1. 在 Azure 门户中的 **Slack** 应用程序集成页上，单击“单一登录”。

    ![配置单一登录][4]

2. 在“单一登录”对话框中，选择“基于 SAML 的单一登录”作为“模式”以启用单一登录。
 
    ![配置单一登录](./media/active-directory-saas-slack-tutorial/tutorial_slack_samlbase.png)

3. 在“Slack 域和 URL”部分中，执行以下步骤：

    ![配置单一登录](./media/active-directory-saas-slack-tutorial/tutorial_slack_url.png)

    a. 在“登录 URL”文本框中，使用以下模式键入 URL：`https://<companyname>.slack.com`

    b. 在“标识符”文本框中，键入 URL：`https://slack.com`

    > [!NOTE] 
    > 此值不是真实值。 必须使用实际登录 URL 更新此值。 请联系 [Slack 支持团队](https://slack.com/help/contact)获取此值
     
4. Slack 应用程序需要特定格式的 SAML 断言。 请为此应用程序配置以下声明。 可以在应用程序集成页的“用户属性”部分管理这些属性的值。 以下屏幕截图显示一个示例。
    
    ![配置单一登录](./media/active-directory-saas-slack-tutorial/tutorial_slack_attribute.png)

    > [!NOTE] 
    > 如果用户有使 Office365 分配的**电子邮件地址**，则只会填充该地址，否则**电子邮件地址**声明将不会出现在 SAML 令牌中。

5. 在“单一登录”对话框上的“用户属性”部分中，选择“user.mail”作为**用户标识符**，并针对下表中所示的每一行，执行以下步骤：
    
    | 属性名称 | 属性值 |
    | --- | --- |
    | first_name | user.givenname |
    | last_name | user.surname |
    | User.Email | user.mail |  
    | User.Username | user.userprincipalname |

    a. 单击“属性”打开“编辑属性”对话框并执行以下步骤：

    ![配置单一登录](./media/active-directory-saas-slack-tutorial/tutorial_slack_attribute1.png)

    a. 在“名称”文本框中，键入为该行显示的属性名称。
    
    b. 在“值”列表中，选择为该行显示的属性值。

    c. 将“命名空间”留空。
    
    d.单击“下一步”。 单击 **“确定”**

6. 在“SAML 签名证书”部分中，单击“证书(base64)”，并在计算机上保存证书文件。

    ![配置单一登录](./media/active-directory-saas-slack-tutorial/tutorial_slack_certificate.png)

7. 单击“保存”按钮。

    ![配置单一登录](./media/active-directory-saas-slack-tutorial/tutorial_general_400.png)

8. 在“Slack 配置”部分中，单击“配置 Slack ”打开“配置登录”窗口。 从“快速参考”部分中复制“SAML 实体 ID 和 SAML 单一登录服务 URL”。

    ![配置单一登录](./media/active-directory-saas-slack-tutorial/tutorial_slack_configure.png) 

9.  在另一个 Web 浏览器窗口中，以管理员身份登录到 Slack 公司站点。

10.  导航到“Microsoft Azure AD”，并转到“团队设置”。

     ![在应用端配置单一登录](./media/active-directory-saas-slack-tutorial/tutorial_slack_001.png)

11.  在“团队设置”部分中，单击“身份验证”选项卡，并单击“更改设置”。

     ![在应用端配置单一登录](./media/active-directory-saas-slack-tutorial/tutorial_slack_002.png)

12. 在“SAML 身份验证设置”对话框中，执行以下步骤：

    ![在应用端配置单一登录](./media/active-directory-saas-slack-tutorial/tutorial_slack_003.png)

    a.  在“SAML 2.0 终结点(HTTP)”文本框中，粘贴从 Azure 门户复制的“SAML 单一登录服务 URL”值。

    b.  在“标识提供程序颁发者”文本框中，粘贴从 Azure 门户复制的“SAML 实体 ID”值。

    c.  在记事本中打开下载的证书，将其内容复制到剪贴板，并将其粘贴到“公共证书”文本框中。

    d.单击“下一步”。 根据 Slack 团队的需要配置上述三个设置。 有关设置的详细信息，请在此处查看 **Slack 的 SSO 配置指南**。 `https://get.slack.help/hc/articles/220403548-Guide-to-single-sign-on-with-Slack%60`

    e.  单击“保存配置”。
     
    <!-- Deselect **Allow users to change their email address**.

    e.  Select **Allow users to choose their own username**.

    f.  As **Authentication for your team must be used by**, select **It’s optional**. -->

> [!TIP]
> 之后在设置应用时，就可以在 [Azure 门户](https://portal.azure.com)中阅读这些说明的简明版本了！  从“Active Directory”>“企业应用程序”部分添加此应用后，只需单击“单一登录”选项卡，即可通过底部的“配置”部分访问嵌入式文档。 可在此处阅读有关嵌入式文档功能的详细信息：[ Azure AD 嵌入式文档]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>创建 Azure AD 测试用户
本部分的目的是在 Azure 门户中创建名为 Britta Simon 的测试用户。

![创建 Azure AD 用户][100]

**若要在 Azure AD 中创建测试用户，请执行以下步骤：**

1. 在 **Azure 门户**的左侧导航窗格中，单击“Azure Active Directory”图标。

    ![创建 Azure AD 测试用户](./media/active-directory-saas-slack-tutorial/create_aaduser_01.png) 

2. 若要显示用户列表，请转到“用户和组”，单击“所有用户”。
    
    ![创建 Azure AD 测试用户](./media/active-directory-saas-slack-tutorial/create_aaduser_02.png) 

3. 若要打开“用户”对话框，请在对话框顶部单击“添加”。
 
    ![创建 Azure AD 测试用户](./media/active-directory-saas-slack-tutorial/create_aaduser_03.png) 

4. 在“用户”对话框页上，执行以下步骤：
 
    ![创建 Azure AD 测试用户](./media/active-directory-saas-slack-tutorial/create_aaduser_04.png) 

    a. 在“名称”文本框中，键入 **BrittaSimon**。

    b. 在“用户名”文本框中，键入 BrittaSimon 的“电子邮件地址”。

    c. 选择“显示密码”并记下“密码”的值。

    d.单击“下一步”。 单击“创建”。
 
### <a name="creating-a-slack-test-user"></a>创建 Slack 测试用户

本部分的目的是在 Slack 中创建名为 Britta Simon 的用户。 Slack 支持在默认情况下启用的实时预配。

此部分不存在任何操作项。 尝试访问 Slack 期间，如果尚不存在用户，则会创建一个新用户。

> [!NOTE]
> 如果需要手动创建用户，则需联系 [Slack 支持团队](https://slack.com/help/contact)。

### <a name="assigning-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分中，通过授予 Britta Simon 访问 Slack 的权限，允许她使用 Azure 单一登录。

![分配用户][200] 

**要将 Britta Simon 分配到 Slack，请执行以下步骤：**

1. 在 Azure 门户中打开应用程序视图，导航到目录视图，接着转到“企业应用程序”，并单击“所有应用程序”。

    ![分配用户][201] 

2. 在应用程序列表中，选择“Slack”。

    ![配置单一登录](./media/active-directory-saas-slack-tutorial/tutorial_slack_app.png) 

3. 在左侧菜单中，单击“用户和组”。

    ![分配用户][202] 

4. 单击“添加”按钮。 然后在“添加分配”对话框中选择“用户和组”。

    ![分配用户][203]

5. 在“用户和组”对话框的“用户”列表中，选择“Britta Simon”。

6. 在“用户和组”对话框中单击“选择”按钮。

7. 在“添加分配”对话框中单击“分配”按钮。
    
### <a name="testing-single-sign-on"></a>测试单一登录

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的“Slack”磁贴时，用户应自动登录到 Slack 应用程序。

## <a name="additional-resources"></a>其他资源

* [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](active-directory-saas-tutorial-list.md)
* [什么是使用 Azure Active Directory 的应用程序访问和单一登录？](active-directory-appssoaccess-whatis.md)



<!--Image references-->

[1]: ./media/active-directory-saas-slack-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-slack-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-slack-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-slack-tutorial/tutorial_general_04.png

[100]: ./media/active-directory-saas-slack-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-slack-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-slack-tutorial/tutorial_general_201.png
[202]: ./media/active-directory-saas-slack-tutorial/tutorial_general_202.png
[203]: ./media/active-directory-saas-slack-tutorial/tutorial_general_203.png

