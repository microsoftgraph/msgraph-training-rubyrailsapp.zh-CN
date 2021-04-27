<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="d1226-101">在此练习中，你将将 Microsoft Graph合并到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="d1226-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="d1226-102">对于此应用程序，你将使用[httparty](https://github.com/jnunemaker/httparty) gem 调用 Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="d1226-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="d1226-103">创建Graph帮助程序</span><span class="sxs-lookup"><span data-stu-id="d1226-103">Create a Graph helper</span></span>

1. <span data-ttu-id="d1226-104">创建用于管理所有 API 调用的帮助程序。</span><span class="sxs-lookup"><span data-stu-id="d1226-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="d1226-105">在 CLI 中运行以下命令以生成帮助程序。</span><span class="sxs-lookup"><span data-stu-id="d1226-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="d1226-106">打开 **./app/helpers/graph_helper.rb，** 将内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="d1226-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

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

<span data-ttu-id="d1226-107">花些时间查看此代码执行哪些功能。</span><span class="sxs-lookup"><span data-stu-id="d1226-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="d1226-108">它通过 gem 向请求的终结点进行简单的 GET 或 POST `httparty` 请求。</span><span class="sxs-lookup"><span data-stu-id="d1226-108">It makes a simple GET or POST request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="d1226-109">它将在 标头中发送访问 `Authorization` 令牌，并包括传递的任何查询参数。</span><span class="sxs-lookup"><span data-stu-id="d1226-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="d1226-110">例如，若要使用 `make_api_call` 方法执行 GET 操作， `https://graph.microsoft.com/v1.0/me?$select=displayName` 可以如下所示调用它：</span><span class="sxs-lookup"><span data-stu-id="d1226-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call 'GET', '/v1.0/me', access_token, {}, { '$select': 'displayName' }
```

<span data-ttu-id="d1226-111">当你在应用中实现更多 Microsoft Graph功能时，你稍后将基于这一点进行构建。</span><span class="sxs-lookup"><span data-stu-id="d1226-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="d1226-112">从 Outlook 获取日历事件</span><span class="sxs-lookup"><span data-stu-id="d1226-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="d1226-113">在 CLI 中，运行以下命令以添加新控制器。</span><span class="sxs-lookup"><span data-stu-id="d1226-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index new
    ```

1. <span data-ttu-id="d1226-114">将新路由添加到 **./config/routes.rb**。</span><span class="sxs-lookup"><span data-stu-id="d1226-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. <span data-ttu-id="d1226-115">向日历帮助程序Graph一个[获取日历视图的方法](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0)。</span><span class="sxs-lookup"><span data-stu-id="d1226-115">Add a new method to the Graph helper to [get a calendar view](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span></span> <span data-ttu-id="d1226-116">打开 **./app/helpers/graph_helper.rb，** 然后向模块中添加以下 `GraphHelper` 方法。</span><span class="sxs-lookup"><span data-stu-id="d1226-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="d1226-117">考虑此代码将执行什么工作。</span><span class="sxs-lookup"><span data-stu-id="d1226-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="d1226-118">将调用的 URL 为 `/v1.0/me/calendarview`。</span><span class="sxs-lookup"><span data-stu-id="d1226-118">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="d1226-119">`Prefer: outlook.timezone`标头使结果中的开始时间和结束时间根据用户的时区进行调整。</span><span class="sxs-lookup"><span data-stu-id="d1226-119">The `Prefer: outlook.timezone` header causes the start and end times in the results to be adjusted to the user's time zone.</span></span>
        - <span data-ttu-id="d1226-120">和 `startDateTime` `endDateTime` 参数设置视图的起始和结束。</span><span class="sxs-lookup"><span data-stu-id="d1226-120">The `startDateTime` and `endDateTime` parameters set the start and end of the view.</span></span>
        - <span data-ttu-id="d1226-121">参数 `$select` 将每个事件返回的字段限制为视图将实际使用的字段。</span><span class="sxs-lookup"><span data-stu-id="d1226-121">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
        - <span data-ttu-id="d1226-122">参数 `$orderby` 按开始时间对结果进行排序。</span><span class="sxs-lookup"><span data-stu-id="d1226-122">The `$orderby` parameter sorts the results by start time.</span></span>
        - <span data-ttu-id="d1226-123">参数 `$top` 将结果限制为 50 个事件。</span><span class="sxs-lookup"><span data-stu-id="d1226-123">The `$top` parameter limits the results to 50 events.</span></span>
    - <span data-ttu-id="d1226-124">为了成功响应，它将返回键中包含的项目 `value` 数组。</span><span class="sxs-lookup"><span data-stu-id="d1226-124">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="d1226-125">向帮助程序添加新Graph，以根据时区名称Windows [IANA](https://www.iana.org/time-zones)时区标识符。</span><span class="sxs-lookup"><span data-stu-id="d1226-125">Add a new method to the Graph helper to lookup an [IANA time zone identifier](https://www.iana.org/time-zones) based on a Windows time zone name.</span></span> <span data-ttu-id="d1226-126">这是必需的，因为 Microsoft Graph可以将时区作为Windows时区名称返回，并且 Ruby **DateTime** 类需要 IANA 时区标识符。</span><span class="sxs-lookup"><span data-stu-id="d1226-126">This is necessary because Microsoft Graph can return time zones as Windows time zone names, and the Ruby **DateTime** class requires IANA time zone identifiers.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. <span data-ttu-id="d1226-127">打开 **./app/controllers/calendar_controller.rb，** 并将其全部内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="d1226-127">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

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

1. <span data-ttu-id="d1226-128">重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="d1226-128">Restart the server.</span></span> <span data-ttu-id="d1226-129">登录并单击导航 **栏中** 的"日历"链接。</span><span class="sxs-lookup"><span data-stu-id="d1226-129">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="d1226-130">如果一切正常，应在用户日历上看到事件被 JSON 卸载。</span><span class="sxs-lookup"><span data-stu-id="d1226-130">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="d1226-131">显示结果</span><span class="sxs-lookup"><span data-stu-id="d1226-131">Display the results</span></span>

<span data-ttu-id="d1226-132">现在，您可以添加 HTML 以更用户友好的方式显示结果。</span><span class="sxs-lookup"><span data-stu-id="d1226-132">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="d1226-133">打开 **./app/views/calendar/index.html.erb，** 并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="d1226-133">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="d1226-134">该操作将循环遍历事件集合，并针对每个事件添加一个表格行。</span><span class="sxs-lookup"><span data-stu-id="d1226-134">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="d1226-135">从 `render json: @events` `index` **./app/controllers/calendar_controller.rb 中的操作中删除行**。</span><span class="sxs-lookup"><span data-stu-id="d1226-135">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="d1226-136">刷新页面，应用现在应呈现事件表。</span><span class="sxs-lookup"><span data-stu-id="d1226-136">Refresh the page and the app should now render a table of events.</span></span>

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
