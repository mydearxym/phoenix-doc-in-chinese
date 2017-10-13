### 通道

频道 (Channels) 是 Phoenix 中一个激动人心并且十分强大的功能模块，它允许我们轻松的给应用添加软实时（soft-realtime）功能。Channels 的理念十分简单 -- 即发送和接收消息。发送者 (Senders) 按照主题（topics）广播消息，接收者（Receivers）通过订阅主题来接收这些消息。`发送者` 和 `接收者` 可以随时互换角色。

对于 Elixir 这样一个基于消息传递的语言，你也许会好奇为什么它还需要一个额外的收发消息的机制。使用 Channels, 发送者和接收者都可以不是 Elixir 的进程 (原文: Elixir processess)。 它可以是其他任何东西 --- 一个 JavaScript 客户端，一个 iOS 应用，其他的Phoenix 应用，甚至我们的手表等等，另外，消息广播可以有多个接收者，而 Elixir processes 只能一对一的通信。


"Channel" 这个词实际上是由很多组件组成的一个层的一个简写。我们先来看看它有那些部分组成，以便我们接下来更好的了解它。


## JS 文档

新生成的 Phoenix 应用自带一个 javascript 的客户端，文档在[https://hexdocs.pm/phoenix/js/](https://hexdocs.pm/phoenix/js/)

## The Moving Parts

### Socket Handlers

Phoenix 会维护一个单一的连接到服务器，然后你的 `channel sockets` 会多路复用这个连接。Socket 处理单元, 比如 `lib/hello_web/channels/user_socket.ex`, 标识定位一个具体的 socket 的模块，并可以让你为所有的 channels 设置默认的参数。

### 通道路由

通道路由定义在 Socket handlers 中，比如 `lib/hello_web/channels/user_socket.ex`, 它和其他路由是独立开的。它匹配一个主题（topic string）并且将请求分发到对应的 Channel 模块。 `*` 表示通配符，在接下来的例子中， 对 `sample_topic:pizza` 和 `sample_topic:oranges` 的请求都会被分发到 `SampleTopicChannel` 中去。

```elixir
channel "sample_topic:*", HelloWeb.SampleTopicChannel
```

### 频道（Channels）

Channels 处理客户端的事件，所以和控制器有点类似，但它们也有两个主要的区别。 Channel 事件是双工的 ， 并且它的生命周期并不随单一的 请求/发送 周期的结束而结束。Channels 是 Phoenix 应用中实时通信组件的最高层级的抽象。

每个 Channel 都会实现这一个或多个回调函数 --- `join/3`, `terminate/2`, `handle_in/3`, and `handle_out/3`。

### 发布订阅（PubSub）

Phoenix 的发布订阅层由 `Phoenix.PubSub` 模块以及 `GenServer`s 的各种适配器模块们组成 (a variety of modules for different adapters and their `GenServer`s)。实现 Channel 通信的各种功能 -- 订阅/取消订阅 主题， 对某一主题广播消息等。

如果需要，我们也可以定义我们自己的 PubSub 适配器。详情可以在 [Phoenix.PubSub docs](http://hexdocs.pm/phoenix_pubsub/Phoenix.PubSub.html) 查看。

需要注意的是，这些模块是在 Phoenix 内部使用的。Channels 在幕后使用他们来实现功能，作为普通用户，我们并不需要直接在我们的应用中使用他们。

### 消息（Messages）

`Phoenix.Socket.Message` 模块定义了一个含有如下键值的结构体来表示一个合法的消息。详细信息请看 [Phoenix.Socket.Message docs](http://hexdocs.pm/phoenix/Phoenix.Socket.Message.html)。

  - `topic` - 主题字符串或者主题:子主题 (The string topic or topic:subtopic pair namespace ), 比如 “messages”, “messages:123”
  - `event` - 事件名字, 比如 “phx_join”
  - `payload` - The message payload
  - `ref` - The unique string ref

### 主题 (Topics)

主题是字符串的标识符 (string identifiers ) - 确保消息去到正确的地方。就像我们之前看到的，主题字符串可以使用通配符，这使得它可以支持 "topic:subtopic" 这样的命名惯例。很多时候，你可能会在模型层用记录IDs组合成主题，比如 `"users:123"`。

### 传输 (Transports)

传输层 (The transport layer ) 是正真干活的地方, `Phoenix.Channel.Transport` 模块分发进出通道 (Channel) 的所有消息。

### 传输适配器 (Transport Adapters)

默认的传输机制是通过 WebSockets (如果客户端不支持会自动降级到 LongPolling) 。 使用其他适配器也是可以的，并且如果我们也可以在遵循适配器协议（adapter contract）自己编写一个，`Phoenix.Transports.WebSocket`有相应的例子。

### 客户端的库 (Client Libraries)

#### 官方

+ JavaScript
  - [phoenix.js](https://github.com/phoenixframework/phoenix/blob/v1.3/assets/js/phoenix.js)

#### 第三方

+ Swift (iOS)
  - [SwiftPhoenix](https://github.com/davidstump/SwiftPhoenixClient)
+ Java (Android)
  - [JavaPhoenixChannels](https://github.com/eoinsha/JavaPhoenixChannels)
+ C#
  - [PhoenixSharp](https://github.com/Mazyod/PhoenixSharp)
  - [dn-phoenix](https://github.com/jfis/dn-phoenix)

## 综合实列

让我们创建一个简单的聊天应用来实践一下。
在[生成一个新的 Phoenix 应用](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/A_%E8%B5%B7%E6%AD%A5.html) 后, 我们会看到 endpoint 已经在 `lib/hello_web/endpoint.ex` 文件中设置好了。

```elixir
defmodule HelloWeb.Endpoint do
  use Phoenix.Endpoint, otp_app: :hello

  socket "/socket", HelloWeb.UserSocket
  ...
end
```

在文件 `lib/hello_web/channels/user_socket.ex` 中，我们上面指向的 `HelloWeb.UserSocket` 也已经自动生成，我需要确保消息被路由到正确的 channel，让我们注释掉 "rooms:*" 的定义。

```elixir
defmodule HelloWeb.UserSocket do
  use Phoenix.Socket

  ## Channels
  channel "room:*", HelloWeb.RoomChannel
  ...
```

现在，无论何时一个客户发送一条以 `"rooms:"` 开头的消息，它都会被路由到我们的 RoomChannel, 下一步，我们需要定义一个 `HelloWeb.RooChannel` 模块来管理我们的聊天室消息。


### 加入频道 (Joining Channels)

现在我们面临的首要问题就是怎样授权客户加入一个指定的主题，对于授权验证，我们必须在`lib/hello_web/channels/room_channel.ex` 实现 `join/3` 回调函数。

```elixir
defmodule HelloWeb.RoomChannel do
  use Phoenix.Channel

  def join("room:lobby", _message, socket) do
    {:ok, socket}
  end
  def join("room:" <> _private_room_id, _params, _socket) do
    {:error, %{reason: "unauthorized"}}
  end
end
```

对我们的聊天应用来说，我们允许任何人加入到 `"rooms:lobby"` 主题，但其他房间被认为是私有的并需要其他手段验证，比如说使用数据库等等。目前我们暂且不考虑这些，但你可以在完成这个简单例子之后自己去摸索。 授权一个 socket 加入一个主题 (topic), 我们返回 `{:ok, socket}` 或者 `{:ok, reply, socket}`。如果是拒绝加入，我们则返回 `{:error, reply}`。更多关于授权的细节可以查看 [`Phoenix.Token` documentation](http://hexdocs.pm/phoenix/Phoenix.Token.html)。

现在，让我们在 `assets/js/socket.js` 中改动一些代码使我们的客户端加入 "rooms:lobby"。


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

然后，我们要确保 `assets/js/socket.js` 被引入我们应用的 javascript 文件。只需将 `assets/js/app.js` 最后一行反注释掉即可。

```javascript
...
import socket from "./socket"
```

保存文件以后你会看到浏览器自动刷新了 --- 得益于 Phoenix live reloader。如果一切顺利，我们会看到浏览器的 JavaScript 终端里看到 "Joined successfully" 字样。我们的客户端和服务器现在可以通过一个持续存在的通道聊天了，现在我们再在前端部分添加一些代码。

在 `lib/hello_web/templates/page/index.html.eex` 文件中, 我们将已存在的代码替换为如下：

```html
<div id="messages"></div>
<input id="chat-input" type="text"></input>
```

然后在 `assets/js/socket.js` 中添加一些事件监听 (event listeners)。

```javascript
...
let channel           = socket.channel("rooms:lobby", {})
let chatInput         = document.querySelector("#chat-input")
let messagesContainer = document.querySelector("#messages")

chatInput.addEventListener("keypress", event => {
  if(event.keyCode === 13){
    channel.push("new_msg", {body: chatInput.value})
    chatInput.value = ""
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
let chatInput         = document.querySelector("#chat-input")
let messagesContainer = document.querySelector("#messages")

chatInput.addEventListener("keypress", event => {
  if(event.keyCode === 13){
    channel.push("new_msg", {body: chatInput.value})
    chatInput.value = ""
  }
})

channel.on("new_msg", payload => {
  let messageItem = document.createElement("li");
  messageItem.innerText = `[${Date()}] ${payload.body}`
  messagesContainer.appendChild(messageItem)
})

channel.join()
  .receive("ok", resp => { console.log("Joined successfully", resp) })
  .receive("error", resp => { console.log("Unable to join", resp) })

export default socket
```

接下来，我们来处理的 server 端的事件来完善这个应用。


### Incoming Events

我们使用 `handle_in/3` 来处理到来的 events, 我们可以在 event 上使用模式匹配，比如 "new_msg", 然后将客户端传递来的 payload 提取出来。在我们的应用中，我们只是使用 `broadcast!/3` 简单的将这条消息发送给所有订阅了 `rooms:lobby` 的用户 。

```elixir
defmodule HelloWeb.RoomChannel do
  use Phoenix.Channel

  def join("room:lobby", _message, socket) do
    {:ok, socket}
  end
  def join("room:" <> _private_room_id, _params, _socket) do
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

与连接结构体（connection structs）`%Plug.Con{}` 类似, 我们也可以在一个 channel socket 上覆盖赋值 。 `Phoenix.Socket.assign/3` 被方便的集成到了 channel 模块，简写作 `assign/3`。

```elixir
socket = assign(socket, :user, msg["user"])
```

Sockets 将这些值以 map 的形式存储在 `socket.assigns`。

#### 使用 Token 鉴权

当我们连接服务器的时候通常需要对客户端进行鉴权， 这可以简单的通过 [Phoenix.Token](https://hexdocs.pm/phoenix/Phoenix.Token.html) 分四步完成。

**Step 1 - Assign a Token in the Connection**

Let's say we have an authentication plug in our app called `OurAuth`. When `OurAuth` authenticates a user, it sets a value for the `:current_user` key in `conn.assigns`. Since the `current_user` exists, we can simply assign the user's token in the connection for use in the layout. We can wrap that behavior up in a private function plug, `put_user_token/2`. This could also be put in its own module as well. To make this all work, we just add `OurAuth` and `put_user_token/2` to the browser pipeline.

```elixir
pipeline :browser do
  # ...
  plug OurAuth
  plug :put_user_token
end

defp put_user_token(conn, _) do
  if current_user = conn.assigns[:current_user] do
    token = Phoenix.Token.sign(conn, "user socket", current_user.id)
    assign(conn, :user_token, token)
  else
    conn
  end
end
```

Now our `conn.assigns` contains the `current_user` and `user_token`.

**Step 2 - Pass the Token to the JavaScript**

Next we need to pass this token to JavaScript. We can do so inside a script tag in `web/templates/layout/app.html.eex`, as follows:

```html
<script>window.userToken = "<%= assigns[:user_token] %>";</script>
```

**Step 3 - Pass the Token to the Socket Constructor and Verify**

We also need to pass the `:params` to the socket constructor and verify the user token in the `connect/2` function. To do so, edit `web/channels/user_socket.ex`, as follows:

```elixir
def connect(%{"token" => token}, socket) do
  # max_age: 1209600 is equivalent to two weeks in seconds
  case Phoenix.Token.verify(socket, "user socket", token, max_age: 1209600) do
    {:ok, user_id} ->
      {:ok, assign(socket, :current_user, user_id)}
    {:error, reason} ->
      :error
  end
end
```

In our JavaScript, we can use the token set previously when to pass the token when constructing the Socket:

```javascript
let socket = new Socket("/socket", {params: {token: window.userToken}})
```

We used `Phoenix.Token.verify/4` to verify the user token provided by the client. `Phoenix.Token.verify/4` returns either `{:ok, user_id}` or `{:error, reason}`. We can pattern match on that return in a `case` statement. With a verified token, we set the user's id as the value to `:current_user` in the socket. Otherwise, we return `:error`.

**Step 4 - Connect to the socket in JavaScript**

With authentication set up, we can connect to sockets and channels from JavaScript.

```javascript
let socket = new Socket("/socket", {params: {token: window.userToken}})
socket.connect()
```

Now that we are connected, we can join channels with a topic:

```elixir
let channel = socket.channel("topic:subtopic", {})
channel.join()
  .receive("ok", resp => { console.log("Joined successfully", resp) })
  .receive("error", resp => { console.log("Unable to join", resp) })

export default socket
```

Note that token authentication is preferable since it's transport agnostic and well-suited for long running-connections like channels, as opposed to using sessions or authentication approaches.

#### Fault Tolerance and Reliability Guarantees

Servers restart, networks split, and clients lose connectivity. In order to design robust systems, we need to understand how Phoenix responds to these events and what guarantees it offers.

### Handling Reconnection

Clients subscribe to topics, and Phoenix stores those subscriptions in an in-memory ETS table. If a channel crashes, the clients will need to reconnect to the topics they had previously subscribed to. Fortunately, the Phoenix JavaScript client knows how to do this. The server will notify all the clients of the crash. This will trigger each client's `Channel.onError` callback. The clients will attempt to reconnect to the server using an exponential back off strategy. Once they reconnect, they'll attempt to rejoin the topics they had previously subscribed to. If they are successful, they'll start receiving messages from those topics as before.

### Resending Client Messages

Channel clients queue outgoing messages into a `PushBuffer`, and send them to the server when there is a connection. If no connection is available, the client holds on to the messages until it can establish a new connection. With no connection, the client will hold the messages in memory until it establishes a connection, or until it receives a `timeout` event. The default timeout is set to 5000 milliseconds. The client won't persist the messages in the browser's local storage, so if the browser tab closes, the messages will be gone.

### Resending Server Messages

Phoenix uses an at-most-once strategy when sending messages to clients. If the client is offline and misses the message, Phoenix won't resend it. Phoenix doesn't persist messages on the server. If the server restarts, unsent messages will be gone. If our application needs stronger guarantees around message delivery, we'll need to write that code ourselves. Common approaches involve persisting messages on the server and having clients request missing messages. For an example, see Chris McCord's Phoenix training: [client code](https://github.com/chrismccord/elixirconf_training/blob/master/web/static/js/app.js#L38-L39) and [server code](https://github.com/chrismccord/elixirconf_training/blob/master/web/channels/document_channel.ex#L13-L19).

### Presence

Phoenix ships with a way of handling online users that is built on top of Phoenix.PubSub and Phoenix channels. The usage of presence is covered in the [presence guide](presence.html).

#### 实列应用

这个实列的源码在 (https://github.com/chrismccord/phoenix_chat_example).

你也可以访问一个线上demo (http://phoenixchat.herokuapp.com/).

