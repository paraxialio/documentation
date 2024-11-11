# PHPAttackMiddleware

This feature requires Paraxial.io for Ruby version `1.3.0` or higher.

Most Ruby on Rails applications do not have routes ending in `.php`, so if an IP does this, that is a strong signal they are malicious and should be banned. Consider the following middleware:

```
module Paraxial
  class PHPAttackMiddleware
    VALID_LENGTHS = [:hour, :day, :week, :infinity]

    def initialize(app, length: :hour)
      @app = app
      if VALID_LENGTHS.include?(length)
        @ban_length = length
      else
        puts "[Paraxial] PHPAttackMiddleware invalid ban length: #{length}, using hour"
        @ban_length = :hour
      end
    end

    def call(env)
      request = ActionDispatch::Request.new(env)

      if request.path.downcase.end_with?('.php')
        Paraxial.ban_ip_msg(request.remote_ip, @ban_length, "Sent request ending in .php")
        # Return a 404 response if the request path ends with '.php'
        [404, { 'Content-Type' => 'text/plain' }, ['Not Found']]
      else
        # Pass the request to the next middleware or the application
        @app.call(env)
      end
    end
  end
```

This is the source code for the `Paraxial::PHPAttackMiddleware`. Example usage:

`config/application.rb`

```
module SampleApp
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 7.0

    Dir[Rails.root.join('lib', 'middleware', '*.{rb}')].each { |file| require file }
    # IpFilterMiddleware is defined in your application
    # on a paid plan, it contains Paraxial.record(request, status)
    # Putting it first allows you to see the banned requests in the 
    # Paraxial.io backend
    config.middleware.use IpFilterMiddleware
    config.middleware.use Paraxial::PHPAttackMiddleware, length: :week
  end
end
```

With the above configuration, any IP that sends a request ending in `.php` will be banned. Below is an example of a ban notification from the Paraxial.io Slack App:

<img src="../assets/php_ban.png" alt="rule" width="550"/>

If you would like to implement your own middleware logic for banning an IP address, use the function:

`Paraxial.ban_ip_msg(ip, ban_length, message)`

With the following arguments:

```
ip - string, same format as `request.remote_ip` above
ban_length - atom, options are :hour, :day, :week, and :infinity 
message - string, the reason the IP was banned
```
