# LiveView Bot Defense

The Paraxial.io [User Manual](user_manual.html#4-defining-rules) has section `4. Defining Rules`, which covers how to define a rule in the Paraxial.io backend for normal HTTP requests. There are some conditions where this is not enough, for example:

- In a LiveView app, you want to rate limit an action that is triggered via a websocket 
- In a Phoenix app with multiple endpoints, you want to rate limit requests with a specific `conn.host`. 

Paraxial.io agent `2.7.6` introduced the `Paraxial.check_rate/6` function for these use cases. For example:

```
def handle_event("login", %{"user" => user_params}, socket) do
  ip_string = socket.assigns.address |> :inet.ntoa() |> to_string()
  key = "user-login-#{ip_string}"
  seconds = 5
  count = 5
  ban_length = "hour"
  ip = socket.assigns.address
  msg = "`> 5 requests in 5 seconds to login from #{ip_string}`"

  case Paraxial.check_rate(key, seconds, count, ban_length, ip, msg) do
    {:allow, _} ->
      do_handle_login(user_params)
    {:deny, _} -> 
      conn
      |> put_resp_content_type("text/html")
      |> send_resp(429, "Rate limited")
end
```

It is recommended to keep the `seconds` value under 60 to avoid excessive memory usage. In the above example the rate limiting key is the incoming IP address. This could also be changed to include the `conn.host` value, user email, or any value you would like to rate limit on. 

The main benefit of this rate limiting over using an open-source version is the banning of IPs will be tracked on the Paraxial.io backend. You can also receive an alert when the rule is triggered via the Paraxial.io Slack App:

<img src="../assets/lv0.png" alt="lv" width="500"/>
