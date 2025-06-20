# Agent Quick Install

## Introduction 

This is a concise list of steps to install the Paraxial.io agent. It is intended for users who have already read the Getting Started guide. 

## Mix install paraxial

```elixir
{:paraxial, "~> 2.8.4"}
```

## Config

```elixir
config :paraxial,
  paraxial_api_key: System.get_env("PARAXIAL_API_KEY"),  # Required
  fetch_cloud_ips: true,                                 # Optional, set to true if using Paraxial.AssignCloudIP or Paraxial.BlockCloudIP
  exploit_guard: :monitor                                # Optional, set to :monitor or :block 
```

Set the `PARAXIAL_API_KEY` environment variable to keep this secret out of source code. 

## Edit `endpoint.ex`

For bot defense:

```elixir
  plug RemoteIp
  plug Paraxial.AllowedPlug
  plug Paraxial.RecordPlug
  plug HavanaWeb.Router
  plug Paraxial.RecordPlug
```

## (Optional) Only send events for specific routes

To only collect data for specific routes, set your configuration at compile time to the code below, by editing your `config/dev.exs`, `config/test.exs`, and `config/prod.exs` files. 

Do NOT set `only:` or `except:` at runtime, the agent uses metaprogramming to generate code at compile time. If you set `only:` or `except:` at runtime, the agent will ignore the config and send data for all routes. As of `2.0.0`, you will get an error (see below).

To have the agent only send data to the backend for the following:

- GET /users/log_in
- POST /users/log_in
- GET /users/:id/settings

```elixir
config :paraxial,
  paraxial_api_key: System.get_env("PARAXIAL_API_KEY"),  
  ... 
  only: [
    %{path: "/users/log_in", method: "GET"},
    %{path: "/users/log_in", method: "POST"},
    %{path: "/users/:id/settings", method: "POST"}
  ]
```

To send events for all routes, except the following:

- GET /health_check
- GET /users/:id/status

```elixir
config :paraxial,
  paraxial_api_key: System.get_env("PARAXIAL_API_KEY"),  
  ... 
  except: [
    %{path: "/health_check", method: "GET"},
    %{path: "/users/:id/status", method: "GET"}
  ]
```
 
After changing `only/except` in your compile time configuration, you must run:

```
mix deps.compile paraxial --force
```

Or you will get an error:

```
@ air % mix phx.server
ERROR! the application :paraxial has a different value set for key :except during runtime compared to compile time. Since this application environment entry was marked as compile time, this difference can lead to different behaviour than expected:

  * Compile time value was not set
  * Runtime value was set to: [%{method: "GET", path: "/health_check"}, %{method: "GET", path: "/users/:id/status"}]

To fix this error, you might:

  * Make the runtime value match the compile time one

  * Recompile your project. If the misconfigured application is a dependency, you may need to run "mix deps.compile paraxial --force"

  * Alternatively, you can disable this check. If you are using releases, you can set :validate_compile_env to false in your release configuration. If you are using Mix to start your system, you can pass the --no-validate-compile-env flag
```


## (Optional) Assigns paraxial_current_user

```elixir
defmodule HavanaWeb.Router do
  ...
  pipeline :browser do
    ...
    plug Paraxial.CurrentUserPlug
  end
```

Note: This essentially does `assign(conn, :paraxial_current_user, conn.assigns.current_user.email)`, so make sure `conn.assigns.current_user.email` exists. 

## (Optional) Send :paraxial_login_user_name via assigns

In your application code, determine how a user's login attempt flows through the code. You are looking for the line right before the user's provided email and password are checked against the database. Once you find that location, re-write the conn with: 

`conn = assign(conn, :paraxial_login_user_name, email)`

Where `email` is the user provided string for a login attempt. 

## (Optional) Send login success true or false

`conn = assign(conn, :paraxial_login_success, false)`


