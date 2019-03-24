# Clusters

OK, onto the main event.

We're going to create two servers (running agents in server mode). Let's call these (imaginatively) Server A and Server B.

Server A is going to host the following services:
* hello-world 
* goodbye-world

Server B is going to host the following services:
* hello-world
* foo-bar

The hello-world service is going to be replicated across both servers.

We're also going to have a daft client app. This client app is going to invoke the services in a continuous loop and just write the responses out to stdout. Yes, it's really going to be that exciting.

We're going to give in and use some Linux clients which we'll deploy using Vagrant. If you've not done this, then you'll need [Vagrant](https://www.vagrantup.com/downloads.html) and a provider [VirtualBox](https://www.virtualbox.org/). Run the Vagrant file from https://github.com/hashicorp/consul/tree/master/demo/vagrant-cluster with `vagrant up`.

You should be able to use `vagrant ssh n1` and `vagrant ssh n2` to get into the boxes. If you're playing along on Windows then the version of `ssh` that comes with Windows won't cut the mustard and you'll need to disable it. The following environment variable setting for PowerShell ought to sort that out.

> $Env:VAGRANT_PREFER_SYSTEM_BIN += 0



