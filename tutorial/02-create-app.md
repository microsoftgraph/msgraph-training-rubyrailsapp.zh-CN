<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="61623-101">在本练习中，将使用["Ruby On Rails"](https://rubyonrails.org/)生成 web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="61623-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span> <span data-ttu-id="61623-102">如果尚未安装 Rails，可以从命令行界面（CLI）使用以下命令进行安装。</span><span class="sxs-lookup"><span data-stu-id="61623-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

```Shell
gem install rails
```

<span data-ttu-id="61623-103">打开您的 CLI，导航到您有权创建文件的目录，并运行以下命令以创建新的 Rails 应用程序。</span><span class="sxs-lookup"><span data-stu-id="61623-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

```Shell
rails new graph-tutorial
```

<span data-ttu-id="61623-104">Rails 将创建一个名`graph-tutorial`为 "搭建基架" 的新目录，并为 Rails 应用程序。</span><span class="sxs-lookup"><span data-stu-id="61623-104">Rails creates a new directory called `graph-tutorial` and scaffolds a Rails app.</span></span> <span data-ttu-id="61623-105">导航到此新目录，然后输入以下命令以启动本地 web 服务器。</span><span class="sxs-lookup"><span data-stu-id="61623-105">Navigate to this new directory and enter the following command to start a local web server.</span></span>

```Shell
rails server
```

<span data-ttu-id="61623-106">打开浏览器，并导航到 `http://localhost:3000`。</span><span class="sxs-lookup"><span data-stu-id="61623-106">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="61623-107">如果一切正常，你将看到一个 "Yay！</span><span class="sxs-lookup"><span data-stu-id="61623-107">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="61623-108">你正在进行滑轨！ "</span><span class="sxs-lookup"><span data-stu-id="61623-108">You're on Rails!"</span></span> <span data-ttu-id="61623-109">消息。</span><span class="sxs-lookup"><span data-stu-id="61623-109">message.</span></span> <span data-ttu-id="61623-110">如果看不到该消息，请查看[Rails 入门指南](http://guides.rubyonrails.org/)。</span><span class="sxs-lookup"><span data-stu-id="61623-110">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

<span data-ttu-id="61623-111">在继续操作之前，请先安装您将使用的一些其他宝石：</span><span class="sxs-lookup"><span data-stu-id="61623-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="61623-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) ，用于处理登录和 OAuth 令牌流。</span><span class="sxs-lookup"><span data-stu-id="61623-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="61623-113">[omniauth-](https://github.com/cookpad/omniauth-rails_csrf_protection)将 csrf 保护添加到 omniauth 的 rails_csrf_protection。</span><span class="sxs-lookup"><span data-stu-id="61623-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="61623-114">[httparty](https://github.com/jnunemaker/httparty)调用 Microsoft Graph 的过程。</span><span class="sxs-lookup"><span data-stu-id="61623-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="61623-115">[nokogiri](https://github.com/sparklemotion/nokogiri)处理电子邮件的 HTML 正文。</span><span class="sxs-lookup"><span data-stu-id="61623-115">[nokogiri](https://github.com/sparklemotion/nokogiri) to process HTML bodies of email.</span></span>
- <span data-ttu-id="61623-116">[activerecord-](https://github.com/rails/activerecord-session_store)用于在数据库中存储会话的 session_store。</span><span class="sxs-lookup"><span data-stu-id="61623-116">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

<span data-ttu-id="61623-117">在 CLI 中运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="61623-117">Run the following commands in your CLI.</span></span>

```Shell
bundle add omniauth-oauth2
bundle add omniauth-rails_csrf_protection
bundle add httparty
bundle add nokogiri
bundle add activerecord-session_store
rails generate active_record:session_migration
```

<span data-ttu-id="61623-118">最后一个命令生成如下所示的输出：</span><span class="sxs-lookup"><span data-stu-id="61623-118">The last command generates output like the following:</span></span>

```Shell
create  db/migrate/20180618172216_add_sessions_table.rb
```

<span data-ttu-id="61623-119">打开创建的文件并找到以下行。</span><span class="sxs-lookup"><span data-stu-id="61623-119">Open the file that was created and locate the following line.</span></span>

```ruby
class AddSessionsTable < ActiveRecord::Migration
```

<span data-ttu-id="61623-120">将该行更改为以下代码行。</span><span class="sxs-lookup"><span data-stu-id="61623-120">Change that line to the following.</span></span>

```ruby
class AddSessionsTable < ActiveRecord::Migration[5.2]
```

> [!NOTE]
> <span data-ttu-id="61623-121">这假设您使用的是5.2： x。</span><span class="sxs-lookup"><span data-stu-id="61623-121">This assumes that you are using Rails 5.2.x.</span></span> <span data-ttu-id="61623-122">如果使用的是其他版本，则将`5.2`替换为您的版本。</span><span class="sxs-lookup"><span data-stu-id="61623-122">If you are using a different version, replace `5.2` with your version.</span></span>

<span data-ttu-id="61623-123">保存文件并运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="61623-123">Save the file and run the following command.</span></span>

```Shell
rake db:migrate
```

<span data-ttu-id="61623-124">最后，将 Rails 配置为使用新的会话存储。</span><span class="sxs-lookup"><span data-stu-id="61623-124">Finally, configure Rails to use the new session store.</span></span> <span data-ttu-id="61623-125">在`./config/initializers`目录中创建一个`session_store.rb`名为的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="61623-125">Create a new file called `session_store.rb` in the `./config/initializers` directory, and add the following code.</span></span>

```ruby
Rails.application.config.session_store :active_record_store, key: '_graph_app_session'
```

## <a name="design-the-app"></a><span data-ttu-id="61623-126">设计应用程序</span><span class="sxs-lookup"><span data-stu-id="61623-126">Design the app</span></span>

<span data-ttu-id="61623-127">首先更新应用程序的全局布局。</span><span class="sxs-lookup"><span data-stu-id="61623-127">Start by updating the global layout for the app.</span></span> <span data-ttu-id="61623-128">打开`./app/views/layouts/application.html.erb`并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="61623-128">Open `./app/views/layouts/application.html.erb` and replace its contents with the following.</span></span>

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Ruby Graph Tutorial</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
      integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
      integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
      integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
      integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <%= link_to "Ruby Graph Tutorial", root_path, class: "navbar-brand" %>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
          aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <%= link_to "Home", root_path, class: "nav-link#{' active' if controller.controller_name == 'home'}" %>
            </li>
            <% if @user_name %>
              <li class="nav-item" data-turbolinks="false">
                <a class="nav-link" href="#">Calendar</a>
              </li>
            <% end %>
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            <% if @user_name %>
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">
                  <% if @user_avatar %>
                    <img src=<%= @user_avatar %> class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  <% else %>
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  <% end %>
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0"><%= @user_name %></h5>
                  <p class="dropdown-item-text text-muted mb-0"><%= @user_email %></p>
                  <div class="dropdown-divider"></div>
                  <a href="#" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            <% else %>
              <li class="nav-item">
                <a href="#" class="nav-link">Sign In</a>
              </li>
            <% end %>
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      <% if @errors %>
        <% @errors.each do |error| %>
          <div class="alert alert-danger" role="alert">
            <p class="mb-3"><%= error[:message] %></p>
            <%if error[:debug] %>
              <pre class="alert-pre border bg-light p-2"><code><%= error[:debug] %></code></pre>
            <% end %>
          </div>
        <% end %>
      <% end %>
      <%= yield %>
    </main>
  </body>
</html>
```

<span data-ttu-id="61623-129">此代码添加简单样式的[引导](http://getbootstrap.com/)，并添加一些简单图标的[字体](https://fontawesome.com/)。</span><span class="sxs-lookup"><span data-stu-id="61623-129">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="61623-130">它还定义具有导航栏的全局布局。</span><span class="sxs-lookup"><span data-stu-id="61623-130">It also defines a global layout with a nav bar.</span></span>

<span data-ttu-id="61623-131">现在打开`./app/assets/stylesheets/application.css`并将以下项添加到文件末尾。</span><span class="sxs-lookup"><span data-stu-id="61623-131">Now open `./app/assets/stylesheets/application.css` and add the following to the end of the file.</span></span>

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

<span data-ttu-id="61623-132">现在将默认页面替换为新页面。</span><span class="sxs-lookup"><span data-stu-id="61623-132">Now replace the default page with a new one.</span></span> <span data-ttu-id="61623-133">使用以下命令生成主页控制器。</span><span class="sxs-lookup"><span data-stu-id="61623-133">Generate a home page controller with the following command.</span></span>

```Shell
rails generate controller Home index
```

<span data-ttu-id="61623-134">然后，将`index` `Home`控制器上的操作配置为应用程序的默认页面。</span><span class="sxs-lookup"><span data-stu-id="61623-134">Then configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="61623-135">打开`./config/routes.rb`并将内容替换为以下内容</span><span class="sxs-lookup"><span data-stu-id="61623-135">Open `./config/routes.rb` and replace the contents with the following</span></span>

```ruby
Rails.application.routes.draw do
  get 'home/index'
  root 'home#index'

  # Add future routes here

end
```

<span data-ttu-id="61623-136">现在打开`./app/view/home/index.html.erb`文件，并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="61623-136">Now open the `./app/view/home/index.html.erb` file and replace its contents with the following.</span></span>

```html
<div class="jumbotron">
  <h1>Ruby Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from Ruby</p>
  <% if @user_name %>
    <h4>Welcome <%= @user_name %>!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  <% else %>
    <a href="#" class="btn btn-primary btn-large">Click here to sign in</a>
  <% end %>
</div>
```

<span data-ttu-id="61623-137">保存所有更改，然后重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="61623-137">Save all of your changes and restart the server.</span></span> <span data-ttu-id="61623-138">现在，应用程序看起来应非常不同。</span><span class="sxs-lookup"><span data-stu-id="61623-138">Now, the app should look very different.</span></span>

![重新设计的主页的屏幕截图](./images/create-app-01.png)
