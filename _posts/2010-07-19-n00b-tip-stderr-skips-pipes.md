---
layout: post
title: Linix tip - stderr skips pipes
date: 2010-07-19 09:49:10.000000000 -07:00
type: post
published: true
status: publish
categories:
- Tech
- Tip
tags: []
---
What does this line of bash output (and where?):
{% highlight bash %}
echo error > /dev/stderr | cat 2>/dev/null
{% endhighlight %}
I print "error" to stderr and pipe to cat while sending stderr to the abyss, right? Yes, I do... the pipe doesn't grab stderr. Check this out:

{% highlight bash %}
echo error > /dev/stderr | cat >&/dev/null
{% endhighlight %}

You still see "error" being printed out, because cat isn't doing it. The pipe doesn't pick up stderr, only stdout. If you need both (like in cron or if you want to mail the output of commands), be sure to use >& when redirecting:

{% highlight bash %}
echo error 2>&1 | mail -s "automated emailz" somebody@example.com
{% endhighlight %}

The moral of the story is that stderr is for errors. For stuff like "HOLY SHIT THIS BROKE ALL KINDS OF SHIT!" which is very helpful to see immediately, not when the rest of the pipes finish reading/writing. So if you want to pass stderr along, either rethink what you're doing, or use 2>&1.
