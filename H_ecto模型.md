
### ecto 模型

今天绝大多数的 web 应用需要某种形式的数据存储。在 Elixir 生态圈中， Ecto 可以助我们一臂之力。

Ecto 目前有下列数据库的适配器：

* PostgreSQL
* MySQL
* MSSQL
* SQLite3
* MongoDB

新生成的 Phoenix 应用默认集成了 Ecto 以及 PostgreSQL 数据库的适配器。

如果想对 Ecto 有一个快速全面的了解，请查看[Ecto 入门手册](https://hexdocs.pm/ecto/getting-started.html)。想要概览 Phoenix相关的所有 Ecto mix 相关任务，请查看[mix tasks guide](http:www.phoenixframework.org/docs/mix-tasks#section-ecto-specific-mix-tasks).

这篇指南假设我们是使用 Ecto 来生成的工程。如果我们在使用一个老版本的 Phoenix 应用，或者我们在生成项目时使用了`--no-ecto` 选项。请阅读下面的章节 'Integrating Ecto into an Existing Application'

这篇指南同时假设我们使用的数据库是 PostgreSQL, 如果你要使用 MySQL, 请查看[MySQL 指南](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/%E8%BF%9B%E9%98%B6%E5%86%85%E5%AE%B9/E_%E4%BD%BF%E7%94%A8MySQL.html)。


默认的 Postgres 配置有一个名字和密码都为 'postgres' 的超级账户。 如果你查看```config/dev.exs ``` 你会看见 Phoenix 已经为你生成了。如果你机器上的数据库没雨这个账号，你可以通过终端命令 `psql` 连接到数据库，并执行下列命令：

```
CREATE USER postgres;
ALTER USER postgres PASSWORD 'postgres';
ALTER USER postgres WITH SUPERUSER;
```

现在我们已经将 Ecto 和 PostgreSQL 安装并配置好了，使用 Ecto 最简单的方式就是先用`phx.gen.schema` 生成一个 Ecto *schema* 了。 Ecto 的 schema 简单的说就是 Elixir 数据类型和微博数据的映射关系, 比如和数据库的表。 让我生成一个 `User` schema, 包含`name`, `email`, `bio`, 和 `number_of_pets` 字段.

```console
$ mix phx.gen.schema User users name:string email:string \
bio:string number_of_pets:integer

* creating ./lib/hello/user.ex
* creating priv/repo/migrations/20170523151118_create_user.exs

Remember to update your repository by running migrations:

   $ mix ecto.migrate
```

这个命令生成了一些文件啊，首先，我们有了 `user.ex` 文件， 包含我们传递进来的 schema 以及相关字段的定义信息。然后，一个位于 `priv/repo/migrations` 的迁移文件(migration file), 用来在数据库中根据刚才的 Schema 创建相应的数据表。

现在我们根据提示来运行 migration. 如果我们的我们的库还没有创建，记得先运行 `ecto.create`.

```console
$ mix ecto.migrate
Compiling 1 file (.ex)
Generated hello app

[info]  == Running Hello.Repo.Migrations.CreateHello.User.change/0 forward

[info]  create table users

[info]  == Migrated in 0.0s
```

Mix 默认我们处于开发环境，除非你显式的指定环境 `MIX_ENV=another_environment mix some_task` 。这样 mix 会从传入的环境参数自动指定数据库名字的后缀。

现在当我们登入数据库服务器，并连接到 `hello_dev` 数据库，我们应该能看到刚才我们创建的 `users`表了， Ecto 还会自动为我们生成一个整数 `id` 字段作为主键。


```console
$ psql -U postgres

Type "help" for help.

postgres=# \connect hello_dev
You are now connected to database "hello_dev" as user "postgres".
hello_dev=# \d
                List of relations
 Schema |       Name        |   Type   |  Owner
--------|-------------------|----------|----------
 public | schema_migrations | table    | postgres
 public | users             | table    | postgres
 public | users_id_seq      | sequence | postgres
(3 rows)
hello_dev=# \q
```

现在我们来看看 `phx.gen.schema` 在 `priv/repo/migrations` 目录下生成的迁移文件,除了我们指定的字段外，还使用 `timestamps/0` 函数自动为我们生成了 `inserted_at`和 `updated_at` 字段。

```elixir
defmodule Hello.Repo.Migrations.CreateHello.User do
  use Ecto.Migration

  def change do
    create table(:users) do
      add :name, :string
      add :email, :string
      add :bio, :string
      add :number_of_pets, :integer

      timestamps()
    end

  end
end
```

在实际的 `users` 表中会被转换成： 

```console
hello_dev=# \d users
Table "public.users"
Column         |            Type             | Modifiers
---------------|-----------------------------|----------------------------------------------------
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

注意，尽管我们没有迁移文件中列出 `id` 字段， 该字段还是被默认为主键添加在数据表中。


## The Repo

`Hello.Repo` 模块是我们操作数据库的基础部分, 定义在 `lib/hello/repo.ex` 中：

```elixir
defmodule Hello.Repo do
  use Ecto.Repo, otp_app: :hello

  @doc """
  Dynamically loads the repository url from the
  DATABASE_URL environment variable.
  """
  def init(_, opts) do
    {:ok, Keyword.put(opts, :url, System.get_env("DATABASE_URL"))}
  end
end
```

这里的 repo 有三个主要作用 - 1.导入 `Ecto.Repo` 中的通用查询函数 。 2. 将`opt_app` 名字设置成我们项目的名字。 3. 通过传入的参数初始化数据库适配器。 我们会在后面详细谈及。

当 `phx.new` 生成项目时，它也会包含一些基本的数据库设置，在文件中
`config/dev.exs` 中: 

```elixir
...
# Configure your database
config :hello, Hello.Repo,
  adapter: Ecto.Adapters.Postgres,
  username: "postgres",
  password: "postgres",
  database: "hello_dev",
  hostname: "localhost",
  pool_size: 10
...
```

你可以根据自己项目的实际需求来更改，同时 Phoenix 还提供了针对不同环境的配置文件 `config/test.exs` and `config/prod.secret.exs`.从 `opt_app` 名字和 repo 模块开始，然后设置适配器 - 我们目前用的是 Postgres, 同时还配置了登陆信息，当然，你可以根据自己的项目需求去修改。

类似的, Phoenix 还提供了针对不同环境的配置文件 `config/test.exs` 和 `config/prod.secret.exs`, 同样可根据你自己的项目需求去修改。


#### The Schema

Ecto Schema 定义我们的 Elixir 数据和外部数据集的映射关系 。同时还是我们定义`关系`的地方,比如，我们的 `User` 模型可能包含很多 `Post` 模型，然后每一个 `Post` 属于一个 `User`。模型同时帮助我们处理数据验证，以及结合 `changesets` 对数据进行清洗转换等。

这是一个 Phoenix 应用生成的 `User` 模型的例子。

```elixir
defmodule Hello.User do
  use Ecto.Schema
  import Ecto.Changeset
  alias Hello.User


  schema "users" do
    field :bio, :string
    field :email, :string
    field :name, :string
    field :number_of_pets, :integer

    timestamps()
  end

  @doc false
  def changeset(%User{} = user, attrs) do
    user
    |> cast(attrs, [:name, :email, :bio, :number_of_pets])
    |> validate_required([:name, :email, :bio, :number_of_pets])
  end
end
```

上面的 schema 部分很好理解，我们接下来看看 changesets。

#### Changesets and Validations

Changesets 定义了了一个在渲染之前清洗转换数据的机制，这些转换包括验证必要数据、数据验证、过滤掉无关的参数等等, 同时 Ecto Repos 还会根据实际变动的数据"最小化"的更新数据库。

让我们看看一个默认的 changeset 。

```elixir
  def changeset(%User{} = user, attrs) do
    user
    |> cast(attrs, [:name, :email, :bio, :number_of_pets])
    |> validate_required([:name, :email, :bio, :number_of_pets])
  end
```

现在，我们在模型的处理流上有两个部分，第一步，我们将请求参数和需要校验的字段传入 `cast/3` , cast 第一个参数是struct (由 pipeline 传递过来)， 然后 params 是可能需要更新的请求参数，最后一个是需要被更新的参数列表。 另外`cast/3` 只抓取 schema 中定义的字段。 接下来, `validate_required/3` 检查 `cast/3` 返回的数据是不是包换所需的字段，默认情况下， schema 中所有字段都是必须提供的。

我们可以使用 iex 来验证一下， 通过 `iex -S mix` 。为了少打点字看着方便，我们给 `Hello.User` 模型起个别名：

```console
$ iex -S mix

iex> alias Hello.User
Hello.User
```

然后使用一个空的 `User` struct 来创建一个 changeset, 不带参数。

```console
iex> changeset = User.changeset(%User{}, %{})

#Ecto.Changeset<action: nil, changes: %{},
 errors: [name: {"can't be blank", [validation: :required]},
  email: {"can't be blank", [validation: :required]},
  bio: {"can't be blank", [validation: :required]},
  number_of_pets: {"can't be blank", [validation: :required]}],
 data: #Hello.User<>, valid?: false>
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

现在应该只有 `name`, `email` 和 `bio` 是不能为空了，我们通过在 `iex` 里运行`recompile()` 来验证一下: 

```elixir
iex> recompile()
Compiling 1 file (.ex)
:ok

iex> changeset = User.changeset(%User{}, %{})
#Ecto.Changeset<action: nil, changes: %{},
 errors: [name: {"can't be blank", [validation: :required]},
  email: {"can't be blank", [validation: :required]},
  bio: {"can't be blank", [validation: :required]}],
 data: #Hello.User<>, valid?: false>

iex> changeset.errors
[name: {"can't be blank", [validation: :required]},
 email: {"can't be blank", [validation: :required]},
 bio: {"can't be blank", [validation: :required]}]
```

那如果我们传递一个 schema 中不存在的字段呢，让我做个小实验, 添加一个合法但多余的 params 参数 `random_key: "random value"`：

```console
iex> params = %{name: "Joe Example", email: "joe@example.com", bio: "An example to all", number_of_pets: 5, random_key: "random value"}
%{email: "joe@example.com", name: "Joe Example", bio: "An example to all",
number_of_pets: 5, random_key: "random value"}
```

然后我们用这个新的 `params` 来创建一个 changeset 。

```console
iex> changeset = User.changeset(%User{}, params)
#Ecto.Changeset<action: nil,
 changes: %{bio: "An example to all", email: "joe@example.com",
   name: "Joe Example", number_of_pets: 5}, errors: [],
 data: #Hello.User<>, valid?: true>
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
  def changeset(%User{} = user, attrs) do
    user
    |> cast(attrs, [:name, :email, :bio, :number_of_pets])
    |> validate_required([:name, :email, :bio, :number_of_pets])
    |> validate_length(:bio, min: 2)
  end
```

这时如果我们尝试在创建用户的时给 bio 字段一个 'A', 就会得到错误：

```console
iex> changeset = User.changeset(%User{}, %{bio: "A"})
iex> changeset.errors[:bio]
{"should be at least %{count} character(s)",
 [count: 2, validation: :length, min: 2]}
```

类似的，我们可以限制最大长度：

```elixir
  def changeset(%User{} = user, attrs) do
    user
    |> cast(attrs, [:name, :email, :bio, :number_of_pets])
    |> validate_required([:name, :email, :bio, :number_of_pets])
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

## 数据持久化

目前为止我们谈论了不少关于 migrations 和 data-storage 的内容，但我们还没有将任何 schema 或者 changesets 持久化到数据库中。 Ecto Repo 作为存储层的接口，为我们封装了的底层细节，比如不同数据库适配器的通信，连接池，错误处理等等，作为调用者，我们只需关心 `获取` 和 `保存` 数据

让我们再次调出 iex, 使用 `iex -S mix`, 然后手动在数据库中插入一些用户信息。

```console
iex> alias Hello.{Repo, User}
[Hello.Repo, Hello.User]
iex> Repo.insert(%User{email: "user1@example.com"})
[debug] QUERY OK db=4.6ms
INSERT INTO "users" ("email","inserted_at","updated_at") VALUES ($1,$2,$3) RETURNING "id" ["user1@example.com", {{2017, 5, 23}, {19, 6, 4, 822044}}, {{2017, 5, 23}, {19, 6, 4, 822055}}]
{:ok,
 %Hello.User{__meta__: #Ecto.Schema.Metadata<:loaded, "users">,
  bio: nil, email: "user1@example.com", id: 3,
  inserted_at: ~N[2017-05-23 19:06:04.822044], name: nil, number_of_pets: nil,
  updated_at: ~N[2017-05-23 19:06:04.822055]}}

iex> Repo.insert(%User{email: "user2@example.com"})
[debug] QUERY OK db=5.1ms
INSERT INTO "users" ("email","inserted_at","updated_at") VALUES ($1,$2,$3) RETURNING "id" ["user2@example.com", {{2017, 5, 23}, {19, 6, 8, 452545}}, {{2017, 5, 23}, {19, 6, 8, 452556}}]
{:ok,
 %Hello.User{__meta__: #Ecto.Schema.Metadata<:loaded, "users">,
  bio: nil, email: "user2@example.com", id: 4,
  inserted_at: ~N[2017-05-23 19:06:08.452545], name: nil, number_of_pets: nil,
  updated_at: ~N[2017-05-23 19:06:08.452556]}}
```

注意： 在 dev 模式下可以看到 debug 日志，我们插入几条数据后再将它们读取出来：

```console
iex> Repo.all(User)
[debug] QUERY OK source="users" db=2.7ms
SELECT u0."id", u0."bio", u0."email", u0."name", u0."number_of_pets", u0."inserted_at", u0."updated_at" FROM "users" AS u0 []
[%Hello.User{__meta__: #Ecto.Schema.Metadata<:loaded, "users">,
  bio: nil, email: "user1@example.com", id: 3,
  inserted_at: ~N[2017-05-23 19:06:04.822044], name: nil, number_of_pets: nil,
  updated_at: ~N[2017-05-23 19:06:04.822055]},
 %Hello.User{__meta__: #Ecto.Schema.Metadata<:loaded, "users">,
  bio: nil, email: "user2@example.com", id: 4,
  inserted_at: ~N[2017-05-23 19:06:08.452545], name: nil, number_of_pets: nil,
  updated_at: ~N[2017-05-23 19:06:08.452556]}]
```

不能更简单了！ `Repo.all/1` 使用一个数据源，在当前是 `User` schema, 然后将其转化为对应的 SQL 查询语句送入数据库，取回数据后，Repo 又根据 User schema 把数据转回Elixir 的数据结构。 不只是这种简单查询 -- Ecto 包含了一整套的 DSL 查询语言和强大的特性， 比如 SQL 注入攻击防护, 查询的编译时优化等等。我们来试试：

```console
iex> import Ecto.Query
Ecto.Query

iex> Repo.all(from u in User, select: u.email)
[debug] QUERY OK source="users" db=2.4ms
SELECT u0."email" FROM "users" AS u0 []
["user1@example.com", "user2@example.com"]
```

首先，我们引入 `Ecto.Query`, 它从 Ecto 的查询 DSL 中导入 `from`, 然后，我们创建一个选择所有用户的 email 的查询。

我们再看一个例子：


```console
iex)> Repo.one(from u in User, where: ilike(u.email, "%1%"),
                               select: count(u.id))
[debug] QUERY OK source="users" db=1.6ms SELECT count(u0."id") FROM "users" AS u0 WHERE (u0."email" ILIKE '%1%') []
1
```

查询用户 emaill 中包含 "1" 的用户总数, 这只是 Ecto 能力的冰山一角，其他的比如`sub-querying`, `interval queries`, 以及 `advanced select statements`, 我们再看
一个例子： 将用户 id 和 email 以 map 形式查询出来: 

```console
iex> Repo.all(from u in User, select: %{u.id => u.email})
[debug] QUERY OK source="users" db=0.9ms
SELECT u0."id", u0."email" FROM "users" AS u0 []
[%{3 => "user1@example.com"}, %{4 => "user2@example.com"}]
```

很 cool 对吧，这个查询在从数据库获取用户 email 的同时，高效的将结果转换成 map ,你可以查看 [Ecto.Query documentation](https://hexdocs.pm/ecto/Ecto.Query.html#content)里的更多案例。

除了插入以外，还有 `Repo.update/1` and `Repo.delete/1` 等修改和删除数据。 它们还有对应的批量操作版本：  `Repo.insert_all`, `Repo.update_all`, and `Repo.delete_all`。

关于 Ecto 的更多内容查看这里 [Ecto documentation](https://hexdocs.pm/ecto/)

在后面的 [context] 章节中，我们还会学习如何将 Ecto 与业务逻辑更好的结合起来以及怎样用 Phoenix 的新特性构建可扩展的，强健的应用。
