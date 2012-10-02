---
layout: post
title: 3 new destination markets in some months
excerpt: It's been a bit of a busy time lately as we launched three new platforms
        in the last time and that brought with it a lot of challenges.
        Fortunately it also pushed me to complete our migration to our new
        backend which is now entirely message driven and based only on the
        finest stuff one finds in the moment.
---

Three new destination markets in some months
============================================

It's been a bit of a busy time lately as we launched three new platforms
in the last time and that brought with it a lot of challenges.
Fortunately it also pushed me to complete our migration to our new
backend which is now entirely message driven and based only on the
finest stuff one finds in the moment.

The backend engine is [RabbitMQ][1] based and persists messages to
[CouchDB][2] if necessary. We have a simple home grown workflow engine
that enables us to easily customise and change flows like what happens
if a domain gets registered or exactly when we charge a user. I spoke
about an early version of this workflow engine last year at the erlounge
Wellington
(http://norbu09.org/2009/10/15/one-system-to-rule-them-all.html). This
backend engine is a scalable hub for tasks and the main challenge was to
make our frontend stupid simple and more or less only a wrapper for
tasks happening in the backend. We have now nearly all the logic moved
and the frontend is quite slim, holding only the e-commerce and user
profile functionality whereas all the domain specific functionality
moved out.

The next step was to make our frontend not only simple but also
brandable and multi currency/language. This is the moment where you
notice that strings are everywhere and solid internal APIs pay off big
time. This challenge was a big one but we managed to master it and have
now an [English][3], [German][4] and [Dutch][5] version online selling
in USD, EUR and [NZD][6]. The frontend is a generic one that has many
configuration knobs that can tweak the functionality but it is still
only one code base for all projects. The template sets are separate so
each language has its own template set. The two English speaking
platforms share a template set but have different currencies - the
content however is not 100% the same on the two. All this needs lots of
config variables to check against and Template::Toolkit was a big help
here.

As you can see there went a lot of effort into designing a flexible
system that can act in different environments and still be only
one system. Most of the content is now served out of CouchDB and we
further reduced the amount of data stored in PostgreSQL as well as hard
coded texts in templates. We rewrote our shopping card from scratch and
the checkout process is really simple now as most of the heavy lifting
is now in the backend. Stuff we are working on in the moment is Session
handling and user management in CouchDB with a goal to strip out
PostgreSQL entirely (the according Catalyst::Plugins are on the way).
Don't get me wrong, we love PostgreSQL but there is no need for it
anymore once we have the remaining things ported over to CouchDB. It
simplifies our stack and reduces our admin overhead.

Speaking of admin overhead - we also run all setups (including local
development VMs) off a puppet setup and code/template/CouchDB rollout is
done via capistrano which turns out to work quite well with catalyst and
perl. We roll out a new frontend webserver in about half an hour 
from bare debian to running frontend with the most time being spent
waiting for puppet and no manual step other then installing puppet.

You see we have been busy, if you find bugs, report them. We know that
the performance in the DNS editor is still crap but we try to sort that
and there are some rough edges here and there but most of the platforms
should run smoothly. It was a steep learning curve once again but we
managed to create a scalable system that is extremely flexible and fast.

... hope you like it ...

[1]: http://rabbitmq.com
[2]: http://couchdb.apache.org
[3]: http://iWantMyName.com
[6]: http://iWantMyName.co.nz
[4]: http://meinName.com
[5]: http://benikvrij.nl
