### 路由

路由是 `Phoenix` 应用的重要组成部分，她将对应的 HTTP 请求映射到 controller/action, 处理实时 `channel` ，还为路由之前的中间件定义了一系列的转换功能。

`Phoenix` 默认生成的路由文件 `web/router.ex` 内容如下：

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", HelloPhoenix do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
  end

  # Other scopes may use custom stacks.
  # scope "/api", HelloPhoenix do
  #   pipe_through :api
  # end
end
```

你在项目创建时起的名字会替代在实例中的 `HelloPhoenix` 位置（当前项目名为 hello_phoenix），包括路由和控制器。

这个模块的第一行 `use HelloPhenix.Web, :router` 使得 Phoneix 路由相关的函数在我们这个路由范围内可用。

Scopes 会在其他章节详细说明，所以这里我们先跳过 `scope "/", HelloPhoenix do`这部分，`pipe_through :brower`
也会在之后的 `Pipeline` 章节提及。我们现在只需知道 `pipelines` 允许在不同类型的路由上执行一系列的中间件转换操作。

在这个区块中，我们实际定义的路由如下：

```elixir
  get "/", PageController, :index
```

`get` 是一个 Phoenix 的宏，她会实际展开为 `match/3` 函数，对应 HTTP 的 GET 请求，类似的宏
还有 POST, PUT, PATCH, DELETE, OPTIONS, CONNECT, TRACE 和 HEAD。

这些宏的第一个参数是`路径`，这里是根路径`/`, 另外两个参数是我们处理这个请求对应的 controller
和 action 的名字。另外这些宏也接受除此之外的另一些参数，我们将在之后讨论。

这些宏会展开成 match 函数，看起来如下：

```elixir
  def match(conn, "GET", ["/"])
```
`match/3`的函数体建立连接并触发对应的 controller/action。

当我们添加更多的路由时，这个模块的结构就像是一段包含了多个 Elixir 函数的代码段。执行规则是自顶向下，
匹配第一个找到的路由规则，一旦匹配成功，剩余的代码将不会再执行。

如果我们创建一个有歧义的路由，虽然会编译通过，但会得到一个警告，让我们看一个实际的例子：

在`scope "/", HelloPhoenix do `代码块的底部再追加一条路由：

```elixir
get "/", RootController, :index
```

然后在项目的根目录运行 `$ mix compile`,我们会看到编译器的警告：

```text
web/router.ex:1: warning: this clause cannot match because a previous clause at line 1 always matches
Compiled web/router.ex

这条语句不会被匹配，因为之前的那条路由总是会被命中。
```

### 检查 Routes

Phoenix 提供了很酷的工具用来输出当前的路由规则：`phoenix.routes`。

让我们看一个实际的例子：到最近创建的项目根目录输入 `mix phoenix.routes` (如果你没有安装依赖请先运行
一下 `mix do deps.get, compile`), 你将会看到如下输出, 内容是我们项目目前唯一有的路由：

```console
$ mix phoenix.routes
page_path  GET  /  HelloPhoenix.PageController :index
```

`page_path` 是一个 helper 的名字，我们会在以后讨论。

### 资源

路由模块除了支持如`get`, `post` 和 `put`等 HTTP 动词以外，还支持其他一些宏，其中很重要的一个
就是 `资源(resources)` --- 他会展开产生八个 match 函数。

让我们在 `web/router.ex` 文件中添加一个资源 (resource)。

```elixir
scope "/", HelloPhoenix do
  pipe_through :browser # Use the default browser stack

  get "/", PageController, :index
  resources "/users", UserController
end
```
注意我们并没有创建 `UserController`，这里只是演示路由功能。

然后我们去项目根目录执行： `mix phoenix.routes`

你会看到如下类似的输出，当然 `HelloPhoenix` 会变成你自己项目的名字。

```elixir
user_path  GET     /users           HelloPhoenix.UserController :index
user_path  GET     /users/:id/edit  HelloPhoenix.UserController :edit
user_path  GET     /users/new       HelloPhoenix.UserController :new
user_path  GET     /users/:id       HelloPhoenix.UserController :show
user_path  POST    /users           HelloPhoenix.UserController :create
user_path  PATCH   /users/:id       HelloPhoenix.UserController :update
           PUT     /users/:id       HelloPhoenix.UserController :update
user_path  DELETE  /users/:id       HelloPhoenix.UserController :delete
```
这是一个标准的 HTTP 动词，path 和 controller/action 的对应列表 ( 原文: the standard matrix of HTTP verbs )，我们一个一个讨论，顺序可能稍有不同。

 * 针对 `/users`          的 GET    请求会触发 `index` action, 显示所有的 users
 * 针对 `/users/:id`      的 GET    请求会触发 `show` action, 显示这个 id 对应的用户。
 * 针对 `/users/new`      的 GET    请求会触发 `new` action, 发送一个创建新用户的表单。
 * 针对 `/users`          的 POST   请求会触发 `create` action, 保存一个新用户到数据库。
 * 针对 `/users/:id/edit` 的 GET    请求会触发 `edit` action, 会先从数据库取出该 :id 用户对应的数据，然后返回一个编辑该用户的表单。
 * 针对 `/users/:id`      的 PATCH  请求会触发 `update` action, 更新指定 :id 对应的用户的信息。
 * 针对 `/users/:id`      的 PUT    请求会触发 `update` action, 更新指定 :id 对应的用户的信息。
 * 针对 `/users`          的 DELETE 请求会触发 `delete` action, 从数据库中删除指定 :id 的用户。

如果我们不需要所有的路由，还可以使用 `:only` 和 `:except` 选项。

比如我们有一个只读的 posts 资源，我们可以这样定义：

```elixir
resources "posts", PostController, only: [:index, :show]
```
运行 `$ mix phoenix.routes` 会看到现在我们的路由只定义了 `index` 和 `show` 规则。

```elixir
post_path  GET     /posts HelloPhoenix.PostController :index
post_path  GET     /posts/:id HelloPhoenix.PostController :show
```

类似的，如果我们有一个 comments 资源，但我们不想定义删除的路由操作，我们可以这么干：

```elixir
resources "comments", CommentController, except: [:delete]
```

运行 `$ mix phoenix.routes` 会发现，除了 `delete` 其他的操作都定义了。

```elixir
comment_path  GET     /comments HelloPhoenix.CommentController :index
comment_path  GET     /comments/:id/edit HelloPhoenix.CommentController :edit
comment_path  GET     /comments/new HelloPhoenix.CommentController :new
comment_path  GET     /comments/:id HelloPhoenix.CommentController :show
comment_path  POST    /comments HelloPhoenix.CommentController :create
comment_path  PATCH   /comments/:id HelloPhoenix.CommentController :update
              PUT     /comments/:id HelloPhoenix.CommentController :update
```

### 路径 Helpers ( Path Helpers )

`Path helpers` 是一些 `Router.Helpers` 模块动态产生的函数（对每个应用独立），就我们目前的应用来讲就
是 `HelloPhoenix.Router.Helpers`。她的命名遵循在 router 中定义的 controller 的规则。我们的 controller
是`HelloPhoenix.PageController`,`page_path`会返回项目目录的根地址。

好了，让我们看一个实际的例子，在项目根目录运行 `$ iex -S mix` 然后按照下面的例子运行：

```elixir
iex> HelloPhoenix.Router.Helpers.page_path(HelloPhoenix.Endpoint, :index)
"/"
```

这是很有用的的，意味着我们可以在模板中用 `page_path` 代表项目的根目录。

```html
<a href="<%= page_path(@conn, :index) %>">To the Welcome Page!</a>
```
`page_path` 函数使用 `use HelloPhoenix.Web, :view` 被引入模板。
更多的细节在 [视图指北](https://github.com/mydearxym/phoenix-doc-in-chinese/blob/master/E_%E8%A7%86%E5%9B%BE.md)。

这为我们省去了大量的体力工作，因为`page_path`是动态生成的，即便我们在 router 中改了路径，这个 helper 还是会一样的工作。

### 更多的路径 Helpers

当我们运行 `phoenix.routes` 后，会列出 `user_path` 的列表，下面的例子是我们如何转换这些 helper 。

```elixir
iex> import HelloPhoenix.Router.Helpers
iex> alias HelloPhoenix.Endpoint
iex> user_path(Endpoint, :index)
"/users"

iex> user_path(Endpoint, :show, 17)
"/users/17"

iex> user_path(Endpoint, :new)
"/users/new"

iex> user_path(Endpoint, :create)
"/users"

iex> user_path(Endpoint, :edit, 37)
"/users/37/edit"

iex> user_path(Endpoint, :update, 37)
"/users/37"

iex> user_path(Endpoint, :delete, 17)
"/users/17"
```
那查询字符串呢？ Phoenix也为你想到了，你可以加一个可选的字典类型的，helper 函数会将这些参数拼接到生成的路径上。

```elixir
iex> user_path(Endpoint, :show, 17, admin: true, active: false)
"/users/17?admin=true&active=false"
```
用 `_url` 取代 `_path` 会得到完整路径。

```elixir
iex(3)> user_url(Endpoint, :index)
"http://localhost:4000/users"
```

我们很快会写关于 `endpoints` 的文档，现在你只需把她看成是一个处理从请求到路由这个中间过程即可。这包括启动
app/server, 应用配置，为每个请求应用基本的 plugs 中间件 ( and applying the plugs common to all requests )。

`_url` 函数根据配置信息取得 host, port, proxy port 和 ssl 信息生成完整的 url 。我们会在专门的章节里讨论这些。
现在，我们可以打开 `/config/dev.exs`看看这些参数配置。

### 资源嵌套

在 Phoenix router 里嵌套资源也很容易实现。比如我们有个 posts 的资源和 users 有一对多的关联，也即，一个 user 可
以创建多个 posts , 一个 post 只属于一个 user 。 我们可以在 `web/router.ex`里这么写：

```elixir
resources "users", UserController do
  resources "posts", PostController
end
```
现在我们运行 `$ mix phoenix.routes`, 可以看到如下结果：

```elixir
. . .
user_post_path  GET     users/:user_id/posts HelloPhoenix.PostController :index
user_post_path  GET     users/:user_id/posts/:id/edit HelloPhoenix.PostController :edit
user_post_path  GET     users/:user_id/posts/new HelloPhoenix.PostController :new
user_post_path  GET     users/:user_id/posts/:id HelloPhoenix.PostController :show
user_post_path  POST    users/:user_id/posts HelloPhoenix.PostController :create
user_post_path  PATCH   users/:user_id/posts/:id HelloPhoenix.PostController :update
                PUT     users/:user_id/posts/:id HelloPhoenix.PostController :update
user_post_path  DELETE  users/:user_id/posts/:id HelloPhoenix.PostController :delete
```

我们看到每个到 posts 的路由被限制在 user ID 之后，比如第一个，我们会触发 `PostController` `index`
action, 但我们必须传入 `user_id`， 这意味这我们只能显示这个给定的 user 的 posts, 其他的路由也类似。

当我们用 helper 生成嵌套路径的时候，我们需要手动传入 IDs ， 比如对于 `show` 路由， `42` 是 `user_id`，
`17` 是 `post_id`，记得给我们的 `HelloPhoenix.endpoint` 起别名。

```elixir
iex> alias HelloPhoenix.Endpoint
iex> HelloPhoenix.Router.Helpers.user_post_path(Endpoint, :show, 42, 17)
"/users/42/posts/17"
```
同样的，当我们给最后一个函数添加键值对时，也会拼接生成查询字符串。

```elixir
iex> HelloPhoenix.Router.Helpers.user_post_path(Endpoint, :index, 42, active: true)
"/users/42/posts?active=true"
```

### 作用域路由（scopes routs）

作用域是一种给路由添加基于统一前缀和一些 plug 中间件群组的机制。 我们可以利用这种机制为路由添加管理员功能，提
供带版本号的 API 等等。 比如说有用户在我们网站上发表了一条评论，这些评论同时需要被一个管理员审核，从字面意思上
来看，这些资源是不同的，他们很可能也不共用 controller, 所以我们把路由分开定义：

用户这边是典型的"资源"类型：

```elixir
/reviews
/reviews/1234
/reviews/1234/edit

and so on
```
管理员部分则是加个前缀 `admin`:

```elixir
/admin/reviews
/admin/reviews/1234
/admin/reviews/1234/edit

等等。。。
```
我们可以用一个作用域选项 `/admin` 来达到同样的目的：

```elixir
scope "/admin" do
  pipe_through :browser

  resources "/reviews", HelloPhoenix.Admin.ReviewController
end
```
注意 Phoenix 会假定我们的路径以`/`开始, 但这是可选的，所以`scope "/admin" do` 和 `scope "admin" do` 的结果是一样的。

另外，如果按照上面的定义，我们需要指定控制器的全名 `HelloPhoenix.Admin.ReviewController`, 我们之后会解决这个问题。

再次运行 `$ mix phoenix.routes`, 结果如下：

```elixir
. . .
review_path  GET     /reviews HelloPhoenix.ReviewController :index
review_path  GET     /reviews/:id/edit HelloPhoenix.ReviewController :edit
review_path  GET     /reviews/new HelloPhoenix.ReviewController :new
review_path  GET     /reviews/:id HelloPhoenix.ReviewController :show
review_path  POST    /reviews HelloPhoenix.ReviewController :create
review_path  PATCH   /reviews/:id HelloPhoenix.ReviewController :update
             PUT     /reviews/:id HelloPhoenix.ReviewController :update
review_path  DELETE  /reviews/:id HelloPhoenix.ReviewController :delete
. . .
review_path  GET     /admin/reviews HelloPhoenix.Admin.ReviewController :index
review_path  GET     /admin/reviews/:id/edit HelloPhoenix.Admin.ReviewController :edit
review_path  GET     /admin/reviews/new HelloPhoenix.Admin.ReviewController :new
review_path  GET     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :show
review_path  POST    /admin/reviews HelloPhoenix.Admin.ReviewController :create
review_path  PATCH   /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
             PUT     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
review_path  DELETE  /admin/reviews/:id HelloPhoenix.Admin.ReviewController :delete
```

看起来不错，但有个问题，虽然路由部分是对的，但是每一行用户和管理员的路径 helper `review_path`都是一样的
，这会导致错误，我们可以加一个 `as: :admin`选项来解决 (原文过于啰嗦，这里有所略过)：

```elixir
scope "/", HelloPhoenix do
  pipe_through :browser
  . . .
  resources "/reviews", ReviewController
  . . .
end

scope "/admin", as: :admin do
  resources "/reviews", HelloPhoenix.Admin.ReviewController
end
```

这时再运行`$ mix phoenix.routes`就能得到正确的结果了。

```elixir
. . .
      review_path  GET     /reviews HelloPhoenix.ReviewController :index
      review_path  GET     /reviews/:id/edit HelloPhoenix.ReviewController :edit
      review_path  GET     /reviews/new HelloPhoenix.ReviewController :new
      review_path  GET     /reviews/:id HelloPhoenix.ReviewController :show
      review_path  POST    /reviews HelloPhoenix.ReviewController :create
      review_path  PATCH   /reviews/:id HelloPhoenix.ReviewController :update
                   PUT     /reviews/:id HelloPhoenix.ReviewController :update
      review_path  DELETE  /reviews/:id HelloPhoenix.ReviewController :delete
. . .
admin_review_path  GET     /admin/reviews HelloPhoenix.Admin.ReviewController :index
admin_review_path  GET     /admin/reviews/:id/edit HelloPhoenix.Admin.ReviewController :edit
admin_review_path  GET     /admin/reviews/new HelloPhoenix.Admin.ReviewController :new
admin_review_path  GET     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :show
admin_review_path  POST    /admin/reviews HelloPhoenix.Admin.ReviewController :create
admin_review_path  PATCH   /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
                   PUT     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
admin_review_path  DELETE  /admin/reviews/:id HelloPhoenix.Admin.ReviewController :delete
```

现在路径 helpers 能返回我们想要的结果了, 我们在 shell 里自己试试吧，
运行  `$ iex -S mix`

```elixir
iex(1)> HelloPhoenix.Router.Helpers.review_path(Endpoint, :index)
"/reviews"

iex(2)> HelloPhoenix.Router.Helpers.admin_review_path(Endpoint, :show, 1234)
"/admin/reviews/1234"
```

如果管理员需要处理其他的资源呢？ 我们可以这样直接追加在后面：

```elixir
scope "/admin", as: :admin do
  pipe_through :browser

  resources "/images", HelloPhoenix.Admin.ImageController
  resources "/reviews", HelloPhoenix.Admin.ReviewController
  resources "/users", HelloPhoenix.Admin.UserController
end
```
运行 `$ mix phoenix.routes` 结果如下：

```elixir
. . .
 admin_image_path  GET     /admin/images HelloPhoenix.Admin.ImageController :index
 admin_image_path  GET     /admin/images/:id/edit HelloPhoenix.Admin.ImageController :edit
 admin_image_path  GET     /admin/images/new HelloPhoenix.Admin.ImageController :new
 admin_image_path  GET     /admin/images/:id HelloPhoenix.Admin.ImageController :show
 admin_image_path  POST    /admin/images HelloPhoenix.Admin.ImageController :create
 admin_image_path  PATCH   /admin/images/:id HelloPhoenix.Admin.ImageController :update
                   PUT     /admin/images/:id HelloPhoenix.Admin.ImageController :update
 admin_image_path  DELETE  /admin/images/:id HelloPhoenix.Admin.ImageController :delete
admin_review_path  GET     /admin/reviews HelloPhoenix.Admin.ReviewController :index
admin_review_path  GET     /admin/reviews/:id/edit HelloPhoenix.Admin.ReviewController :edit
admin_review_path  GET     /admin/reviews/new HelloPhoenix.Admin.ReviewController :new
admin_review_path  GET     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :show
admin_review_path  POST    /admin/reviews HelloPhoenix.Admin.ReviewController :create
admin_review_path  PATCH   /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
                   PUT     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
admin_review_path  DELETE  /admin/reviews/:id HelloPhoenix.Admin.ReviewController :delete
  admin_user_path  GET     /admin/users HelloPhoenix.Admin.UserController :index
  admin_user_path  GET     /admin/users/:id/edit HelloPhoenix.Admin.UserController :edit
  admin_user_path  GET     /admin/users/new HelloPhoenix.Admin.UserController :new
  admin_user_path  GET     /admin/users/:id HelloPhoenix.Admin.UserController :show
  admin_user_path  POST    /admin/users HelloPhoenix.Admin.UserController :create
  admin_user_path  PATCH   /admin/users/:id HelloPhoenix.Admin.UserController :update
                   PUT     /admin/users/:id HelloPhoenix.Admin.UserController :update
  admin_user_path  DELETE  /admin/users/:id HelloPhoenix.Admin.UserController :delete
```

不错，正是我们想要的，不过我们可以让这变得更简单。注意对于上面每个资源，我们都要在控制器前面手动加
上 `HelloPhoenix.Admin`, 这很枯燥并且容易产生错误，我们可以在 scope 的后面加上`HelloPhoenix.Admin`
选项，其他的问题Phoenix会帮我们生成完整的控制器名称，像这样：

```elixir
scope "/admin", HelloPhoenix.Admin, as: :admin do
  pipe_through :browser

  resources "/images",  ImageController
  resources "/reviews", ReviewController
  resources "/users",   UserController
end
```

现在我们运行 `$ mix phoenix.routes` 会发现结果和上面一样。

自然的，我们可以嵌套我们应用里的所有路由，简单为我们的应用的指定一个别名，就可以省去控制器名字前的重复了。

实际上 Phoenix 已经这么做了：

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  scope "/", HelloPhoenix do
    pipe_through :browser

    get "/images", ImageController, :index
    resources "/reviews", ReviewController
    resources "/users",   UserController
  end
end
```
再次运行： `$ mix phoenix.routes` , 控制器的名字都符合预期。

```elixir
image_path   GET     /images            HelloPhoenix.ImageController :index
review_path  GET     /reviews           HelloPhoenix.ReviewController :index
review_path  GET     /reviews/:id/edit  HelloPhoenix.ReviewController :edit
review_path  GET     /reviews/new       HelloPhoenix.ReviewController :new
review_path  GET     /reviews/:id       HelloPhoenix.ReviewController :show
review_path  POST    /reviews           HelloPhoenix.ReviewController :create
review_path  PATCH   /reviews/:id       HelloPhoenix.ReviewController :update
             PUT     /reviews/:id       HelloPhoenix.ReviewController :update
review_path  DELETE  /reviews/:id       HelloPhoenix.ReviewController :delete
  user_path  GET     /users             HelloPhoenix.UserController :index
  user_path  GET     /users/:id/edit    HelloPhoenix.UserController :edit
  user_path  GET     /users/new         HelloPhoenix.UserController :new
  user_path  GET     /users/:id         HelloPhoenix.UserController :show
  user_path  POST    /users             HelloPhoenix.UserController :create
  user_path  PATCH   /users/:id         HelloPhoenix.UserController :update
             PUT     /users/:id         HelloPhoenix.UserController :update
  user_path  DELETE  /users/:id         HelloPhoenix.UserController :delete
```

作用域同样可以被嵌套，就像资源一样，比如说我们为 images, reviews 和用户增加了版本控制，我们可以这样定义路由：

```elixir
scope "/api", HelloPhoenix.Api, as: :api do
  pipe_through :api

  scope "/v1", V1, as: :v1 do
    resources "/images",  ImageController
    resources "/reviews", ReviewController
    resources "/users",   UserController
  end
end
```

运行`$ mix phoenix.routes`结果如下：

```elixir
 api_v1_image_path  GET     /api/v1/images HelloPhoenix.Api.V1.ImageController :index
 api_v1_image_path  GET     /api/v1/images/:id/edit HelloPhoenix.Api.V1.ImageController :edit
 api_v1_image_path  GET     /api/v1/images/new HelloPhoenix.Api.V1.ImageController :new
 api_v1_image_path  GET     /api/v1/images/:id HelloPhoenix.Api.V1.ImageController :show
 api_v1_image_path  POST    /api/v1/images HelloPhoenix.Api.V1.ImageController :create
 api_v1_image_path  PATCH   /api/v1/images/:id HelloPhoenix.Api.V1.ImageController :update
                    PUT     /api/v1/images/:id HelloPhoenix.Api.V1.ImageController :update
 api_v1_image_path  DELETE  /api/v1/images/:id HelloPhoenix.Api.V1.ImageController :delete
api_v1_review_path  GET     /api/v1/reviews HelloPhoenix.Api.V1.ReviewController :index
api_v1_review_path  GET     /api/v1/reviews/:id/edit HelloPhoenix.Api.V1.ReviewController :edit
api_v1_review_path  GET     /api/v1/reviews/new HelloPhoenix.Api.V1.ReviewController :new
api_v1_review_path  GET     /api/v1/reviews/:id HelloPhoenix.Api.V1.ReviewController :show
api_v1_review_path  POST    /api/v1/reviews HelloPhoenix.Api.V1.ReviewController :create
api_v1_review_path  PATCH   /api/v1/reviews/:id HelloPhoenix.Api.V1.ReviewController :update
                    PUT     /api/v1/reviews/:id HelloPhoenix.Api.V1.ReviewController :update
api_v1_review_path  DELETE  /api/v1/reviews/:id HelloPhoenix.Api.V1.ReviewController :delete
  api_v1_user_path  GET     /api/v1/users HelloPhoenix.Api.V1.UserController :index
  api_v1_user_path  GET     /api/v1/users/:id/edit HelloPhoenix.Api.V1.UserController :edit
  api_v1_user_path  GET     /api/v1/users/new HelloPhoenix.Api.V1.UserController :new
  api_v1_user_path  GET     /api/v1/users/:id HelloPhoenix.Api.V1.UserController :show
  api_v1_user_path  POST    /api/v1/users HelloPhoenix.Api.V1.UserController :create
  api_v1_user_path  PATCH   /api/v1/users/:id HelloPhoenix.Api.V1.UserController :update
                    PUT     /api/v1/users/:id HelloPhoenix.Api.V1.UserController :update
  api_v1_user_path  DELETE  /api/v1/users/:id HelloPhoenix.Api.V1.UserController :delete
```

有趣的是，我们可以利用路由定义相同的作用域，只要你确保他们之间不会相互冲突即可，否则你会得到之前的错误：

```console
warning: this clause cannot match because a previous clause at line 16 always matches
```

```elixir
defmodule HelloPhoenix.Router do
  use Phoenix.Router
  . . .
  scope "/", HelloPhoenix do
    pipe_through :browser

    resources "users", UserController
  end

  scope "/", AnotherApp do
    pipe_through :browser

    resources "posts", PostController
  end
  . . .
end
```

运行`$ mix phoenix.routes`, 得到如下结果.

```elixir
user_path  GET     /users           HelloPhoenix.UserController :index
user_path  GET     /users/:id/edit  HelloPhoenix.UserController :edit
user_path  GET     /users/new       HelloPhoenix.UserController :new
user_path  GET     /users/:id       HelloPhoenix.UserController :show
user_path  POST    /users           HelloPhoenix.UserController :create
user_path  PATCH   /users/:id       HelloPhoenix.UserController :update
           PUT     /users/:id       HelloPhoenix.UserController :update
user_path  DELETE  /users/:id       HelloPhoenix.UserController :delete
post_path  GET     /posts           AnotherApp.PostController :index
post_path  GET     /posts/:id/edit  AnotherApp.PostController :edit
post_path  GET     /posts/new       AnotherApp.PostController :new
post_path  GET     /posts/:id       AnotherApp.PostController :show
post_path  POST    /posts           AnotherApp.PostController :create
post_path  PATCH   /posts/:id       AnotherApp.PostController :update
           PUT     /posts/:id       AnotherApp.PostController :update
post_path  DELETE  /posts/:id       AnotherApp.PostController :delete
```

### Pipelines

现在是时候谈论一下我们最开始看到的那几行了：` pipe_throuph :browser` 。

还记得我们在 [概览章节](https://github.com/mydearxym/phoenix-doc-in-chinese/blob/master/%E6%A6%82%E8%A7%88.md)
中我们把 plugs 描述成一组按顺序执行的任务( being stacked and executable in a pre-determined order ), 就像管道
(pipeline) 一样，现在我们来看看 plug 在 router 中是怎样工作的。

管道(Pipelines) 是一些简单的 plugs 按照一定的顺序集合起来, 并取一个名字。它可以在某一个请求上执行特定的操作。
Phoenix 默认为我们提供了一些任务，我们也可以定制它们来满足自己的需求。

新创建的 Phoenix 应用定义了两个 pipelines `:browser` 和 `:api`。我们稍后会接触到， 现在我们先来看看在 EndPoint
中 plug 的工作流。

##### The Endpoint Plugs

Endpoints 为每个请求安排一组 plug 任务, 并在请求到达路由层的 `:browser`, `:api` 以及自定义的 pipelines 之前被
执行。 默认的 Endpoint plugs 做了很多工作，以下排名分先后。

- [Plug.Static](http://hexdocs.pm/plug/Plug.Static.html) - 伺服静态资源，因为这个 plug 是在 logger 之前被记录
  的，所以并不会被记录到日志里。

- [Plug.Logger](http://hexdocs.pm/plug/Plug.Logger.html) - 记录请求信息。

- [Phoenix.CodeReloader](http://hexdocs.pm/phoenix/Phoenix.CodeReloader.html) - 这个 plug 可以自动对 web 目录
  下的代码具有自动刷新功能， Phoenix 默认已配置。

- [Plug.Parsers](http://hexdocs.pm/plug/Plug.Parsers.html) - 使用自带的解析器解析请求，默认是 url ,multipart
  和 json （原文: parsers urlencoded, multipart and json (with poison) ）, 如果不能识别请求中的 content-type 则不解析。

- [Plug.MethodOverride](http://hexdocs.pm/plug/Plug.MethodOverride.html) - 将 POST 请求转化为合适的 PUT, PATCH 或者 DELETE。

- [Plug.Head](http://hexdocs.pm/plug/Plug.Head.html) - 将 HEAD 请求转换为 GET 请求并去除响应的 body
  (converts HEAD requests to GET requests and strips the response body)

- [Plug.Session](http://hexdocs.pm/plug/Plug.Session.html) - 一个 session 管理的 plug , 注意 `fetch_session/2`
  还是需要被调用因为这个 plug 只是决定 session 怎样被获得。(Note that `fetch_session/2` must still be explicitly called before using the session as this plug just sets up how the session is fetched)

- [Plug.Router](http://hexdocs.pm/plug/Plug.Router.html) - 将 router 要用到请求周期中。(plugs a router into the request cycle)

##### `:browser` 和 `:api` Pipelines

Phoenix 默认定义了两个 pipeline, `:browser` 和 `:api`。如果我们在 scope 中使用了 `pipe_through/1` 它们的其中一个， 请求命中路由规则就会被触发。

就像它们名字所代表的意思那样，`:browser` pipeline 为浏览器的渲染请求做准备，而 `:api` pipeline 为数据请求做准备。

`:browser` pipeline 有 5 个 plugs:

- `plug :accepts, ["html"]` 定义请求格式或者决定哪些接受哪些格式。

- `:fetch_session`, 一般来讲，获取 session 信息，并让其在整个连接中可用。

- `:fetch_flash` 会抓取任何被设置的 flash 信息。

- `:protect_from_forgery` 和 `:put_secure_browser_headers` 则确保请求不跨域。

目前, `:api` pipeline 只定义了 `plug :accepts, ["json"]`。

pipeline 只在 scope 中起作用，如果没有定义 scope ，路由会在每条规则触发 pipeline,  如果我们在嵌套的 scope
中调用 `pipe_through/1` , 那么之后在这个嵌套的范围内起作用。

我们再来看看另一个新生成的 Phoenix 应用，这次我们将 api 作用域部分的注释打开，然后添加一条新的路由。

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", HelloPhoenix do
    pipe_through :browser

    get "/", PageController, :index
  end

  # Other scopes may use custom stacks.
  scope "/api", HelloPhoenix do
    pipe_through :api

    resources "reviews", ReviewController
  end
end
```

当服务器接收到一个请求，请求总是会穿过 Endpoint 中的 plugs ,然后再命中路由。

比如，一个请求命中了 `GET /` 这个路由，在到达 `PageController` `index` action 之前， 路由器( router ) 会首先
执行 `:brower` pipeline -- 它会获取 session，flash 数据，并执行跨域保护。

相反的，如果请求命中了 `resources/2` 定义的路由, 路由器会在它到达 `HelloPhoenix.ReviewController` 之前把它交给 `:api` pipeline (目前什么都没干)。

如果我们的应用只为浏览器提供渲染页面的工作，那么我们可以把简单的将 `api` 部分删除：

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipe_through :browser

  get "/", HelloPhoenix.PageController, :index

  resources "reviews", HelloPhoenix.ReviewController
end
```

将scopes 删除会迫使 `:browser` pipeline 对每一个路由生效。

再深入一点，如果我们想同时使用 `:browser` 和其他一些自定义的 pipelines， 我们可以简单的给 `pipe_through`
传递一个数组，Phoenix 会将它们顺序执行。

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end
  ...

  scope "/reviews" do
    # Use the default browser stack.
    pipe_through [:browser, :review_checks, :other_great_stuff]

    resources "reviews", HelloPhoenix.ReviewController
  end
end
```

这是一个在嵌套作用域中使用不同 pipelines 的例子。

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end
  ...

  scope "/", HelloPhoenix do
    pipe_through :browser

    resources "posts", PostController

    scope "/reviews" do
      pipe_through :review_checks

      resources "reviews", ReviewController
    end
  end
end
```

上面这个例子中，所有的路由都会经过 `:browser` pipeline, 因为 `/` 作用域包含了所有的路由，但只有
`/reviews` 会经过 `:review_checks` pipeline。

##### 创建新的 pipelines

Phoenix 允许我们在路由中创建自定义的 pipelines . 并且这极其简单。你只需要调用 `pipeline/2` 宏，一个新的
pipeline 的名字（以 atom 的形式）, 以及一个作用域即可

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipeline :review_checks do
    plug :ensure_authenticated_user
    plug :ensure_user_owns_review
  end

  scope "/reviews", HelloPhoenix do
    pipe_through :review_checks

    resources "reviews", ReviewController
  end
end
```

### 通道路由

通道（`Channels`）是 Phoenix 框架中的令人激动的，实时的组件，通道根据一个特定主题处理来自 socket 上的信息，
通道通过 socket 和主题(topic) 来确定路由。我们会在[通道指北](https://github.com/mydearxym/phoenix-doc-in-chinese/blob/master/G_%E9%80%9A%E9%81%93.md)
中详细谈论。

我们将 socket 处理函数挂载到我们的 endpoint (位置在 `lib/hello_phoenix/endpoint.ex`)Socket 处理函数
处理权限和和通道路由。

```elixir
defmodule HelloPhoenix.Endpoint do
  use Phoenix.Endpoint

  socket "/socket", HelloPhoenix.UserSocket
  ...
end
```

下一步，我们打开 `web/channels/user_socket.ex`文件，用 channel/3 宏定义我们的路由。这个路由会将一个主题
（topic）映射给一个通道，如果我们有一个通道叫 `RoomChannel` 另有一个主题叫 `"rooms.*"`, 代码看起来如下：

```elixir
defmodule HelloPhoenix.UserSocket do
  use Phoenix.Socket

  channel "rooms:*", HelloPhoenix.RoomChannel
  ...
end
```

主题（Topics）只是简单的字符串。 这里所用的是一个惯用法`主题：子主题` 。`*`是一个匹配任何子主题的通配符，所以
`"rooms:lobby"` 和 `"rooms:kitchen"`同样会匹配这个路由。

Phoenix 将 socket 传输层抽象成两种机制 -- WebSockets 和 Long-Polling. 如果我们希望确定 socket 的类型，我
们可以用`via`指定,像这样：

```elixir
channel "rooms:*", HelloPhoenix.RoomChannel, via: [Phoenix.Transports.WebSocket]
```

每个 socket 可以处理多个通道：

```elixir
channel "rooms:*", HelloPhoenix.RoomChannel, via: [Phoenix.Transports.WebSocket]
channel "foods:*", HelloPhoenix.FoodChannel
```

也可以设置多个 socket 的处理逻辑：

```elixir
socket "/socket", HelloPhoenix.UserSocket
socket "/admin-socket", HelloPhoenix.AdminSocket do
```

### 总结

路由是一个大话题，我们已经谈了很多，现在总结一下：

* 以 HTTP 动词开头的宏会被展开为一个 match 函数。
* 以 resources 开头的宏会被展开为 8 个match函数。
* Resources 可以用 `only:`或 `except:` 限制生成的函数个数。
* 所有的路由都可以被嵌套。
* 所有的路由都可以以一个给定的路径作为作用域。
* 可以用 `as: ` 选项减少重复。
* 可以用在作用域路由上使用 helper 选项避免多余的输入。
