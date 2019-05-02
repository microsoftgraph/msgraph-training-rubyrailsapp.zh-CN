<!-- markdownlint-disable MD002 MD041 -->

在本练习中, 将把 Microsoft Graph 合并到应用程序中。 对于此应用程序, 您将使用[httparty](https://github.com/jnunemaker/httparty) Gem 对 Microsoft Graph 进行调用。

## <a name="create-a-graph-helper"></a>创建图表帮助程序

创建帮助程序以管理所有 API 调用。 在 CLI 中运行以下命令以生成帮助程序。

```Shell
rails generate helper Graph
```

打开新创建`./app/helpers/graph_helper.rb`的文件, 并将内容替换为以下内容。

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

请花点时间查看此代码的功能。 它通过`httparty` gem 向请求的终结点发出简单的 GET 请求。 它将发送`Authorization`标头中的访问令牌, 并包含任何传递的查询参数。

例如, 若要使用`make_api_call`方法获取 GET `https://graph.microsoft.com/v1.0/me?$select=displayName`, 可以按如下所示调用它:

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

你将在稍后将更多 Microsoft Graph 功能实施到应用中时生成此功能。

## <a name="get-calendar-events-from-outlook"></a>从 Outlook 获取日历事件

让我们先添加查看用户日历上的事件的功能。 在 CLI 中, 运行以下命令以添加新的控制器。

```Shell
rails generate controller Calendar index
```

现在, 我们已提供了路线, 请更新**** 导航栏中的 "日历`./app/view/layouts/application.html.erb` " 链接以使用它。 将行替换`<a class="nav-link" href="#">Calendar</a>`为以下代码行。

```html
<%= link_to "Calendar", {:controller => :calendar, :action => :index}, class: "nav-link#{' active' if controller.controller_name == 'calendar'}" %>
```

将新方法添加到 Graph 帮助程序以[列出用户的事件](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events)。 打开`./app/helpers/graph_helper.rb`并将以下方法添加到`GraphHelper`模块中。

```ruby
def get_calendar_events(token)
  get_events_url = '/v1.0/me/events'

  query = {
    '$select': 'subject,organizer,start,end',
    '$orderby': 'createdDateTime DESC'
  }

  response = make_api_call get_events_url, token, query

  raise response.parsed_response.to_s || "Request returned #{response.code}" unless response.code == 200
  response.parsed_response['value']
end
```

考虑此代码执行的操作。

- 将调用的 URL 为`/v1.0/me/events`。
- `$select`参数将为每个事件返回的字段限制为只显示我们的视图实际使用的字段。
- `$orderby`参数按其创建日期和时间对结果进行排序, 最新项目最先开始。
- 若要成功进行响应, 它将返回包含在`value`键中的项的数组。

现在, 您可以对此进行测试。 打开`./app/controllers/calendar_controller.rb`并更新`index`操作以调用此方法并呈现结果。

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

重新启动服务器。 登录并单击导航栏中的 "**日历**" 链接。 如果一切正常, 应在用户的日历上看到一个 JSON 转储的事件。

## <a name="display-the-results"></a>显示结果

现在, 您可以添加 HTML 和 CSS 以以更用户友好的方式显示结果。

打开`./app/views/calendar/index.html.erb`并将其内容替换为以下内容。

```html
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    <% @events.each do |event| %>
      <tr>
        <td><%= event['organizer']['emailAddress']['name'] %></td>
        <td><%= event['subject'] %></td>
        <td><%= event['start']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
        <td><%= event['end']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

这将遍历一组事件并为每个事件添加一个表行。 删除中`render json: @events` `index`操作中`./app/controllers/calendar_controller.rb`的行, 应用现在应呈现一个事件表。

![事件表的屏幕截图](./images/add-msgraph-01.png)