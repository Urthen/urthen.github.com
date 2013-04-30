---
layout: post
title: "Deploying a blog to Github in 30 minutes or less"
description: "Introduction to installing and setting up Jekyll"
category: 
tags: ["blog", "github", "jekyll"]
---
{% include JB/setup %}

# An introduction to installing and setting up Jekyll

There's a whole lot of ways out there to start a blog. From [WordPress](http://wordpress.org/) to [Blogger](http://www.blogger.com/) to [Tumblr](http://www.tumblr.com/), you can set up your own soapbox in a matter of moments. Some of us, however, desire a more hands-on approach. Fortunately there's a way to have a completely self-managed blog without having to spend a dime on hosting costs.

Enter [Github Pages](http://pages.github.com/). GitHub will let you create and host static HTML pages, or pages written with [Markdown](https://help.github.com/articles/github-flavored-markdown). Additionally, it'll let you deploy with a templating language called [Jekyll](https://github.com/mojombo/jekyll), allowing you to quickly and easily deploy blog-like pages. A final piece of the puzzle: [Jekyll Bootstrap](http://www.jekyllbootstrap.com/), a fully-featured themable blog framework that takes most of the work out of setting up a simple blog. There's excellent documentation on the Jekyll Bootstrap site detailing the very few steps neccesary to get yourself up and running. I'll be going over everything from "Hm, I should start a blog..." to "Hey, that was easy!"

## Installation

First, you'll want to make sure you've got [Ruby](http://www.ruby-lang.org/en/) installed. If you're running a Mac, it probably already is. Once you've got that, install Jekyll with `gem install jekyll`. Now create the `your-username.github.com` repository in GitHub... replace `your-username` with your actual GitHub username, of course. That's it for prerequisites, and if you're on schedule, that should have only taken about ten minutes or so.

Next ten minutes will be to set up and configure Jekyll Bootstrap. Following the instructions from the Bootstrap site, enter the following commands in your preferred projects directory (I like `~/Projects/` myself):

	$ git clone https://github.com/plusjade/jekyll-bootstrap.git your-username.github.com
	$ cd your-username.github.com
	$ git remote set-url origin git@github.com:your-username/your-username.github.com.git
	$ git push origin master

You should now be able to go to `https://your-username.github.com/` and see the Jekyll Bootstrap demo live, as well as run `rake preview` and visit `http://localhost:4000/` to see it hosted on your machine.

## Configuration

If you'd like to use a theme, now would be a good time to install one. Visit the [Theme browser](http://themes.jekyllbootstrap.com/) and choose one - you can adjust it later, of course, but it's just a starting point. Click "Install Theme" at the bottom left to be given a commandline to install the theme to your project. You should be given the choice to switch to that theme immediately; you can switch later with `rake theme:switch theme-name` if you wish.

Now let's do some configuration. `index.md` describes a few, but you'll really want to investigate `_config.yml` for the full list. At the least, you'll want to check out the following:

	title
	tagline
	author
	production_url
	JB
	- comments
	- analytics

For the Comments and Analytics sections, you'll want to set yourself up some external services. That'll be the next ten minutes, unless you've done this before. First, head to [Disqus](http://disqus.com/) and create an account for yourself there, and register your site. Add your site shortname to the configuration. Next, head to [Google Analytics](http://www.google.com/analytics/) and sign up if you haven't already. Create a new account to track your website. You should recieve a tracking code that looks something like `UA-12345-1`. Add that to the analytics section.

Your blog should be set up, at least initially. Commit and push your repository and check it online. Time to start your first post!

## Next Steps

Jekyll Bootstrap comes with a sample post in the `_posts/core-samples` directory. You'll want to read this, as it contains useful information on how the Jekyll engine works. After you've read through it, delete the entire `core-samples` directory as you'll want to start your blog fresh. You can start the first post from the commandline; for example, this is the command I used to create this post: `rake post title="Deploying a blog to Github in 30 minutes or less"`

If you've been chugging along, you should reach this point in just around 30 minutes. Now, comes the hard part: What do you write about? Unfortunately, I can't help you there.
