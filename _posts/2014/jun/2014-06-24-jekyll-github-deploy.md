---
layout: post
title: "Deploy Jekyll to Github Pages"
date: 2014-06-24
tags: jekyll github ruby
description:
  Jekyll site can easily be deployed to Github Pages
  even with custom plugins; just use my jgd Ruby gem
keywords:
  - jekyll
  - jekyll github
  - jekyll github pages
  - deploy jekyll to github
  - deploy jekyll to github pages
  - how to deploy jekyll to github
  - custom plugins with jekyll github
  - jekyll github safe mode
---

This blog is written in [Jekyll](http://jekyllrb.com/) and is hosted at
[Github Pages](https://pages.github.com/). It uses half a dozen custom
plugins, which [are not allowed there](https://help.github.com/articles/using-jekyll-plugins-with-github-pages).

Here is how I deploy it:

{% highlight bash %}
$ jgd
{% endhighlight %}

That's it. [jgd](http://rubygems.org/gems/jgd) is my
Ruby gem (stands for "Jekyll Github Deploy"), which does
the trick. Here is
[what it does](https://github.com/yegor256/jekyll-github-deploy/blob/master/bash/deploy.sh):

<!--more-->

1. It clones your existing repository from the current
directory to a temporary one (guessing the URL of the repo from `.git/config` file).

2. Runs `jekyll build` in that temporary directory,
which saves the output in another temporary directory.

3. Checks out `gh-pages` branch or creates one if it doesn't exist.

4. Copies the content of the site built by `jekyll build` into the branch, thus overwriting existing files, commits and pushes to Github.

5. Cleans up all temporary directories.

Using this gem is very easy. Just install it with
`gem install jgd` and then run in the root directory of your Jekyll blog.

What is important is that your Jekyll site files be
located in the root directory of the repository. Just
as they do on this blog; see [its sources in Github](https://github.com/yegor256/blog).

You can easily integrate `jgd` with Travis. See
[`.travis.yml`](https://github.com/yegor256/blog/blob/master/.travis.yml) of this blog.

Full documentation about the gem is located
[here](https://github.com/yegor256/jekyll-github-deploy).
