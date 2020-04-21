<!-- markdownlint-disable MD002 MD041 -->

在本练习中，将把 Microsoft Graph 合并到应用程序中。 对于此应用程序，您将使用[httparty](https://github.com/jnunemaker/httparty) Gem 对 Microsoft Graph 进行调用。

## <a name="create-a-graph-helper"></a>创建图表帮助程序

1. 创建帮助程序以管理所有 API 调用。 在 CLI 中运行以下命令以生成帮助程序。

    ```Shell
    rails generate helper Graph
    ```

1. **/App/helpers/graph_helper rb** ，并将内容替换为以下内容。

    ```ruby
    require 'httparty'

    # Graph API helper methods
    module GraphHelper
      GRAPH_HOST = 'https://graph.microsoft.com'.freeze

      def make_api_call(endpoint, token, params = nil)
        headers = {
          Authorization: "Bearer #{token}"
        }

        query = params || {}

        HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                     headers: headers,
                     query: query
      end
    end
    ```

请花点时间查看此代码的功能。 它通过`httparty` gem 向请求的终结点发出简单的 GET 请求。 它将发送`Authorization`标头中的访问令牌，并包含任何传递的查询参数。

例如，若要使用`make_api_call`方法获取 GET `https://graph.microsoft.com/v1.0/me?$select=displayName`，可以按如下所示调用它：

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

你将在稍后将更多 Microsoft Graph 功能实施到应用中时生成此功能。

## <a name="get-calendar-events-from-outlook"></a>从 Outlook 获取日历事件

1. 在 CLI 中，运行以下命令以添加新的控制器。

    ```Shell
    rails generate controller Calendar index
    ```

1. 将新的路由添加到 **/config/routes.rb**。

    ```ruby
    get 'calendar', to: 'calendar#index'
    ```

1. 将新方法添加到 Graph 帮助程序以[列出用户的事件](/graph/api/user-list-events?view=graph-rest-1.0)。 打开 **/app/helpers/graph_helper rb**并将以下方法添加到`GraphHelper`模块中。

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    考虑此代码执行的操作。

    - 将调用的 URL 为`/v1.0/me/events`。
    - `$select`参数将为每个事件返回的字段限制为只显示我们的视图实际使用的字段。
    - `$orderby`参数按其创建日期和时间对结果进行排序，最新项目最先开始。
    - 若要成功进行响应，它将返回包含在`value`键中的项的数组。

1. 打开 **/app/controllers/calendar_controller rb** ，并将其全部内容替换为以下内容。

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        @events = get_calendar_events access_token || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            message: 'Microsoft Graph returned an error getting events.',
            debug: e
          }
        ]
      end
    end
    ```

1. 重新启动服务器。 登录并单击导航栏中的 "**日历**" 链接。 如果一切正常，应在用户的日历上看到一个 JSON 转储的事件。

## <a name="display-the-results"></a>显示结果

现在，您可以添加 HTML 以以更用户友好的方式显示结果。

1. 打开 **/app/views/calendar/index.html.erb** ，并将其内容替换为以下内容。

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    这将遍历一组事件并为每个事件添加一个表行。

1. 从`render json: @events` **/app/controllers/calendar_controller**的`index`操作中移除该行。

1. 刷新页面，应用现在应呈现一个事件表。

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
