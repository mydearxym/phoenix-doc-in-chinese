### 通道

频道 (Channels) 是 Phoenix 中一个激动人心并且十分强大的功能，它允许我们轻松的给应用添加软实时（soft-realtime）功能。Channels 的理念十分简单 -- 即发送和接收消息。发送者 (Senders) 按照主题（topics）广播消息，接收者（Receivers）通过订阅主题来接收这些消息。`发送者` 和 `接收者` 可以随时互换角色。

对于 Elixir 这样一个基于消息传递的语言，你也许会好奇为什么它还需要一个额外的收发消息的机制。使用 Channels, 发送者和接收者都可以不是 Elixir 的进程 (原文: Elixir processess)。 它可以是其他任何东西 --- 一个 JavaScript 客户端，一个 iOS 应用，其他的Phoenix 应用，甚至我们的手表等，另外，消息广播可以有多个接收者，而 Elixir processes 只能一对一的通信。


"Channel" 这个词实际上是由很多组件组成的一个层的一个简写。我们先来看看它有那些部分组成，以便我们接下来更好的了解它。

#### The Moving Parts

- Socket Handlers

Phoenix 会维护一个单一的连接到服务器，然后你的 `channel sockets` 会多路复用这个连接。Socket handlers, 比如 `web/channels/user_socket.ex`, 标识定位一个具体的 socket 的模块，并可以让你设置默认的消息。(原文：allow you to set default socket assigns for use in all channels.)


- 通道路由

通道路由定义在 Socket handlers 中，比如 `web/channels/user_socket.ex`, 它和其他路由是独立开的。它匹配一个主题（topic string）并且将请求分发到对应的 Channel 模块。 `*` 表示通配符，在接下来的例子中， 对 `sample_topic:pizza` 和 `sample_topic:oranges` 的请求都会被分发到 `SampleTopicChannel` 中去。

```elixir
channel "sample_topic:*", HelloPhoenix.SampleTopicChannel
```

- 频道（Channels）

Channels 处理客户端的事件，所以和控制器有点类似，但它们也有两个主要的区别。 Channel 事件是双工的 ， 并且它的生命周期并不随单一的 请求/发送 周期的结束而结束。Channels 是 Phoenix 应用中实时通信组件的最高层级的抽象(highest level abstraction)。

每个 Channel 都会实现这一个或多个回调函数 --- `join/3`, `terminate/2`, `handle_in/3`, and `handle_out/3`。

- 发布订阅（PubSub）

Phoenix 的发布订阅层由 `Phoenix.PubSub` 模块以及 `GenServer`s 的各种适配器模块们组成 (a variety of modules for different adapters and their `GenServer`s)。实现 Channel 通信的各种功能 -- 订阅/取消订阅 主题， 对某一主题广播消息等。

如果需要，我们也可以定义我们自己的 PubSub 适配器。详情可以在 [Phoenix.PubSub docs](http://hexdocs.pm/phoenix_pubsub/Phoenix.PubSub.html) 查看。


需要注意的是，这些模块是在 Phoenix 内部使用的。Channels 在幕后使用他们来实现功能，作为终端用户（end users），我们并不需要直接在我们的应用中使用他们。


- 消息（Messages）

`Phoenix.Socket.Message` 模块定义了一个含有如下键值的结构体来表示一个合法的消息。详细信息请看 [Phoenix.Socket.Message docs](http://hexdocs.pm/phoenix/Phoenix.Socket.Message.html)。

  - `topic` - 主题字符串或者主题:子主题 (The string topic or topic:subtopic pair namespace ), 比如 “messages”, “messages:123”
  - `event` - 事件名字, 比如 “phx_join”
  - `payload` - The message payload
  - `ref` - The unique string ref

- 主题 (Topics)

主题是字符串的标识符 (string identifiers ) - 确保消息去到正确的地方。就像我们之前看到的，主题字符串可以使用通配符，这使得它可以支持 "topic:subtopic" 这样的命名惯例。很多时候，你可能会在模型层（model layer）用记录IDs （record IDs）组合成主题，比如 `"users:123"`。


- 传输 (Transports)

传输层 (The transport layer ) 是正真干活的地方(the rubber meets the road), `Phoenix.Channel.Transport` 模块分发进出通道 (Channel) 的所有消息。

- 传输适配器 (Transport Adapters)

默认的传输机制是通过 WebSockets (如果客户端不支持会自动降级到 LongPolling) 。 使用其他适配器也是可以的，并且如果我们也可以在遵循适配器协议（adapter contract）自己编写一个，`Phoenix.Transports.WebSocket`有相应的例子。


- 客户端的库 (Client Libraries)

目前 Phoenix 自带 JavaScript 客户端，Phoenix 1.0 以后也发布了 [iOS](https://github.com/davidstump/SwiftPhoenixClient), [Android](https://github.com/eoinsha/JavaPhoenixChannels), 和 [C#](https://github.com/livehelpnow/CSharpPhoenixClient) 客户端。

## 综合实列

让我们创建一个简单的聊天应用来实践一下。在 [生成一个新的 Phoenix 应用](http://www.phoenixframework.org/docs/up-and-running) 后, 我们会看到 endpoint 已经在 `lib/hello_phoenix/endpoint.ex` 文件中设置好了。

```elixir
defmodule HelloPhoenix.Endpoint do
  use Phoenix.Endpoint, otp_app: :hello_phoenix

  socket "/socket", HelloPhoenix.UserSocket
  ...
end
```

在文件 `web/channels/user_socket.ex` 中，我们上面指向的 `HelloPhoenix.UserSocket` 也已经自动生成，我需要确保消息被路由到正确的 channel，让我们注释掉 "rooms:*" 的定义。


```elixir
defmodule HelloPhoenix.UserSocket do
  use Phoenix.Socket

  ## Channels
  channel "rooms:*", HelloPhoenix.RoomChannel
  ...
```

现在，无论何时一个客户发送一条以 `"rooms:"` 开头的消息，它都会被路由到我们的 RoomChannel, 下一步，我们需要定义一个 `HelloPhoenix.RooChannel` 模块来管理我们的聊天室消息。


### 加入频道 (Joining Channels)

channels 面临的首要问题就是怎样授权客户加入一个指定的主题，对于授权验证，我们必须实现 `join/3` 回调函数（在`web/channels/room?channel.ex` 文件中）。


```elixir
defmodule HelloPhoenix.RoomChannel do
  use Phoenix.Channel

  def join("rooms:lobby", _message, socket) do
    {:ok, socket}
  end
  def join("rooms:" <> _private_room_id, _params, _socket) do
    {:error, %{reason: "unauthorized"}}
  end
end
```

对我们的聊天应用来说，我们允许任何人加入到 `"rooms:lobby"` 主题，但其他房间被认为是私有的并需要其他手段验证，比如说使用数据库等等。目前我们暂且不考虑这些，但你可以在完成这个简单例子之后自己去摸索。 授权一个 socket 加入一个主题 (topic), 我们返回 `{:ok, socket}` 或者 `{:ok, reply, socket}`。如果是拒绝加入，我们则返回 `{:eror, reply}`。更多关于授权的细节可以查看 [`Phoenix.Token` documentation](http://hexdocs.pm/phoenix/Phoenix.Token.html)。

现在，让我们在 `web/static/js/socket.js` 中改动一些代码使我们的客户端加入 "rooms:lobby"。


```javascript
...
socket.connect()

// Now that you are connected, you can join channels with a topic:
let channel = socket.channel("rooms:lobby", {})
channel.join()
  .receive("ok", resp => { console.log("Joined successfully", resp) })
  .receive("error", resp => { console.log("Unable to join", resp) })

export default socket
```
然后，我们要确保 `web/static/js/socket.js` 被引入我们应用的 javascript 文件。只需将 `web/static/js/app.js` 最后一行反注释掉即可。

```javascript
...
import socket from "./socket"
```

保存文件以后你会看到浏览器自动刷新了 --- 得益于 Phoenix live reloader。如果一切顺利，我们会看到浏览器的 JavaScript 终端里看到 "Joined successfully" 字样。我们的客户端和服务器现在可以通过一个持续存在的通道聊天了，现在我们再在前端部分添加一些代码。

在 `web/templates/page/index.html.eex` 文件中, 我们将已存在的代码替换为如下：

```html
<div id="messages"></div>
<input id="chat-input" type="text"></input>
```

我们还需要在 `web/templates/layout/app.html.eex` 布局文件中引入 jQuery。

```html
  ...
    <%= render @view_module, @view_template, assigns %>

  </div> <!-- /container -->
  <script src="//code.jquery.com/jquery-1.11.3.min.js"></script>
  <script src="<%= static_path(@conn, "/js/app.js") %>"></script>
</body>
```

然后在 `web/static/js/socket.js` 中添加一些事件监听 (event listeners)。

```javascript
...
let channel           = socket.channel("rooms:lobby", {})
let chatInput         = $("#chat-input")
let messagesContainer = $("#messages")

chatInput.on("keypress", event => {
  if(event.keyCode === 13){
    channel.push("new_msg", {body: chatInput.val()})
    chatInput.val("")
  }
})

channel.join()
  .receive("ok", resp => { console.log("Joined successfully", resp) })
  .receive("error", resp => { console.log("Unable to join", resp) })

export default socket
```

我们监听 "new_msg" 然后将其添加到页面的消息容器中。

```javascript
...
let channel           = socket.channel("rooms:lobby", {})
let chatInput         = $("#chat-input")
let messagesContainer = $("#messages")

chatInput.on("keypress", event => {
  if(event.keyCode === 13){
    channel.push("new_msg", {body: chatInput.val()})
    chatInput.val("")
  }
})

channel.on("new_msg", payload => {
  messagesContainer.append(`<br/>[${Date()}] ${payload.body}`)
})

channel.join()
  .receive("ok", resp => { console.log("Joined successfully", resp) })
  .receive("error", resp => { console.log("Unable to join", resp) })

export default socket
```

接下来，我们来处理的 server 端的事件来完善这个应用。


### Incoming Events

我们使用 `handle_in/3` 来处理到来的 events, 我们可以在 event 上使用模式匹配，比如 "new_msg", 然后将客户端传递来的 payload 提取出来。在我们的应用中，我们只是简单的将这条消息发送给所有订阅了 `rooms:lobby` 的用户, 使用 `broadcast!/3`。

```elixir
defmodule HelloPhoenix.RoomChannel do
  use Phoenix.Channel

  def join("rooms:lobby", _message, socket) do
    {:ok, socket}
  end
  def join("rooms:" <> _private_room_id, _params, _socket) do
    {:error, %{reason: "unauthorized"}}
  end

  def handle_in("new_msg", %{"body" => body}, socket) do
    broadcast! socket, "new_msg", %{body: body}
    {:noreply, socket}
  end

  def handle_out("new_msg", payload, socket) do
    push socket, "new_msg", payload
    {:noreply, socket}
  end
end
```

`broadcast!/3` 函数会根据 `socket` 上的主题通知所有的订阅客户端, 然后触发他们的 `handle_out/3` 回调，`handle_out/3` 并不是必须的，但它允许我们在消息到达每个客户端之前对 broadcasts 的内容进行定制和过滤。默认情况下，`handle_out/3` 只是简单的将消息发送到我们的客户端，就像我们要求的那样，我们在这里引入它是为了展示其强大的消息定制和过滤功能，我们来看一个实际的例子。


#### 拦截发送事件 (Intercepting Outgoing Events)

我们不打算在这里具体实现，但是你可以想象一下，如果我们的聊天应用允许用户忽略新加入用户的消息。我们就可以使用 `handle_out/3` 回调拦截发往客户端的消息来实现这个功能 （这里我们假设我们的 User 模型定义了 `ignoring?/2` 函数，并且我们可以使用 `assigns` 传递一个用户进来。 ）。

```elixir
intercept ["user_joined"]

def handle_out("user_joined", msg, socket) do
  if User.ignoring?(socket.assigns[:user], msg.user_id) do
    {:noreply, socket}
  else
    push socket, "user_joined", msg
  end
end
```

差不多了，现在当你打开多个浏览器标签你就可以看到消息被推送到所有的客户端了。

#### Socket Assigns

与连接结构体（connection structs）`%Plug.Con{}` 类似, 我们也可以在一个 channel socket 上覆盖赋值 (assign values)。 `Phoenix.Socket.assign/3` 被方便的集成到了 channel 模块，简写作 `assign/3`。

```elixir
socket = assign(socket, :user, msg["user"])
```

Sockets 将这些值以 map 的形式存储在 `socket.assigns`。


#### 实列应用

这个实列的源码在 (https://github.com/chrismccord/phoenix_chat_example).

你也可以访问一个线上demo (http://phoenixchat.herokuapp.com/).

