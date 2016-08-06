### 模板

模板 (Templats) 名如其意：即我们传入数据给文件然后完成 HTTP 响应。对于一个 web 应用来说这些响应通常是完整的 HTML 文档。对 API 来说，它们通常是 JSON 或者 XML 格式的数据。模板的内容通常是 html 标签，但也有一些 `Elixir` 代码混在其中（这些代码 Phoenix 会为我们编译和执行）。事实上 Phoenix 的模板都是预编译的，所以渲染它们非常快。

`EEx` 是 Phoenix 默认的模板系统，它很像 Ruby 里的 ERB。`EEx` 是 Elixir 语言的一部分，在生成新应用的时候，Phoenix 使用 EEx 模板系统来创建诸如路由以及主要的视图文件。

我们在 [View Guide](http://www.phoenixframework.org/docs/views) 中提到过，默认情况下，模板（templates）在 `web/templates` 目录下，每一个 视图 (view) 对应一个目录，里面包含它们自己的渲染模板的视图模块 (view module)。(原文：Each directory has its own view module to render the templates in it.)


### 例子

我们已经在前面或多或少的使用了一些模板的功能，尤其是在 [Adding Pages Guide](http://www.phoenixframework.org/docs/adding-pages) 和 [Views Guide](http://www.phoenixframework.org/docs/views)。 We may cover some of the same territory here, but we will certainly add some new information.


##### web.ex

Phoenix 默认会生成一个 `web/web.ex` 文件来管理常用的 `imports` 和 `aliases`。在 `view` 代码快中的配置对所有的模板(templates)起作用。

我们来给现有的应用加一点代码来做个实验。

首先我们在 `web/router.ex` 文件中添加一条新的路由。

```elixir
defmodule HelloPhoenix.Router do
  ...

  scope "/", HelloPhoenix do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
    get "/test", PageController, :test
  end

  # Other scopes may use custom stacks.
  # scope "/api", HelloPhoenix do
  #   pipe_through :api
  # end
end
```

然后在 `web/controllers/page_controller.ex` 文件中定义一个 `text/2` action。


```elixir
defmodule HelloPhoenix.PageController do
  ...

  def test(conn, _params) do
    render conn, "test.html"
  end
end
```

我们将要创建一个显示是哪个 controller 和 action 在处理请求的函数。

为了完成这个功能，我们需要在 `web/web.ex` 文件中从 `Phoenix.Controller` 模块里引入 `action_name/1` 和 `controller_module/1` 函数。


```elixir
  def view do
    quote do
      use Phoenix.View, root: "web/templates"

      # Import convenience functions from controllers
      import Phoenix.Controller, only: [get_csrf_token: 0, get_flash: 2, view_module: 1,
                                        action_name: 1, controller_module: 1]

      ...
    end
  end
```

接下来，我们在 `web/views/page_view.ex` 文件的底部定义一个 `handler_info/1` 函数 (使用我们刚才引入的 `controller_module/1` 和 `action_name/1` 函数。 ), 我们还会定义一个 `connection_keys/1` 函数稍后使用。


```elixir
defmodule HelloPhoenix.PageView do
  use HelloPhoenix.Web, :view

  def handler_info(conn) do
    "Request Handled By: #{controller_module conn}.#{action_name conn}"
  end

  def connection_keys(conn) do
    conn
    |> Map.from_struct()
    |> Map.keys()
  end
end
```

截至目前，我们有了路由，有了 controller action, 修改了 view , 现在我们需要一个模板显示 `handler_info/1` 函数的返回值。现在来添加一个  `web/templates/page/text.html.eex`, 内容如下：

```html
<div class="jumbotron">
  <p><%= handler_info @conn %></p>
</div>
```

注意 `@conn` 在模板中是可用的。

现在当我们访问 [localhost:4000/test](http://localhost:4000/test) ，就能看到 `Elixir.HelloPhoenix.PageController.test` 返给我们的页面了。

我们可以在 `web/views` 中的任何一个 view 文件中定义函数。但需要注意，只有当 view 文件的名字和 templates 的名字相匹配，里面的定义的函数才是可用的，比如说，之前的`handler_info` 函数，只有在 `web/templates/page` 的模板（templates）里面才是可用的。


##### 显示列表 ( Displaying Lists )

截至目前，我们仅仅在模板（templates）中显示了简单的字符串，在另外一些 guides 中显示了整数（intergers）。我们怎样将这所有元素显示成一个 list 呢？

答案是使用 Elixir 的列表解析（list comprehensions）。

我们现在改动一下 `web/templates/page/test.html.eex` , 让它能显示 `conn` 结构体中的内容。

我们增加一个 header 和一个列表解析，像这样：


```html
<div class="jumbotron">
  <p><%= handler_info @conn %></p>

  <h3>Keys for the conn Struct</h3>

  <%= for key <- connection_keys @conn do %>
    <p><%= key %></p>
  <% end %>
</div>
```

我们把 `connection_keys` 函数的返回值作为列表遍历的数据源，注意我们使用了 `<%=`， 如果你不加 `=` , 什么都不会显示出来。

现在访问 [localhost:4000/test](http://localhost:4000/test) ，就能看到 conn 结构体中所有的 key 了。

#####  在模板之中渲染模板（ Render templates within templates ）

在上面的例子中，实际显示值的代码是相当简单的:

```html
<p><%= key %></p>
```

但实际情况往往比较复杂，如果我们在这里放置过多的代码会导致难以阅读。

最简单的解决方法就是使用另一个模板！ 模板 (Templates) 仅仅只是函数，所以就像正常代码一样，我们可以将一些小的，目的明确的函数组合成一个大的模板，让设计变得更简洁。我们之前在 布局（Layout）的例子中已经见到过，布局就是由模板一层层拼装起来的。

让我们把上面的片段变成一个小模板，我们创建一个 `web/templates/page/key.html.eex` 文件，内容如下：

```html
<p><%= @key %></p>
```
我们需要把  `key` 变成 `@key` 因为这是一个新模板， 并不包含列表解析的内容。我们通过 `assigns` map 的方法将值传进来显示。

现在我们有了新模板，只要简单的将原先列表解析中的内容替换成模板即可：

```html
<div class="jumbotron">
  <p><%= handler_info @conn %></p>

  <h3>Keys for the conn Struct</h3>

  <%= for key <- connection_keys @conn do %>
    <%= render "key.html", key: key %>
  <% end %>
</div>
```

再次访问 [localhost:4000/test](http://localhost:4000/test) . 页面和之前显示的结果一样。

##### 在视图中共享模板 (Shared Templates Across Views)

经常的，我们发现应用中有很多地方都需要渲染相同的一些数据，这里的最佳实践是将这些模板放在公共的目录下，让其能在整个应用中被访问到。

让我们把之前的模板变成一个公共 view 。

`key.html.eex` 当前是被 `HelloPhoenix.PageView` 模块渲染的。这里我们直接使用 render 调用是默认假设当前使用的就是 `HelloPhoenix.PageView`, 我们可以详细写明，像这样：


```html
<div class="jumbotron">
  ...

  <%= for key <- connection_keys @conn do %>
    <%= render HelloPhoenix.PageView, "key.html", key: key %>
  <% end %>
</div>
```

因为我们想让这个模板在 `web/templates/shared/` 目录下, 我们先添加一个公共的视图 `web/views/shared_view.ex`。

```elixir
defmodule HelloPhoenix.SharedView do
  use HelloPhoenix.Web, :view
end
```
然后我们将 `key.html.eex` 文件从 `web/templates/page` 目录里移动到 `web/templates/shared/` 目录下，然后我们就可以把  `web/template/page.text.html.eex` 中的 render 调用改为如下 (使用 `HelloPhoenix.SharedView`)：

```html
<%= for key <- connection_keys @conn do %>
  <%= render HelloPhoenix.SharedView, "key.html", key: key %>
<% end %>
```
再次访问 [localhost:4000/test](http://localhost:4000/test) , 页面内容和之前一样。

