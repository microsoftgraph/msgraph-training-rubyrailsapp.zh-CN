<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="25684-101">在此部分中，您将添加在用户日历上创建事件的能力。</span><span class="sxs-lookup"><span data-stu-id="25684-101">In this section you will add the ability to create events on the user's calendar.</span></span>

1. <span data-ttu-id="25684-102">打开 **./app/helpers/graph_helper.rb，** 将以下方法添加到 **Graph** 类。</span><span class="sxs-lookup"><span data-stu-id="25684-102">Open **./app/helpers/graph_helper.rb** and add the following method to the **Graph** class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="CreateEventSnippet":::

1. <span data-ttu-id="25684-103">打开 **./app/controllers/calendar_controller，** 将以下路由添加到 **CalendarController** 类。</span><span class="sxs-lookup"><span data-stu-id="25684-103">Open **./app/controllers/calendar_controller** and add the following route to the **CalendarController** class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/calendar_controller.rb" id="CreateEventRouteSnippet":::

1. <span data-ttu-id="25684-104">打开 **./config/routes.rb** 并添加新路由。</span><span class="sxs-lookup"><span data-stu-id="25684-104">Open **./config/routes.rb** and add the new route.</span></span>

    ```ruby
    post 'calendar/new', :to => 'calendar#create'
    ```

1. <span data-ttu-id="25684-105">打开 **./app/views/calendar/new.html.erb，** 并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="25684-105">Open **./app/views/calendar/new.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/new.html.erb" id="NewEventFormSnippet":::

1. <span data-ttu-id="25684-106">保存更改并刷新应用。</span><span class="sxs-lookup"><span data-stu-id="25684-106">Save your changes and refresh the app.</span></span> <span data-ttu-id="25684-107">在 **"日历** "页上，选择 **"新建事件"** 按钮。</span><span class="sxs-lookup"><span data-stu-id="25684-107">On the **Calendar** page, select the **New event** button.</span></span> <span data-ttu-id="25684-108">填写表单，然后选择" **创建** "以创建新事件。</span><span class="sxs-lookup"><span data-stu-id="25684-108">Fill in the form and select **Create** to create a new event.</span></span>

    ![新事件表单的屏幕截图](images/create-event-01.png)
