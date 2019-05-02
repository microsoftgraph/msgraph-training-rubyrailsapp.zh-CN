<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="9c7a0-101">在本练习中, 将把 Microsoft Graph 合并到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="9c7a0-102">对于此应用程序, 您将使用[httparty](https://github.com/jnunemaker/httparty) Gem 对 Microsoft Graph 进行调用。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="9c7a0-103">创建图表帮助程序</span><span class="sxs-lookup"><span data-stu-id="9c7a0-103">Create a Graph helper</span></span>

<span data-ttu-id="9c7a0-104">创建帮助程序以管理所有 API 调用。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="9c7a0-105">在 CLI 中运行以下命令以生成帮助程序。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-105">Run the following command in your CLI to generate the helper.</span></span>

```Shell
rails generate helper Graph
```

<span data-ttu-id="9c7a0-106">打开新创建`./app/helpers/graph_helper.rb`的文件, 并将内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-106">Open the newly created `./app/helpers/graph_helper.rb` file and replace the contents with the following.</span></span>

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

<span data-ttu-id="9c7a0-107">请花点时间查看此代码的功能。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="9c7a0-108">它通过`httparty` gem 向请求的终结点发出简单的 GET 请求。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-108">It makes a simple GET request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="9c7a0-109">它将发送`Authorization`标头中的访问令牌, 并包含任何传递的查询参数。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="9c7a0-110">例如, 若要使用`make_api_call`方法获取 GET `https://graph.microsoft.com/v1.0/me?$select=displayName`, 可以按如下所示调用它:</span><span class="sxs-lookup"><span data-stu-id="9c7a0-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

<span data-ttu-id="9c7a0-111">你将在稍后将更多 Microsoft Graph 功能实施到应用中时生成此功能。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="9c7a0-112">从 Outlook 获取日历事件</span><span class="sxs-lookup"><span data-stu-id="9c7a0-112">Get calendar events from Outlook</span></span>

<span data-ttu-id="9c7a0-113">让我们先添加查看用户日历上的事件的功能。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-113">Let's start by adding the ability to view events on the user's calendar.</span></span> <span data-ttu-id="9c7a0-114">在 CLI 中, 运行以下命令以添加新的控制器。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-114">In your CLI, run the following command to add a new controller.</span></span>

```Shell
rails generate controller Calendar index
```

<span data-ttu-id="9c7a0-115">现在, 我们已提供了路线, 请更新\*\*\*\* 导航栏中的 "日历`./app/view/layouts/application.html.erb` " 链接以使用它。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-115">Now that we have the route available, update the **Calendar** link in the navbar in `./app/view/layouts/application.html.erb` to use it.</span></span> <span data-ttu-id="9c7a0-116">将行替换`<a class="nav-link" href="#">Calendar</a>`为以下代码行。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-116">Replace the line `<a class="nav-link" href="#">Calendar</a>` with the following.</span></span>

```html
<%= link_to "Calendar", {:controller => :calendar, :action => :index}, class: "nav-link#{' active' if controller.controller_name == 'calendar'}" %>
```

<span data-ttu-id="9c7a0-117">将新方法添加到 Graph 帮助程序以[列出用户的事件](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events)。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-117">Add a new method to the Graph helper to [list the user's events](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events).</span></span> <span data-ttu-id="9c7a0-118">打开`./app/helpers/graph_helper.rb`并将以下方法添加到`GraphHelper`模块中。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-118">Open `./app/helpers/graph_helper.rb` and add the following method to the `GraphHelper` module.</span></span>

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

<span data-ttu-id="9c7a0-119">考虑此代码执行的操作。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-119">Consider what this code is doing.</span></span>

- <span data-ttu-id="9c7a0-120">将调用的 URL 为`/v1.0/me/events`。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-120">The URL that will be called is `/v1.0/me/events`.</span></span>
- <span data-ttu-id="9c7a0-121">`$select`参数将为每个事件返回的字段限制为只显示我们的视图实际使用的字段。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-121">The `$select` parameter limits the fields returned for each events to just those our view will actually use.</span></span>
- <span data-ttu-id="9c7a0-122">`$orderby`参数按其创建日期和时间对结果进行排序, 最新项目最先开始。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-122">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
- <span data-ttu-id="9c7a0-123">若要成功进行响应, 它将返回包含在`value`键中的项的数组。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-123">For a successful response, it returns the array of items contained in the `value` key.</span></span>

<span data-ttu-id="9c7a0-124">现在, 您可以对此进行测试。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-124">Now you can test this.</span></span> <span data-ttu-id="9c7a0-125">打开`./app/controllers/calendar_controller.rb`并更新`index`操作以调用此方法并呈现结果。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-125">Open `./app/controllers/calendar_controller.rb` and update the `index` action to call this method and render the results.</span></span>

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

<span data-ttu-id="9c7a0-126">重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-126">Restart the server.</span></span> <span data-ttu-id="9c7a0-127">登录并单击导航栏中的 "**日历**" 链接。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-127">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="9c7a0-128">如果一切正常, 应在用户的日历上看到一个 JSON 转储的事件。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-128">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="9c7a0-129">显示结果</span><span class="sxs-lookup"><span data-stu-id="9c7a0-129">Display the results</span></span>

<span data-ttu-id="9c7a0-130">现在, 您可以添加 HTML 和 CSS 以以更用户友好的方式显示结果。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-130">Now you can add HTML and CSS to display the results in a more user-friendly manner.</span></span>

<span data-ttu-id="9c7a0-131">打开`./app/views/calendar/index.html.erb`并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-131">Open `./app/views/calendar/index.html.erb` and replace its contents with the following.</span></span>

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

<span data-ttu-id="9c7a0-132">这将遍历一组事件并为每个事件添加一个表行。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-132">That will loop through a collection of events and add a table row for each one.</span></span> <span data-ttu-id="9c7a0-133">删除中`render json: @events` `index`操作中`./app/controllers/calendar_controller.rb`的行, 应用现在应呈现一个事件表。</span><span class="sxs-lookup"><span data-stu-id="9c7a0-133">Remove the `render json: @events` line from the `index` action in `./app/controllers/calendar_controller.rb` and the app should now render a table of events.</span></span>

![事件表的屏幕截图](./images/add-msgraph-01.png)