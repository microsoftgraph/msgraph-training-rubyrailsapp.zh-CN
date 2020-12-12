<!-- markdownlint-disable MD002 MD041 -->

在此部分中，您将添加在用户日历上创建事件的能力。

1. 打开 **./app/helpers/graph_helper.rb，** 将以下方法添加到 **Graph** 类。

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="CreateEventSnippet":::

1. 打开 **./app/controllers/calendar_controller，** 将以下路由添加到 **CalendarController** 类。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/calendar_controller.rb" id="CreateEventRouteSnippet":::

1. 打开 **./config/routes.rb** 并添加新路由。

    ```ruby
    post 'calendar/new', :to => 'calendar#create'
    ```

1. 打开 **./app/views/calendar/new.html.erb，** 并将其内容替换为以下内容。

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/new.html.erb" id="NewEventFormSnippet":::

1. 保存更改并刷新应用。 在 **"日历** "页上，选择 **"新建事件"** 按钮。 填写表单，然后选择" **创建** "以创建新事件。

    ![新事件表单的屏幕截图](images/create-event-01.png)
