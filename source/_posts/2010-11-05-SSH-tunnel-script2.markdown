---
layout: post
title: SSH tunnel script(2)
excerpt: I work quite a bit with daemons on remote hosts that are
         normally not directly listening to the public internet. I use SSH
         tunnels a lot and had [previously used a simplistic script][1]
         to deal with it.
---

SSH tunnel script(2)
====================

I work quite a bit with daemons on remote hosts that are normally not
directly listening to the public internet. I use SSH tunnels a lot and
had [previously used a simplistic script][1] to deal with it.

I got tired of having to reconnect SSH tunnels when I changed
environments and wanted [autossh][2] to handle the connections. I hacked
around a bit with a shell script and then came up with his perl version.
The config file is a standard git like config that is quite handy as it
handles groups of servers syntax wise.

The calls to `tunnel` take a tunnel definition from the config file and
start the tunnel with the defined user. It is really written for a safe
environment and not nailed down for security in the moment but feel free
to play with it.

The config file can deal with inheritance and checks the standard directories
(/etc, ~/ and .) and looks like this:

    [backend "rabbitmq"]
        local  = 5672
        remote = 5672
        host   = RABBIT_HOST
        user   = USER_WITH_SSH_KEYS

    [backend "couchdb"]
        local  = 5986
        remote = 5984
        host   = COUCHDB_HOST
        user   = USER_WITH_SSH_KEYS

    [frontend "couchdb"]
        local  = 5984
        remote = 5984
        host   = OTHER_COUCHDB_HOST
        user   = USER_WITH_SSH_KEYS

The tunnel script is then called with:
    
    tunnel backend.rabbitmq
    tunnel frontend.couchdb
    ...

It does some basic checking if a port is used already.

The script can be found in my [`/bin` dump on github][3]

[1]: http://norbu09.org/2009/08/04/simple-ssh-tunnel-script.html
[2]: http://www.harding.motd.ca/autossh/
[3]: https://github.com/norbu09/bin/blob/master/tunnel
