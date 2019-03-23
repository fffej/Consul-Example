
# Agents

An agent, started by `consul agent` is required on every machine that is part of the cluster. An agent can run in one of two modes:

* _client_ - A client is an agent that forwards all remote procedure calls to a server. 

* _server_ - A server is an agent with an expanded set of responsibilities. Servers are where the data is stored and replicated and it's recommended to have 3-5 servers to avoid failure scenarios.

To start an agent in client mode, you just need something like this.

```

> consul agent --data-dir=./temp
==> Starting Consul agent...
==> Consul agent running!
           Version: 'v1.4.4'
           Node ID: 'f40baca6-1cd5-909b-5ac1-64f4ab00308d'
         Node name: 'DESKTOP-CGDRMHI'
        Datacenter: 'dc1' (Segment: '')
            Server: false (Bootstrap: false)
       Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: -1, DNS: 8600)
      Cluster Addr: 192.168.0.15 (LAN: 8301, WAN: 8302)
           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false

==> Log data will now stream in as it occurs:

    2019/03/23 07:51:42 [INFO] serf: EventMemberJoin: DESKTOP-CGDRMHI 192.168.0.15

```

You'll notice it says "Datacenter" is dc1. In Consul, a datacenter is simply an area of the network that is connected, reliable and has high bandwidth. You might have your local area network being a single data center, but you wouldn't then try to add your private cloud in their. Instead, you'd create a second data center. That's way beyond what we're going to do here.

The `data-dir` location provides a directory for storing state. You need this in either mode. The important thing here is that this directory should be durable across reboots AND it must support file system locking.

The agent will run forever. In this case, we're running a single client and it's got no servers. This is particularly boring. All your going to see is the message `No known Consul servers` (or similar) repeated on the screen ad-infintum.

To terminate the agent, press Ctrl+C. If you do so, you'll get some polite messages about leaving the cluster and then it should gracefully shutdown.

Let's run an agent as a server instead.

> consul agent --data-dir=./temp -dev

The `-dev` flag runs it as a server, but in development mode. It doesn't write any data to disk (gulp!) and everything in stored in memory (double gulp!). It's not intended for production, but that's cool since you're not going to be building a production environment by following this tutorial, right?

You'll see very different log messages when running as server including plenty of messages beginning `raft:`. As mentioned up above, you'd general have a cluster of 3-5 servers. [Raft](https://raft.github.io/) is a consensus protocol that ensures multiple servers agree even in difficult states such as server failure.

# Interacting with the Server

There's two ways of interacting with the server. 

From the command line I can execute CLI commands like this:

> consule members

```
Node             Address         Status  Type    Build  Protocol  DC   Segment
DESKTOP-CGDRMHI  127.0.0.1:8301  alive   server  1.4.4  2         dc1  <all>
```
If you look on the server (you're still running that right?), you'll see this actually gets mapped to an http endpoint.

> http: Request GET /v1/agent/members?segment=_all (1.0301ms) from=127.0.0.1:58444

You can have an explore of this (there's a nice HTTP endpoint) by browsing to localhost:8500

<INSERT AN IMAGE>

From this UI, you can see the state of the `consul` cluster, the items in the Key/Value store, permissions (ACL / Intentions) and finally (and probably most importantly) the services that are available.
