---
layout: default
title: Addumb.com
---
# Oh, hello.
I'm a Systems Engineering-type person in [Teh San Francisco Bay Area](http://en.wikipedia.org/wiki/San_Francisco_Bay_Area) for some tech company, probably (this is always outdated). It's a pretty sweet gig. You should give it a try if you can.
Oh hey! I have some stupid stuff for you on the internets I don't actually do anything with:

 * My launchpad.net account: [https://launchpad.net/~addumb](https://launchpad.net/~addumb)
 * My github account: [http://github.com/addumb](http://github.com/addumb)
   * I open-sourced [python-aliyun](https://github.com/quixey/python-aliyun/)
   * I have a python project starter template at [python-example](https://github.com/addumb/python-example/)
   * And I have a pretty handy "sshec2" command and some others in [tools](https://github.com/addumb/tools)
 * My [résumé]({{ baseurl }}/resume.html)

Here are some posts maybe?
{% for post in site.posts %}
 * [{{ post.title }}]({{ post.url }})
{% endfor %}
