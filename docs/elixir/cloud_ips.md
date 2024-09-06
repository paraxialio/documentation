# Cloud IP Matching

By default, several Cloud hosting IP ranges are defined in the Paraxial agent:

- AWS
- Azure
- GCP
- Digital Ocean
- Oracle

This is useful because a login request coming from a rented Cloud IP server is most likely a bot, and should be blocked. To make this data available locally in your agent, ensure `fetch_cloud_ips: true` is set:

```elixir
config :paraxial,
  paraxial_api_key: System.get_env("PARAXIAL_API_KEY"),
  fetch_cloud_ips: true
```

## Relevant Plugs

There are two plugs related to Cloud IP matching:

`Paraxial.AssignCloudIP`

`Paraxial.BlockCloudIP`


`Paraxial.AssignCloudIP` If the `remote_ip` of an incoming request matching a cloud provider IP address, this plug will add metadata to the conn via an assigns. For example, if a conn's remote_ip matches aws, this plug will do `assigns(conn, :paraxial_cloud_ip, :aws)`. Use this if your application has branching logic based on if an incoming conn.remote_ip is from a rented server.

`Paraxial.BlockCloudIP` - When a conn matches a cloud provider IP, the assign is updated and the conn is halted, with a 404 response sent to the client. Use this to block cloud IPs, for example in your router's authentication pipeline.

## FAQ

### Will Paraxial.BlockCloudIP block Google's Crawler?

No, Google's Cloud Platform is hosted on a different IP range from Googlebot. Google will still be able to index your site, you are only blocking requests from GCP servers that anyone can rent. 

### What if I want to allow a specific Cloud IP? For example a client has a cloud-hosted VPN with a cloud IP.

Add the IP address to your site's Allow List, and it will no longer be blocked by `Paraxial.BlockCloudIP`  

