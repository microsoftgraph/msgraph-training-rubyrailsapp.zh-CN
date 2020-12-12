<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将扩展上一练习中的应用程序，以支持使用 Azure AD 进行身份验证。 这是获取调用 Microsoft Graph 所需的 OAuth 访问令牌所必需的。 在此步骤中，你将 [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) 到应用程序中集成，并创建自定义 OmniAuth 策略。

1. 创建单独的文件来保存应用 ID 和密码。 创建在 `oauth_environment_variables.rb` **./config** 文件夹中调用的新文件，并添加以下代码。

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. 替换为 `YOUR_APP_ID_HERE` 应用程序注册门户中的应用程序 ID，并 `YOUR_APP_SECRET_HERE` 替换为生成的密码。

    > [!IMPORTANT]
    > 如果你使用的是源代码管理（如 git），那么现在应该将文件从源代码管理中排除，以避免意外泄露应用 ID 和 `oauth_environment_variables.rb` 密码。

1. 打开 **./config/environment.rb，** 在行前添加以下 `Rails.application.initialize!` 代码。

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a>设置 OmniAuth

你已安装了该，但为了使它与 Azure OAuth 终结点一起工作，你需要 `omniauth-oauth2` [创建 OAuth2 策略](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy)。 这是一个 Ruby 类，用于定义向 Azure 提供程序提出 OAuth 请求的参数。

1. 创建在 `microsoft_graph_auth.rb` **./lib**"** 文件夹中调用的新文件，并添加以下代码。

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    花些时间查看此代码执行哪些功能。

    - 它设置 `client_options` 指定 Microsoft 标识平台终结点。
    - 它指定应在 `scope` 授权阶段发送参数。
    - 它将 `id` 用户属性映射为用户的唯一 ID。
    - 它使用访问令牌从 Microsoft Graph 检索用户配置文件以填写 `raw_info` 哈希。
    - 它会覆盖回调 URL，以确保它与应用注册门户中注册的回调匹配。

1. 创建在 `omniauth_graph.rb` **./config/initializers** 文件夹中调用的新文件，并添加以下代码。

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    此代码将在应用启动时执行。 它将使用提供程序加载 OmniAuth 中间件，该中间件使用 `microsoft_graph_auth` **oauth_environment_variables.rb 中设置的环境变量进行配置**。

## <a name="implement-sign-in"></a>实现登录

现已配置 OmniAuth 中间件，可以继续向应用添加登录。

1. 在 CLI 中运行以下命令，生成用于登录和注销的控制器。

    ```Shell
    rails generate controller Auth
    ```

1. 打开 **./app/controllers/auth_controller.rb**。 将回调方法添加到 `AuthController` 类。 OAuth 流完成后，OmniAuth 中间件将调用此方法。

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    现在，这一切就是呈现 OmniAuth 提供的哈希。 在继续之前，你将使用此验证登录是否正常工作。

1. 将路由添加到 **./config/routes.rb**。

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', :to => 'auth#callback', :via => [:get, :post]
    ```

1. 启动服务器并浏览到 `https://localhost:3000` 。 单击登录按钮，应重定向到 `https://login.microsoftonline.com` 。 使用 Microsoft 帐户登录，并同意请求的权限。 浏览器重定向到应用，显示 OmniAuth 生成的哈希。

    ```json
    {
      "provider": "microsoft_graph_auth",
      "uid": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
      "info": {
        "name": null
      },
      "credentials": {
        "token": "eyJ0eXAi...",
        "refresh_token": "OAQABAAA...",
        "expires_at": 1529517383,
        "expires": true
      },
      "extra": {
        "raw_info": {
          "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users(displayName,mail,mailboxSettings,userPrincipalName)/$entity",
          "displayName": "Lynne Robbins",
          "mail": "LynneR@contoso.OnMicrosoft.com",
          "userPrincipalName": "LynneR@contoso.OnMicrosoft.com",
          "id": "d294e784-840e-4f9f-bb1e-95c0a75f2f18@2d18179c-4386-4cbd-8891-7fd867c4f62e",
          "mailboxSettings": {
            "archiveFolder": "AAMkAGI2...",
            "timeZone": "Pacific Standard Time",
            "delegateMeetingMessageDeliveryOptions": "sendToDelegateOnly",
            "dateFormat": "M/d/yyyy",
            "timeFormat": "h:mm tt",
            "automaticRepliesSetting": {
              "status": "disabled",
              "externalAudience": "all",
              "internalReplyMessage": "",
              "externalReplyMessage": "",
              "scheduledStartDateTime": {
                "dateTime": "2020-12-09T17:00:00.0000000",
                "timeZone": "UTC"
              },
              "scheduledEndDateTime": {
                "dateTime": "2020-12-10T17:00:00.0000000",
                "timeZone": "UTC"
              }
            },
            "language": {
              "locale": "en-US",
              "displayName": "English (United States)"
            },
            "workingHours": {
              "daysOfWeek": [
                "monday",
                "tuesday",
                "wednesday",
                "thursday",
                "friday"
              ],
              "startTime": "08:00:00.0000000",
              "endTime": "17:00:00.0000000",
              "timeZone": {
                "name": "Pacific Standard Time"
              }
            }
          }
        }
      }
    }
    ```

## <a name="storing-the-tokens"></a>存储令牌

现在，你可以获取令牌，是时候实现在应用中存储令牌的方法了。 由于这是一个示例应用，为了简单起见，你将将它们存储在会话中。 实际应用将使用更可靠的安全存储解决方案，如数据库。

1. 打开 **./app/controllers/application_controller.rb**。 将下列方法添加到 `ApplicationController` 类。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    该方法将 OmniAuth 哈希作为参数，并提取相关信息位，然后将该信息存储在会话中。

1. 将访问器函数添加到类，以检索用户名、电子邮件地址和访问令牌 `ApplicationController` ，以从会话退出。

    ```ruby
    def user_name
      session[:user_name]
    end

    def user_email
      session[:user_email]
    end

    def user_timezone
      session[:user_timezone]
    end

    def access_token
      session[:graph_token_hash][:token]
    end
    ```

1. 将以下代码添加到将在处理任何 `ApplicationController` 操作之前运行的类。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    此方法设置布局在 **(l.erb** application.htm中用于) 导航栏中显示用户信息的变量。 通过在此处添加它，不必在单个控制器操作中添加此代码。 但是，这也将对中的操作运行 `AuthController` ，这不是最佳操作。

1. 将以下代码添加到 `AuthController` **./app/controllers/auth_controller.rb** 中的类，以跳过之前的操作。

    ```ruby
    skip_before_action :set_user
    ```

1. 更新 `callback` 类中的函数以在会话中存储令牌并 `AuthController` 重定向回主页。 将现有的 `callback` 函数替换为以下内容。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a>实现注销

测试此新功能之前，请添加一种注销方法。

1. 将以下操作添加到 `AuthController` 类。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. 将此操作添加到 **./config/routes.rb**。

    ```ruby
    get 'auth/signout'
    ```

1. 重新启动服务器并完成登录过程。 你最终应返回主页，但 UI 应更改以指示你已登录。

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. 单击右上角的用户头像以访问 **"注销"** 链接。 单击 **"注销** "可重置会话，并返回到主页。

    ![包含"注销"链接的下拉菜单屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>刷新令牌

如果你仔细查看 OmniAuth 生成的哈希，你会注意到哈希中有两个标记： `token` 和 `refresh_token` 。 其中的值 `token` 是访问令牌，在 API 调用标头中 `Authorization` 发送。 这是允许应用代表用户访问 Microsoft Graph 的令牌。

但是，此令牌是短期的。 令牌在颁发后一小时过期。 这是值 `refresh_token` 变得有用的位置。 刷新令牌允许应用请求新的访问令牌，而无需用户重新登录。 更新令牌管理代码以实施令牌刷新。

1. 打开 **./app/controllers/application_controller.rb，** 在顶部 `require` 添加以下语句：

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. 将下列方法添加到 `ApplicationController` 类。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    此方法使用 [oauth2](https://github.com/oauth-xx/oauth2) (依赖的) 刷新令牌并 `omniauth-oauth2` 更新会话。

1. 将当前 `access_token` 方法替换为以下内容。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    它将首先检查令牌是否接近过期，而不是仅从会话返回令牌。 如果是，它将在返回令牌之前刷新。
