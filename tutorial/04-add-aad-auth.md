<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6322e-101">在本练习中，你将扩展上一练习中的应用程序，以支持 Azure AD 的身份验证。</span><span class="sxs-lookup"><span data-stu-id="6322e-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="6322e-102">若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph，这是必需的。</span><span class="sxs-lookup"><span data-stu-id="6322e-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="6322e-103">在此步骤中，将[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem 集成到应用程序中，并创建自定义 omniauth 策略。</span><span class="sxs-lookup"><span data-stu-id="6322e-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

1. <span data-ttu-id="6322e-104">创建一个单独的文件来保存您的应用程序 ID 和密码。</span><span class="sxs-lookup"><span data-stu-id="6322e-104">Create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="6322e-105">在/config 文件夹中创建`oauth_environment_variables.rb`一个名 **./config**为的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="6322e-105">Create a new file called `oauth_environment_variables.rb` in the **./config** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. <span data-ttu-id="6322e-106">将`YOUR_APP_ID_HERE`替换为应用程序注册门户中的应用程序 ID， `YOUR_APP_SECRET_HERE`并将替换为您生成的密码。</span><span class="sxs-lookup"><span data-stu-id="6322e-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="6322e-107">如果您使用的是源代码管理（如 git），现在可以从源代码管理中排除`oauth_environment_variables.rb`该文件，以避免无意中泄漏您的应用程序 ID 和密码。</span><span class="sxs-lookup"><span data-stu-id="6322e-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="6322e-108">打开 **/config/environment.rb** ，并在`Rails.application.initialize!`行前面添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="6322e-108">Open **./config/environment.rb** and add the following code before the `Rails.application.initialize!` line.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a><span data-ttu-id="6322e-109">安装 OmniAuth</span><span class="sxs-lookup"><span data-stu-id="6322e-109">Setup OmniAuth</span></span>

<span data-ttu-id="6322e-110">你已安装了`omniauth-oauth2` gem，但为了使其能够使用 Azure OAuth 终结点，你需要[创建一个 OAuth2 策略](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy)。</span><span class="sxs-lookup"><span data-stu-id="6322e-110">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="6322e-111">这是一个 Ruby 类，用于定义用于向 Azure 提供程序发出 OAuth 请求的参数。</span><span class="sxs-lookup"><span data-stu-id="6322e-111">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

1. <span data-ttu-id="6322e-112">在 "/lib" \* `microsoft_graph_auth.rb` \* 文件夹 **./lib**中创建一个名为的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="6322e-112">Create a new file called `microsoft_graph_auth.rb` in the **./lib**\`\*\* folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    <span data-ttu-id="6322e-113">请花点时间查看此代码的功能。</span><span class="sxs-lookup"><span data-stu-id="6322e-113">Take a moment to review what this code does.</span></span>

    - <span data-ttu-id="6322e-114">它将设置`client_options`为指定 Microsoft identity platform 终结点。</span><span class="sxs-lookup"><span data-stu-id="6322e-114">It sets the `client_options` to specify the Microsoft identity platform endpoints.</span></span>
    - <span data-ttu-id="6322e-115">它指定应在`scope`授权阶段发送参数。</span><span class="sxs-lookup"><span data-stu-id="6322e-115">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
    - <span data-ttu-id="6322e-116">它将用户`id`的属性映射为用户的唯一 ID。</span><span class="sxs-lookup"><span data-stu-id="6322e-116">It maps the `id` property of the user as the unique ID for the user.</span></span>
    - <span data-ttu-id="6322e-117">它使用访问令牌从 Microsoft Graph 检索用户的配置文件，以填写`raw_info`哈希值。</span><span class="sxs-lookup"><span data-stu-id="6322e-117">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
    - <span data-ttu-id="6322e-118">它将重写回调 URL，以确保它与应用注册门户中的注册回调相匹配。</span><span class="sxs-lookup"><span data-stu-id="6322e-118">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

1. <span data-ttu-id="6322e-119">创建一个名`omniauth_graph.rb`为 " **/config/initializers** " 文件夹的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="6322e-119">Create a new file called `omniauth_graph.rb` in the **./config/initializers** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    <span data-ttu-id="6322e-120">此代码将在应用程序启动时执行。</span><span class="sxs-lookup"><span data-stu-id="6322e-120">This code will execute when the app starts.</span></span> <span data-ttu-id="6322e-121">它使用`microsoft_graph_auth`提供程序加载 OmniAuth 中间件，并使用在**oauth_environment_variables rb**中设置的环境变量进行配置。</span><span class="sxs-lookup"><span data-stu-id="6322e-121">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in **oauth_environment_variables.rb**.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="6322e-122">实施登录</span><span class="sxs-lookup"><span data-stu-id="6322e-122">Implement sign-in</span></span>

<span data-ttu-id="6322e-123">现在已配置 OmniAuth 中间件，您可以转到将登录添加到应用中。</span><span class="sxs-lookup"><span data-stu-id="6322e-123">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span>

1. <span data-ttu-id="6322e-124">在 CLI 中运行以下命令，以生成用于登录和注销的控制器。</span><span class="sxs-lookup"><span data-stu-id="6322e-124">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

    ```Shell
    rails generate controller Auth
    ```

1. <span data-ttu-id="6322e-125">**/App/controllers/auth_controller rb**。</span><span class="sxs-lookup"><span data-stu-id="6322e-125">Open **./app/controllers/auth_controller.rb**.</span></span> <span data-ttu-id="6322e-126">向`AuthController`类中添加回调方法。</span><span class="sxs-lookup"><span data-stu-id="6322e-126">Add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="6322e-127">OAuth 流完成后，OmniAuth 中间件将调用此方法。</span><span class="sxs-lookup"><span data-stu-id="6322e-127">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    <span data-ttu-id="6322e-128">现在，所有这些都是呈现由 OmniAuth 提供的哈希值。</span><span class="sxs-lookup"><span data-stu-id="6322e-128">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="6322e-129">在继续操作之前，你将使用此操作来验证登录是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="6322e-129">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="6322e-130">将路由添加到 **/config/routes.rb**。</span><span class="sxs-lookup"><span data-stu-id="6322e-130">Add the routes to **./config/routes.rb**.</span></span>

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
    ```

1. <span data-ttu-id="6322e-131">启动服务器并浏览到`https://localhost:3000`。</span><span class="sxs-lookup"><span data-stu-id="6322e-131">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="6322e-132">单击 "登录" 按钮，您应会被重定向`https://login.microsoftonline.com`到。</span><span class="sxs-lookup"><span data-stu-id="6322e-132">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="6322e-133">使用你的 Microsoft 帐户登录，并同意请求的权限。</span><span class="sxs-lookup"><span data-stu-id="6322e-133">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="6322e-134">浏览器重定向到应用程序，显示由 OmniAuth 生成的哈希值。</span><span class="sxs-lookup"><span data-stu-id="6322e-134">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

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

## <a name="storing-the-tokens"></a><span data-ttu-id="6322e-135">存储令牌</span><span class="sxs-lookup"><span data-stu-id="6322e-135">Storing the tokens</span></span>

<span data-ttu-id="6322e-136">现在，您可以获取令牌，现在是时候实现将它们存储在应用程序中的方法了。</span><span class="sxs-lookup"><span data-stu-id="6322e-136">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="6322e-137">由于这是一个示例应用程序，因此为简单起见，你将把它们存储在会话中。</span><span class="sxs-lookup"><span data-stu-id="6322e-137">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="6322e-138">实际应用程序将使用更可靠的安全存储解决方案，就像数据库一样。</span><span class="sxs-lookup"><span data-stu-id="6322e-138">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="6322e-139">**/App/controllers/application_controller rb**。</span><span class="sxs-lookup"><span data-stu-id="6322e-139">Open **./app/controllers/application_controller.rb**.</span></span> <span data-ttu-id="6322e-140">将下列方法添加到 `ApplicationController` 类。</span><span class="sxs-lookup"><span data-stu-id="6322e-140">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    <span data-ttu-id="6322e-141">该方法采用 OmniAuth 哈希作为参数并提取相关的信息位，然后将其存储在会话中。</span><span class="sxs-lookup"><span data-stu-id="6322e-141">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

1. <span data-ttu-id="6322e-142">向`ApplicationController`类中添加访问器函数，以检索会话中的用户名、电子邮件地址和访问令牌。</span><span class="sxs-lookup"><span data-stu-id="6322e-142">Add accessor functions to the `ApplicationController` class to retrieve the user name, email address, and access token back out of the session.</span></span>

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

1. <span data-ttu-id="6322e-143">将以下代码添加到将`ApplicationController`在处理任何操作之前运行的类中。</span><span class="sxs-lookup"><span data-stu-id="6322e-143">Add the following code to the `ApplicationController` class that will run before any action is processed.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    <span data-ttu-id="6322e-144">此方法设置 erb （在**应用程序**中）中使用的变量，以在导航栏中显示用户的信息。</span><span class="sxs-lookup"><span data-stu-id="6322e-144">This method sets the variables that the layout (in **application.html.erb**) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="6322e-145">通过在此处添加它，无需在每个单个控制器操作中添加此代码。</span><span class="sxs-lookup"><span data-stu-id="6322e-145">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="6322e-146">但是，这也将对中的`AuthController`操作运行，这并不是最佳的。</span><span class="sxs-lookup"><span data-stu-id="6322e-146">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span>

1. <span data-ttu-id="6322e-147">将以下代码添加到`AuthController` **auth_controller/app/controllers/** 中的类中，以跳过 before 操作。</span><span class="sxs-lookup"><span data-stu-id="6322e-147">Add the following code to the `AuthController` class in **./app/controllers/auth_controller.rb** to skip the before action.</span></span>

    ```ruby
    skip_before_action :set_user
    ```

1. <span data-ttu-id="6322e-148">更新`AuthController`类`callback`中的函数以将标记存储在会话中，并重定向回主页面。</span><span class="sxs-lookup"><span data-stu-id="6322e-148">Update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="6322e-149">将现有的 `callback` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="6322e-149">Replace the existing `callback` function with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="6322e-150">实现注销</span><span class="sxs-lookup"><span data-stu-id="6322e-150">Implement sign-out</span></span>

<span data-ttu-id="6322e-151">在测试此新功能之前，请添加一种注销方式。</span><span class="sxs-lookup"><span data-stu-id="6322e-151">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="6322e-152">将以下操作添加到`AuthController`类中。</span><span class="sxs-lookup"><span data-stu-id="6322e-152">Add the following action to the `AuthController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. <span data-ttu-id="6322e-153">将此操作添加到 **/config/routes.rb**。</span><span class="sxs-lookup"><span data-stu-id="6322e-153">Add this action to **./config/routes.rb**.</span></span>

    ```ruby
    get 'auth/signout'
    ```

1. <span data-ttu-id="6322e-154">重新启动服务器并完成登录过程。</span><span class="sxs-lookup"><span data-stu-id="6322e-154">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="6322e-155">您应该最后返回到主页，但 UI 应更改以指示您已登录。</span><span class="sxs-lookup"><span data-stu-id="6322e-155">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. <span data-ttu-id="6322e-157">单击右上角的用户头像以访问 "**注销**" 链接。</span><span class="sxs-lookup"><span data-stu-id="6322e-157">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="6322e-158">单击 "**注销**" 重置会话并返回到主页。</span><span class="sxs-lookup"><span data-stu-id="6322e-158">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![带有 "注销" 链接的下拉菜单的屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="6322e-160">刷新令牌</span><span class="sxs-lookup"><span data-stu-id="6322e-160">Refreshing tokens</span></span>

<span data-ttu-id="6322e-161">如果您仔细查看 OmniAuth 生成的哈希值，您会注意到哈希中有两个标记`token` ： `refresh_token`和。</span><span class="sxs-lookup"><span data-stu-id="6322e-161">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="6322e-162">中`token`的值是访问令牌，它是在 API 调用的`Authorization`标头中发送的。</span><span class="sxs-lookup"><span data-stu-id="6322e-162">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="6322e-163">这是允许应用代表用户访问 Microsoft Graph 的令牌。</span><span class="sxs-lookup"><span data-stu-id="6322e-163">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="6322e-164">但是，此令牌的生存期较短。</span><span class="sxs-lookup"><span data-stu-id="6322e-164">However, this token is short-lived.</span></span> <span data-ttu-id="6322e-165">令牌在发出后会过期一小时。</span><span class="sxs-lookup"><span data-stu-id="6322e-165">The token expires an hour after it is issued.</span></span> <span data-ttu-id="6322e-166">这是`refresh_token`值变得有用的地方。</span><span class="sxs-lookup"><span data-stu-id="6322e-166">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="6322e-167">刷新令牌允许应用在不要求用户重新登录的情况下请求新的访问令牌。</span><span class="sxs-lookup"><span data-stu-id="6322e-167">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="6322e-168">更新令牌管理代码以实现令牌刷新。</span><span class="sxs-lookup"><span data-stu-id="6322e-168">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="6322e-169">打开 **/app/controllers/application_controller rb**并在顶部添加以下`require`语句：</span><span class="sxs-lookup"><span data-stu-id="6322e-169">Open **./app/controllers/application_controller.rb** and add the following `require` statements at the top:</span></span>

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. <span data-ttu-id="6322e-170">将下列方法添加到 `ApplicationController` 类。</span><span class="sxs-lookup"><span data-stu-id="6322e-170">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    <span data-ttu-id="6322e-171">此方法使用[oauth2](https://github.com/oauth-xx/oauth2) gem （ `omniauth-oauth2` gem 的依赖项）刷新标记，并更新会话。</span><span class="sxs-lookup"><span data-stu-id="6322e-171">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

1. <span data-ttu-id="6322e-172">将当前`access_token`方法替换为以下项。</span><span class="sxs-lookup"><span data-stu-id="6322e-172">Replace the current `access_token` method with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    <span data-ttu-id="6322e-173">它将首先检查它是否接近到期，而不是仅从会话中返回令牌。</span><span class="sxs-lookup"><span data-stu-id="6322e-173">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="6322e-174">如果是，则在返回令牌之前将其刷新。</span><span class="sxs-lookup"><span data-stu-id="6322e-174">If it is, then it will refresh before returning the token.</span></span>
