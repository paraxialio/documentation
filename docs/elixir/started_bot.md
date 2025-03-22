# Getting Started with Paraxial.io for Elixir - Bot Defense

This guide is for Paraxial.io Bot Defense. A paid account is required. If you are on the free tier, email `support@paraxial.io` to request a 2 week free trial.

You will setup the following:

- Ingesting HTTP traffic
- Rate limiting - Ban IPs that do too many login attempts
- Honeypots - Ban IPs that submit a fake form
- Banning bots scanning for `.php` routes
- Blocking data center/cloud IP attacks


## Create Account and Install Paraxial.io
You can skip this section if you have already installed Paraxial.io by following the [Getting Started guide for Application Secure.](./started_bot.md)

Create your account here - [https://app.paraxial.io/users/register](https://app.paraxial.io/users/register)

Confirm your email and create your first site:

![start](./assets/started/ST0-create-potion-shop-green.png)

`Site Name` - Cannot contain spaces.

`Timezone` - Select the timezone where you are located

`Environment` - Where is this application deployed? You should create one site for each environment.

`Web facing` - Does this application face the public internet? This is a label set by the account owner, not detected by the agent. 

`PII (Personal Identifiable Information)` - Does this application handle sensitive user data? This is a label set by the account owner, Paraxial.io cannot access PII. 

---

Install the Paraxial.io agent. The agent is written in Elixir, and installed as a Hex dependency - [https://hex.pm/packages/paraxial](https://hex.pm/packages/paraxial)

`mix.exs`
```
{:paraxial, "~> 2.8.2"}
```

```
mix deps.get
```

The package is installed as a normal Elixir dependency. Once you confirm the install was successful, go to your site in the web interface of [app.paraxial.io](https://app.paraxial.io/sites) -> Site Settings -> Site API Key. 

With the private API key, configure your project:

`config/dev.exs`
```
config :paraxial,
  paraxial_api_key: System.get_env("PARAXIAL_API_KEY")
```


## Ingest HTTP Traffic

Before installing Paraxial.io, create a new Git branch, then run `mix test` before making any code changes. If any tests are failing, make a note of that fact before you continue with the install. 

Run:

```
mix paraxial.scan
```

To ensure you have the dependency installed and API configured correctly. If the scan runs and uploads successfully, continue. 

Now that you are on a new branch, and have confirmed that your install is working, add the following plugs to record HTTP traffic:

`endpoint.ex`
```
  # plug RemoteIp               # This plug is optional, only needed if you are behind a proxy
  plug Paraxial.AllowedPlug     # Determine if an incoming request is allowed based on ban list
  plug Paraxial.RecordPlug      # To record requests that do not match the router
  plug YourProjectNameWeb.Router   # Change to your own project name
  plug Paraxial.RecordPlug      # To record requests that did match the router
```

Note that `plug Paraxial.RecordPlug` appearing twice is intentional, 

Paraxial.io uses the value of `conn.remote_ip` for bot defense. If you are behind a proxy, every HTTP request will have the same IP. This can be fixed via the [RemoteIP](https://github.com/ajvondrak/remote_ip) plug. For example, `plug RemoteIp, headers: ["fly-client-ip"]` is specific to fly.io deployments. Your configuration may be different. 

Now start your application, make sure the configuration is correct, and send some local HTTP requests:

```
@ potion_shop % mix phx.server
Generated carafe app
[info] [Paraxial] v2.8.2 URL and API key found. Agent will be started  <-- This is what you want to see
[info] Access CarafeWeb.Endpoint at http://localhost:4000
[info] GET /
[info] Sent 200 in 58ms
[info] GET /
[info] Sent 200 in 3ms
```

The requests will show as from localhost:

![asset](./assets/started/ST2-bot-defense.png)

If your application has a large amount of HTTP traffic (> 100,000 HTTP requests/month), you can restrict sending traffic to only specific routes. For detailed instructions see the Bot Defense documentation page. 

## Rate Limiting with Rules

Now that you have the appropriate plug in place (`plug Paraxial.AllowedPlug`), define the following bot defense rule:

If an IP sends more than 5 login attempts in 10 seconds, ban for 1 week:

![rule](./assets/started/ST3-define-rule.png)

You can use a similar route in your own application during testing. Now send a few HTTP requests by refreshing the page and observe a rule event has been created:

![rule](./assets/started/ST4-rule-event.png)

By default, the Paraxial.io agent does not send user email's to the backend. You can get the history of login attempts for an IP address, instructions are on the full Bot Defense page. 


## HTML Honeypot

Find a controller that does not require auth, create the following action:

```
# Values for second argument (length) are :hour, :day, :week, :infinity

def honeypot_ban(conn, _params) do
  Paraxial.ban_ip(conn.remote_ip, :week, "Triggered honeypot form, ban for 1 week")
  json(conn, %{ok: "system online"})
end
```

In your router, find a scope that goes through the :browser pipeline (for CSRF protection) and does not require auth, for example:

```
  scope "/", ParaxWeb do
    pipe_through :browser

    post "/customer", PageController, :honeypot_ban
  end
```

Create the form, with CSRF protection:

```
<%= form_for @conn, Routes.page_path(@conn, :honeypot_ban), [style: "display:none !important"], fn f -> %>
  <%= text_input f, :email, tabindex: -1 %>
  <%= text_input f, :password, tabindex: -1 %>
  <%= submit "Register" %>
<% end %>
```

LiveView version:

```
def render(assigns) do
  ~H"""
  <.form for={@form} id="customer_form" action={~p"/customer"} style="display:none !important">
    <.input field={@form[:email]} type="email" label="Email" tabindex="-1" />
    <.input field={@form[:password]} type="password" label="Password" tabindex="-1" />
    <button>Register</button>
  </.form>
  ...
```

```
# @form needs a value
def mount(_params, _session, socket) do
  form = to_form(%{}, as: "user")
  {:ok, assign(socket, form: form)}
end
```

Note that you have have to change the values here, for example your application may not have a `PageController`, you will have to substitute with a public controller action.

To test your implementation, use inspect element in your browser to remove `display:none !important` on the rendered page, then submit the form. If you get banned, the setup was successful. 


## Banning Malicious Clients 

## Data Center and Cloud IPs





