# 模型

在[Ecto 模型指南](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/H_ecto%E6%A8%A1%E5%9E%8B.html)
中我们生成了一个 user 的 HTML 资源。这为我们生成了一些模块，包括一个 user 模型和一个 user 模型的测试用例。 在
本章中，我们将以测试先行的方式来跑一遍 Ecto 模型指南里的那些例子。

对于还没有完整看过 Ecto 模型指南的朋友，一个简单的办法是，看看本章最后 "生成 HTML 资源" 部分。

先运行 `mix test` 确保我们的项目目前是无恙的。

```console
$ mix test
................

Finished in 0.6 seconds (0.5s on load, 0.1s on tests)
16 tests, 0 failures

Randomized with seed 638414
```

好的，我们现有的 16 个测试都是通过的。

## 测试驱动的 Changeset

我们会专注在 `test/models/user_test.exs` 文件，现在先来大致预览一下：

```elixir
defmodule HelloPhoenix.UserTest do
  use HelloPhoenix.ModelCase

  alias HelloPhoenix.User

  @valid_attrs %{bio: "some content", email: "some content", name: "some content", number_of_pets: 42}
  @invalid_attrs %{}

  test "changeset with valid attributes" do
    changeset = User.changeset(%User{}, @valid_attrs)
    assert changeset.valid?
  end

  test "changeset with invalid attributes" do
    changeset = User.changeset(%User{}, @invalid_attrs)
    refute changeset.valid?
  end
end
```

在第一行, 我们使用了 `use HelloPhoenix.ModelCase`, 它定义在 `test/support/model_case.ex`。
`HelloPhoenix.ModelCase` 负责将我们所有的模型引入并取别名。 `HelloPhoenix.ModelCase` 会同时同时在数据库中运行
所有的模块测试，除非我们在一独立的测试用例上使用了 `:async` 选项。

> 注意：我们不应该在与数据库模型测试的时候使用 `:async` 选项，这将会造成难于预测的结果并可能产生死锁。

我们还可以在 `HelloPhoenix.ModelCase` 中定义 helper 函数来帮助我们测试模型。 其中内建了一个 `errors_on` 函数，
我们很快会看到它是如何工作的。

我们将 `HelloPhoenix.User` 做了别名处理，所以可以直接使用 `%User()` 而不是 `%HelloPhoenix.User{}` 。

我们同时定义了模块属性 `@valid_attrs` 和 `@invalid_attrs` 以便于可以在所有测试函数中访问。

我们从生成的 `HelloPhoenix.UserTest` 中得到的测试属性是可用的，但是我们改动一下使其更真实。我们唯一要改动的是
`:email`, 因为它有个额外的 '@' 规则, 其他的字段随意即可 (The other changes are just cosmetic)。

```elixir
defmodule HelloPhoenix.UserTest do
  use HelloPhoenix.ModelCase

  alias HelloPhoenix.User

  @valid_attrs %{bio: "my life", email: "pat@example.com", name: "Pat Example", number_of_pets: 4}
  @invalid_attrs %{}

  ...
end
```

我们同时需要改变 `test/controllers/user_controller_test.exs` 中的模块属性 `@valid_attrs` 已使其开起来一致。

```elixir
defmodule HelloPhoenix.UserControllerTest do
  use HelloPhoenix.ConnCase

  alias HelloPhoenix.User
  @valid_attrs %{bio: "my life", email: "pat@example.com", name: "Pat Example", number_of_pets: 4}
  @invalid_attrs %{}

  ...
end
```

现在运行测试，十六个测试用例依然通过。

#### Number of Pets

Phoenix 为我们模型生成了所有的必须字段， 但用户的 `the number of pets` 是可选的 (While Phoenix generated our
model with all of the fields required, the number of pets a user has is optional in our domain.)。

让我们在测试中证实一下。

我们可以在 `@valid_attrs` 中删除 `:number_of_pets` 键值对，然后用这些新属性做一个新的 changeset , 看看这个
changeset 是否合法。

```elixir
defmodule HelloPhoenix.UserTest do
  ...

  test "number_of_pets is not required" do
    changeset = User.changeset(%User{}, Map.delete(@valid_attrs, :number_of_pets))
    assert changeset.valid?
  end
end
```

现在，我们运行下测试：

```console
$ mix test
.............

  1) test number_of_pets is not required (HelloPhoenix.UserTest)
     test/models/user_test.exs:19
     Expected truthy, got false
     code: changeset.valid?()
     stacktrace:
       test/models/user_test.exs:21

...

Finished in 0.4 seconds (0.2s on load, 0.1s on tests)
17 tests, 1 failure

Randomized with seed 780208
```

失败了 -- 为了修正它，我们需要在 `web/models/user.ex` 文件中的 `validate_required/3` 函数中去除
`:number_of_pets`属性。

```elixir
defmodule HelloPhoenix.User do
  ...

  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:name, :email, :bio, :number_of_pets])
    |> validate_required([:name, :email, :bio])
  end
end
```

再运行一次：

```console
$ mix test
.................

Finished in 0.3 seconds (0.2s on load, 0.09s on tests)
17 tests, 0 failures

Randomized with seed 963040
```

#### Bio 属性

在 Ecto 模型指南中，我们给 `:bio` 字段设置了两个限制。第一是字段长度不能少于连个字符，让我们用刚学到的方法给它写个测试。

首先，我们改变 `:bio` 属性让其只包含一个字符。让后我们创建一个 changeset 看其是否合法。

```elixir
defmodule HelloPhoenix.UserTest do
  ...

  test "bio must be at least two characters long" do
    attrs = %{@valid_attrs | bio: "I"}
    changeset = User.changeset(%User{}, attrs)
    refute changeset.valid?
  end
end
```

运行测试，在意料中的失败了：

```console
$ mix test
.....

  1) test bio must be at least two characters long (HelloPhoenix.UserTest)
     test/models/user_test.exs:24
     Expected false or nil, got true
     code: changeset.valid?()
     stacktrace:
       test/models/user_test.exs:27

............

Finished in 0.3 seconds (0.2s on load, 0.09s on tests)
18 tests, 1 failure

Randomized with seed 327779
```

测试是失败了，但是错误信息却不是很明确，我们在验证 `:bio` 属性的长度，错误信息却是 "Expected false or nil,
got true"，并没有提到 `:bio` 字段。

我们改进一下错误信息：

我们不改动设置 `:bio` 值的那行，但 assert 那行，我们用从 `ModelCase` 中得到的 `errors_on/2` 函数来生成一个错误
列表，然后检查 `:bio` 属性的错误是否在其中。

```elixir
defmodule HelloPhoenix.UserTest do
  ...

  test "bio must be at least two characters long" do
    attrs = %{@valid_attrs | bio: "I"}
    assert {:bio, "should be at least 2 character(s)"} in errors_on(%User{}, attrs)
  end
end
```

> 注意：`ModelCase.errors_on/2` 返回一个关键字列表(keyword list)，并且其中每一个元素是一个元组 (tuple)。

现在运行测试，我们得到了一个不同的错误信息。

```console
$ mix test
...............

  1) test bio must be at least two characters long (HelloPhoenix.UserTest)
     test/models/user_test.exs:24
     Assertion with in failed
     code: {:bio, "should be at least 2 character(s)"} in errors_on(%User{}, attrs)
     lhs:  {:bio,
            "should be at least 2 character(s)"}
     rhs:  []

..

Finished in 0.4 seconds (0.2s on load, 0.1s on tests)
18 tests, 1 failure

Randomized with seed 435902
```

显示这个错误在模型的 changeset 中。

```console
code: {:bio, "should be at least 2 character(s)"} in errors_on(%User{}, attrs)
```

我们看到左边的值是错误：

```console
lhs:  {:bio, "should be at least 2 character(s)"}
```

但是右边的值却是空的：

```console
rhs:  []
```

为空的原因是我们还没有验证 `:bio` 属性的最短长度。

我们的测试已经提示了，现在让我们加上那个限制规则。

```elixir
defmodule HelloPhoenix.User do
  ...

  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:name, :email, :bio, :number_of_pets])
    |> validate_required([:name, :email, :bio])
    |> validate_length(:bio, min: 2)
  end
end
```

现在运行测试就通过了。

```console
$ mix test
..................

Finished in 0.3 seconds (0.2s on load, 0.09s on tests)
18 tests, 0 failures

Randomized with seed 305958
```

让我们再使用 `errors_on/2` 给 `:bio` 字段添加一个 140 字的限制。

在我们谢测试之前，我们怎样优雅的制造一个长字符串呢？ 在 `HelloPhoenix.ModelCase` 中创建一个函数去做这件事是个
好办法，我们就叫他`long_string/1` 吧，它在字符串后面添加指定长度的 'a'。

```elixir
defmodule HelloPhoenix.ModelCase do
  ...

  def long_string(length) do
    Enum.reduce (1..length), "", fn _, acc ->  acc <> "a" end
  end
end
```

然后我们使用 `long_string/1` 来给 `:bio` 设置一个新值。

```elixir
defmodule HelloPhoenix.UserTest do
  ...

  test "bio must be at most 140 characters long" do
    attrs = %{@valid_attrs | bio: long_string(141)}
    assert {:bio, "should be at most 140 character(s)"} in errors_on(%User{}, attrs)
  end
end
```

运行测试，失败了。

```console
$ mix test
....

  1) test bio must be at most 140 characters long (HelloPhoenix.UserTest)
     test/models/user_test.exs:29
     Assertion with in failed
     code: {:bio, {:bio, "should be at most 140 character(s)"} in errors_on(%User{}, attrs)
     lhs:  {:bio,
            "should be at most 120 character(s)"}

..............

Finished in 0.3 seconds (0.2s on load, 0.1s on tests)
19 tests, 1 failure

Randomized with seed 593838
```

为了让测试通过，我们给 `:bio` 再添加最长长度限制。

```elixir
defmodule HelloPhoenix.User do
  ...

  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:name, :email, :bio, :number_of_pets])
    |> validate_required([:name, :email, :bio])
    |> validate_length(:bio, min: 2)
    |> validate_length(:bio, max: 140)
  end
end
```

现在测试没问题了。

```console
$ mix test
...................

Finished in 0.4 seconds (0.3s on load, 0.1s on tests)
19 tests, 0 failures

Randomized with seed 468975
```

#### Email 属性

我们剩下最后一个字段了，目前 `:email` 只是和其他字符串一样，我们需要它至少匹配一个 "@", 现在并没有一个可靠的办
法来测试 email 地址的真实性，但我们可以在发送 email 之前过滤掉一些非法地址。

接下来的事情就很熟悉了， 我们改变 `:email` 的值，看 `errors_on` 生成的错误中是否有 email 相关的部分。

```elixir
defmodule HelloPhoenix.UserTest do
  ...

  test "email must contain at least an @" do
    attrs = %{@valid_attrs | email: "fooexample.com"}
    assert {:email, "has invalid format"} in errors_on(%User{}, attrs)
  end
end
```

运行测试，发现有点 `errors_on/2` 返回的是空。

```console
$ mix test
................

  1) test email must contain at least an @ (HelloPhoenix.UserTest)
     test/models/user_test.exs:34
     Assertion with in failed
     code: {:email, "has invalid format"} in errors_on(%User{}, attrs)
     lhs:  {:email, "has invalid format"}
     rhs:  []
     stacktrace:
       test/models/user_test.exs:36

...

Finished in 0.4 seconds (0.2s on load, 0.1s on tests)
20 tests, 1 failure

Randomized with seed 962127
```

我们给 email 字段一个简单的校验。

```elixir
defmodule HelloPhoenix.User do
  ...

  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:name, :email, :bio, :number_of_pets])
    |> validate_required([:name, :email, :bio])
    |> validate_length(:bio, min: 2)
    |> validate_length(:bio, max: 140)
    |> validate_format(:email, ~r/@/)
  end
end
```

现在测试重新通过了。

```console
$ mix test
....................

Finished in 0.3 seconds (0.2s on load, 0.09s on tests)
20 tests, 0 failures

Randomized with seed 330955
```

### [附] 生成 HTML 资源

我们假设你已经安装了 PostgreSQL 数据库，并使用正确的配置生成了一个默认的应用

现在，我们使用下列参数运行 `phoenix.gen.html`。

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

Remember to update your repository by running migrations:

    $ mix ecto.migrate
```

然后我们根据提示将 `resources "/users", UserController` 添加到路由文件 `web/router.ex` 中去。

```elixir
defmodule HelloPhoenix.Router do
  ...

  scope "/", HelloPhoenix do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
    resources "/users", UserController
  end

  # Other scopes may use custom stacks.
  # scope "/api", HelloPhoenix do
  #   pipe_through :api
  # end
end
```

然后，我们可以使用 `ecto.create` 创建数据库了。

```console
$ mix ecto.create
The database for HelloPhoenix.Repo has been created.
```

然后我们使用 `ecto.migrate` 运行迁移任务创建 `users` 表。

```console
$ mix ecto.migrate

[info]  == Running HelloPhoenix.Repo.Migrations.CreateUser.change/0 forward

[info]  create table users

[info]  == Migrated in 0.0s
```

现在，我们可以开始探索测试了。
