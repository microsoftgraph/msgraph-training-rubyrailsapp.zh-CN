# <a name="how-to-run-the-completed-project"></a>如何运行已完成的项目

## <a name="prerequisites"></a>先决条件

若要在此文件夹中运行已完成的项目，您需要以下各项：

- [开发计算机上安装了 Ruby。](https://www.ruby-lang.org/en/downloads/) 如果没有 Ruby，请访问上一链接，查看下载选项。  (**注意：** 本教程是使用 Ruby 版本 2.6.5 编写的。 本指南中的步骤可能与其他版本一样，但尚未测试。) 
- 具有邮箱的个人 Microsoft 帐户Outlook.com Microsoft 工作或学校帐户。

如果你没有 Microsoft 帐户，有几个选项可以获取免费帐户：

- 你可以 [注册新的个人 Microsoft 帐户](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。
- 你可以 [注册 Office 365 开发人员计划](https://developer.microsoft.com/office/dev-program) ，获取免费的 Office 365 订阅。

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a>向 Azure Active Directory 管理中心注册 Web 应用程序

1. 打开浏览器，并转到 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。 使用 **个人帐户**（亦称为“Microsoft 帐户”）或 **工作或学校帐户** 登录。

1. 选择左侧导航栏中的“**Azure Active Directory**”，再选择“**管理**”下的“**应用注册**”。

    ![应用注册的屏幕截图 ](/tutorial/images/aad-portal-app-registrations.png)

1. 选择“新注册”。 在“注册应用”页上，按如下方式设置值。

    - 将“名称”设置为“`Ruby Graph Tutorial`”。
    - 将“受支持的帐户类型”设置为“任何组织目录中的帐户和个人 Microsoft 帐户”。
    - 在“重定向 URI”下，将第一个下拉列表设置为“`Web`”，并将值设置为“`http://localhost:3000/auth/microsoft_graph_auth/callback`”。

    !["注册应用程序"页的屏幕截图](/tutorial/images/aad-register-an-app.png)

1. 选择 **“注册”**。 在 **Ruby Graph 教程** 页面上，复制应用程序 (**客户端**) ID 并保存它，下一步中将需要该值。

    ![新应用注册的应用程序 ID 屏幕截图](/tutorial/images/aad-application-id.png)

1. 选择“管理”下的“证书和密码”。 选择“新客户端密码”按钮。 在“说明”中输入值，并选择一个“过期”选项，再选择“添加”。

    !["添加客户端密码"对话框的屏幕截图](/tutorial/images/aad-new-client-secret.png)

1. 离开此页前，先复制客户端密码值。 将在下一步中用到它。

    > [!IMPORTANT]
    > 此客户端密码不会再次显示，所以请务必现在就复制它。

    ![新添加的客户端密码的屏幕截图](/tutorial/images/aad-copy-client-secret.png)

## <a name="configure-the-sample"></a>配置示例

1. 将文件 `./config/oauth_environment_variables.rb.example` 重命名为 `oauth_environment_variables.rb` 。
1. 编辑 `oauth_environment_variables.rb` 文件并做出以下更改。
    1. 替换为 `YOUR_APP_ID_HERE` 从 **应用** 注册门户获得的应用程序 ID。
    1. 替换为 `YOUR APP PASSWORD HERE` 从应用注册门户获得的密码。
1. 在命令行界面 (CLI) ，导航到此目录并运行以下命令以安装要求。

    ```Shell
    bundle install
    ```

1. 在 CLI 中，运行以下命令以安装包装包。

    ```Shell
    yarn install
    ```

1. 在 CLI 中，运行以下命令以初始化应用的数据库。

    ```Shell
    rake db:migrate
    ```

## <a name="run-the-sample"></a>运行示例

1. 在 CLI 中运行以下命令以启动应用程序。

    ```Shell
    rails server
    ```

1. 打开浏览器，然后转到 `http://localhost:3000`。
