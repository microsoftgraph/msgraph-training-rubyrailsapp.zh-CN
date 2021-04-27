<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b9100-101">在此练习中，你将使用 [Ruby on Rails](https://rubyonrails.org/) 生成 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="b9100-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span>

1. <span data-ttu-id="b9100-102">如果尚未安装 Rails，可以通过以下命令从命令行界面 (CLI) 安装它。</span><span class="sxs-lookup"><span data-stu-id="b9100-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    gem install rails -v 6.1.3.1
    ```

1. <span data-ttu-id="b9100-103">打开 CLI，导航到你拥有创建文件权限的目录，然后运行以下命令以创建新的 Rails 应用。</span><span class="sxs-lookup"><span data-stu-id="b9100-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

    ```Shell
    rails new graph-tutorial
    ```

1. <span data-ttu-id="b9100-104">导航到此新目录并输入以下命令以启动本地 Web 服务器。</span><span class="sxs-lookup"><span data-stu-id="b9100-104">Navigate to this new directory and enter the following command to start a local web server.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="b9100-105">打开浏览器，并导航到 `http://localhost:3000`。</span><span class="sxs-lookup"><span data-stu-id="b9100-105">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="b9100-106">如果一切正常，你将看到"Yay！</span><span class="sxs-lookup"><span data-stu-id="b9100-106">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="b9100-107">你使用 Rails！"</span><span class="sxs-lookup"><span data-stu-id="b9100-107">You're on Rails!"</span></span> <span data-ttu-id="b9100-108">消息。</span><span class="sxs-lookup"><span data-stu-id="b9100-108">message.</span></span> <span data-ttu-id="b9100-109">如果你看不到该消息，请查看 [Rails 入门指南](http://guides.rubyonrails.org/)。</span><span class="sxs-lookup"><span data-stu-id="b9100-109">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

## <a name="install-gems"></a><span data-ttu-id="b9100-110">安装 gem</span><span class="sxs-lookup"><span data-stu-id="b9100-110">Install gems</span></span>

<span data-ttu-id="b9100-111">在继续之前，请安装一些稍后将使用的其他 gem：</span><span class="sxs-lookup"><span data-stu-id="b9100-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="b9100-112">[omniauth-oauth2，](https://github.com/omniauth/omniauth-oauth2) 用于处理登录和 OAuth 令牌流。</span><span class="sxs-lookup"><span data-stu-id="b9100-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="b9100-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) 向 OmniAuth 添加 CSRF 保护。</span><span class="sxs-lookup"><span data-stu-id="b9100-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="b9100-114">[用于调用](https://github.com/jnunemaker/httparty)Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="b9100-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="b9100-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) 用于存储数据库中的会话。</span><span class="sxs-lookup"><span data-stu-id="b9100-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

1. <span data-ttu-id="b9100-116">打开 **./Gemfile** 并添加以下行。</span><span class="sxs-lookup"><span data-stu-id="b9100-116">Open **./Gemfile** and add the following lines.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. <span data-ttu-id="b9100-117">在 CLI 中，运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="b9100-117">In your CLI, run the following command.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="b9100-118">在 CLI 中，运行以下命令以配置数据库以存储会话。</span><span class="sxs-lookup"><span data-stu-id="b9100-118">In your CLI, run the following commands to configure the database for storing sessions.</span></span>

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. <span data-ttu-id="b9100-119">在 `session_store.rb` **./config/initializers** 目录中创建名为 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="b9100-119">Create a new file called `session_store.rb` in the **./config/initializers** directory, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="b9100-120">设计应用</span><span class="sxs-lookup"><span data-stu-id="b9100-120">Design the app</span></span>

<span data-ttu-id="b9100-121">在此部分中，你将为应用创建基本 UI。</span><span class="sxs-lookup"><span data-stu-id="b9100-121">In this section you'll create the basic UI for the app.</span></span>

1. <span data-ttu-id="b9100-122">打开 **./app/views/layouts/application.html.erb，** 并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="b9100-122">Open **./app/views/layouts/application.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    <span data-ttu-id="b9100-123">此代码为[简单样式设置添加 Bootstrap，](http://getbootstrap.com/)为一些简单图标添加[Fabric Core。](https://developer.microsoft.com/fluentui#/get-started#fabric-core)</span><span class="sxs-lookup"><span data-stu-id="b9100-123">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) for some simple icons.</span></span> <span data-ttu-id="b9100-124">它还定义具有导航栏的全局布局。</span><span class="sxs-lookup"><span data-stu-id="b9100-124">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="b9100-125">打开 **./app/assets/stylesheets/application.css，** 将以下内容添加到文件的末尾。</span><span class="sxs-lookup"><span data-stu-id="b9100-125">Open **./app/assets/stylesheets/application.css** and add the following to the end of the file.</span></span>

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. <span data-ttu-id="b9100-126">使用下面的命令生成主页控制器。</span><span class="sxs-lookup"><span data-stu-id="b9100-126">Generate a home page controller with the following command.</span></span>

    ```Shell
    rails generate controller Home index
    ```

1. <span data-ttu-id="b9100-127">将 `index` 控制器上的 `Home` 操作配置为应用的默认页面。</span><span class="sxs-lookup"><span data-stu-id="b9100-127">Configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="b9100-128">打开 **./config/routes.rb，** 并将其内容替换为以下内容</span><span class="sxs-lookup"><span data-stu-id="b9100-128">Open **./config/routes.rb** and replace its contents with the following</span></span>

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. <span data-ttu-id="b9100-129">打开 **./app/view/home/index.html.erb，** 并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="b9100-129">Open **./app/view/home/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. <span data-ttu-id="b9100-130">在 **./app/assets/images** **no-profile-photo.png** 名为no-profile-photo.pngPNG 文件。</span><span class="sxs-lookup"><span data-stu-id="b9100-130">Add a PNG file named **no-profile-photo.png** in the **./app/assets/images** directory.</span></span>

1. <span data-ttu-id="b9100-131">保存全部更改，重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="b9100-131">Save all of your changes and restart the server.</span></span> <span data-ttu-id="b9100-132">现在，应用看起来应该非常不同。</span><span class="sxs-lookup"><span data-stu-id="b9100-132">Now, the app should look very different.</span></span>

    ![重新设计的主页的屏幕截图](./images/create-app-01.png)
