---
layout: post
title: Introducing check_http
url:  /blog/2012/08/27/introducing-check-http/
categories:
  - monitoring
---

#{{ page.title }}

Monitoring tooling has been a fascination of mine since I began hacking on computing systems.  Every ops engineer wants to write their own.  I get that.  I'm one of those guys, too.

However, these endeavours always end up spitting out large, monolithic applications that just aren't as good as the already existing (and somewhat crappy) monolithic applications.  I'd prefer to avoid that approach, if only to make it a point that the monitoring story can never be finalized.  We will never reach an end state, but the more flexible our tooling the better.

[Traditional Unix philsophy](http://u-tx.net/ccritics/art-of-unix-programming.html) gives us something to go on here.  What if, instead of writing large applications, we just wrote a bunch of tiny tools.  AND - rather than writing each of these tiny tools as sexy network daemons (which we all want to do, right?), what if we focused on implementing them as simple command-line utilities wherever practical?  Less tight coupling and all that.  We can always turn them into sexy daemons later. The end result is something more akin to Legos than to pre-fabbed plastic castles.  It's composable.  It's Voltron (h/t to my friend [obfuscurity](http://obfuscurity.com/2011/07/Robots-Are-Cool-and-Shit-But-Seriously)).

That's what I'm trying to get going with [`check_http`](http://github.com/gorsuch/check_http), something I see as a small piece of the modern monitoring story.

Let's take a look at how this thing currently behaves:

{% highlight bash %}
$ check_http https://github.com
return_code=ok total_time=0.869853 connect_time=0.110924 namelookup_time=0.023941 effective_url=https://github.com primary_ip=207.97.227.239 response_code=200 redirect_count=0 url=https://github.com
{% endhighlight %}

A few things to note here:

1. It emits a single line of text arranged in `key=value` format
2. The data is rich
3. It takes no action outside of emitting these results
4. It makes no judgement about the target's availability

Further, the tool is capable of taking in a list of urls:

`urls.txt`:
{% highlight bash %}
$ check_http https://github.com
http://gorsuch.github.com
http://www.heroku.com
https://github.com
{% endhighlight %}

{% highlight bash %}
$ cat urls.txt | check_http
return_code=ok total_time=0.227701 connect_time=0.095336 namelookup_time=0.028544 effective_url=http://gorsuch.github.com primary_ip=204.232.175.78 response_code=200 redirect_count=0 url=http://gorsuch.github.com
return_code=ok total_time=0.281105 connect_time=0.087912 namelookup_time=0.029107 effective_url=http://www.heroku.com primary_ip=174.129.23.118 response_code=200 redirect_count=0 url=http://www.heroku.com
return_code=ok total_time=0.353444 connect_time=0.094452 namelookup_time=0.026816 effective_url=https://github.com primary_ip=207.97.227.239 response_code=200 redirect_count=0 url=https://github.com
{% endhighlight %}

Do you see where this starts to take us?  I'm talkin' about `pipelines`.  Eventually, if I have my way, we end up with a toolchain that might look like this:

{% highlight bash %}
$ scheduler | check_http | judge >$(log) >$(graph) >$(notify)
{% endhighlight %}

In the above example of fictional tools, we have a `scheduler` that periodically pushes urls into `check_http`.  The results are then fed into our `judge` process, which interprets the results.  His output feeds into some sort of logger and metrics system.  Notifications are sent out as needed.

Each piece is well understood and easy to reason about.  The codebase for each building block is likely small ([proof!](https://github.com/gorsuch/check_http/blob/master/lib/check_http/checker.rb)).  It is quite flexible - use what you want and introduce whatever you feel is lacking.  Throw away things without much guilt, knowing that your investment was quite low.

[Bug reports welcome, feedback more so](https://github.com/gorsuch/check_http/issues/new). There will be more to come over the following days and weeks.

Also, a big shout out to [Hans Hasselberg](https://github.com/i0rek), the maintainer of [Ethon](https://github.com/typhoeus/ethon).  `check_http` is really nothing more than a gentle wrapper around that.
