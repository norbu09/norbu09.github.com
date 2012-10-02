---
layout: post
title: git via HTTP (startup automation 3)
excerpt: To enable read only repositories via HTTP is pretty straight
         forward. I'll cover the basic setup based on nginx and our
         previously discussed gitosis setup here.
---

Startup automation - Part 3
===========================

git via HTTP with nginx
-----------------------

Just in case you don't know (very unlikely) or never played with nginx
before. This is _the_ web server and frontend HTTP loadbalancer all your
HTTP based setups should have anyway. There is documentation in English
on the nginx wiki (<a href="http://wiki.nginx.org">http://wiki.nginx.org</a>) 
and if you get stuck there is a lot of stuff just on google. The rails
community love nginx and they tend to document well - a thanks at this
point to those guys.

The setup is really straight forward but has some minor cave eats that
can cause you grief and debugging hours I want to save you.

First of all you need nginx installed on your system - obviously. I
assume that your webroot is under `/var/www`. We need a directory called
`git` under it and need to link the repositories in that you want to
share.

{% highlight sh %}
$ mkdir /var/www/git
$ cd /var/www/git
$ ln -s /srv/gitosis/repositories/[repository].git [repository]
{% endhighlight %}

This includes some assumptions. I take that you have more repositories
in your gitosis setup than public repositories you want to share or make
available via http. If this is not the case you can simply point nginx
at your gitosis repository path and add a simple rewrite rule that cuts
off the trailing `.git` of the repository path.

In nginx we need some configuration that acually serves those
repositories now. That is initially really straight forward:

{% highlight sh %}
server {
    listen   80;
    server_name  git.[your.domain.tld];
    access_log  /var/log/nginx/git.access.log;
    location / {
        root   /var/www/git/;
    }
}
{% endhighlight %}

This is the first very basic configuration for nginx. All we have to do
now is enabling a simple hook in our git repository we want to serve
and we are done.

To do that we need to go to our gitosis repository (you can simply `cd` to
the linked directory now) and edit the `hooks/post-update` hook. Add the
following line (or uncomment it) and make sure the file has the
executable bit set (`chmod +x hooks/post-update`).

{% highlight sh %}
exec git-update-server-info
{% endhighlight %}

This post update hook needs to be run once now. You can either push
something to the repository to trigger the hook or `su` to the git user
and simply run the hook. The `su` is important to not screw up the
permissions.

Speaking of permissions - we need to make the repository readable by the
nginx user. The easy way is to add `www-data` to the git group that owns
the git repository. The git group should only have read access to the
git repositories anyway so it does not open up a too big hole in your
security.

Once all that is done you can test your setup with the following
command:

{% highlight sh %}
git ls-remote
http://git.[your.domain.tld]/[repository] master
{% endhighlight %}

This should come back with something like this:

`171bb2f12ceb908fd4802857a2f577a1739f8d1f        refs/heads/master`

Securing the setup with "basic auth" and HTTPS
----------------------------------------------

Now comes the fun part. After we have this setup up and running we can
start to use it for rollout systems like capistrano and have read only
repositories that we serve via HTTPS and basic auth without the need of
deploying SSH keys all over for access to gitosis.

All we have to do now is change our nginx configuration slightly. We
need to enable HTTPS and we can even have different auth files for
different projects. This might be interesting if you run several
projects on your gitosis host with different committers.  

{% highlight sh %}
server {
    listen   443;
    server_name  git.[your.domain.tld];
    access_log  /var/log/nginx/git.access.log;

    ssl  on;
    ssl_certificate  certs/git.pem;
    ssl_certificate_key  certs/git.key;

    ssl_session_timeout  5m;

    ssl_protocols  SSLv2 SSLv3 TLSv1;
    ssl_ciphers  ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
    ssl_prefer_server_ciphers   on;

    location / {
        root   /var/www/git/;
        auth_basic            "Restricted";
        auth_basic_user_file  conf.d/htpasswd.general;
    }

    location /[repository 1] {
        root   /var/www/git/;
        autoindex  on;
        auth_basic            "Restricted";
        auth_basic_user_file  conf.d/htpasswd.[group 1];
    }
    location /[repository 2] {
        root   /var/www/git/;
        autoindex  on;
        auth_basic            "Restricted";
        auth_basic_user_file  conf.d/htpasswd.[group 2];
    }
}
{% endhighlight %}

Reload your nginx and you have your repositories served up under HTTPS
and only need to create the htaccess files now. This is simply done with
the `htaccess` program that is part of apache (or the apache utils with
come as a separate package).
Also note that your `root` directory is always the base git webroot.

some oddities
-------------

While testing I used self signed certificates and as the host is only
used in non public ways I will stay with them. This caused some googling
however as git came up with the following error:

`fatal: https://[user:pass@git.host]/[repository]/info/refs download error - SSL certificate problem, verify that the CA cert is OK. `

This is due to the self signed cert and has to be overridden with a
environment variable. A simple

{% highlight sh %}
$ export GIT_SSL_NO_VERIFY=true
{% endhighlight %}

did the trick. If you have a rollout user you want this parameter to be
in his `.profile` or your shell specific equivalent (eg `.bashrc`).

