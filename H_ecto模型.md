### ecto 模型

今天绝大多数的 web 应用需要某种形式的数据存储。在 Elixir 生态圈中， Ecto 可以助我们一臂之力。 Ecto 目前
有下列数据库的适配器：

* PostgreSQL
* MySQL
* MSSQL
* SQLite3
* MongoDB

新生成的 Phoenix 应用默认集成了 Ecto 以及 PostgreSQL 数据库的适配器。

如果相对 Ecto 有一个快速全面的了解，请查看 [Ecto 入门手册](https://hexdocs.pm/ecto/getting-started.html)

这篇指南假设我们是使用 Ecto 来生成的工程。如果我们在使用一个老版本的 Phoenix 应用，或者我们在生成项目时使用了
`--no-ecto` 选项。请阅读下面的章节 'Integrating Ecto into an Existing Application'

这篇指南同时假设我们使用的数据库是 PostgreSQL, 如果你要使用 MySQL, 请查看
[MySQL 指南](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/%E8%BF%9B%E9%98%B6%E5%86%85%E5%AE%B9/E_%E4%BD%BF%E7%94%A8MySQL.html)。

当我们安装并配置好了 `Ecto` 以及 `PostgreSQL`, 最简单的使用 Ecto 模型的方法就是使用 `phoenix.gen.html` 任务生
成一个 `资源`。我们就来生成一个包含 `name`, `email`, `bio` 和 `number_of_pets` 字段的 *User* 模型。

```console
$ mix phoenix.gen.html User users name:string email:string bio:string number_of_pets:integer
* creating priv/repo/migrations/20150409213440_create_user.exs
* creating web/models/user.ex
* creating test/models/user_test.exs
* creating web/controllers/user_controller.ex
* creating web/templates/user/edit.html.eex
* creating web/templates/user/form.html.eex
* creating web/templates/user/index.html.eex
* creating web/templates/user/new.html.eex
* creating web/templates/user/show.html.eex
* creating web/views/user_view.ex
* creating test/controllers/user_controller_test.exs

Add the resource to your browser scope in web/router.ex:

    resources "/users", UserController

and then update your repository by running migrations:

    $ mix ecto.migrate
```

注意这个任务生成了很多东西： 一个迁移(migration), 一个控制器，一个控制器的测试，一个模型，一个模型测试，一个视
图，以及一些模板。( a migration, a controller, a controller test, a model, a model test, a view, and
a number of templates.)

让我们先来看看 'web/router.ex' 中添加的这行 `resources "/users", UserController`:

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router
. . .

  scope "/", HelloPhoenix do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
    resources "/users", UserController
  end
. . .
end
```

接下来，我们运行我们的迁移任务：

```console
$ mix ecto.migrate
Compiled lib/hello_phoenix.ex
Compiled web/models/user.ex
Compiled web/views/error_view.ex
Compiled web/controllers/page_controller.ex
Compiled web/views/page_view.ex
Compiled web/router.ex
Compiled web/views/layout_view.ex
Compiled web/controllers/user_controller.ex
Compiled lib/hello_phoenix/endpoint.ex
Compiled web/views/user_view.ex
Generated hello_phoenix.app
** (Postgrex.Error) FATAL (invalid_catalog_name): database "hello_phoenix_dev" does not exist
    lib/ecto/adapters/sql/worker.ex:29: Ecto.Adapters.SQL.Worker.query!/4
    lib/ecto/adapters/sql.ex:187: Ecto.Adapters.SQL.use_worker/3
    lib/ecto/adapters/postgres.ex:58: Ecto.Adapters.Postgres.ddl_exists?/3
    lib/ecto/migration/schema_migration.ex:19: Ecto.Migration.SchemaMigration.ensure_schema_migrations_table!/1
    lib/ecto/migrator.ex:36: Ecto.Migrator.migrated_versions/1
    lib/ecto/migrator.ex:134: Ecto.Migrator.run/4
    (mix) lib/mix/cli.ex:55: Mix.CLI.run_task/2
```

啊哦! 这里的错误消息提示我们现在并没有创建 Ecto 所期望的数据库，目前，我们希望创建的数据库是
`hello_phoenix_dev` - `_dev` 后缀表明现在是测试数据库。

Ecto 解决这个问题很简单，只需要运行： `ecto.create` 任务:

```console
$ mix ecto.create
The database for repo HelloPhoenix.Repo has been created.
```

Mix 会默认我们处于 开发环境，除非我们使用`MIX_ENV=another_environment`明确指定当前环境. 我们的Ecto 任务会从
Mix 中得到环境变量, 并由此决定我们数据库名称的后缀。

现在我们的迁移任务可以正常运行了：

```console
$ mix ecto.migrate
[info] == Running HelloPhoenix.Repo.Migrations.CreateUser.change/0 forward
[info] create table users
[info] == Migrated in 0.3s
```

在我们陷入细节之前，先来做点小实验，我们在项目的根目录运行 `mix phoenix.server` 启动服务器，然后打开
[users index](http://localhost:4000/users) 页面, 我们可以点击 "new user" 生成一些新用户然后编辑或删除他们。默
认情况下， Ecto 会默认模型上的所有字段都是必填的，（我们之后会知道如何修改它），如果我们在创建或者更新的时候不
提供某些字段，我们会看到相应的错误提示信息，我的资源生成器给了我们一个完整的操作和显示 user 记录的手脚架。

好了，我们再看看细节。

如果我们登陆进数据库服务器并连接到 `hello_phoenix_dev` 数据库，我们应该会看到我们的 `users` 表。Ecto 默认我们
使用 id 作为主键，所以我们也会看到一列整型类型的列。


```console
=# \connect hello_phoenix_dev
You are now connected to database "hello_phoenix_dev" as user "postgres".
hello_phoenix_dev=# \d
List of relations
Schema |       Name        |   Type   |  Owner
--------+-------------------+----------+----------
public | schema_migrations | table    | postgres
public | users             | table    | postgres
public | users_id_seq      | sequence | postgres
(3 rows)
```

如果我们看看 `phoenix.gen.html` 生成的迁移任务 (位于：priv/repo/migrations), 我们会看到它会自动添加我们指定的
字段，同样还会自动加入一个 timestamp 列 --- 它由 `timestamps/0` 函数产生，会生成 `inserted_at` 和 `updated_at`
两个字段。

```elixir
defmodule HelloPhoenix.Repo.Migrations.CreateUser do
  use Ecto.Migration

  def change do
    create table(:users) do
      add :name, :string
      add :email, :string
      add :bio, :string
      add :number_of_pets, :integer

      timestamps
    end
  end
end

```

让我们看看实际生成的 `users` 表。

```console
hello_phoenix_dev=# \d users
Table "public.users"
Column     |            Type             | Modifiers
----------------+-----------------------------+----------------------------------------------------
id             | integer                     | not null default nextval('users_id_seq'::regclass)
name           | character varying(255)      |
email          | character varying(255)      |
bio            | character varying(255)      |
number_of_pets | integer                     |
inserted_at    | timestamp without time zone | not null
updated_at     | timestamp without time zone | not null
Indexes:
"users_pkey" PRIMARY KEY, btree (id)
```

注意，尽管我们没有在迁移任务中显示的指定 id 字段，它还是作为默认的主键被添加了。

#### The Repo

`HelloPhoenix.Repo` 模块是 phoenix 应用处理数据库的基石，它被自动生成在 `lib/hello_phoenix/repo.ex`, 内容如下：

```elixir
defmodule HelloPhoenix.Repo do
  use Ecto.Repo, otp_app: :hello_phoenix
end
```
它包含两个主要任务：从 `Ecto.Repo` 中继承所有的基础请求函数，另外将 `opt_app` 的名字和我们的项目名称对应起来。

当 `phoenix.new` 生成我们项目时，也自动产生了一些基础配置信息，让我们来看看 `config/dev.exs`。

```elixir
. . .
# Configure your database
config :hello_phoenix, HelloPhoenix.Repo,
adapter: Ecto.Adapters.Postgres,
username: "postgres",
password: "postgres",
database: "hello_phoenix_dev"
. . .
```

从 `opt_app` 名字和 repo 模块开始，然后设置适配器 - 我们目前用的是 Postgres, 同时还配置了登陆信息，当然，你可
以根据自己的项目需求去修改。

类似的配置还有 `config/test.exs` 和 `config/prod.secret.exs`, 同样可根据你自己的项目需求去修改。

#### 模型

Ecto 模型定义我们 schema 的字段以及它们的类型。并且定义一个包含相同字段的结构体。模型还是我们定义`关系`的地方,
比如，我们的 `User` 模型可能包含很多 `Post` 模型，然后每一个 `Post` 属于一个 `User`。模型同时帮助我们处理数据
验证，以及结合 `changesets` 对数据进行清洗转换等。

这是一个 Phoenix 应用生成的 `User` 模型的例子。

```elixir
defmodule HelloPhoenix.User do
  use HelloPhoenix.Web, :model

  schema "users" do
    field :name, :string
    field :email, :string
    field :bio, :string
    field :number_of_pets, :integer

    timestamps()
  end

  @doc """
  Builds a changeset based on the `struct` and `params`.
  """
  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:name, :email, :bio, :number_of_pets])
    |> validate_required([:name, :email, :bio, :number_of_pets])
  end
end

```

上面的 schema 部分很好理解，我们接下来看看 changesets。

#### Changesets and Validations

Changesets 定义了在渲染之前一个转换数据的管道机制，这些转换包括验证必要数据、数据验证、过滤掉无关的参数等等。

让我们看看一个默认的 changeset 。

```elixir
def changeset(struct, params \\ %{}) do
  struct
  |> cast(params, [:name, :email, :bio, :number_of_pets])
  |> validate_required([:name, :email, :bio, :number_of_pets])
end
```

现在，我们在模型的处理流上有两个部分，第一步，我们将请求参数和需要校验的字段传入 `cast/3` , cast 第一个参数是
struct (由 pipeline 传递过来)， 然后 params 是可能需要更新的请求参数，最后一个是需要被更新的参数列表。 另外
`cast/3` 只抓取 schema 中定义的字段。 接下来, `validate_required/3` 检查 `cast/3` 返回的数据是不是包换所需的字
段，默认情况下， schema 中所有字段都是必须提供的。

让我们看看两种验证它们的方式，第一种，也是最容易的方式是在根目录启动我们的应用 `mix phoenix.server`。然后我们
到 [new users page](http://localhost:4000/users/new) ，在不填写任何信息的情况下，点击 "submit" 提交按钮。这时我
们应该会得到错误提示：某些字段不能为空（这里指 schema 中定义的字段）。

另一种方式是使用 iex，我们先停止服务器然后使用 `iex -S mix phoenix.server` 重新打开。为了少打点字看着方便，我
们给 `HelloPhoenix.User` 模型起个别名：

```console
iex(1)> alias HelloPhoenix.User
nil
```

然后使用一个空的 `User` struct 来创建一个 changeset, 不带参数。

```console
iex(2)> changeset = User.changeset(%User{}, %{})
#Ecto.Changeset<action: nil, changes: %{},
  errors: [name: {"can't be blank", []}, email: {"can't be blank", []},
    bio: {"can't be blank", []}, number_of_pets: {"can't be blank", []}],
  data: #HelloPhoenix.User<>, valid?: false>
```

一旦有了 changeset, 我们可以简单的检查其是否合法：

```console
iex(3)> changeset.valid?
false
```

如果不和合法的话，我们可以查看错误在哪里：

```console
iex(4)> changeset.errors
[name: {"can't be blank", []}, email: {"can't be blank", []},
 bio: {"can't be blank", []}, number_of_pets: {"can't be blank", []}]
```

和之前那个例子看到的错误信息一样。

让我们将`number_of_pets`字段变成可选的，很简单：

```elixir
|> validate_required([:name, :email, :bio])
```

现在只有 name, email 和 bio 才是必须字段了。
那如果我们传递一个 schema 中不存在的字段呢，让我做个小实验。还是像之前一样先做个别名。

```console
iex(1)> alias HelloPhoenix.User
nil
```

然后创建一个合法但多余的 params 参数 `random_key: "random value"`：

```console
iex(2)> params = %{name: "Joe Example", email: "joe@example.com", bio: "An example to all", number_of_pets: 5, random_key: "random value"}
%{email: "joe@example.com", name: "Joe Example", bio: "An example to all",
number_of_pets: 5, random_key: "random value"}
```

然后我们用这个新的 `params` 来创建一个 changeset 。

```console
iex(3)> changeset = User.changeset(%User{}, params)
#Ecto.Changeset<action: nil,
  changes: %{bio: "An example to all", email: "joe@example.com",
    name: "Joe Example", number_of_pets: 5}, errors: [], data: #HelloPhoenix.User<>,
    valid?: true>
```

现在新的 changeset 是合法的。

```console
iex(4)> changeset.valid?
true
```

我们也可以查看 changeset 目前的改变 -- 经过转换完成后的一个 map 。

```console
iex(9)> changeset.changes
%{bio: "An example to all", email: "joe@example.com", name: "Joe Example",
number_of_pets: 5}
```

注意 `random_key` 和 `random_value` 已经在最后的 changeset 中被移除了.

当然我们能做的还不止这些，让我们再来看一个更细粒度的校验例子。

比如我们想给简介字段设置一个长度限制，只需要在 pipeline 后面再加一个针对 `bio` 字段的转换规则即可：

```elixir
def changeset(model, params \\ :empty) do
  model
  |> cast(params, @required_fields, @optional_fields)
  |> validate_length(:bio, min: 2)
end
```
这时如果我们尝试在创建用户的时给 bio 字段一个 'A', 就会得到错误：

```text
Oops, something went wrong! Please check the errors below:
Bio should be at least 2 characters
```

类似的，我们可以限制最大长度：

```elixir
def changeset(model, params \\ :empty) do
  model
  |> cast(params, @required_fields, @optional_fields)
  |> validate_length(:bio, min: 2)
  |> validate_length(:bio, max: 140)
end
```

这时如果超过 140 个字符，也会报错：

```text
Oops, something went wrong! Please check the errors below:
Bio should be at most 140 characters
```

我们也可以使用`validate_format`函数执行自定义的校验规则：

```elixir
def changeset(model, params \\ :empty) do
  model
  |> cast(params, @required_fields, @optional_fields)
  |> validate_length(:bio, min: 2)
  |> validate_length(:bio, max: 140)
  |> validate_format(:email, ~r/@/)
end
```

这时，如果我们试图使用 "personexample.com" 作为 email 字段来创建用户，会报错：

```text
Oops, something went wrong! Please check the errors below:
Email has invalid format
```

还有很多校验和转换的例子，请查看 [Ecto Changeset documentation](http://hexdocs.pm/ecto/Ecto.Changeset.html)。

#### Controller Usage

现在，让我们看看如何在项目中使用 Ecto 。幸运的是，Phoenix 给了我们一个例子（`mix phoenix.gen.html`）, 在 `HelloPhoenix.UserController`
中。

让我们来仔细看看 Ecto 是怎么在这个生成的控制器中起作用的。
不过在开始前还是先做一个简单的别名映射，以节省体力（之后可以用 `%User{}` 代替 `%HelloPhoenix.User{}` 了）。

```elixir
defmodule HelloPhoenix.UserController do
. . .
  alias HelloPhoenix.User
. . .
end
```

我们来看看第一个 action, `index`.

```elixir
def index(conn, _params) do
  users = Repo.all(User)
  render(conn, "index.html", users: users)
end
```

这个 action 的目的是从数据库中获取到所有的 users 信息，并用 index.html.eex 这个模板将他们显示出来。我们使用了
内建的 `Repo.all/1` 请求来完成 （User 是 HelloPhoenix.User 的别名，之前已经解释过），就这么简单。

注意这里我们并没有使用 changeset, changeset 假设你写入数据到数据库时不干净，需要"清洗和校验", 但从数据库中取得
已经存在其中的数据时就不需要了，因为这些数据已经是合法的了。


现在，再来看看 new action, 注意这里我们使用了一个 changeset, 即便 new action 本身并不需要任何参数（它只是发送
一个空表单到前端），这里需要 changeset 的理由是我们在其他页面比如创建页面出错时可能会重定向到 new action, 需要
显示错误信息什么的。

```elixir
def new(conn, _params) do
  changeset = User.changeset(%User{})
  render(conn, "new.html", changeset: changeset)
end
```

接下来是将参数传递过来创建 user：

```elixir
def create(conn, %{"user" => user_params}) do
  changeset = User.changeset(%User{}, user_params)

  case Repo.insert(changeset) do
    {:ok, _user} ->
      conn
      |> put_flash(:info, "User created successfully.")
      |> redirect(to: user_path(conn, :index))
    {:error, changeset} ->
      render(conn, "new.html", changeset: changeset)
  end
end
```

我们先通过 `模式匹配` 得到 user 的参数，然后用这些参数创建一个 changeset，并调用 `Repo.insert/1`, 如果这个
changeset是合法的，它会返回 {:ok, user} 包含刚刚插入的 user 模型。 再之后，我们设置一条 flash 信息，最后重定向
到 index 首页。

如果插入失败了，我们重新渲染 new.html 页面给用户，并显示错误信息。

在 show action 中，我们用内建的 `Repo.get!/2` 函数，根据请求的 id 找到目标 user， 这里没有使用 changeset 是因
为我们任务数据库里的数据是合法的。

和 index 类似。

```elixir
def show(conn, %{"id" => id}) do
  user = Repo.get!(User, id)
  render(conn, "show.html", user: user)
end
```

在 edit action 中， 我们结合 show 和 index 来使用 `Ecto`, 我们先用`模式匹配`从请求参数中取到 id， 然后使用
`Repo.get!/2`从数据库中获取用户，我们同时还是用了 changeset, 因为当用户使用 `PUT` 请求时， 参数可能包含错误。

```elixir
def edit(conn, %{"id" => id}) do
  user = Repo.get!(User, id)
  changeset = User.changeset(user)
  render(conn, "edit.html", user: user, changeset: changeset)
end
```

update action 几乎和 create 那个一样，唯一的区别是我们使用了 `Repo.update/1` 而不是 `Repo.insert/1`。
`Repo.update/1` 和 changeset 一起使用时自动检查那些字段被更新了，如果没有字段更新，那么 `Repo.update/1` 不会向
数据库中写入任何数据， `Repo.insert/1` 则是每次都将所有数据写入数据库。

```elixir
def update(conn, %{"id" => id, "user" => user_params}) do
  user = Repo.get!(User, id)
  changeset = User.changeset(user, user_params)

  case Repo.update(changeset) do
    {:ok, user} ->
      conn
      |> put_flash(:info, "User updated successfully.")
      |> redirect(to: user_path(conn, :show, user))
    {:error, changeset} ->
      render(conn, "edit.html", user: user, changeset: changeset)
  end
end
```

最后，我们看看 delete action, 我们使用`模式匹配`取到 id , 然后使用 `Repo.get!/2` 取到user， 然后只需简单的调用
`Repo.delete!/1`, 设置一条 flash 消息，然后重定向到 index 首页。

```elixir
def delete(conn, %{"id" => id}) do
  user = Repo.get!(User, id)

  # Here we use delete! (with a bang) because we expect
  # it to always work (and if it does not, it will raise).
  Repo.delete!(user)

  conn
  |> put_flash(:info, "User deleted successfully.")
  |> redirect(to: user_path(conn, :index))
end
```

我们简单学习了一下如何在控制器中使用 Ecto , 更详细的内容请查看 [Ecto documentation](http://hexdocs.pm/ecto/)。

### 模型关系和依赖

假设我们正在编写一个很简单的视频分享类的应用，为了吸引用户，我们先来创建一个 video 模型。

```console
$ mix phoenix.gen.model Video videos name:string approved_at:datetime description:text likes:integer views:integer user_id:references:users
* creating priv/repo/migrations/20150611051558_create_video.exs
* creating web/models/video.ex
* creating test/models/video_test.exs

$ mix ecto.migrate
```

如果我们要编写一个现代的网页应用，只处理独立的表是不行的，我们需要将数据关联起来，类似于 Ruby 的 ActiveRecord，
Ecto 也提供了类似的语法糖来处理这些关系，比如：

`Schema.has_many/3` 声明`一对多`关系,比如，在我们的视频分享应用中，一个 user 可能会上传多个 video 。

`Schema.belongs_to/3` 声明父子式的 `属于` 关系 比如，一个 video 只属于一个 user。

`Schema.has_one/3` 声明`一对一`关系。和 `has_many` 类似，只是返回的不是模型的集合而是一个模型。比如用户上传了
很多 video, 但他可能只喜欢其中的一个。

这是一个在 `web/models/user.ex` 中声明 `has_many` 的例子：

```elixir
defmodule HelloPhoenix.User do
. . .
  schema "users" do
    field :name, :string
    field :email, :string
    field :bio, :string
    field :number_of_pets, :integer

    has_many :videos, HelloPhoenix.Video
    timestamps()
  end
. . .
end
```

因为之前我们是使用的生成器来生成的 `Video` 模型，并显示的用`user_id:references:users` 制定了它的从属关系,所以
`belongs_to` 内容被自动添加到了 `web/models/video.ex` 中：

```elixir
defmodule HelloPhoenix.Video do
. . .
  schema "videos" do
    field :name, :string
    field :approved_at, Ecto.DateTime
    field :description, :string
    field :likes, :integer
    field :views, :integer
    belongs_to :user, HelloPhoenix.User

    timestamps()
  end
. . .
end
```

注意我们并没有在 video 模型中声明 `user_id`, 我们把它添加到必须字段中去。

我们可以在 `web/controllers/user_controller.ex` 使用它们：

```elixir
defmodule HelloPhoenix.UserController do
. . .
  def index(conn, _params) do
    users = User |> Repo.all |> Repo.preload([:videos])
    render(conn, "index.html", users: users)
  end

  def show(conn, %{"id" => id}) do
    user = User |> Repo.get!(id) |> Repo.preload([:videos])
    render(conn, "show.html", user: user)
  end
. . .
end
```

因为我们在 `HelloPhoenix.User` 中声明了`数据关系`， `%User{}` 会包含一个 videos 的属性，为了将它显示出来，我们
需要显示的告诉 Ecto。 注意 `Repo.preload/2` 会自动判断是 `一对多` 还是 `多对多`。

### 在现有的项目中集成 Ecto

在一个现有的 Phoenix 应用中使用 Ecto 并不难，一旦安装了 `Ecto` 和 `Postgrex` 依赖项目，使用 mix 任务就可以搞定
了。

#### 添加 Ecto 和 Postgrex 作为依赖项

使用 Ecto, 你可以使用 `phoenix_ecto` 包。 同时我们可以直接添加 `postgres` 包，就像添加项目中其他的依赖一样。

```elixir
defmodule HelloPhoenix.Mixfile do
  use Mix.Project

. . .
  # Specifies your project dependencies
  #
  # Type `mix help deps` for examples and options
  defp deps do
    [{:phoenix, "~> 1.2.0"},
     {:phoenix_ecto, "~> 3.0"},
     {:postgrex, ">= 0.0.0"},
     {:phoenix_html, "~> 2.6"},
     {:phoenix_live_reload, "~> 1.0", only: :dev},
     {:gettext, "~> 0.11",
     {:cowboy, "~> 1.0"},]
  end
end
```

然后运行 `mix do deps.get, compile` 将其添加到我们的项目中。

```console
$ mix do deps.get, compile
Running dependency resolution
Dependency resolution completed successfully
. . .
```

接下来要添加的是我们应用的 repo, 运行 `ecto.gen.repo` 任务。

```console
$ mix ecto.gen.repo
* creating lib/hello_phoenix
* creating lib/hello_phoenix/repo.ex
* updating config/config.exs
Don't forget to add your new repo to your supervision tree
(typically in lib/hello_phoenix.ex):

worker(HelloPhoenix.Repo, [])
```
这个任务同时还为我们的 repo 创建了一个一个目录。

```elixir
defmodule HelloPhoenix.Repo do
  use Ecto.Repo, otp_app: :hello_phoenix
end
```

同时，它自动在我们的配置文件 `config/config.exs` 中添加了一些配置。如果我们对不同的开发环境配置了不同的参数
（事实上也应该这样），我们需要增加类似的配置代码到 `config/dev.exs`, `config/test.exs` 和 `config/prod.secret.exs`。

```elixir
. . .
config :hello_phoenix, HelloPhoenix.Repo,
adapter: Ecto.Adapters.Postgres,
database: "hello_phoenix_repo",
username: "user",
password: "pass",
hostname: "localhost"
```

我们还需要监听 ecto.gen.repo 的输出，并把它作为一个 child worker 加到应用的`监视树` (supervision tree)上。

```elixir
defmodule HelloPhoenix do
  use Application
. . .
    children = [
      # Start the endpoint when the application starts
      supervisor(HelloPhoenix.Endpoint, []),
      # Start the Ecto repository
      supervisor(HelloPhoenix.Repo, []),
      # Here you could define other workers and supervisors as children
      # worker(HelloPhoenix.Worker, [arg1, arg2, arg3]),
    ]
. . .
end
```
