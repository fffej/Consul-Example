# Clusters

OK, onto the main event.

We're going to give in and use some Linux clients which we'll deploy using Vagrant. If you've not done this, then you'll need [Vagrant](https://www.vagrantup.com/downloads.html) and a provider [VirtualBox](https://www.virtualbox.org/). Run the Vagrant file from https://github.com/hashicorp/consul/tree/master/demo/vagrant-cluster with `vagrant up`.

You should be able to use `vagrant ssh n1` and `vagrant ssh n2` to get into the boxes. If you're playing along on Windows then the version of `ssh` that comes with Windows won't cut the mustard and you'll need to disable it. The following environment variable setting for PowerShell ought to sort that out.

> $Env:VAGRANT_PREFER_SYSTEM_BIN += 0

We can spin up the first node by following the instructions [here](https://learn.hashicorp.com/consul/getting-started/join). After `vagrant ssh n1` then start the agent in server mode.

> vagrant@n1:~$ consul agent -server -bootstrap-expect=1 \
    -data-dir=/tmp/consul -node=agent-one -bind=172.20.20.10 \
    -enable-script-checks=true -config-dir=/etc/consul.d

What's new here? The `-enable-script-checks` means that health checks can execute external scripts. You might not want to do that in production! Secondly, the `-bootstrap-expect` flag specifies the number of EXTRA servers we are going to expect. Let's spin that up by `vagrant ssh n2`:

> vagrant@n2~$ consul agent -data-dir=/tmp/consul -node=agent-two \
    -bind=172.20.20.11 -enable-script-checks=true -config-dir=/etc/consul.d

You might be expecting some cleverness here. For example, should the `consul agent` on n2 automatically jopin the network? It doesn't. To join a cluster you must execute the `consul join` command. You can do this from either machine, `consul join <IP>` will join together. Thanks to the gossip protocol you only need to find a single existing member. After joining full membership information is propogated about.

Once you've joined you should be able to see all members with the `consul members` command.

```
vagrant@n2:~$ consul members
Node       Address            Status  Type    Build  Protocol  DC   Segment
agent-one  172.20.20.10:8301  alive   server  1.4.3  2         dc1  <all>
agent-two  172.20.20.11:8301  alive   client  1.4.3  2         dc1  <default>
```




