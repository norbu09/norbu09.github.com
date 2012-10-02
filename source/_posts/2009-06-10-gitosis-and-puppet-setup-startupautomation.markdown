---
layout: post
title: puppet and gitosis (startup automation 1)
excerpt: Startup automation became a big buzz currently but I think it
         has always been key to those who were successful. The speed to
         market is important and this is not only true for small
         companies.

---

Startup automation - Part 1
===========================

Startup automation became a big buzz currently but I think it has always
been key to those who were successful. The speed to market is important
and this is not only true for small companies.

Enough marketing blurb. Truth is we tech guys are lazy and that is a
good habit. Laziness keeps us creative in terms of minimising tasks we
have to do and that means in automating stuff. When you (as the tech
guy) grow a company there are many things only you know and can do. This
is normal. One day your marketing guy wants to release new content not
only once but 5 times and you start to think about a real rollout
process. Tis is not exactly new technology but it keeps you busy for a
while and there is a surprising lack of good tutorials out there.

I played with puppet and gitosis the other day to automate parts of our
environment and came up with the following setup. It runs on a slicehost
slice on debian - being a freebsd and mac guy I can confirm the setup is
not too different on those platforms either :-)

gitosis
-------

gitosis is a git hosting platform that uses virtual users and is managed
entirely with git hooks. This is nice as you simply check out the master
repository and add users and repositories there. Really simple, really
effective.
It looks like there is exactly one post in the wild that describes
gitosis setup and that is from 2007 ([scie.nti.st] [1])

First important thing here is a ssh key. You should have one and used to
use it. If you never heard of ssh keys do your homework first!

The last part of your ssh key is normally username@host. Edit that one
to be only your preferred username - this identifier will be the
reference in your gitosis config for your user.

I started playing with gitosis and came up with the following order:
- install gitosis from your preferred package management system (ports,
  apt, ...)
- copy your ssh public key /tmp on the gitosis host
- initialize the repository
  {% highlight sh %}
  $ sudo -H -u gitosis gitosis-init < /tmp/id_dsa.pub
  {% endhighlight %}
- clone the new master repository 
  {% highlight sh %}
  $ git clone gitosis@GITHOST:gitosis-admin.git
  {% endhighlight %}
- edit gitosis.conf
  {% highlight ini %}
  [group admin]
  members = YOURNAME
  writable = puppet
  {% endhighlight %}
- add the gitosis.conf and commit and push
- you have successfully created your first step in a working gitosis
  setup
- create a local repository called puppet (git init ...)
- tell the git repository where your gitosis server is
  {% highlight sh %}
  $ git remote add origin gitosis@GITHOST:puppet.git
  {% endhighlight %}
- touch a file (you need to have something in the repository to commit)
- add and commit the file
- push the repository to the gitosis server
  {% highlight sh %}
  $ git push origin master:refs/heads/master
  {% endhighlight %}

After this you have two things - a master gitosis repository for your
git server management and a local puppet repository for your future
puppet setup. This puppet setup does not do anything yet so lets wire
that one up to your puppetmaster. For simplicity I assume that the
gitosis server will also be your puppetmaster. Once the underlying
concept is understood you can easily change the puppetmaster to any
other host by only changing the post commit hook in the according git
repository.

puppet
------

Till now we did not bother finding the filesystem location of our new
gitosis setup. Now we need some git hooks in the puppet repository and
we have to find it. Under debain you can find the gitosis repository
structure in `/srv/gitosis` and our puppet repository under
`/srv/gitosis/repositories/puppet.git`.

I started out by creating a `bin` directory for the gitosis user as I
expect to share some code between different hooks. There will be many
"check out this repo to this working directory" hooks for various
purposes. The first script is very basic and does nothing fancy - we can
beef it up later on as we go.

`/srv/gitosis/bin/update.sh`
{% highlight sh %}
#!/bin/sh
umask 002

PFAD=$1
CURR=`pwd`
cd ${PFAD}

env -i /usr/bin/git-pull origin master
env -i git-update-server-info
cd ${CURR}
{% endhighlight %}

This simple script takes one argument which is the destination path and
doe a git pull there. Very generic and not too secure but it does the
trick for the beginning.

Now we need to make this script executable and add it to the
post-receive git hook in the puppet repository. Go to
`/srv/gitosis/repositories/puppet.git/hooks` and edit the `post-receive`
hook. We only need the following line in there for the moment:

{% highlight sh %}
~/bin/update.sh /etc/puppet
{% endhighlight %}

This calls our shiny update script we wrote moments ago. Make sure the
hook script is executable. Now we start setting up puppet. Make sure you
installed puppetmasterd via your favorite package management system. In case you
have a working puppet setup already make sure you back up all your
configuration files. I assume there is nothing here yet and we can axe
the existing stuff and start from scratch.

Remove the existing `/etc/puppet` directory and clone our repository
here.
{% highlight sh %}
$ cd /etc
$ rm -rf puppet
$ git clone /srv/gitosis/repositories/puppet.git
$ chmod -R g+w puppet
{% endhighlight %}

Now we have to modify two hooks to make sure we fix permissions. This is
a ugly hack and only gets us started for now - this should really be
done nicer later on. Edit in the puppet directory under `.git/hooks` the
file `post-merge` and add the following:

{% highlight sh %}
#!/bin/sh

echo "granting rights in `pwd`"
sudo chown -R root:root /etc/puppet
{% endhighlight %}

To make this work you need to make sure that the gitosis user is in your
sodoers file and of course that sudo is installed. Make this file
executable and copy it to the `post-checkout` hook.

Nearly there, test it now and see if you get errors when you push puppet
configuration to the server. If all goes well you should see the puppet
config hitting your `/etc/puppet` directory on the server right after
you pushed your local repository.

How to set up a basic puppet infrastructure will be covered in the next
part of this series. Stay tuned ...

[1]: http://scie.nti.st/2007/11/14/hosting-git-repositories-the-easy-and-secure-way "scie.nti.st"
