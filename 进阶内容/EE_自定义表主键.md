# 自定义主键

有时我们只在一个已经存在的数据库之上来开发我们的应用，我们不能控制数据库的是怎么创建的，而为此修改数据库代价又
比较昂贵。

Ecto 需要一个表中有一个自增的整型字段作为主键。但如果这个现有的数据库是以一个字符串作为主键呢？ 没关系，我们可
以使用自定义的主键来创建模型。Ecto 会像我们有整型主键一样正常工作。

> *注意* : 尽管 Ecto 支持非整型主键，我们最好还是在新项目中使用整型作为主键。

> 另外，这里选择 `name` 只是为了演示的简单性， `name` 并不是主键的最佳选择。

比如我们需要一个 JSON 资源存储一个球队的信息表(每行是一个球员信息)，每个球员有一个 `名字`, 一个 `场上位置`, 以
及一个 `球衣号码`, 现有的数据库需要每个表有一个字符串类型的主键 ( The database that will back this
resource requires that each table have a string for a primary key.)。

我们可以像这样生成资源。

```console
$ mix phoenix.gen.json Player players name:string position:string number:integer
* creating priv/repo/migrations/20150908003815_create_player.exs
* creating web/models/player.ex
* creating test/models/player_test.exs
* creating web/controllers/player_controller.ex
* creating web/views/player_view.ex
* creating test/controllers/player_controller_test.exs
* creating web/views/changeset_view.ex

Add the resource to your api scope in web/router.ex:

    resources "/players", PlayerController

and then update your repository by running migrations:

    $ mix ecto.migrate
```

我们首先在路由中只用 `api` scope 添加资源的路由。

```elixir
. . .
scope "/api", HelloPhoenix do
  pipe_through :api

  resources "/players", PlayerController
end
. . .
```

现在我们需要在生成的文件上做一些小改动。

先来看看迁移任务的文件， `priv/repo/migrations/20150908003815_create_player.exs`。 哦们需要改动两处地方，首先
是传递一个 `primary_key: false` 给 `table/2` 函数让其不生成主键。然后我们给 name 字段的 `add/3` 函数传递
`primary_key: true` 让其成为主键。

```elixir
defmodule HelloPhoenix.Repo.Migrations.CreatePlayer do
  use Ecto.Migration

  def change do
    create table(:players, primary_key: false) do
      add :name, :string, primary_key: true
      add :position, :string
      add :number, :integer

      timestamps
    end
  end
end
```

接下来打开 `web/models/player.ex`, 我们需要添加一行模块属性 `@primary_key {:name, :string, []}` 来描述我们的主
键是一个字符串。然后我们需要告诉 Phoenix 怎样在路由中获取我们的数据的标识。 我们还需要删除 `field :name,
:string`, 因为 :name 现在已经是主键了。

( 原文：
  Let's move on to `web/models/player.ex` next. We'll need to add a module attribute
  `@primary_key {:name, :string, []}` describing our primary key as a string. Then we'll need to tell Phoenix
  how to convert our data structure to an ID that is used in the routes: `@derive {Phoenix.Param, key: :name}`.
  We'll also need to remove the `field :name, :string` line because this is our new primary key. If this seems
  unusual, recall that the schema doesn't list the `id` field in models where `id` is the primary key.
)

```elixir
defmodule HelloPhoenix.Player do
  use HelloPhoenix.Web, :model

  @primary_key {:name, :string, []}
  @derive {Phoenix.Param, key: :name}
  schema "players" do
    field :position, :string
    field :number, :integer

    timestamps
  end
  . . .
```

最后，让我们在 `def render("player.json", %{player: player})` 函数体中删除 `id: player.id` 。

```elixir
defmodule HelloPhoenix.PlayerView do
  use HelloPhoenix.Web, :view

  . . .

  def render("player.json", %{player: player}) do
    %{name: player.name,
      position: player.position,
      number: player.number}
  end
end
```

完成之后，我们来运行迁移任务。

```console
$mix ecto.migrate
```

`players` 表看起来如下：

```sql
hello_phoenix_dev=# \d players
                Table "public.players"
   Column    |            Type             | Modifiers
-------------+-----------------------------+-----------
 name        | character varying(255)      | not null
 position    | character varying(255)      |
 number      | integer                     |
 inserted_at | timestamp without time zone | not null
 updated_at  | timestamp without time zone | not null
Indexes:
    "players_pkey" PRIMARY KEY, btree (name)
```

现在我们有了一个以 `name` 为主键的模型，可以用 `Repo.get!/2` 来查询，同时我们也可以在路由中直接使用 -- `localhost:4000/players/iguberman`。
