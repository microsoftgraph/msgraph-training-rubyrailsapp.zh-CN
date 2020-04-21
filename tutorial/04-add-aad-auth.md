<!-- markdownlint-disable MD002 MD041 -->

在本练习中，你将扩展上一练习中的应用程序，以支持 Azure AD 的身份验证。 若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph，这是必需的。 在此步骤中，将[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem 集成到应用程序中，并创建自定义 omniauth 策略。

1. 创建一个单独的文件来保存您的应用程序 ID 和密码。 在/config 文件夹中创建`oauth_environment_variables.rb`一个名 **./config**为的新文件，并添加以下代码。

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. 将`YOUR_APP_ID_HERE`替换为应用程序注册门户中的应用程序 ID， `YOUR_APP_SECRET_HERE`并将替换为您生成的密码。

    > [!IMPORTANT]
    > 如果您使用的是源代码管理（如 git），现在可以从源代码管理中排除`oauth_environment_variables.rb`该文件，以避免无意中泄漏您的应用程序 ID 和密码。

1. 打开 **/config/environment.rb** ，并在`Rails.application.initialize!`行前面添加以下代码。

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a>安装 OmniAuth

你已安装了`omniauth-oauth2` gem，但为了使其能够使用 Azure OAuth 终结点，你需要[创建一个 OAuth2 策略](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy)。 这是一个 Ruby 类，用于定义用于向 Azure 提供程序发出 OAuth 请求的参数。

1. 在 "/lib" * `microsoft_graph_auth.rb` * 文件夹 **./lib**中创建一个名为的新文件，并添加以下代码。

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    请花点时间查看此代码的功能。

    - 它将设置`client_options`为指定 Microsoft identity platform 终结点。
    - 它指定应在`scope`授权阶段发送参数。
    - 它将用户`id`的属性映射为用户的唯一 ID。
    - 它使用访问令牌从 Microsoft Graph 检索用户的配置文件，以填写`raw_info`哈希值。
    - 它将重写回调 URL，以确保它与应用注册门户中的注册回调相匹配。

1. 创建一个名`omniauth_graph.rb`为 " **/config/initializers** " 文件夹的新文件，并添加以下代码。

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    此代码将在应用程序启动时执行。 它使用`microsoft_graph_auth`提供程序加载 OmniAuth 中间件，并使用在**oauth_environment_variables rb**中设置的环境变量进行配置。

## <a name="implement-sign-in"></a>实施登录

现在已配置 OmniAuth 中间件，您可以转到将登录添加到应用中。

1. 在 CLI 中运行以下命令，以生成用于登录和注销的控制器。

    ```Shell
    rails generate controller Auth
    ```

1. **/App/controllers/auth_controller rb**。 向`AuthController`类中添加回调方法。 OAuth 流完成后，OmniAuth 中间件将调用此方法。

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    现在，所有这些都是呈现由 OmniAuth 提供的哈希值。 在继续操作之前，你将使用此操作来验证登录是否正常工作。

1. 将路由添加到 **/config/routes.rb**。

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
    ```

1. 启动服务器并浏览到`https://localhost:3000`。 单击 "登录" 按钮，您应会被重定向`https://login.microsoftonline.com`到。 使用你的 Microsoft 帐户登录，并同意请求的权限。 浏览器重定向到应用程序，显示由 OmniAuth 生成的哈希值。

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
          "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
          "id": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
          "businessPhones": [
            "+1 425 555 0109"
          ],
          "displayName": "Adele Vance",
          "givenName": "Adele",
          "jobTitle": "Retail Manager",
          "mail": "AdeleV@contoso.onmicrosoft.com",
          "mobilePhone": null,
          "officeLocation": "18/2111",
          "preferredLanguage": "en-US",
          "surname": "Vance",
          "userPrincipalName": "AdeleV@contoso.onmicrosoft.com"
        }
      }
    }
    ```

## <a name="storing-the-tokens"></a>存储令牌

现在，您可以获取令牌，现在是时候实现将它们存储在应用程序中的方法了。 由于这是一个示例应用程序，因此为简单起见，你将把它们存储在会话中。 实际应用程序将使用更可靠的安全存储解决方案，就像数据库一样。

1. **/App/controllers/application_controller rb**。 将下列方法添加到 `ApplicationController` 类。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    该方法采用 OmniAuth 哈希作为参数并提取相关的信息位，然后将其存储在会话中。

1. 向`ApplicationController`类中添加访问器函数，以检索会话中的用户名、电子邮件地址和访问令牌。

    ```ruby
    def user_name
      session[:user_name]
    end

    def user_email
      session[:user_email]
    end

    def access_token
      session[:graph_token_hash][:token]
    end
    ```

1. 将以下代码添加到将`ApplicationController`在处理任何操作之前运行的类中。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    此方法设置 erb （在**应用程序**中）中使用的变量，以在导航栏中显示用户的信息。 通过在此处添加它，无需在每个单个控制器操作中添加此代码。 但是，这也将对中的`AuthController`操作运行，这并不是最佳的。

1. 将以下代码添加到`AuthController` **auth_controller/app/controllers/** 中的类中，以跳过 before 操作。

    ```ruby
    skip_before_action :set_user
    ```

1. 更新`AuthController`类`callback`中的函数以将标记存储在会话中，并重定向回主页面。 将现有的 `callback` 函数替换为以下内容。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a>实现注销

在测试此新功能之前，请添加一种注销方式。

1. 将以下操作添加到`AuthController`类中。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. 将此操作添加到 **/config/routes.rb**。

    ```ruby
    get 'auth/signout'
    ```

1. 重新启动服务器并完成登录过程。 您应该最后返回到主页，但 UI 应更改以指示您已登录。

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. 单击右上角的用户头像以访问 "**注销**" 链接。 单击 "**注销**" 重置会话并返回到主页。

    ![带有 "注销" 链接的下拉菜单的屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>刷新令牌

如果您仔细查看 OmniAuth 生成的哈希值，您会注意到哈希中有两个标记`token` ： `refresh_token`和。 中`token`的值是访问令牌，它是在 API 调用的`Authorization`标头中发送的。 这是允许应用代表用户访问 Microsoft Graph 的令牌。

但是，此令牌的生存期较短。 令牌在发出后会过期一小时。 这是`refresh_token`值变得有用的地方。 刷新令牌允许应用在不要求用户重新登录的情况下请求新的访问令牌。 更新令牌管理代码以实现令牌刷新。

1. 打开 **/app/controllers/application_controller rb**并在顶部添加以下`require`语句：

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. 将下列方法添加到 `ApplicationController` 类。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    此方法使用[oauth2](https://github.com/oauth-xx/oauth2) gem （ `omniauth-oauth2` gem 的依赖项）刷新标记，并更新会话。

1. 将当前`access_token`方法替换为以下项。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    它将首先检查它是否接近到期，而不是仅从会话中返回令牌。 如果是，则在返回令牌之前将其刷新。
