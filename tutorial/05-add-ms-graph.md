<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将将 Microsoft Graph合并到应用程序中。 对于此应用程序，你将使用[httparty](https://github.com/jnunemaker/httparty) gem 调用 Microsoft Graph。

## <a name="create-a-graph-helper"></a>创建Graph帮助程序

1. 创建用于管理所有 API 调用的帮助程序。 在 CLI 中运行以下命令以生成帮助程序。

    ```Shell
    rails generate helper Graph
    ```

1. 打开 **./app/helpers/graph_helper.rb，** 将内容替换为以下内容。

    ```ruby
    require 'httparty'

    # Graph API helper methods
    module GraphHelper
      GRAPH_HOST = 'https://graph.microsoft.com'.freeze

      def make_api_call(method, endpoint, token, headers = nil, params = nil, payload = nil)
        headers ||= {}
        headers[:Authorization] = "Bearer #{token}"
        headers[:Accept] = 'application/json'

        params ||= {}

        case method.upcase
        when 'GET'
          HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                       :headers => headers,
                       :query => params
        when 'POST'
          headers['Content-Type'] = 'application/json'
          HTTParty.post "#{GRAPH_HOST}#{endpoint}",
                        :headers => headers,
                        :query => params,
                        :body => payload ? payload.to_json : nil
        else
          raise "HTTP method #{method.upcase} not implemented"
        end
      end
    end
    ```

花些时间查看此代码执行哪些功能。 它通过 gem 向请求的终结点进行简单的 GET 或 POST `httparty` 请求。 它将在 标头中发送访问 `Authorization` 令牌，并包括传递的任何查询参数。

例如，若要使用 `make_api_call` 方法执行 GET 操作， `https://graph.microsoft.com/v1.0/me?$select=displayName` 可以如下所示调用它：

```ruby
make_api_call 'GET', '/v1.0/me', access_token, {}, { '$select': 'displayName' }
```

当你在应用中实现更多 Microsoft Graph功能时，你稍后将基于这一点进行构建。

## <a name="get-calendar-events-from-outlook"></a>从 Outlook 获取日历事件

1. 在 CLI 中，运行以下命令以添加新控制器。

    ```Shell
    rails generate controller Calendar index new
    ```

1. 将新路由添加到 **./config/routes.rb**。

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. 向日历帮助程序Graph一个[获取日历视图的方法](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0)。 打开 **./app/helpers/graph_helper.rb，** 然后向模块中添加以下 `GraphHelper` 方法。

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    考虑此代码将执行什么工作。

    - 将调用的 URL 为 `/v1.0/me/calendarview`。
        - `Prefer: outlook.timezone`标头使结果中的开始时间和结束时间根据用户的时区进行调整。
        - 和 `startDateTime` `endDateTime` 参数设置视图的起始和结束。
        - 参数 `$select` 将每个事件返回的字段限制为视图将实际使用的字段。
        - 参数 `$orderby` 按开始时间对结果进行排序。
        - 参数 `$top` 将结果限制为 50 个事件。
    - 为了成功响应，它将返回键中包含的项目 `value` 数组。

1. 向帮助程序添加新Graph，以根据时区名称Windows [IANA](https://www.iana.org/time-zones)时区标识符。 这是必需的，因为 Microsoft Graph可以将时区作为Windows时区名称返回，并且 Ruby **DateTime** 类需要 IANA 时区标识符。

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. 打开 **./app/controllers/calendar_controller.rb，** 并将其全部内容替换为以下内容。

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        # Get the IANA identifier of the user's time zone
        time_zone = get_iana_from_windows(user_timezone)

        # Calculate the start and end of week in the user's time zone
        start_datetime = Date.today.beginning_of_week(:sunday).in_time_zone(time_zone).to_time
        end_datetime = start_datetime.advance(:days => 7)

        @events = get_calendar_view access_token, start_datetime, end_datetime, user_timezone || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            :message => 'Microsoft Graph returned an error getting events.',
            :debug => e
          }
        ]
      end
    end
    ```

1. 重新启动服务器。 登录并单击导航 **栏中** 的"日历"链接。 如果一切正常，应在用户日历上看到事件被 JSON 卸载。

## <a name="display-the-results"></a>显示结果

现在，您可以添加 HTML 以更用户友好的方式显示结果。

1. 打开 **./app/views/calendar/index.html.erb，** 并将其内容替换为以下内容。

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    该操作将循环遍历事件集合，并针对每个事件添加一个表格行。

1. 从 `render json: @events` `index` **./app/controllers/calendar_controller.rb 中的操作中删除行**。

1. 刷新页面，应用现在应呈现事件表。

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
