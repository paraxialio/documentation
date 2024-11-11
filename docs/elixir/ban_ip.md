# ban_ip/3 and PHPAttackPlug

These features require Elixir agent version `2.8.0` or higher.

In a Phoenix app you may want to define a plug with some custom logic, for example if an IP sends a request ending in `.php`, this is a strong signal they are not a real user and should be banned. You could write this plug yourself, or use the one included with Paraxial.io. 

`endpoint.ex`
```
  plug RemoteIp, headers: ["fly-client-ip"]    # This is specific to fly.io
  plug Paraxial.AllowedPlug                    # Required to enforce Paraxial.io bans
  plug Paraxial.RecordPlug                     # Optional, requires paid account
  plug Paraxial.PHPAttackPlug, length: :week   # Place before the router
  plug HavanaWeb.Router                        # The value "HavanaWeb" will be different in your own project
  plug Paraxial.RecordPlug                     # Optional, requires paid account
```

```
defmodule Paraxial.PHPAttackPlug do
  @moduledoc """
  Plug to ban IPs sending requests that end in .php

  Most Elixir and Phoenix applications do not have routes ending in .php,
  so this is a strong signal an IP is malicious. The default ban length
  is one hour, this can be configured when setting the plug in your
  endpoint.ex file:

    plug Paraxial.PHPAttackPlug, length: :week
    plug HavanaWeb.Router  # Your application name will be different

  Valid options for :length are :hour, :day, :week, :infinity
  """
  import Plug.Conn
  require Logger

  @valid_lengths [:hour, :day, :week, :infinity]
  @default_length :hour
  @ban_message "Sent request ending in .php"

  def init(opts) do
    length = Keyword.get(opts, :length, @default_length)

    if length in @valid_lengths do
      opts
    else
      Logger.warning("[Paraxial] Invalid option for Paraxial.PHPAttackPlug: #{length}, using #{@default_length}")
      [length: @default_length]
    end
  end

  def call(conn, opts) do
    if php_request?(conn.request_path) do
      length = Keyword.get(opts, :length)

      Task.start(fn ->
        Paraxial.ban_ip(conn.remote_ip, length, @ban_message)
      end)

      conn
      |> halt()
      |> send_resp(403, Jason.encode!(%{"error" => "banned"}))
    else
      conn
    end
  end

  defp php_request?(path) do
    String.ends_with?(String.downcase(path), ".php")
  end
end
```

The above code is the source for `plug Paraxial.PHPAttackPlug`. 

Now you may want to use a custom plug with your own logic in Elixir code. For this, Paraxial.io gives you the function:

`Paraxial.ban_ip(ip, length, message)`

```
  Ban an IP address, both locally and on the Paraxial.io backend.

  Returns the result of an HTTP request, for example:

  {:ok, "ban created"} - returned on successful ban

  {:error, "ban not created"} - returned if you attempt to ban an IP that is already banned

  {:error, "invalid length, valid options are :hour, :day, :week, :infinity"}

  If you are using this function in a blocking content, call with Task.start, https://hexdocs.pm/elixir/1.12/Task.html#start/1

  - `ip` - Format should match conn.remote_ip, which is a tuple, 
           {192, 168, 1, 1} or {8193, 3512, 34211, 0, 0, 35374, 880, 29492}
  - `length` - Valid options are :hour, :day, :week, :infinity
  - `message` - A string comment, for example "Submitted honeypot HTML form"
```

See the example above for how to use this function. It does trigger an HTTP request, so using a Task is helpful to prevent blocking. 

```
Task.start(fn ->
  Paraxial.ban_ip(conn.remote_ip, length, @ban_message)
end)
```
