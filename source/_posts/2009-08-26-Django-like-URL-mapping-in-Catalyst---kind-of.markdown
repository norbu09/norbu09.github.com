---
layout: post
title: Django like URL mapping in Catalyst - kind of
excerpt: Did you ever want to override the way Catalyst builds its path?
         a bit like Rails or Django? Did you miss the handy config file
         and did it also take you half a day to finally figure out how
         to do it and realise that it is straight forward?
---

Django like URL mapping in Catalyst - kind of

The problem I tried to solve is trivial, at least that was what I
thought. I tried to remap URLs in one of our platforms to make it easy
for others to adapt the way we call things. I18N related mostly. I have
most of the things that might change in central config file already but
figuring out how I actually get the URL map in there took a while. I
finally found <a href="http://www.catalystframework.org/calendar/2008/11">
this post from Matt Trout</a> where he outlines several possibilities to
tackle it. Facing lack of time and knowing that the mapping is done by
others ;-) I went with the verbose version. It fits quite nicely in our
way of deploying Catalyst apps with local config files that change
behaviour of the platform.

In your central config file (MyApp.conf) put a snippet like this:

{% highlight xml %}
<Controller Editor>
  <action index>
    Path        /vim
  </action>
  <action content>
    Path        /vim/content
  </action>
  <action tld>
    Path        /vim/tld
  </action>
</Controller>
{% endhighlight %}

This maps the controller `Editor` to the path `/vim` and the two actions
( content and tlds ) to the according endpoints under it. This stupid
snippet cost me a couple of hours to figure out but now live is good and
I can move on :-)

