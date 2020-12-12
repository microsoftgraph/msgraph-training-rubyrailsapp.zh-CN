<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ad324-101">在此练习中，你将扩展上一练习中的应用程序，以支持使用 Azure AD 进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="ad324-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="ad324-102">这是获取调用 Microsoft Graph 所需的 OAuth 访问令牌所必需的。</span><span class="sxs-lookup"><span data-stu-id="ad324-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="ad324-103">在此步骤中，你将 [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) 到应用程序中集成，并创建自定义 OmniAuth 策略。</span><span class="sxs-lookup"><span data-stu-id="ad324-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

1. <span data-ttu-id="ad324-104">创建单独的文件来保存应用 ID 和密码。</span><span class="sxs-lookup"><span data-stu-id="ad324-104">Create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="ad324-105">创建在 `oauth_environment_variables.rb` **./config** 文件夹中调用的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="ad324-105">Create a new file called `oauth_environment_variables.rb` in the **./config** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. <span data-ttu-id="ad324-106">替换为 `YOUR_APP_ID_HERE` 应用程序注册门户中的应用程序 ID，并 `YOUR_APP_SECRET_HERE` 替换为生成的密码。</span><span class="sxs-lookup"><span data-stu-id="ad324-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="ad324-107">如果你使用的是源代码管理（如 git），那么现在应该将文件从源代码管理中排除，以避免意外泄露应用 ID 和 `oauth_environment_variables.rb` 密码。</span><span class="sxs-lookup"><span data-stu-id="ad324-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="ad324-108">打开 **./config/environment.rb，** 在行前添加以下 `Rails.application.initialize!` 代码。</span><span class="sxs-lookup"><span data-stu-id="ad324-108">Open **./config/environment.rb** and add the following code before the `Rails.application.initialize!` line.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a><span data-ttu-id="ad324-109">设置 OmniAuth</span><span class="sxs-lookup"><span data-stu-id="ad324-109">Setup OmniAuth</span></span>

<span data-ttu-id="ad324-110">你已安装了该，但为了使它与 Azure OAuth 终结点一起工作，你需要 `omniauth-oauth2` [创建 OAuth2 策略](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy)。</span><span class="sxs-lookup"><span data-stu-id="ad324-110">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="ad324-111">这是一个 Ruby 类，用于定义向 Azure 提供程序提出 OAuth 请求的参数。</span><span class="sxs-lookup"><span data-stu-id="ad324-111">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

1. <span data-ttu-id="ad324-112">创建在 `microsoft_graph_auth.rb` **./lib**"\*\* 文件夹中调用的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="ad324-112">Create a new file called `microsoft_graph_auth.rb` in the **./lib**\`\*\* folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    <span data-ttu-id="ad324-113">花些时间查看此代码执行哪些功能。</span><span class="sxs-lookup"><span data-stu-id="ad324-113">Take a moment to review what this code does.</span></span>

    - <span data-ttu-id="ad324-114">它设置 `client_options` 指定 Microsoft 标识平台终结点。</span><span class="sxs-lookup"><span data-stu-id="ad324-114">It sets the `client_options` to specify the Microsoft identity platform endpoints.</span></span>
    - <span data-ttu-id="ad324-115">它指定应在 `scope` 授权阶段发送参数。</span><span class="sxs-lookup"><span data-stu-id="ad324-115">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
    - <span data-ttu-id="ad324-116">它将 `id` 用户属性映射为用户的唯一 ID。</span><span class="sxs-lookup"><span data-stu-id="ad324-116">It maps the `id` property of the user as the unique ID for the user.</span></span>
    - <span data-ttu-id="ad324-117">它使用访问令牌从 Microsoft Graph 检索用户配置文件以填写 `raw_info` 哈希。</span><span class="sxs-lookup"><span data-stu-id="ad324-117">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
    - <span data-ttu-id="ad324-118">它会覆盖回调 URL，以确保它与应用注册门户中注册的回调匹配。</span><span class="sxs-lookup"><span data-stu-id="ad324-118">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

1. <span data-ttu-id="ad324-119">创建在 `omniauth_graph.rb` **./config/initializers** 文件夹中调用的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="ad324-119">Create a new file called `omniauth_graph.rb` in the **./config/initializers** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    <span data-ttu-id="ad324-120">此代码将在应用启动时执行。</span><span class="sxs-lookup"><span data-stu-id="ad324-120">This code will execute when the app starts.</span></span> <span data-ttu-id="ad324-121">它将使用提供程序加载 OmniAuth 中间件，该中间件使用 `microsoft_graph_auth` **oauth_environment_variables.rb 中设置的环境变量进行配置**。</span><span class="sxs-lookup"><span data-stu-id="ad324-121">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in **oauth_environment_variables.rb**.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="ad324-122">实现登录</span><span class="sxs-lookup"><span data-stu-id="ad324-122">Implement sign-in</span></span>

<span data-ttu-id="ad324-123">现已配置 OmniAuth 中间件，可以继续向应用添加登录。</span><span class="sxs-lookup"><span data-stu-id="ad324-123">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span>

1. <span data-ttu-id="ad324-124">在 CLI 中运行以下命令，生成用于登录和注销的控制器。</span><span class="sxs-lookup"><span data-stu-id="ad324-124">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

    ```Shell
    rails generate controller Auth
    ```

1. <span data-ttu-id="ad324-125">打开 **./app/controllers/auth_controller.rb**。</span><span class="sxs-lookup"><span data-stu-id="ad324-125">Open **./app/controllers/auth_controller.rb**.</span></span> <span data-ttu-id="ad324-126">将回调方法添加到 `AuthController` 类。</span><span class="sxs-lookup"><span data-stu-id="ad324-126">Add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="ad324-127">OAuth 流完成后，OmniAuth 中间件将调用此方法。</span><span class="sxs-lookup"><span data-stu-id="ad324-127">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    <span data-ttu-id="ad324-128">现在，这一切就是呈现 OmniAuth 提供的哈希。</span><span class="sxs-lookup"><span data-stu-id="ad324-128">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="ad324-129">在继续之前，你将使用此验证登录是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="ad324-129">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="ad324-130">将路由添加到 **./config/routes.rb**。</span><span class="sxs-lookup"><span data-stu-id="ad324-130">Add the routes to **./config/routes.rb**.</span></span>

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', :to => 'auth#callback', :via => [:get, :post]
    ```

1. <span data-ttu-id="ad324-131">启动服务器并浏览到 `https://localhost:3000` 。</span><span class="sxs-lookup"><span data-stu-id="ad324-131">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="ad324-132">单击登录按钮，应重定向到 `https://login.microsoftonline.com` 。</span><span class="sxs-lookup"><span data-stu-id="ad324-132">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="ad324-133">使用 Microsoft 帐户登录，并同意请求的权限。</span><span class="sxs-lookup"><span data-stu-id="ad324-133">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="ad324-134">浏览器重定向到应用，显示 OmniAuth 生成的哈希。</span><span class="sxs-lookup"><span data-stu-id="ad324-134">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

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

## <a name="storing-the-tokens"></a><span data-ttu-id="ad324-135">存储令牌</span><span class="sxs-lookup"><span data-stu-id="ad324-135">Storing the tokens</span></span>

<span data-ttu-id="ad324-136">现在，你可以获取令牌，是时候实现在应用中存储令牌的方法了。</span><span class="sxs-lookup"><span data-stu-id="ad324-136">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="ad324-137">由于这是一个示例应用，为了简单起见，你将将它们存储在会话中。</span><span class="sxs-lookup"><span data-stu-id="ad324-137">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="ad324-138">实际应用将使用更可靠的安全存储解决方案，如数据库。</span><span class="sxs-lookup"><span data-stu-id="ad324-138">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="ad324-139">打开 **./app/controllers/application_controller.rb**。</span><span class="sxs-lookup"><span data-stu-id="ad324-139">Open **./app/controllers/application_controller.rb**.</span></span> <span data-ttu-id="ad324-140">将下列方法添加到 `ApplicationController` 类。</span><span class="sxs-lookup"><span data-stu-id="ad324-140">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    <span data-ttu-id="ad324-141">该方法将 OmniAuth 哈希作为参数，并提取相关信息位，然后将该信息存储在会话中。</span><span class="sxs-lookup"><span data-stu-id="ad324-141">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

1. <span data-ttu-id="ad324-142">将访问器函数添加到类，以检索用户名、电子邮件地址和访问令牌 `ApplicationController` ，以从会话退出。</span><span class="sxs-lookup"><span data-stu-id="ad324-142">Add accessor functions to the `ApplicationController` class to retrieve the user name, email address, and access token back out of the session.</span></span>

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

1. <span data-ttu-id="ad324-143">将以下代码添加到将在处理任何 `ApplicationController` 操作之前运行的类。</span><span class="sxs-lookup"><span data-stu-id="ad324-143">Add the following code to the `ApplicationController` class that will run before any action is processed.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    <span data-ttu-id="ad324-144">此方法设置布局在 **(l.erb** application.htm中用于) 导航栏中显示用户信息的变量。</span><span class="sxs-lookup"><span data-stu-id="ad324-144">This method sets the variables that the layout (in **application.html.erb**) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="ad324-145">通过在此处添加它，不必在单个控制器操作中添加此代码。</span><span class="sxs-lookup"><span data-stu-id="ad324-145">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="ad324-146">但是，这也将对中的操作运行 `AuthController` ，这不是最佳操作。</span><span class="sxs-lookup"><span data-stu-id="ad324-146">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span>

1. <span data-ttu-id="ad324-147">将以下代码添加到 `AuthController` **./app/controllers/auth_controller.rb** 中的类，以跳过之前的操作。</span><span class="sxs-lookup"><span data-stu-id="ad324-147">Add the following code to the `AuthController` class in **./app/controllers/auth_controller.rb** to skip the before action.</span></span>

    ```ruby
    skip_before_action :set_user
    ```

1. <span data-ttu-id="ad324-148">更新 `callback` 类中的函数以在会话中存储令牌并 `AuthController` 重定向回主页。</span><span class="sxs-lookup"><span data-stu-id="ad324-148">Update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="ad324-149">将现有的 `callback` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="ad324-149">Replace the existing `callback` function with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="ad324-150">实现注销</span><span class="sxs-lookup"><span data-stu-id="ad324-150">Implement sign-out</span></span>

<span data-ttu-id="ad324-151">测试此新功能之前，请添加一种注销方法。</span><span class="sxs-lookup"><span data-stu-id="ad324-151">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="ad324-152">将以下操作添加到 `AuthController` 类。</span><span class="sxs-lookup"><span data-stu-id="ad324-152">Add the following action to the `AuthController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. <span data-ttu-id="ad324-153">将此操作添加到 **./config/routes.rb**。</span><span class="sxs-lookup"><span data-stu-id="ad324-153">Add this action to **./config/routes.rb**.</span></span>

    ```ruby
    get 'auth/signout'
    ```

1. <span data-ttu-id="ad324-154">重新启动服务器并完成登录过程。</span><span class="sxs-lookup"><span data-stu-id="ad324-154">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="ad324-155">你最终应返回主页，但 UI 应更改以指示你已登录。</span><span class="sxs-lookup"><span data-stu-id="ad324-155">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. <span data-ttu-id="ad324-157">单击右上角的用户头像以访问 **"注销"** 链接。</span><span class="sxs-lookup"><span data-stu-id="ad324-157">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="ad324-158">单击 **"注销** "可重置会话，并返回到主页。</span><span class="sxs-lookup"><span data-stu-id="ad324-158">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![包含"注销"链接的下拉菜单屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="ad324-160">刷新令牌</span><span class="sxs-lookup"><span data-stu-id="ad324-160">Refreshing tokens</span></span>

<span data-ttu-id="ad324-161">如果你仔细查看 OmniAuth 生成的哈希，你会注意到哈希中有两个标记： `token` 和 `refresh_token` 。</span><span class="sxs-lookup"><span data-stu-id="ad324-161">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="ad324-162">其中的值 `token` 是访问令牌，在 API 调用标头中 `Authorization` 发送。</span><span class="sxs-lookup"><span data-stu-id="ad324-162">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="ad324-163">这是允许应用代表用户访问 Microsoft Graph 的令牌。</span><span class="sxs-lookup"><span data-stu-id="ad324-163">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="ad324-164">但是，此令牌是短期的。</span><span class="sxs-lookup"><span data-stu-id="ad324-164">However, this token is short-lived.</span></span> <span data-ttu-id="ad324-165">令牌在颁发后一小时过期。</span><span class="sxs-lookup"><span data-stu-id="ad324-165">The token expires an hour after it is issued.</span></span> <span data-ttu-id="ad324-166">这是值 `refresh_token` 变得有用的位置。</span><span class="sxs-lookup"><span data-stu-id="ad324-166">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="ad324-167">刷新令牌允许应用请求新的访问令牌，而无需用户重新登录。</span><span class="sxs-lookup"><span data-stu-id="ad324-167">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="ad324-168">更新令牌管理代码以实施令牌刷新。</span><span class="sxs-lookup"><span data-stu-id="ad324-168">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="ad324-169">打开 **./app/controllers/application_controller.rb，** 在顶部 `require` 添加以下语句：</span><span class="sxs-lookup"><span data-stu-id="ad324-169">Open **./app/controllers/application_controller.rb** and add the following `require` statements at the top:</span></span>

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. <span data-ttu-id="ad324-170">将下列方法添加到 `ApplicationController` 类。</span><span class="sxs-lookup"><span data-stu-id="ad324-170">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    <span data-ttu-id="ad324-171">此方法使用 [oauth2](https://github.com/oauth-xx/oauth2) (依赖的) 刷新令牌并 `omniauth-oauth2` 更新会话。</span><span class="sxs-lookup"><span data-stu-id="ad324-171">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

1. <span data-ttu-id="ad324-172">将当前 `access_token` 方法替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="ad324-172">Replace the current `access_token` method with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    <span data-ttu-id="ad324-173">它将首先检查令牌是否接近过期，而不是仅从会话返回令牌。</span><span class="sxs-lookup"><span data-stu-id="ad324-173">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="ad324-174">如果是，它将在返回令牌之前刷新。</span><span class="sxs-lookup"><span data-stu-id="ad324-174">If it is, then it will refresh before returning the token.</span></span>
