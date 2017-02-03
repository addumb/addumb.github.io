---
layout: post
title: I moved addumb.com into GitHub pages
date: 2016-03-07 11:26:00.000000000 -07:00
type: post
published: true
status: publish
description: Moving a blog from wordpress and website from AWS to GitHub Pages.
keywords: github pages, aws migration, wordpress export
categories:
- Tech
- Tip
tags:
- Tech
author:
  login: addumb
  email: adam@addumb.com
  display_name: addumb
  first_name: 'Adam'
  last_name: 'Gray'
---
It was kind of fun :) Believe it or not, I actually have a couple other pages under addumb.com aside from shitty blog posts:

 - [/posts](/posts)
 - [my résumé](/resume.html)
 - my rÃ©sumÃ© (but I have since removed it because Github Pages doesn't like it for some reason.
 - [some social science statistic calculator](/derp.html)
 - [a NATO phonetic alphabet spitter-outer](/nato-phonetic-alphabet-tool.html)

Those were all really easy to move. Moving my shitty blog from wordpress.com into GitHub pages was a little more complicated than all the guides make it seem. At a high level, you'll do these four big steps:

 * Do the normal shit for [GitHub pages](https://pages.github.com/) (make a new repo, setup a jekyll site)
 * Import your Wordpress blog.
 * Fix all the stuff that just broke all over the place

## GitHub Pages
The [GitHub docs](https://help.github.com/categories/github-pages-basics/) on this are pretty straightforward. You should end up with a site with a directory structure like this:

```bash
$ tree -L 2
.
├── CNAME
├── Gemfile
├── README.md
├── _config.yml
├── _includes
│   ├── footer.html
│   ├── head.html
│   ├── header.html
│   └── sidebar.html
├── _layouts
│   ├── default.html
├── derp.html
├── index.md
├── posts.md
├── sitemap.xml
└── vendor
    └── bundle
```
The contents of CNAME here is just "addumb.com" for me. Don't do that for yours. Gemfile only contains 2 lines right now: "source 'https://rubygems.org'" and "gem 'github-pages'". README.md is just a regular GitHub readme. `_config.yml` has some annoying pieces you may need to dig up, particularly this:

```
markdown: kramdown
kramdown:
  input: GFM
  force_wrap: false
```
Then the `_includes` are just your site template pieces, `_layouts` just has a default for now which is basically empty:

```
{% raw %}<!DOCTYPE html>
<html>
{% include head.html %}
<body>
  {% include header.html %}
  {{ content }}
  {% include footer.html %}
</body>
</html>{% endraw %}
```

You should be able to run this to get your jekyll site running locally:

```
$ bundle exec jekyll serve
```

## The Wordpress.com Import Stuff
Now that you have a fancy new GitHub Pages site up and running, it's time to move your shitty blog over to it from Wordpress.com. The steps are very similar for other Wordpress setups, so try to read between the lines.

<span class="label label-warning">Warning</span> Make a new branch in your repo, you're going to totally fuck shit up.

Okay, then follow along here...

 1. Login to your wordpress.com account, then your "site", then export it all as a single XML file.
 1. Add the `jekyll-import` dependency to your `Gemfile` where you should already have `github-pages`:

    ```
    source 'https://rubygems.org'
    gem 'jekyll-import'
    gem 'github-pages'
    ```
    Don't try to add this at the outset. Install github-pages, and then jekyll-import. They have a fake version conflict.
 1. Now run this to get your import started:

    ```
    bundle exec jekyll import wordpressdotcom --source-file ~/Downloads/addumb.wordpress*.xml
    ```
    This will spew all kinds of unhelpful gem dependency failures for a while. One by one, you'll need to fix them. This step is total bullshit.

    ```
    Whoops! Looks like you need to install 'hpricot' before you can use this importer.

    If you're using bundler:
      1. Add 'gem "hpricot"' to your Gemfile
      2. Run 'bundle install'

    If you're not using bundler:
      1. Run 'gem install hpricot'.
    ```
    This is pretty straightforward:

    ```
    echo "gem 'hpricot'" >> Gemfile
    bundle install
    bundle exec jekyll import wordpressdotcom --source-file ~/Downloads/addumb.wordpress*.xml
    ```
    and repeat until you don't get a Gem error.
  1. Okay, now you have a steaming pile of malformatted ".html" files under `_posts`. Each one has "frontmatter" at the top of the page, which is put between YAML comment markers: `---`. The frontmatter is just YAML describing some specifics about each post. Go through each one and clean up the garbage left over, when you're done it should look something like this post's frontmatter:

          ---
          layout: post
          title: I moved addumb.com into GitHub pages
          date: 2016-03-07 11:26:00.000000000 -07:00
          type: post
          published: true
          status: publish
          description: Moving a blog from wordpress and website from AWS to GitHub Pages.
          keywords: github pages, aws migration, wordpress export
          categories:
          - Tech
          - Tip
          tags:
          - Tech
          author:
            login: addumb
            email: adam@addumb.com
            display_name: addumb
            first_name: 'Adam'
            last_name: 'Gray'
          ---

        You should have removed garbage like this:

          meta:
          _publicize_pending: '1'
          _edit_last: '16162427'
          _wp_old_slug: '1'
          original_post_id: '1'

<!-- Atom is telling me to terminate my markdown underline from above, so here:_ -->

## The Layout(s)

The import created a bunch of `_posts` which reference a layout called `post`. What is that and how do you create it and make it somewhat useful? (I cannot give any advice on making it truly useful.)

You might want to make the post layout so that your posts aren't rendered plainly, like [this](/plainpost). Here's a starter, just plop this in `_layouts/post.html`:

    {%raw%}
    <!DOCTYPE html>
    <html>
      <head>
        <title>
        {% if page.title %} {{ page.title }}
        {% else %} {{ site.title }}
        {% endif %}
        </title>
      </head>
      <body>
        <h1>{{ page.title }}</h1>
        {{ content }}
      </body>
    </html>
    {% endraw %}

That will just make a web page with your post's title in the title bar of the browser along with a big header up top, then your unadulterated goodness underneath.


Next, you may want to list your posts in a few different places. Here's how I made the sidebar/bottombar list of posts here. I made a file in `_includes` named `sidebar.html` and it's this:

    {%raw%}
    Other posts...
    <ul>
      {% for post in site.posts %}
      {% if page.url == post.url %}
      <li>&raquo; {{post.title}}</li>
      {% else %}
      <li><a href="{{post.url}}">{{post.title}}</a></li>
      {% endif %}
      {% endfor %}
    </ul>
    {%endraw%}

Then you can update your `post.html` layout to include this on the right-hand side however you'd like.

## SEO

Ha! I have no idea what I'm talking about here, but generally do these things:

 1. Make a [sitemap.xml](https://github.com/addumb/addumb.github.io/blob/master/sitemap.xml).
 1. Add descriptions and keywords to your site and to each post. Check the head layout on this site: [_includes/head.html](https://github.com/addumb/addumb.github.io/blob/master/_includes/head.html)
 1. Add some jQuery, that helps, right??
 1. Bootstrap something, is that still fashionable?

## Walk Away

That's it! That's how I made this thing. It was fun :)
