---
layout: post
title: puppet basics (startup automation 2)
excerpt: In the last part of startup automation I talked you through the
         basics of configuring gitosis and the puppet config directory
         to make deployment of the config files really easy. Obviously
         we need some test clients and a working server now to play. I
         use parallels for local development and normally fire up one VM
         per project. I created an empty debian VM and just did a
         `apt-get install puppet`. This installs all dependencies and
         leaves you with a working client.

---

Startup automation - Part 2
===========================

puppet basics
-------------

In the last part of startup automation I talked you through the basics
of configuring gitosis and the puppet config directory to make
deployment of the config files really easy. Obviously we need some test
clients and a working server now to play. I use parallels for local
development and normally fire up one VM per project. I created an empty
debian VM and just did a `apt-get install puppet`. This installs all
dependencies and leaves you with a working client.

create a basic configuration
----------------------------

we need some basic configuration files before we can start. Use these as
templates, they are more or less standard debian config files so you
might have to tweak paths a bit for your environment:

`puppet.conf`
{% highlight ini %}
[main]
logdir=/var/log/puppet
vardir=/var/lib/puppet
ssldir=/var/lib/puppet/ssl
rundir=/var/run/puppet
factpath=$vardir/lib/facter
pluginsync=true

[puppetmasterd]
templatedir=/var/lib/puppet/templates
{% endhighlight %}

`fileserver.conf`
{% highlight ini %}
# This file consists of arbitrarily named sections/modules
# defining where files are served from and to whom

# Define a section 'files'
# Adapt the allow/deny settings to your needs. Order
# for allow/deny does not matter, allow always takes precedence
# over deny
[files]
path /etc/puppet/files
#  allow *.example.com
#  deny *.evil.example.com
#  allow 192.168.0.0/24

[plugins]
#  allow *.example.com
#  deny *.evil.example.com
#  allow 192.168.0.0/24
{% endhighlight %}

and a basic `manifests/site.pp`

{% highlight ruby %}
# site.pp

filebucket { main: server => 'YOUR_PUPPET_SERVER_HERE' }

# global defaults
File { backup => main }
Exec { path => "/usr/bin:/usr/sbin/:/bin:/sbin" }

Package {
    provider => $operatingsystem ? {
        debian => aptitude,
        redhat => up2date
    }
}
{% endhighlight %}

Make sure you edit the `sites.pp` to actually point to your puppet
server. Add those files, commit and push. Then restart your
puppetmasterd on the server.

We need two shells now, one on the server and one on your client. Fire
up your shell on the server and ask the puppetmaster for pending
certificate requests with the `puppetca` command: 
{% highlight sh %}
$ puppetca --list
{% endhighlight %}
This should return with ` no pending requests` message or so.

Fire up the second shell on your client and request a certificate from
your server. We try it interactive first till things go smooth: 
{% highlight sh %}
$ puppetd --server YOUR_PUPPET_SERVER --waitforcert 60 --test
{% endhighlight %}
This should come back with an error that the cert could not be issued
yet. This is expected as we have not authorized the client yet. To do so
we hop over to our server console again and issue the puppetca command
again. We should have a pending request now from our test client. Go on
and sign it with 
{% highlight sh %}
$ puppetca --sign CLIENT_NAME
{% endhighlight %}
If you rerun the client now with the same command it should be happy and
report success.
