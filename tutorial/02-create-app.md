<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="2ded8-101">在本练习中，将使用["Ruby On Rails"](https://rubyonrails.org/)生成 web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="2ded8-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span>

1. <span data-ttu-id="2ded8-102">如果尚未安装 Rails，可以从命令行界面（CLI）使用以下命令进行安装。</span><span class="sxs-lookup"><span data-stu-id="2ded8-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    gem install rails -v 6.0.2.2
    ```

1. <span data-ttu-id="2ded8-103">打开您的 CLI，导航到您有权创建文件的目录，并运行以下命令以创建新的 Rails 应用程序。</span><span class="sxs-lookup"><span data-stu-id="2ded8-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

    ```Shell
    rails new graph-tutorial
    ```

1. <span data-ttu-id="2ded8-104">导航到此新目录，然后输入以下命令以启动本地 web 服务器。</span><span class="sxs-lookup"><span data-stu-id="2ded8-104">Navigate to this new directory and enter the following command to start a local web server.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="2ded8-105">打开浏览器，并导航到 `http://localhost:3000`。</span><span class="sxs-lookup"><span data-stu-id="2ded8-105">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="2ded8-106">如果一切正常，你将看到一个 "Yay！</span><span class="sxs-lookup"><span data-stu-id="2ded8-106">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="2ded8-107">你正在进行滑轨！ "</span><span class="sxs-lookup"><span data-stu-id="2ded8-107">You're on Rails!"</span></span> <span data-ttu-id="2ded8-108">消息。</span><span class="sxs-lookup"><span data-stu-id="2ded8-108">message.</span></span> <span data-ttu-id="2ded8-109">如果看不到该消息，请查看[Rails 入门指南](http://guides.rubyonrails.org/)。</span><span class="sxs-lookup"><span data-stu-id="2ded8-109">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

## <a name="install-gems"></a><span data-ttu-id="2ded8-110">安装 gem</span><span class="sxs-lookup"><span data-stu-id="2ded8-110">Install gems</span></span>

<span data-ttu-id="2ded8-111">在继续操作之前，请先安装您将使用的一些其他宝石：</span><span class="sxs-lookup"><span data-stu-id="2ded8-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="2ded8-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) ，用于处理登录和 OAuth 令牌流。</span><span class="sxs-lookup"><span data-stu-id="2ded8-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="2ded8-113">[omniauth-](https://github.com/cookpad/omniauth-rails_csrf_protection)将 csrf 保护添加到 omniauth 的 rails_csrf_protection。</span><span class="sxs-lookup"><span data-stu-id="2ded8-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="2ded8-114">[httparty](https://github.com/jnunemaker/httparty)调用 Microsoft Graph 的过程。</span><span class="sxs-lookup"><span data-stu-id="2ded8-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="2ded8-115">[activerecord-](https://github.com/rails/activerecord-session_store)用于在数据库中存储会话的 session_store。</span><span class="sxs-lookup"><span data-stu-id="2ded8-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

1. <span data-ttu-id="2ded8-116">打开 **/Gemfile** ，并添加以下行。</span><span class="sxs-lookup"><span data-stu-id="2ded8-116">Open **./Gemfile** and add the following lines.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. <span data-ttu-id="2ded8-117">在 CLI 中，运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="2ded8-117">In your CLI, run the following command.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="2ded8-118">在 CLI 中，运行以下命令来配置数据库以存储会话。</span><span class="sxs-lookup"><span data-stu-id="2ded8-118">In your CLI, run the following commands to configure the database for storing sessions.</span></span>

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. <span data-ttu-id="2ded8-119">创建一个名`session_store.rb`为 " **/config/initializers**目录" 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="2ded8-119">Create a new file called `session_store.rb` in the **./config/initializers** directory, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="2ded8-120">设计应用程序</span><span class="sxs-lookup"><span data-stu-id="2ded8-120">Design the app</span></span>

<span data-ttu-id="2ded8-121">在本节中，您将为应用程序创建基本 UI。</span><span class="sxs-lookup"><span data-stu-id="2ded8-121">In this section you'll create the basic UI for the app.</span></span>

1. <span data-ttu-id="2ded8-122">打开 **/app/views/layouts/application.html.erb** ，并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="2ded8-122">Open **./app/views/layouts/application.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    <span data-ttu-id="2ded8-123">此代码添加简单样式的[引导](http://getbootstrap.com/)，并添加一些简单图标的[字体](https://fontawesome.com/)。</span><span class="sxs-lookup"><span data-stu-id="2ded8-123">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="2ded8-124">它还定义具有导航栏的全局布局。</span><span class="sxs-lookup"><span data-stu-id="2ded8-124">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="2ded8-125">打开 **/app/assets/stylesheets/application.css** ，并将以下项添加到文件末尾。</span><span class="sxs-lookup"><span data-stu-id="2ded8-125">Open **./app/assets/stylesheets/application.css** and add the following to the end of the file.</span></span>

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. <span data-ttu-id="2ded8-126">使用以下命令生成主页控制器。</span><span class="sxs-lookup"><span data-stu-id="2ded8-126">Generate a home page controller with the following command.</span></span>

    ```Shell
    rails generate controller Home index
    ```

1. <span data-ttu-id="2ded8-127">将`Home`控制器`index`上的操作配置为应用程序的默认页面。</span><span class="sxs-lookup"><span data-stu-id="2ded8-127">Configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="2ded8-128">打开 **/config/routes.rb**并将其内容替换为以下内容</span><span class="sxs-lookup"><span data-stu-id="2ded8-128">Open **./config/routes.rb** and replace its contents with the following</span></span>

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. <span data-ttu-id="2ded8-129">打开 **/app/view/home/index.html.erb** ，并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="2ded8-129">Open **./app/view/home/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. <span data-ttu-id="2ded8-130">保存所有更改，然后重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="2ded8-130">Save all of your changes and restart the server.</span></span> <span data-ttu-id="2ded8-131">现在，应用程序看起来应非常不同。</span><span class="sxs-lookup"><span data-stu-id="2ded8-131">Now, the app should look very different.</span></span>

    ![重新设计的主页的屏幕截图](./images/create-app-01.png)
