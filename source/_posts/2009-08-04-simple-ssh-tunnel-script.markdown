---
layout: post
title: simple ssh tunnel script
excerpt: Ever needed that one script that simply opens up a ssh tunnel
         and closes it after usage again? In the background? Here it is.
---

Simple ssh tunnel script
========================

When replicating CouchDB you need either VPN oder authentication or
simply a SSH tunnel. No big deal normally a simple `ssh -L...` does the
trick but I ended up with those SSH sessions that blocked ports and hung
around and it was not the way to go in terms of automating rollouts
where you need that port once and then never again.

A bit of googling brought up a script of the <a href="http://wiki.bacula.org/doku.php?id=sshtunnel">
bacula guys</a> that had the same problem and I adopted their script a
bit to fit my needs. Here is what I came up with:

{% highlight sh %}
#!/bin/sh
# Establishes a self-killing SSH tunnel to the
# given SSH server, and forwards the correct
# ports for couchdb usage.

USER=[YOUR USER HERE]
HOME=$(grep "^$USER:" /etc/passwd | cut -d : -f 6)
CLIENT=$1
SSH=/usr/bin/ssh
DESTPORT=5985

echo "Starting SSH-tunnel to $CLIENT..."
# -f means: go into background
# -C means: use compression
# -2 means: only use SSH2
# -L 5985:localhost:5984 means: when forward 5985 to destination:5984
# sleep 60 is a simple command that will execute on the server and does
# nothing for 60 seconds,
# then it exits. This keeps ssh running for 60 seconds. Once we connect
# to the FD, that
# connection will keep ssh running even beyond the 60 seconds.
# Using this approach, we do not need to tear down the tunnel later, it
# disconnects itself
# automagically.
${SSH} -fC2 -L ${DESTPORT}:localhost:5984 ${CLIENT} sleep 60 >/dev/null
2>/dev/null
# give ssh a little time to establish the connection.
sleep 10
{% endhighlight %}

Put it in a file called `sshtunnel` and make it executable. Usage is as
simple as `./sshtunnel user@host` or without the user if it is the same
you are using in the `$USER` variable.
