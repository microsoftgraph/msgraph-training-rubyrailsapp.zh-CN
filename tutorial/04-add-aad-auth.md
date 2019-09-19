<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ebf8e-101">在本练习中，你将扩展上一练习中的应用程序，以支持 Azure AD 的身份验证。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="ebf8e-102">若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph，这是必需的。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="ebf8e-103">在此步骤中，将[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem 集成到应用程序中，并创建自定义 omniauth 策略。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

<span data-ttu-id="ebf8e-104">首先，创建一个单独的文件来保存您的应用程序 ID 和密码。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-104">First, create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="ebf8e-105">在`./config`文件夹中创建一个`oauth_environment_variables.rb`名为的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-105">Create a new file called `oauth_environment_variables.rb` in the `./config` folder, and add the following code.</span></span>

```ruby
ENV['AZURE_APP_ID'] = 'YOUR_APP_ID_HERE'
ENV['AZURE_APP_SECRET'] = 'YOUR_APP_SECRET_HERE'
ENV['AZURE_SCOPES'] = 'openid profile email offline_access user.read calendars.read'
```

<span data-ttu-id="ebf8e-106">将`YOUR_APP_ID_HERE`替换为应用程序注册门户中的应用程序 ID， `YOUR_APP_SECRET_HERE`并将替换为您生成的密码。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="ebf8e-107">如果您使用的是源代码管理（如 git），现在可以从源代码管理中排除`oauth_environment_variables.rb`该文件，以避免无意中泄漏您的应用程序 ID 和密码。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

<span data-ttu-id="ebf8e-108">现在，添加代码以加载此文件（如果存在）。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-108">Now add code to load this file if it's present.</span></span> <span data-ttu-id="ebf8e-109">打开`./config/environment.rb`文件，并在`Rails.application.initialize!`行前面添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-109">Open the `./config/environment.rb` file and add the following code before the `Rails.application.initialize!` line.</span></span>

```ruby
# Load OAuth settings
oauth_environment_variables = File.join(Rails.root, 'config', 'oauth_environment_variables.rb')
load(oauth_environment_variables) if File.exist?(oauth_environment_variables)
```

## <a name="setup-omniauth"></a><span data-ttu-id="ebf8e-110">安装 OmniAuth</span><span class="sxs-lookup"><span data-stu-id="ebf8e-110">Setup OmniAuth</span></span>

<span data-ttu-id="ebf8e-111">你已安装了`omniauth-oauth2` gem，但为了使其能够使用 Azure OAuth 终结点，你需要[创建一个 OAuth2 策略](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy)。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-111">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="ebf8e-112">这是一个 Ruby 类，用于定义用于向 Azure 提供程序发出 OAuth 请求的参数。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-112">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

<span data-ttu-id="ebf8e-113">在`./lib`文件夹中创建一个`microsoft_graph_auth.rb`名为的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-113">Create a new file called `microsoft_graph_auth.rb` in the `./lib` folder, and add the following code.</span></span>

```ruby
require 'omniauth-oauth2'

module OmniAuth
  module Strategies
    # Implements an OmniAuth strategy to get a Microsoft Graph
    # compatible token from Azure AD
    class MicrosoftGraphAuth < OmniAuth::Strategies::OAuth2
      option :name, :microsoft_graph_auth

      DEFAULT_SCOPE = 'openid email profile User.Read'.freeze

      # Configure the Microsoft identity platform endpoints
      option  :client_options,
              site:          'https://login.microsoftonline.com',
              authorize_url: '/common/oauth2/v2.0/authorize',
              token_url:     '/common/oauth2/v2.0/token'

      # Send the scope parameter during authorize
      option :authorize_options, [:scope]

      # Unique ID for the user is the id field
      uid { raw_info['id'] }

      # Get additional information after token is retrieved
      extra do
        {
          'raw_info' => raw_info
        }
      end

      def raw_info
        # Get user profile information from the /me endpoint
        @raw_info ||= access_token.get('https://graph.microsoft.com/v1.0/me').parsed
      end

      def authorize_params
        super.tap do |params|
          params['scope'.to_sym] = request.params['scope'] if request.params['scope']
          params[:scope] ||= DEFAULT_SCOPE
        end
      end

      # Override callback URL
      # OmniAuth by default passes the entire URL of the callback, including
      # query parameters. Azure fails validation because that doesn't match the
      # registered callback.
      def callback_url
        options[:redirect_uri] || (full_host + script_name + callback_path)
      end
    end
  end
end
```

<span data-ttu-id="ebf8e-114">请花点时间查看此代码的功能。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-114">Take a moment to review what this code does.</span></span>

- <span data-ttu-id="ebf8e-115">它将设置`client_options`为指定 Microsoft identity platform 终结点。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-115">It sets the `client_options` to specify the Microsoft identity platform endpoints.</span></span>
- <span data-ttu-id="ebf8e-116">它指定应在`scope`授权阶段发送参数。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-116">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
- <span data-ttu-id="ebf8e-117">它将用户`id`的属性映射为用户的唯一 ID。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-117">It maps the `id` property of the user as the unique ID for the user.</span></span>
- <span data-ttu-id="ebf8e-118">它使用访问令牌从 Microsoft Graph 检索用户的配置文件，以填写`raw_info`哈希值。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-118">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
- <span data-ttu-id="ebf8e-119">它将重写回调 URL，以确保它与应用注册门户中的注册回调相匹配。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-119">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

<span data-ttu-id="ebf8e-120">现在我们已定义策略，我们需要将 OmniAuth 配置为使用它。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-120">Now that we've defined the strategy, we need to configure OmniAuth to use it.</span></span> <span data-ttu-id="ebf8e-121">在`./config/initializers`文件夹中创建一个`omniauth_graph.rb`名为的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-121">Create a new file called `omniauth_graph.rb` in the `./config/initializers` folder, and add the following code.</span></span>

```ruby
require 'microsoft_graph_auth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :microsoft_graph_auth,
           ENV['AZURE_APP_ID'],
           ENV['AZURE_APP_SECRET'],
           scope: ENV['AZURE_SCOPES']
end
```

<span data-ttu-id="ebf8e-122">此代码将在应用程序启动时执行。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-122">This code will execute when the app starts.</span></span> <span data-ttu-id="ebf8e-123">它将加载具有`microsoft_graph_auth`提供程序的 OmniAuth 中间件，并配置了中`oauth_environment_variables.rb`设置的环境变量。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-123">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in `oauth_environment_variables.rb`.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="ebf8e-124">实施登录</span><span class="sxs-lookup"><span data-stu-id="ebf8e-124">Implement sign-in</span></span>

<span data-ttu-id="ebf8e-125">现在已配置 OmniAuth 中间件，您可以转到将登录添加到应用中。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-125">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span> <span data-ttu-id="ebf8e-126">在 CLI 中运行以下命令，以生成用于登录和注销的控制器。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-126">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

```Shell
rails generate controller Auth
```

<span data-ttu-id="ebf8e-127">打开 `./app/controllers/auth_controller.rb` 文件。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-127">Open the `./app/controllers/auth_controller.rb` file.</span></span> <span data-ttu-id="ebf8e-128">将下列方法添加到 `AuthController` 类。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-128">Add the following method to the `AuthController` class.</span></span>

```ruby
def signin
  redirect_to '/auth/microsoft_graph_auth'
end
```

<span data-ttu-id="ebf8e-129">所有此方法都将重定向到 OmniAuth 预期调用我们的自定义策略的路由。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-129">All this method does is redirect to the route that OmniAuth expects to invoke our custom strategy.</span></span>

<span data-ttu-id="ebf8e-130">接下来，向`AuthController`类中添加回调方法。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-130">Next, add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="ebf8e-131">OAuth 流完成后，OmniAuth 中间件将调用此方法。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-131">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Temporary for testing!
  render json: data.to_json
end
```

<span data-ttu-id="ebf8e-132">现在，所有这些都是呈现由 OmniAuth 提供的哈希值。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-132">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="ebf8e-133">在继续操作之前，我们将使用此操作来验证登录是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-133">We'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="ebf8e-134">在测试之前，我们需要将路由添加到`./config/routes.rb`。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-134">Before we test, we need to add the routes to `./config/routes.rb`.</span></span>

```ruby
get 'auth/signin'

# Add route for OmniAuth callback
match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
```

<span data-ttu-id="ebf8e-135">现在，更新视图以使用该`signin`操作。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-135">Now update the views to use the `signin` action.</span></span> <span data-ttu-id="ebf8e-136">打开`./app/views/layouts/application.html.erb`。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-136">Open `./app/views/layouts/application.html.erb`.</span></span> <span data-ttu-id="ebf8e-137">将行替换`<a href="#" class="nav-link">Sign In</a>`为以下代码行。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-137">Replace the line `<a href="#" class="nav-link">Sign In</a>` with the following.</span></span>

```html
<%= link_to "Sign In", {:controller => :auth, :action => :signin}, :class => "nav-link" %>
```

<span data-ttu-id="ebf8e-138">打开`./app/views/home/index.html.erb`文件，并将`<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>`该行替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-138">Open the `./app/views/home/index.html.erb` file and replace the `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` line with the following.</span></span>

```html
<%= link_to "Click here to sign in", {:controller => :auth, :action => :signin}, :class => "btn btn-primary btn-large" %>
```

<span data-ttu-id="ebf8e-139">启动服务器并浏览到`https://localhost:3000`。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-139">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="ebf8e-140">单击 "登录" 按钮，您应会被重定向`https://login.microsoftonline.com`到。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-140">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="ebf8e-141">使用你的 Microsoft 帐户登录，并同意请求的权限。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-141">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="ebf8e-142">浏览器重定向到应用程序，显示由 OmniAuth 生成的哈希值。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-142">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

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

## <a name="storing-the-tokens"></a><span data-ttu-id="ebf8e-143">存储令牌</span><span class="sxs-lookup"><span data-stu-id="ebf8e-143">Storing the tokens</span></span>

<span data-ttu-id="ebf8e-144">现在，您可以获取令牌，现在是时候实现将它们存储在应用程序中的方法了。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-144">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="ebf8e-145">由于这是一个示例应用程序，因此为简单起见，你将把它们存储在会话中。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-145">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="ebf8e-146">实际应用程序将使用更可靠的安全存储解决方案，就像数据库一样。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-146">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="ebf8e-147">打开 `./app/controllers/application_controller.rb` 文件。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-147">Open the `./app/controllers/application_controller.rb` file.</span></span> <span data-ttu-id="ebf8e-148">你将在此处添加所有令牌管理方法。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-148">You'll add all of our token management methods here.</span></span> <span data-ttu-id="ebf8e-149">由于其他所有控制器都会继承`ApplicationController`该类，因此它们可以使用这些方法访问令牌。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-149">Because all of the other controllers inherit the `ApplicationController` class, they'll be able to use these methods to access the tokens.</span></span>

<span data-ttu-id="ebf8e-150">将下列方法添加到 `ApplicationController` 类。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-150">Add the following method to the `ApplicationController` class.</span></span> <span data-ttu-id="ebf8e-151">该方法采用 OmniAuth 哈希作为参数并提取相关的信息位，然后将其存储在会话中。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-151">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

```ruby
def save_in_session(auth_hash)
  # Save the token info
  session[:graph_token_hash] = auth_hash.dig(:credentials)
  # Save the user's display name
  session[:user_name] = auth_hash.dig(:extra, :raw_info, :displayName)
  # Save the user's email address
  # Use the mail field first. If that's empty, fall back on
  # userPrincipalName
  session[:user_email] = auth_hash.dig(:extra, :raw_info, :mail) ||
                         auth_hash.dig(:extra, :raw_info, :userPrincipalName)
end
```

<span data-ttu-id="ebf8e-152">现在，添加用于从会话中检索用户名、电子邮件地址和访问令牌的访问器函数。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-152">Now add accessor functions to retrieve the user name, email address, and access token back out of the session.</span></span>

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

<span data-ttu-id="ebf8e-153">最后，添加一些将在处理任何操作之前运行的代码。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-153">Finally, add some code that will run before any action is processed.</span></span>

```ruby
before_action :set_user

def set_user
  @user_name = user_name
  @user_email = user_email
end
```

<span data-ttu-id="ebf8e-154">此方法设置布局（in `application.html.erb`）用来在导航栏中显示用户信息的变量。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-154">This method sets the variables that the layout (in `application.html.erb`) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="ebf8e-155">通过在此处添加它，无需在每个单个控制器操作中添加此代码。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-155">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="ebf8e-156">但是，这也将对中的`AuthController`操作运行，这并不是最佳的。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-156">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span> <span data-ttu-id="ebf8e-157">将下面的代码添加到`AuthController`中`./app/controllers/auth_controller.rb`的类，以跳过 before 操作。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-157">Add the following code to the `AuthController` class in `./app/controllers/auth_controller.rb` to skip the before action.</span></span>

```ruby
skip_before_action :set_user
```

<span data-ttu-id="ebf8e-158">然后，更新`AuthController`类`callback`中的函数以存储会话中的标记，并重定向回主页面。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-158">Then, update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="ebf8e-159">将现有的 `callback` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-159">Replace the existing `callback` function with the following.</span></span>

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Save the data in the session
  save_in_session data

  redirect_to root_url
end
```

## <a name="implement-sign-out"></a><span data-ttu-id="ebf8e-160">实现注销</span><span class="sxs-lookup"><span data-stu-id="ebf8e-160">Implement sign-out</span></span>

<span data-ttu-id="ebf8e-161">在测试此新功能之前，请添加一种注销方式。将以下操作添加到`AuthController`类中。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-161">Before you test this new feature, add a way to sign out. Add the following action to the `AuthController` class.</span></span>

```ruby
def signout
  reset_session
  redirect_to root_url
end
```

<span data-ttu-id="ebf8e-162">将此操作添加`./config/routes.rb`到。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-162">Add this action to `./config/routes.rb`.</span></span>

```ruby
get 'auth/signout'
```

<span data-ttu-id="ebf8e-163">现在，更新视图以使用该`signout`操作。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-163">Now update the view to use the `signout` action.</span></span> <span data-ttu-id="ebf8e-164">打开`./app/views/layouts/application.html.erb`。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-164">Open `./app/views/layouts/application.html.erb`.</span></span> <span data-ttu-id="ebf8e-165">将此行`<a href="#" class="dropdown-item">Sign Out</a>`替换为：</span><span class="sxs-lookup"><span data-stu-id="ebf8e-165">Replace the line `<a href="#" class="dropdown-item">Sign Out</a>` with:</span></span>

```html
<%= link_to "Sign Out", {:controller => :auth, :action => :signout}, :class => "dropdown-item" %>
```

<span data-ttu-id="ebf8e-166">重新启动服务器并完成登录过程。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-166">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="ebf8e-167">您应该最后返回到主页，但 UI 应更改以指示您已登录。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-167">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

<span data-ttu-id="ebf8e-169">单击右上角的用户头像以访问 "**注销**" 链接。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-169">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="ebf8e-170">单击 "**注销**" 重置会话并返回到主页。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-170">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![带有 "注销" 链接的下拉菜单的屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="ebf8e-172">刷新令牌</span><span class="sxs-lookup"><span data-stu-id="ebf8e-172">Refreshing tokens</span></span>

<span data-ttu-id="ebf8e-173">如果您仔细查看 OmniAuth 生成的哈希值，您会注意到哈希中有两个标记`token` ： `refresh_token`和。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-173">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="ebf8e-174">中`token`的值是访问令牌，它是在 API 调用的`Authorization`标头中发送的。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-174">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="ebf8e-175">这是允许应用代表用户访问 Microsoft Graph 的令牌。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-175">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="ebf8e-176">但是，此令牌的生存期较短。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-176">However, this token is short-lived.</span></span> <span data-ttu-id="ebf8e-177">令牌在发出后会过期一小时。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-177">The token expires an hour after it is issued.</span></span> <span data-ttu-id="ebf8e-178">这是`refresh_token`值变得有用的地方。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-178">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="ebf8e-179">刷新令牌允许应用在不要求用户重新登录的情况下请求新的访问令牌。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-179">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="ebf8e-180">更新令牌管理代码以实现令牌刷新。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-180">Update the token management code to implement token refresh.</span></span>

<span data-ttu-id="ebf8e-181">打开`./app/controllers/application_controller.rb`并在顶部添加`require`以下语句：</span><span class="sxs-lookup"><span data-stu-id="ebf8e-181">Open `./app/controllers/application_controller.rb` and add the following `require` statements at the top:</span></span>

```ruby
require 'microsoft_graph_auth'
require 'oauth2'
```

<span data-ttu-id="ebf8e-182">然后，将以下方法添加到`ApplicationController`类中。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-182">Then add the following method to the `ApplicationController` class.</span></span>

```ruby
def refresh_tokens(token_hash)
  oauth_strategy = OmniAuth::Strategies::MicrosoftGraphAuth.new(
    nil, ENV['AZURE_APP_ID'], ENV['AZURE_APP_SECRET']
  )

  token = OAuth2::AccessToken.new(
    oauth_strategy.client, token_hash[:token],
    refresh_token: token_hash[:refresh_token]
  )

  # Refresh the tokens
  new_tokens = token.refresh!.to_hash.slice(:access_token, :refresh_token, :expires_at)

  # Rename token key
  new_tokens[:token] = new_tokens.delete :access_token

  # Store the new hash
  session[:graph_token_hash] = new_tokens
end
```

<span data-ttu-id="ebf8e-183">此方法使用[oauth2](https://github.com/oauth-xx/oauth2) gem （ `omniauth-oauth2` gem 的依赖项）刷新标记，并更新会话。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-183">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

<span data-ttu-id="ebf8e-184">现在，将此方法用于。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-184">Now put this method to use.</span></span> <span data-ttu-id="ebf8e-185">若要执行此操作， `access_token`请将`ApplicationController`类中的访问器设置得更聪明一点。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-185">To do that, make the `access_token` accessor in the `ApplicationController` class a bit smarter.</span></span> <span data-ttu-id="ebf8e-186">它将首先检查它是否接近到期，而不是仅从会话中返回令牌。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-186">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="ebf8e-187">如果是，则在返回令牌之前将其刷新。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-187">If it is, then it will refresh before returning the token.</span></span> <span data-ttu-id="ebf8e-188">将当前`access_token`方法替换为以下项。</span><span class="sxs-lookup"><span data-stu-id="ebf8e-188">Replace the current `access_token` method with the following.</span></span>

```ruby
def access_token
  token_hash = session[:graph_token_hash]

  # Get the expiry time - 5 minutes
  expiry = Time.at(token_hash[:expires_at] - 300)

  if Time.now > expiry
    # Token expired, refresh
    new_hash = refresh_tokens token_hash
    new_hash[:token]
  else
    token_hash[:token]
  end
end
```
