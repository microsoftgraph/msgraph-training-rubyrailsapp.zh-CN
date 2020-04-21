<!-- markdownlint-disable MD002 MD041 -->

在本练习中，将使用["Ruby On Rails"](https://rubyonrails.org/)生成 web 应用程序。

1. 如果尚未安装 Rails，可以从命令行界面（CLI）使用以下命令进行安装。

    ```Shell
    gem install rails -v 6.0.2.2
    ```

1. 打开您的 CLI，导航到您有权创建文件的目录，并运行以下命令以创建新的 Rails 应用程序。

    ```Shell
    rails new graph-tutorial
    ```

1. 导航到此新目录，然后输入以下命令以启动本地 web 服务器。

    ```Shell
    rails server
    ```

1. 打开浏览器，并导航到 `http://localhost:3000`。 如果一切正常，你将看到一个 "Yay！ 你正在进行滑轨！ " 消息。 如果看不到该消息，请查看[Rails 入门指南](http://guides.rubyonrails.org/)。

## <a name="install-gems"></a>安装 gem

在继续操作之前，请先安装您将使用的一些其他宝石：

- [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) ，用于处理登录和 OAuth 令牌流。
- [omniauth-](https://github.com/cookpad/omniauth-rails_csrf_protection)将 csrf 保护添加到 omniauth 的 rails_csrf_protection。
- [httparty](https://github.com/jnunemaker/httparty)调用 Microsoft Graph 的过程。
- [activerecord-](https://github.com/rails/activerecord-session_store)用于在数据库中存储会话的 session_store。

1. 打开 **/Gemfile** ，并添加以下行。

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. 在 CLI 中，运行以下命令。

    ```Shell
    bundle install
    ```

1. 在 CLI 中，运行以下命令来配置数据库以存储会话。

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. 创建一个名`session_store.rb`为 " **/config/initializers**目录" 的新文件，并添加以下代码。

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a>设计应用程序

在本节中，您将为应用程序创建基本 UI。

1. 打开 **/app/views/layouts/application.html.erb** ，并将其内容替换为以下内容。

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    此代码添加简单样式的[引导](http://getbootstrap.com/)，并添加一些简单图标的[字体](https://fontawesome.com/)。 它还定义具有导航栏的全局布局。

1. 打开 **/app/assets/stylesheets/application.css** ，并将以下项添加到文件末尾。

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. 使用以下命令生成主页控制器。

    ```Shell
    rails generate controller Home index
    ```

1. 将`Home`控制器`index`上的操作配置为应用程序的默认页面。 打开 **/config/routes.rb**并将其内容替换为以下内容

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. 打开 **/app/view/home/index.html.erb** ，并将其内容替换为以下内容。

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. 保存所有更改，然后重新启动服务器。 现在，应用程序看起来应非常不同。

    ![重新设计的主页的屏幕截图](./images/create-app-01.png)
