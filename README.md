##Loggster Middleware

Let’s write our own logging middleware. We’ll call the class Loggster! :-) What we are aiming for is middleware, that will wrap our Rack application and will log the requests that come in to a logfile. Sounds simple? Lets see…

```ruby
# loggster.ru
class Loggster
  def initialize app
    @app = app
  end

  def call env
    start_time = Time.now
    status, headers, body = @app.call env
    end_time = Time.now

    Dir.mkdir('logs') unless File.directory?('logs')
    File.open('logs/server.log', 'a+') do |f|
      f.write("[#{Time.now}] \"#{env['REQUEST_METHOD']} #{env['PATH_INFO']}\" #{status} Delta: #{end_time - start_time}s \n")
    end

    [status, headers, body]
  end
end

class RackApp
  def self.call env
    [200, {'Content-Type' => 'text/html'}, ['Hi!']]
  end
end

use Loggster
run RackApp
```

The Rack application, or, the RackApp class just returns a HTTP 200 with a “Hi!” message. Simple stuff. So, what does Loggster do? When we add the use Loggster line in the file, we tell Rack to wrap the request handler a.k.a. RackApp with Loggster. 

This means that `Loggster#call` will call RackApp.call, it will write to the log file and return the full response that RackApp returned. The “magic” call method, is really simple. It checks if there’s a directory called “logs” and creates it if it does not exist. 
After, it creates a server.log file and inside, it writes the current time, the request method (i.e. GET), the URL that the web client requested and the response HTTP status.

If we run this file with `rackup loggster.ru` and visit the URL, we’ll see that a directory called `logs` and a file `server.log` inside, have been created. So, how does the output of our simple logger looks?

➜ cat logs/server.log
[2015-06-27 01:24:37 +0200] "GET /something" 200 Delta: 1.3s
[2015-06-27 01:24:37 +0200] "GET /favicon.ico" 200 Delta: 0.3s
[2015-06-27 01:24:37 +0200] "GET /favicon.ico" 200 Delta: 0.2s
As you can see, I requested localhost:<port-number>/something via my browser. The browser requested the path and it also requested the favicon twice.

As you can see, I requested localhost:<port-number>/something via my browser. The browser requested the path and it also requested the favicon twice.

###Outro

As you can see, writing Rack middleware can be pretty simple. Of course, it all depends on what functionality you would like the middleware to have. That being said, the rules (or constraints) that Rack imposes when writing middleware are tiny and very clear.
