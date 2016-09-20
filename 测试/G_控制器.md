# 测试 Controllers

我们来看看怎样用测试驱动一个提供 JSON api 的 controller。

Phoenix 有一个生成 JSON 资源的生成器，像这样：

### 起步

```bash
mix phoenix.gen.json Thing things some_attr:string another_attr:string
```

上面的例子中，`Thing` 是模型， `things` 是数据库表名， `some_attr` 和
`another_attr` 是在表中类型为 string 的列。注意先不要运行这行代码，我们接下来通
过测试驱动的办法来探明这个命令都做了些什么。

我们先来创建一个 `User` 模型。 如果你对这个生成器不熟悉，可以参考
[this section of the Mix guide](http://www.phoenixframework.org/docs/mix-tasks#section--mix-phoenix-gen-model-).

```bash
$ mix phoenix.gen.model User users name:string email:string
```

然后运行迁移：

```bash
$ mix ecto.migrate
```

### 测试驱动

我们需要的是一个包含标准 CRUD actions 的控制器。因为我们用的是测试驱动的思路，先
来在 `test/controllers` 目录中创建一个 `user_controller_test.exs` 文件。

```elixir
# test/controllers/user_controller_test.exs

defmodule HelloPhoenix.UserControllerTest do
  use HelloPhoenix.ConnCase, async: true

end
```

TDD 的方式有很多种，我们这里假设每一个 action 在理想情况下的功能，然后在处理可能
发生的错误。

```elixir
# test/controllers/user_controller_test.exs

defmodule HelloPhoenix.UserControllerTest do
  use HelloPhoenix.ConnCase, async: true

  test "index/2 responds with all Users"

  describe "create/2" do
    test "Creates, and responds with a newly created user if attributes are valid"
    test "Returns an error and does not create a user if attributes are invalid"
  end

  describe "show/2" do
    test "Responds with a newly created user if the user is found"
    test "Responds with a message indicating user not found"
  end

  describe "update/2" do
    test "Edits, and responds with the user if attributes are valid"
    test "Returns an error and does not edit the user if attributes are invalid"
  end

  test "delete/2 and responds with :ok if the user was deleted"

end
```

这里我们测试一个标准的 JSON API 的 CRUD 接口， 其中的 `index` 和 `delete` 我们只
测试了理想情况(happy path), 实际项目中可能会遇到跨域或者权限什么的,这里先不考虑。

`Create`, `show` 和 `update` 有一些共同的特点，就是需要某种办法来找到某个资源，
这个资源可能是不存在的，所以我们把他们用 `describe` 组织起来。

让我们来运行测试：

```bash
$ mix test test/controllers/user_controller_test.exs
```

我们得到了 8 个错误提示我们 "Not yet implemented", 因为我们的测试块中还没有任何
代码。

让我们来添加第一个测试，从 `index/2` 开始。

```elixir
defmodule HelloPhoenix.UserControllerTest do
  use HelloPhoenix.ConnCase, async: true

  alias HelloPhoenix.{Repo, User}

  test "index/2 responds with all Users", %{conn: conn} do
    users = [ User.changeset(%User{}, %{name: "John", email: "john@example.com"}),
              User.changeset(%User{}, %{name: "Jane", email: "jane@example.com"}) ]

    Enum.each(users, &Repo.insert!(&1))

    response = conn
    |> get(user_path(conn, :index))
    |> json_response(200)

    expected = %{
      "data" => [
        %{ "name" => "John", "email" => "john@example.com" },
        %{ "name" => "Jane", "email" => "jane@example.com" }
      ]
    }

    assert response == expected
  end
```

我们来仔细看看。 每一个测试用例都有一个 conn 传递进来，所以我们可以使用模式匹配
来得到我们需要的 conn。我们创建 users, 然后使用 `get` 函数来模拟 `GET` 请求到
`UserController` index action， 然后把结果通过合适的 HTTP 状态码传递给
`json_response/2`， 然后将返回的 json 数据和我们 `expected` 变量做对比，测试是否
一致。

我们期望的 JSON 返回值是一个顶层有一个 "data" 键，其中包含一个 users 列表的结构，
列表中的每个 user 包含 `"name"` 和 `"email"` 属性。

当我们运行测试会提示我们没有 `user_path` 函数。

接下来，在我们的路由中，在 API pipe 那里添加一个 `User` 资源 :

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
    resources "/users", HelloPhoenix.UserController
  end

  #...
```

再次运行，我们会得到一个新的错误，我们并没有 `UserController`。 让我们来添加一个,
使用 `index/2` action 返回所有 users：

```elixir
defmodule HelloPhoenix.UserController do
  use HelloPhoenix.Web, :controller

  alias HelloPhoenix.{User, Repo}

  def index(conn, _params) do
    users = Repo.all(User)
    render conn, "index.json", users: users
  end

end
```

再次运行，又提示我们没有视图 (view)。我们来添加一个。格式要像test中指明的那样，
顶层是一个 `"data"` key，包含一个 users 列表，列表中的每个 user 包含 `"name"`和
`"email"`字段。

```elixir
defmodule HelloPhoenix.UserView do
  use HelloPhoenix.Web, :view

  def render("index.json", %{users: users}) do
    %{data: render_many(users, HelloPhoenix.UserView, "user.json")}
  end

  def render("user.json", %{user: user}) do
    %{name: user.name, email: user.email}
  end

end
```

这样，我们的测试通过了。

接下来我们再看看测试中的 `show/2` 部分。

```elixir
  describe "show/2" do
    test "Responds with a newly created user if the user is found"
    test "Responds with a message indicating user not found"
  end
```

我们使用指定行号的方式来运行这个测试(注意根据你自己文件的行号调整)：

```bash
$ mix test test/controllers/user_controller_test.exs:32
```

测试不出意外的失败了，因为我们还没有实现。。。

让我们来构建一个测试描述请求 `show/2` 成功时的样子。

```elixir
test "Reponds with a newly created user if the user is found", %{conn: conn} do
  user = User.changeset(%User{}, %{name: "John", email: "john@example.com"})
  |> Repo.insert!

  response = conn
  |> get(user_path(conn, :show, user.id))
  |> json_response(200)

  expected = %{ "data" => %{ "name" => "John", "email" => "john@example.com" } }

  assert response == expected
end
```

这和我们之前的 `index/2` 测试很像， 除了 `show/2` 需要一个 user id, 并且我们的数
据是一个单独的 JSON 结构体而不是一个数组。

运行测试提示我们需要一个 `show/2` action, 让我们来添加一个：

```elixir
defmodule HelloPhoenix.UserController do
  use HelloPhoenix.Web, :controller

  alias HelloPhoenix.{User, Repo}

  def index(conn, _params) do
    users = Repo.all(User)
    render conn, "index.json", users: users
  end

  def show(conn, %{"id" => id}) do
    case Repo.get(User, id) do
      user -> render conn, "show.json", user: user
    end
  end
end
```

你也许会注意到我们只处理了成功找到一个 user 的情况。当我们做 TDD 的时候我们需要
尽快让测试通过，我们之后会添加更多的代码来处理没有找到的错误情况。

再次运行测试提示我们需要一个 `render/2` 函数来匹配 `"show.json"`:

```elixir
defmodule HelloPhoenix.UserView do
  use HelloPhoenix.Web, :view

  def render("index.json", %{users: users}) do
    %{data: render_many(users, HelloPhoenix.UserView, "user.json")}
  end

  def render("show.json", %{user: user}) do
    %{data: render_one(user, HelloPhoenix.UserView, "user.json")}
  end

  def render("user.json", %{user: user}) do
    %{name: user.name, email: user.email}
  end

end
```

再次运行，测试通过了。

最后我们来处理一下在 `show/2` 中没有找到 user 的情况。

实际上处理办法有很多，一个可能的解决办法如下：

我们使用一个不存在的 user id 来请求 `user_path`, 它会返回一个 404 状态码和一个错
误信息：

```elixir
test "Responds with a message indicating user not found", %{conn: conn} do
  response = conn
  |> get(user_path(conn, :show, 300))
  |> json_response(404)

  expected = %{ "error" => "User not found." }

  assert response == expected
end
```

我们想通过一个 HTTP 状态码告诉请求者这个资源没有找到，并且还提供了一个错误信息。

控制器端添加：

```elixir
def show(conn, %{"id" => id}) do
  case Repo.get(User, id) do
    nil -> conn |> put_status(404) |> render("error.json")
    user -> render conn, "show.json", user: user
  end
end
```

视图添加：

```elixir
def render("error.json", _assigns) do
  %{error: "User not found."}
end
```

这样，测试就通过了。

剩下的测试就留作练习吧， 祝你测试愉快!
