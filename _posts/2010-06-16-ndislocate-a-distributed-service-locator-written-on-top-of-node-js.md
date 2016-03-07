---
layout: post
title: ndislocate - A distributed service locator, written on top of Node.js
date: 2010-06-16 19:19:25.000000000 -07:00
permalink: /2010/06/16/ndislocate-a-distributed-service-locator-written-on-top-of-node-js/
type: post
published: true
status: publish
categories: []
tags: []
author:
  login: addumb
  email: adam@addumb.com
  display_name: addumb
  first_name: ''
  last_name: ''
---
[http://github.com/pquerna/ndislocate](http://github.com/pquerna/ndislocate)

This looks pretty cool! My coworker, [Vijay](http://github.com/lukatmyshu),  showed me this. Built on node.js. A simple distributed service locator. Well... a service locator service. Like a factory factory? Whatever, I want to check it out...

{% highlight bash %}
git clone git://github.com/ry/node
cd node
#WARNING: it didn't pick up my libevent properly,... whatever though, I'm just seein' how it works
./configure && make && sudo make install
cd ..
git clone git://github.com/pquerna/ndislocate
cd ndislocate
echo "{ }" > test.json
./ndislocate.js this_isnt_even_read_so_why_bother_requiring_a_third_param
{% endhighlight %}

Voila! It told me that it found my local HTTP, HTTPS and SSH servers. I'll fiddle more tomorrow, I suppose.
