<!-- markdownlint-disable MD002 MD041 -->

在本练习中, 将使用["Ruby On Rails"](https://rubyonrails.org/)生成 web 应用程序。 如果尚未安装 Rails, 可以从命令行界面 (CLI) 使用以下命令进行安装。

```Shell
gem install rails
```

打开您的 CLI, 导航到您有权创建文件的目录, 并运行以下命令以创建新的 Rails 应用程序。

```Shell
rails new graph-tutorial
```

Rails 将创建一个名`graph-tutorial`为 "搭建基架" 的新目录, 并为 Rails 应用程序。 导航到此新目录, 然后输入以下命令以启动本地 web 服务器。

```Shell
rails server
```

打开浏览器，并导航到 `http://localhost:3000`。 如果一切正常, 你将看到一个 "Yay! 你正在进行滑轨! " 消息。 如果看不到该消息, 请查看[Rails 入门指南](http://guides.rubyonrails.org/)。

在继续操作之前, 请先安装您将使用的一些其他宝石:

- [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) , 用于处理登录和 OAuth 令牌流。
- [httparty](https://github.com/jnunemaker/httparty)调用 Microsoft Graph 的过程。
- [nokogiri](https://github.com/sparklemotion/nokogiri)处理电子邮件的 HTML 正文。
- [activerecord-session_store](https://github.com/rails/activerecord-session_store)用于将会话存储在数据库中。

在 CLI 中运行以下命令。

```Shell
bundle add omniauth-oauth2
bundle add httparty
bundle add nokogiri
bundle add activerecord-session_store
rails generate active_record:session_migration
```

最后一个命令生成如下所示的输出:

```Shell
create  db/migrate/20180618172216_add_sessions_table.rb
```

打开创建的文件并找到以下行。

```ruby
class AddSessionsTable < ActiveRecord::Migration
```

将该行更改为以下代码行。

```ruby
class AddSessionsTable < ActiveRecord::Migration[5.2]
```

> [!NOTE]
> 这假设您使用的是 5.2: x。 如果使用的是其他版本, 则将`5.2`替换为您的版本。

保存文件并运行以下命令。

```Shell
rake db:migrate
```

最后, 将 Rails 配置为使用新的会话存储。 在`./config/initializers`目录中创建一个`session_store.rb`名为的新文件, 并添加以下代码。

```ruby
Rails.application.config.session_store :active_record_store, key: '_graph_app_session'
```

## <a name="design-the-app"></a>设计应用程序

首先更新应用程序的全局布局。 打开`./app/views/layouts/application.html.erb`并将其内容替换为以下内容。

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

此代码添加简单样式的[引导](http://getbootstrap.com/), 并添加一些简单图标的[字体](https://fontawesome.com/)。 它还定义具有导航栏的全局布局。

现在打开`./app/assets/stylesheets/application.css`并将以下项添加到文件末尾。

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

现在将默认页面替换为新页面。 使用以下命令生成主页控制器。

```Shell
rails generate controller Home index
```

然后, 将`index` `Home`控制器上的操作配置为应用程序的默认页面。 打开`./config/routes.rb`并将内容替换为以下内容

```ruby
Rails.application.routes.draw do
  get 'home/index'
  root 'home#index'

  # Add future routes here

end
```

现在打开`./app/view/home/index.html.erb`文件, 并将其内容替换为以下内容。

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

保存所有更改, 然后重新启动服务器。 现在, 应用程序看起来应非常不同。

![重新设计的主页的屏幕截图](./images/create-app-01.png)
