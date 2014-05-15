---
layout: post
title: "why i chose jekyll/octopress over wordpress"
date: 2012-09-09 21:24
comments: false
categories: tech
---
When I chose to revamp AdrianArtiles.com from a static site to something that I can share work and writings on, I quickly set to work whiteboarding everything I wanted from the new site. This narrowed my choice to two options for the framework; wordpress, and octopress/jekyll.
<!--more-->
Now let me start by saying I love wordpress. I've used it for several years and have relied on it for several projects for both myself and for clients. Initially I thought I would be using wordpress, but after further consideration it started to seem like it wasn't the right fit. 

My first requirement that it had to be easy to use. When I say "easy to use", I mean so in every sense; installing it, maintaining it, updating it, writing entries for it, hacking it up and customizing it to whatever I need, and so on. I have enough things that require effort and attention in my life, AdrianArtiles.com doesn't need to be one of them.

Now wordpress is not difficult to use. I've installed it countless times on vps's, amazon instances, shared hosting, heroku, etc., etc.. Authoring a new post only takes a few minutes. Assuming you know PHP, altering a theme, plugin, or anything really only takes a minimal amount of fiddling.

But for my needs it seemed a little heavy. First of all, I didn't want to deal with a database. I wanted my new site to be as simple and light weight as possible, and I felt there was something out there lighter than wordpress that can do what I wanted.

Now if I just wanted a care-free, "no frills" blogging solution, I could do that anywhere. I could easily turn to tumblr and be done with it, but I also wanted to be able to hack it up and customize it however I wanted, control the experience as much or as little as I wanted to, or maybe repurpose a part of the site for something else.

So the solution would have to be light-weight and flexible. Bonus points for using a technology that I like working with that helps me achieve this. Something in ruby perhaps?

With these requirements I set out to see what best matched the bill.

### enter octopress ###

I heard about [Jekyll](https://github.com/mojombo/jekyll) before, but at the time I didn't give it much thought. "Why would I want to work with something so bare-bones when there are so many other full featured and customizable options" I asked outloudly to my computer screen. I laughed to myself and installed wordpress for the umpteenth time. 

Fast forward to today and my quest for my new framework. I came across [Octopress](http://octopress.org/), which is built on top of jekyll. It was polished, feature-filled, easy to use (if you're a developer of any sort), and lightweight.

What I loved about Octopress/Jekyll was the overhead. It was just files. No database, and I get everything I want. I just write the posts that I want locally with any text editor (currently vim, with the very occasional sublime text), `rake generate`, and I have the files that are my site! Now I can take those files and host them anywhere and I have my site. So I can put it on an amazon instance, heroku, appfog, a laptop connected to the internet in a basement, whatever, anything that can host a few files. No database to configure, maintain, or worry myself with.

Since it's written in ruby, if I wanted to get in there and change something up I easily could. Most of the configuration I wanted could be achieved with plugins, themes, and old fashioned css and javascript. Now I could SEO, tweak, and design to my heart's content, which so far as turned into this theme (which I made open sourced and available via [github](https://github.com/sevenadrian/foxslide)).

Besides being able to use whatever text editor I like, I'd be writing in [markdown](http://daringfireball.net/projects/markdown/). Markdown is an easy to use "shorthand" for writing for websites. It makes creating links, headers, quotes, lists, those kind of things, very easy to do.

Another advantage is *version control*. If you are a software developer and you don't use version control, you are a danger to yourself and to others around you. For those who use version control, the advantages in using it for a blog should be obvious. For those not familiar with version control, take a look at a very popular distributed version control called [git](http://git-scm.com/). Basically it allows you to save a "copy" of your site at every step of the way, and giving you the ability to see changes over time and revert to a specific point at any time. Use a hosted git solution like [github](http://github.com) and never have to worry about losing any part of your website again.

So far I'm happy with the decision I've made. I got everything I was looking for in the package I want. Now let's hope I write often enough to take advantage of it.