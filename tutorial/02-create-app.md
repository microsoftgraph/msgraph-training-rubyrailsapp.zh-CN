<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将使用 [Ruby on Rails](https://rubyonrails.org/) 生成 Web 应用。

1. 如果尚未安装 Rails，可以通过以下命令从命令行界面 (CLI) 安装它。

    ```Shell
    gem install rails -v 6.1.3.1
    ```

1. 打开 CLI，导航到你拥有创建文件权限的目录，然后运行以下命令以创建新的 Rails 应用。

    ```Shell
    rails new graph-tutorial
    ```

1. 导航到此新目录并输入以下命令以启动本地 Web 服务器。

    ```Shell
    rails server
    ```

1. 打开浏览器，并导航到 `http://localhost:3000`。 如果一切正常，你将看到"Yay！ 你使用 Rails！" 消息。 如果你看不到该消息，请查看 [Rails 入门指南](http://guides.rubyonrails.org/)。

## <a name="install-gems"></a>安装 gem

在继续之前，请安装一些稍后将使用的其他 gem：

- [omniauth-oauth2，](https://github.com/omniauth/omniauth-oauth2) 用于处理登录和 OAuth 令牌流。
- [omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) 向 OmniAuth 添加 CSRF 保护。
- [用于调用](https://github.com/jnunemaker/httparty)Microsoft Graph。
- [activerecord-session_store](https://github.com/rails/activerecord-session_store) 用于存储数据库中的会话。

1. 打开 **./Gemfile** 并添加以下行。

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. 在 CLI 中，运行以下命令。

    ```Shell
    bundle install
    ```

1. 在 CLI 中，运行以下命令以配置数据库以存储会话。

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. 在 `session_store.rb` **./config/initializers** 目录中创建名为 的新文件，并添加以下代码。

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a>设计应用

在此部分中，你将为应用创建基本 UI。

1. 打开 **./app/views/layouts/application.html.erb，** 并将其内容替换为以下内容。

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    此代码为[简单样式设置添加 Bootstrap，](http://getbootstrap.com/)为一些简单图标添加[Fabric Core。](https://developer.microsoft.com/fluentui#/get-started#fabric-core) 它还定义具有导航栏的全局布局。

1. 打开 **./app/assets/stylesheets/application.css，** 将以下内容添加到文件的末尾。

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. 使用下面的命令生成主页控制器。

    ```Shell
    rails generate controller Home index
    ```

1. 将 `index` 控制器上的 `Home` 操作配置为应用的默认页面。 打开 **./config/routes.rb，** 并将其内容替换为以下内容

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. 打开 **./app/view/home/index.html.erb，** 并将其内容替换为以下内容。

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. 在 **./app/assets/images** **no-profile-photo.png** 名为no-profile-photo.pngPNG 文件。

1. 保存全部更改，重新启动服务器。 现在，应用看起来应该非常不同。

    ![重新设计的主页的屏幕截图](./images/create-app-01.png)
