# Bot Defense

---
**Paid Feature**

This is a paid Paraxial.io feature. It is not available in the free tier.

---

This page is a tutorial to send HTTP events to the Paraxial.io backend. You should complete the [Getting Started](./start.md) guide first. 

## Middleware

Update the middleware you already have with the `Paraxial.record(request, status)` method:

`lib/middleware/ip_filter_middleware.rb`

```
require 'paraxial'

class IpFilterMiddleware
  def initialize(app)
    @app = app
  end

  def call(env)
    request = ActionDispatch::Request.new(env)

    Paraxial.block_cloud_ip(request, ['/login'])
    Paraxial.req_allowed?(request)

    if env['paraxial.deny']
      status = 403
      Paraxial.record(request, status)
      [status, {'Content-Type' => 'text/plain'}, ['Forbidden']]
    else
      status, headers, response = @app.call(env)
      Paraxial.record(request, status)
      [status, headers, response]
    end
  end
end
```

This does the following:

1. Creates a new request via middleware.
2. If the request is coming from a cloud server IP, and matches the `/login` route, do not allow the request. Note that your own authentication route may be different.
3. Check if the request is allowed, setting the `env` value.
4. If the request was denied, set it to 403 and return forbidden. If it is allowed, pass it through. In both cases the request will be recorded and sent to the Paraxial.io backend.

Now update the application file to ensure the middleware is loaded:

`config/application.rb`

```
require_relative 'boot'
require 'rails/all'

Bundler.require(*Rails.groups)

module SampleApp
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 7.0

    # [Paraxial.io] Add these lines to ensure IpFilterMiddleware is loaded
    Dir[Rails.root.join('lib', 'middleware', '*.{rb}')].each { |file| require file }
    config.middleware.use IpFilterMiddleware

  end
end
```

Ensure you have the `PARAXIAL_API_KEY` value set and start rails:

```
@ sample_app % rails s               
=> Booting Puma
=> Rails 7.1.3.4 application starting in development 
=> Run `bin/rails server --help` for more startup options
[Paraxial] Agent starting...
[Paraxial] API key detected, agent starting
[Paraxial] Exploit Guard, no configuration exists, will not run
[Paraxial] Cloud IPs set
Puma starting in single mode...
* Puma version: 6.4.2 (ruby 3.1.2-p20) ("The Eagle of Durango")
*  Min threads: 5
*  Max threads: 5
*  Environment: development
*          PID: 36781
* Listening on http://127.0.0.1:3000
* Listening on http://[::1]:3000
Use Ctrl-C to stop
```

Load the web page in your browser. Then, go to your Paraxial.io site:

<img src="../assets/bot_http.png" alt="gl" width="800"/>

If you can see requests in HTTP Tail, it is working.
