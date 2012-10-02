---
layout: post
title: lean prototyping stack
excerpt: For my new startup I play with a lean prototyping stack that
         will enable me to prototype and evaluate ideas really fast and
         test on the market.
---

lean prototyping stack
======================

For my new startup I play with a lean prototyping stack that will enable
me to prototype and evaluate ideas really fast and test on the market.
The idea is to bump out a new idea, prove it and if it works scale it
further. In order to not having to rewrite the entire app once it is
clear that the idea is worth pushing I tried to come up with a nice
stack that enables me to prototype fast and scale if I have to.

Running against all trends I use perl (neither hip nor trendy) and the
quite new [Mojolicious framework][1] to do the frontend part. In
Mojolicious::Lite I can prototype a small application in no time and can
host it really simple, worst case as a CGI. Mojolicious has a very
powerful HTTP client integrated that makes programming against JSON
based HTTP APIs really straight forward.

    $client->get($url)->success->json->{rows}->[0]->{doc}

This is a request to the second part of my stack: [CouchDB][2]. CouchDB is a
much loved citizen in my tool chain and there are no big surprises that
I picked it as my storage layer. The simplicity and elegance of CouchDB
in this environment makes it even more compelling.

When writing a small web app I normally hit the moment where I either
want to do something asynchronous or where tasks are taking too long
to wait for. Sending mails is one of those tasks. You are normally not
interested in the outcome, if things go wrong you either tell the user
via AJAX or not at all but you don't want to wait for it. I am used to
have some sort of backend that takes care of all that. Problem with a
small prototyping framework is that you don't want to write a full blown
backend. [RabbitMQ][3] to the rescue. RabbitMQ has a JSON RPC interface
that mimics AMQP via a HTTP/JSON interface. This fits perfectly into the
rest of my little framework as it is the same set of protocols again and
if I need to scale I can write native AMQP daemons.

Scaling out to may small prototypes is as easy as creating the new
CouchDB databases. Most probably there is a backend daemon for what we
need already written from an earlier project or we write a quick and
dirty script in _any_ language we like and process the asynchronous
messages. Frontwise the interface is simple enough and pure HTTP/JSON
requests all over. Starting the Mojolicious app is in the smallest form
one file that I can use as a CGI right away.

I'll push some libraries that I need on the way to [github][4] as usual. If
it turns out to work as nice and reusable as it looks in the moment I am
happy to share more if there is interest.

[1]: http://mojolicious.org
[2]: http://couchdb.org
[3]: http://rabbitmq.com
[4]: http://github.com/norbu09
