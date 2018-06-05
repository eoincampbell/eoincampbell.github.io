---
layout: post
title: Migrating from Wordpress to Jekyll
description: A simple approach to migrating a wordpress blog to Jekyll and Github Pages.
tags: 
- jekyll 
- wordpress 
- blog 
- linux 
- wsl 
- github
- ubuntu
- ruby
categories: 
- coding
- random
published: true
---

Recently I had the displeasure of Wordpress screwing up on me yet again. I've been paying a small but measurable amount to a colo based in Ireland for about the past 8 years to provide a small VPS running wordpress. At the time I set it up it was great. But times have changes. The VPS is underpowered. I don't have the need for a remote windows box anymore with Azure MPN/Dev subscriptions giving me all the free .NET hosting I need. And, Wordpress is a slow hunking mess. I really don't need a big kludge of a CMS running on MySQL for a personal blog with minimal traffic.

## Jekyll

After a little bit of research, I decided to give [Jekyll][jekyll] a try. Jekyll is a simple, blog-aware, static site generator built with Ruby. It also happens to be the engine behind [Github Pages][github-pages] as well. So you can setup your jekyll blog, upload it to your Github pages repo in your github account and Github will autogenerate your site for you.

## Ruby Environment

In order to build and run Jekyll locally, you'll need access to a ruby dev environment. The general consensus from my bit of research was to do this on a linux distro. Thankfully this is much easier for a windows nerd like me with the new Windows Subsystem for Linux. Simply enable the WSL from powershell and then install your preferred distro from the Microsoft Store. I decided to install Ubuntu.

```powershell
#Enable Windows Subsystem for Linux
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
#Reboot
``` 

Once installed I needed to run some package updates and upgrades. This will be slightly different for each distribution.

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install ruby-full build-essential 
gem install bundler jekyll
```

## Github Pages

Once you have `jekyll` and `bundle` installed, go ahead and create a github pages repo. I created mine at [eoincampbell.github.io][eoincampbell-github-io]. The name of this repo is the same as your github account. You can [browse the folder structure of my site here][eoincampbell-github-io-repo] to get a sense of the content/folder structure.

Next check out the empty repo to your local machine, and run the jekyll create command in it.

```bash
jekyll new your-site-name
bundle exec jekyll serve
```

There's tonnes on information Jekyll and how to [configure][jekyll-config] and [theme][jekyll-themes] it on their site.

## Migrating Wordpress Comments

Before shutting down my wordpress account, I wanted to retain the comment history for my old site. Jekyll has built in comment/discussion support using [Disqus][disqus].

You can export all your wordpress content by logging into your wordpress admin section, and running the **Export** tool under the tools menu. You should select to export all content including posts, pages and comments.

Next create an account on Disqus and login to the [Disqus Importer][disqus-importer]. Here you'll be able to upload your wordpress xml data dump. Comments are stored against your domain name, and post urls. That means so long as your new site has the same domain and same permalinks, your comments will just appear.

## Migrating Wordpress Posts

Finally, you'll want to convert your old wordpress blog posts into markdown files.

Thankfully this a ruby 1 liner. (though you may need to install the gem first)

```bash
ruby -rubygems -e 
    'require "jekyll/migrators/wordpressdotcom"; 
    Jekyll::WordpressDotCom.process("/path/to/wordpress.xml")'
```

## Gotchas

The whole process was pretty painless to be honest, but I did get caught out by a couple of issues, mostly due to the fact that it's a long time since I dabbled with Linux and it kinda has some sharp corners.

There's a good article [here on how to install ruby on ubuntu][wsl-ubuntu]. Originally I missed adding the system Gems' to my path which caused some headaches.

Another issue I ran into had to do with some dependencies after changing from the Default Jekyll gem to using the github pages version. It looked for a dependency called nokogiri which was missing a further dependency called Zlib, and I needed to install Zlib manuaully via apt-get.

Finally, although the jekyll markdown format supports html, you'll probably find that you'll need to do a pass of your old posts to clean them up a little and get the formatting just right.

Have fun

***~Eoin Campbell***

[jekyll]: https://jekyllrb.com/docs/home
[github-pages]: https://pages.github.com/
[disqus]: https://disqus.com/
[disqus-importer]: https://import.disqus.com/
[jekyll-themes]: https://jekyllrb.com/docs/themes/
[jekyll-config]: https://jekyllrb.com/docs/configuration/
[eoincampbell-github-io]: https://eoincampbell.github.io
[eoincampbell-github-io-repo]: https://github.com/eoincampbell/eoincampbell.github.io
[wsl-ubuntu]: https://www.digitalocean.com/community/tutorials/how-to-set-up-a-jekyll-development-site-on-ubuntu-16-04
