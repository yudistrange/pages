---
title: Creating a chat server using Phoenix Channels
date: 2020-08-02 17:20:19
tags:
- elixir
- phoenix
- websockets
---

In this post I will try to create a chat server using [`Channels`](https://hexdocs.pm/phoenix/channels.html) provided by the [Phoenix framework](https://phoenixframework.org/).

Phoenix is the de-facto web framework in Elixir language. Channels are an abstraction that Phoenix provides to multiplex over a group of connections. Channels do not specify the transport mechanism for these connections, they can be `websockets` / `long-polling` / `our own custom transport`.

For the purpose of this post, I will use websockets as the transport mechanism.

First let's create a new phoenix application
```zsh
mix phx.new chatter --no-gettext --no-webpack --no-html --no-dashboard --no-ecto
```

I have used the `--no-<feature>` flags to turn off some of features that are included by default in a Phoenix application. The list of these options can be [found here](https://hexdocs.pm/phoenix/Mix.Tasks.Phx.New.html)

Once the project has been bootstrapped and the dependencies downloaded, you can run the application 
```zsh
cd chatter && mix phx.server
```
This will prompt that the service is accessible at `localhost:4000`. If you go to that url you would be greeted with Phoenix Framework welcome page.

Next let's go through the folder structure of the application for a bit.

- `mix.exs` file stores the deps for the application along with other configurations such as the main application module, the version of elixir to be used with the app etc. `package.json` is the JavaScript equivalent for it.
- `mix.lock` file stores the installed version of the dependencies, along with the git commit hash. It's equivalent to `package.lock` in JavaScript land.
- `/config` folder contains the configuration files for the application. There's one `.exs` file per environment. For instance `dev.exs`, `prod.exs` etc. These configuration files contain the environment specific config variables.
- `/lib` folder contains the elixir source files for the application.

Now let's take the first steps towards creating a chat server. To configure a socket connection in Phoenix you need to edit the `Endpoint` module. More specifically configure the url where you want the client to connect to; the transport mechanism; and the module which handles the incoming connections.

{% codeblock Endpoint.ex lang:elixir %}
socket "/socket", ChatterWeb.UserSocket,
  websocket: true,
  longpoll: false
{% endcodeblock %}

Let's take a look at UserSocket and configure it for incoming connections

{% codeblock UserSocket.ex lang:elixir %}
# Create a channel listening on topic 'chatter:room:*'
channel "chatter:room:*", ChatterWeb.ChatterRoomChannel

# handle incoming connections
def connect(_params, socket, _connect_info) do
  {:ok, socket}
end
{% endcodeblock %}

The `connect` function can be modified to build user authentication, but we can tackle that later. For now let's go ahead an create a channel module which will handle the 









A phoenix application bootstrapped with the `webpack` feature enabled bundles a few JavaScript files in the `/assets/js` directory. We will take a look at `socket.js` and `app.js`

socket.js describes how the frontend is connecting to the Phoenix channel. If you did not generate a socket.js file, you can take a look at the following snippet.


{% codeblock app.js lang:javascript %}
import {Socket} from "phoenix"

// Connect to a Phoenix socket at '/socket' url
let socket = new Socket("/socket", {params: {}})
socket.connect()

// Once the socket is connected subscribe to a channel with topic
// as 'topic:subtopic'
let channel = socket.channel("topic:subtopic", {}) 

// Log to the console when receiving messages from the backend
channel.join()
  .receive("ok", resp => { console.log("Joined successfully", resp) })
  .receive("error", resp => { console.log("Unable to join", resp) })
{% endcodeblock %}


let's edit the app.js to include socket.js, so that a socket is created as soon as the page is loaded.
{% codeblock app.js lang:javascript %}
import "../css/app.scss"
import socket from "./socket"
import "phoenix_html"
{% endcodeblock %}

If you check the browser's console now, it would be littered with the "Unable to join" log messages. Let's fix that.

So in Elixir the socket configuration will sit by default in the `Endpoint` module
``` elixir
socket "/socket", ChatterWeb.UserSocket,
  websocket: true,
  longpoll: false
```
Herein we are specifying that the application will listen to socket connections at the `/socket` endpoint. The `websocket: true` configuration also tells the application to only use websockets for this endpoint. Notice that the module serving this socket is `ChatterWeb.UserSocket`. You can create multiple socket endpoints each with differing (or same) handler modules


