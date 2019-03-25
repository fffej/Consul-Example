# Consul Services

Consul is a service mesh tool. Every node that provides a service must run a `consul agent` and register with consul. Why would you do this? 

* Service Discovery - client can find (using DNS or HTTP) the services they depend on.
* Monitoring - Once registered, you can provide health checks, the allows monitoring and (for example) routing to be redirected to healthy nodes
* Shared storage - Services can use Consul's key value store (this is consistent across the areas)
* Security communication - Consul does all the TLS magic and can enforce who can talk to what.

By using Consul you can avoid doing all this gubbins in your application code and do it via configuration instead.

# Creating a Service using the CLI

I'm assuming you've got a development server running (if not, run `consul agent --data-dir=./temp -dev` and you should be set.). Let's register a service that does nothing.

`> consul services register -name=banana`

If you look at the output in your server you should see that this has translated to a `PUT` to register this service. You can see the services available by executing `consul catalog services` or by browsing http://localhost:8500/ and having a nosey around. 

Creating it from the command line doesn't offer a whole host of flexibility. Let's remove this service

`> consul services deregister -id=banana`

If you do your browsing again, you should see it's all disappeared.

# Creating a Service using a Service Definition

To configure a service, we can define it using a lump of JSON. To make sure that the service is found, we pass in an extra argument on startup:

`> consul agent --data-dir=./temp -dev --config-dir=./config`

In that directory (`./config`) let's put a minimal service definition in a filel called `banana.json`. Note the file extension MUST be `.json` or `.hcl` for it to be picked up.

```
{
  "service": {
      "name": "banana"    
  }
}
```

If you restart your `consul agent` (exit with Ctrl + C and run it again), you'll see that the service has been picked up and you can see it in the same way you did before. Note, there's an API for doing all of this dynamically - you wouldn't have to restart the server if you were doing it properly.

# Exposing a real service using Connect

This [service definition](https://www.consul.io/docs/agent/services.html) is very flexible. In this section, we're going to expose a service running on our local machine.

Let's create a web server. Now I'm on Windows, so I'm going to do the following entirely comprehsensible Powershell to serve "Hello world". If you execute the below in an admin (for the port!) PowerShell then you'll find you have a web server at http://localhost:8000/ that serves us a hello world JSON.

> $Hso=New-Object Net.HttpListener;$Hso.Prefixes.Add("http://+:8000/");$Hso.Start();While ($Hso.IsListening){$HC=$Hso.GetContext();$HRes=$HC.Response;$HRes.Headers.Add("Content-Type","application/json");$enc = [System.Text.Encoding]::UTF8; $Buf=$enc.GetBytes("{ 'hello': 'world'}");$HRes.ContentLength64=$Buf.Length;$HRes.OutputStream.Write($Buf,0,$Buf.Length);$HRes.Close()};$Hso.Stop()

Now we want to add this as a service, so in our `./config/` directory let's create a service definition.

```
{
  "service": {
    "name": "hello-world",
    "port" : 8000
  }
}
```

If we restart then we'll see that we do indeed have a "hello-world" service registered, but that's it. Setting the port just makes that meta data available, it does nothing else. What we want to do is create a proxy to the original address. This per-service proxy will transparently handle inbound and outbound service connections, automatically doing all the TLS cleverness.

[Consul Connect](https://www.consul.io/docs/connect/index.html) is the magic we need and this is configured in the service definition file too. This time, we'll configure it with the Connect cleverness. Edit your `./config/hello-world.json` to look like:

~~~
{
  "service": {
    "name": "hello-world",
    "port" : 8000,
    "connect": { "sidecar_service": {} }
  }
}
~~~~

After restarting the service, you'll see that Consul has registered a sidecar process for our "hello world" service. It's important to note that this just says a proxy should be running but doesn't actually start the process for you.

Let's start the proxy in a separate window with:

> consul connect proxy -sidecar-for hello-world

This create the sidecar for our "hello-world" service. This proxy is the one that represents a specific service (hello-world). It accepts inbound connections on a dynamically allocated port, verifies and thorizes the TLS connection and proxiues back a standard TCP connection to the process.

Now let's create a proxy for accessing the service. We do this (again) with `consul connect proxy`. This creates a new service that can access any other service you have permission for.

> consul connect proxy -service biddly -upstream hello-world:9000

This creates a new service, `biddly` that forwards to our `hello-world` service running on port 9000.

If this has all worked then:

* http://localhost:8000/ will return you a boring `{ 'hello' : 'world'}` set of data. This is coming directly from the Powershell webserver command you put in.

* http://127.0.0.1:9000 will return you exactly the same boring data, BUT the connection between this and your `localhost:8000` is TLS encrypted. The only thing unencrypted is the loopback-only connections.

This is already seeming like a fair bit of work, but in the next section we'll see why this is important.









