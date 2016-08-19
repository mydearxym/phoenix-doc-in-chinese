# Channels

作为开发者我们深切的了解测试的价值，她确保了我们能在坚实的基础上构建将来的代码。 Phoenix 也不例外， 为项目的不
同部分提供了简单一致的测试方法，包括 `Channels`。

在 Channels 指南中, 我们知道 "Channel" 是一个由不同部分构成的一层通信机制(a layered system with different
components)。所以在这种情况下，仅仅编写单元测试是不够的，我们需要将相关部分联合起来进行集成测试，这包括正确的
定义我们的 channel 路由，channal 模块，已经其回调函数；还包括更底层的`订阅发布`和 `传输`配置，使其按照我们的
期望正确工作 (lower-level layers such as the PubSub and Transport are configured correctly and
are working as intended)。


#### Channel 生成器

Phoenix 为 Channel 内建了一个 Mix 任务， 可以帮我们快速生成一个基本的 channel 和它的测试。
我们也来创建一个:

```console
$ mix phoenix.gen.channel Room
* creating web/channels/room_channel.ex
* creating test/channels/room_channel_test.exs

Add the channel to your `web/channels/user_socket.ex` handler, for example:

    channel "room:lobby", HelloPhoenix.RoomChannel
```

除了创建上述文件以外，还提示我们将 channel 路由信息添加到 `web/channels/user_socket.ex` 文件中，这很重要，否则
channel 功能是不会起作用的。

#### Channel Test Helpers 模块

在 `test/channels/room_channel_test.exs` 中，我们看到一行 `use MyApp.ChannelCase`。 注意 -- 我们在这篇指南中假
设我们的应用叫做 `MyApp`。

另外，当我们生成了一个新的 Phoenix 应用后，还会自动生成一个 `test/support/channel_case.ex`。这个文件中定义了
`MyApp.ChannelCase`, 我们之后会在所有的集成测试用到它。这个模块中自动了引入了一些方便的功能，比如 Ecto 模型和
请求等 （如果我们使用的是 Ecto 的话）。


其中一些 helpers 函数提供一些函数能在我们的 Channel 中触发回调。另一些是一些特殊能应用在我们 channels 上的
断言(assertions)。

如果我们想自定义一个 helper 函数在我们的 channel 测试时使用，我们可以将其定义在 `MyApp.ChannelCase`中，并确保
`MyApp.ChannelCase`在我们需要使用的地方被引入即可， 比如:

```elixir
defmodule MyApp.ChannelCase do
  ...

  using do
    quote do
      ...
      import MyApp.ChannelCase
    end
  end

  def a_channel_test_helper() do
    # code here
  end
end
```


#### Setup 部分

现在我们知道 Phoenix 为 channels 专门定义了一套测试用例以及大概内容，我们可以再仔细看看
`test/channel/room_channel_test.exs` 中剩下的内容。

首先， 是 setup 部分：

```elixir
setup do
  {:ok, _, socket} =
    socket("user_id", %{some: :assign})
    |> subscribe_and_join(RoomChannel, "room:lobby")

  {:ok, socket: socket}
end
```

`setup/2` 是 Elixir 自带的 `ExUnit` 内建的一个宏。 这里的 `do` 部分会在我们每次运行测试的时候运行。注意返回的
那行`{:ok, socket: socket}`。 这确保了从`subscribe_and_join/3` 返回的 `socket` 在下面所有的测试中可见。这样我
们就不用在下面每个测试用例里面都手动调用 `subscribe_and_join/3` 了。

`subscribe_and_join/3` 模拟了客户端加入 channel 和订阅指定主题 (topic)。之所以这样命名是因为客户端首先要加入一
个channel 才能收到这个 channel 上的事件。


#### 测试一个同步回应

第一个测试用例如下：

```elixir
test "ping replies with status ok", %{socket: socket} do
  ref = push socket, "ping", %{"hello" => "there"}
  assert_reply ref, :ok, %{"hello" => "there"}
end
```

这测试了 `MyApp.RoomChannel` 模块中的这段代码：

```elixir
# Channels can be used in a request/response fashion
# by sending replies to requests from the client
def handle_in("ping", payload, socket) do
  {:reply, {:ok, payload}, socket}
end
```

就像上面注释里提到的那样， `reply` 是同步的因为它模拟了 HTTP 请求的 requests/response 过程。这个同步的 reply
的最佳使用场景是我们处理完某个任务后返给客户端消息。比如，当我们在数据库中存储了某个东西后我们发送给客户端一个
回执。

在 `test "ping replies with status ok", %{socket: socket} do` 这行，我们看到我们已经映射了 `%{socket: socket}`。
这确保了我们能使用 setup 块中的 `socket` 。

我们使用 `push/3` 模拟了客户端给 channel 发送一条消息。在 `ref = push socket, "ping", %{"hello" => "there"}`
中我们使用 `%{"hello" => "there"}` 参数给 channel 发送一个 `ping` 事件。 这会触发
`web/channel/room_channel.ex` 文件中监听 `ping` 事件的 `handle_in/3` 回调函数。注意我们保留了返回值 `ref` 因为
我们要在下一行使用 `assert_reply ref, :ok, %{"hello" => "there"}` 来测试服务器的返回值，我们断言服务器返回了一
个同步应答 `:ok, %{"hello" => "there"}`。这样就确定了 `handle_in/3` 回调确实被 `ping` 触发了。

#### 测试广播

一个常见的需求是接收一个客户端的消息然后将其广播给所有订阅了该主题的客户端。这个常见需求在 Phoenix 也很容表达，
并且在生成的 `MyApp.RoomChannel` 文件的 `handle_in/3` 回调中可以直接使用：

```elixir
def handle_in("shout", payload, socket) do
  broadcast socket, "shout", payload
  {:noreply, socket}
end
```

它对应的测试如下：

```elixir
test "shout broadcasts to room:lobby", %{socket: socket} do
  push socket, "shout", %{"hello" => "all"}
  assert_broadcast "shout", %{"hello" => "all"}
end
```

我们使用了和之前相同的办法取到 `socket`, 同样使用 `push/3` 给 channel 一个 "shout" 事件，参数为 `%{"hello" =>
"all"}` 。

因为 `handle_in/3` 回调里面接收到 "shout" 后是广播的，所有订阅 `"room:lobby"` 的客户端都应该收到广播消息，测试
中我们使用 `assert_broadcast "shout", %{"hello" => "all"}` 来验证。

#### 测试服务器端的异步广播

我们 `MyApp.RoomChannelTest` 中的最后一个测试是验证服务器的广播被推送到了客户端。和我们上面的测试不同，我们现
在是间接测试我 channel 上的 `handle_out/3` 是否被触发了。这个 `handle_in/3` 函数定义在 `MyApp.RoomChannel`中:

```elixir
def handle_out(event, payload, socket) do
  push socket, event, payload
  {:noreply, socket}
end
```
因为 `handle_out/3` 只会在我们调用 `broadcast/3` 时被触发，我们需要在测试中使用 `broadcast_from` 或
`broadcast_from!` 来模拟这一点。 他俩唯一的区别是 `broadcast_from!` 会在广播失败的时候触发错误。

这行 `broadcast_from! socket, "broadcast", %{"some" => "data"}` 会触发我们的 `handle_out/3` 回调，并将得到的参
数原封不动的返回回去。对应的测试就是 `assert_push "broadcast", %{"some" => "data"}`.


#### 总结回顾

在这个章节中我们讨论并使用了 `MyApp.ConnCase` 中的特殊断言和一些 helper 函数，为我们测试 channel 提供了方便。
我们发现这里的 API 和 Phoenix Channels 部分很相似， 这种一致性为我们减轻了不少负担。

`MyApp.ChannelCase` 中还有很多的 helpers, 如果你对它们感兴趣可以查看 [`Phoenix.ChannelTest`](http://hexdocs.pm/phoenix/Phoenix.ChannelTest.html)。
