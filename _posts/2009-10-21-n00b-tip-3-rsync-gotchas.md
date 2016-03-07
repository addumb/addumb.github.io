---
layout: post
title: Linux tip 3 - rsync gotchas
date: 2009-10-21 18:46:35.000000000 -07:00
permalink: /2009/10/21/n00b-tip-3-rsync-gotchas/
type: post
published: true
status: publish
categories:
- Tech
- Tip
tags: []
author:
  login: addumb
  email: adam@addumb.com
  display_name: addumb
  first_name: ''
  last_name: ''
---
Use rsync a lot? Do you use it just infrequently enough to have to check the man page every time? Me too. I always have trouble with out it interprets trailing slashes on paths.

 * Copy entire directory to remote parent directory:

    {% highlight bash %}
    rsync /path/without/slash host:/path/to/parent/with/slash/
    {% endhighlight %}

 * Copy contents of one directory to another:

     {% highlight bash %}
    rsync -r /path/to/directory/* host:/path/to/directory/
    {% endhighlight %}

 * Copy a directory into a *new* directory on another host:


   {% highlight bash %}
    rsync /path/without/slash host:/path/to/new/directory
    {% endhighlight %}

One more thing, I almost always use `-aP`, but for some high-latency connections you'll probably also want to pass `-z` for compressing the stream.
