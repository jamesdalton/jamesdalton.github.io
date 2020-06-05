---
layout: post
title: "New Home and New Platform"
description: "Why I moved from GoDaddy and WordPress to Jekyll and GitHub Pages."
categories: Jekyll
excerpt_separator: <!--more-->
---

My site also has a new URL. It is now <https://blog.james-dalton.net>. If you bookmarked <http://james-dalton.net> you will want to update your bookmark. For now, it redirects here. I changed the URL so I could use this domain for more than just this blog. Not that I have any plans to do that yet, but at least now I can. I also pick up HTTPS for free from GitHub Pages.

<!--more-->

## Why the Move

Let me start by saying this move was **not** prompted by any support or technical issues with GoDaddy or WordPress. I still have my domain registered through GoDaddy and my new e-mail address for the blog (blog@james-dalton.net) is through them as well.

I was, however, frustrated with WordPress. I know I just said I didn't move for technical reasons. WordPress is not a bad platform, just overkill for what I wanted. Initially I thought it would make a great platform for my blog. I learned a lot about WordPress and had fun tinkering with it initially. 

But I **really** like writing in Markdown. If I could get away with writing my thesis in Markdown I would. WordPress would let me do use Markdown, but there was always a battle to get the formatting right when I was ready to publish. I liked writing local and the copy/paste the blog into WordPress. That never worked well for me. I tried writing online, but that didn't work for me either.

I have another post I wrote about a year ago that I never published. Every time I thought about it, I would put if off because I didn't want to fight WordPress to get it formatted right.

## The Search for a new Platform

I started searching about a year ago. I first looked at WordPress competitors, but they were all more than I needed. The ones I looked at were great at what they did, including WordPress. What they did wasn't what I wanted.

Then I remembered a coworker talking about static site generators and GitHub Pages. I wasn't sure how that was going to work. How can you create a blog with a static generator? How would comments work? So, I asked google.

Several hits where top (n) lists. There were a few that made the top of multiple lists. I decided to investigate Jekyll, Hugo and another that I can't remember the name. I installed them and played around with them. I wanted something that was easy to use, easy to style and allowed me to work the way I wanted.

At the same time, I was looking into hosting options. Azure was a possibility; I've worked with it before. What I wanted was a low friction hosting option. I didn't want to have to worry about patching or keeping servers running. While I could do that with Azure, the more I looked at GitHub Pages the better it looked. GitHub Pages was easy to set up, supported custom domains, publishing with Jekyll was as easy as `git push` and is free. :smile:

## Getting Started with Jekyll and GitHub Pages

Jekyll's [Step by Step Tutorial](https://jekyllrb.com/docs/step-by-step/01-setup/) was a great guide. I still go back to it when I have questions. After the tutorial I wanted to work on styling. I didn't want it to look the same as the WordPress blog, but close. I started with the Jekyll default theme, Minima. The sass layout and the simple file layout made it easy to get the blog looking the way I wanted. And then summer school started, and I didn't have time to work with it anymore :disappointed:.

I tried to work with it a little during breaks but didn't get much done until recently. The nice thing about Jekyll is it is so easy to use I had no problems picking up where left off even after not using it for almost a year. The motivation for getting it up and running came when I realized my GoDaddy WordPress subscription would automatically renew at the end of the month.

I finished the styling and copied my posts over. I only had three, so it was easy to do by hand. I wrote all but the first using Markdown, so it was easy to reformat them for Jekyll. The hard part where the code blocks and syntax highlighting. I wanted to keep line numbers, but the built in version with Rouge didn't do what I wanted so I used [linedivs plugin](https://www.bytedude.com/jekyll-syntax-highlighting-and-line-numbers/). A little clean up and everything looked great. Time to set up hosting on GitHub Pages.

And this is where I encountered my first disappointment. GitHub has a [whitelist of plugins allowed](https://pages.github.com/versions/) and linedivs isn't one of them:disappointed:. I had done too much at this point to change my mind. I liked Jekyll and the ease of hosting on GitHub Pages enough that I could do without line numbers. But I hated Minima's default styling for code blocks. I wanted it to look like VS Code's preview pane but was struggling to get it right. I liked the way [Phil Haack's](https://haacked.com/) code blocks looked so I checked out his sass files and finally was able to get it looking the way I wanted. The best part is I can use backtick for my code blocks:grinning:.

I looked at the plugins that are available on GitHub Pages and decided to only actively use four, jekyll-feed, jekyll-seo-tag, jekyll-sitemap, jemoji. I didn't have to do anything for jekyll-feed. I did need to add my Google tracking ID to the `_config.yml` file to enable google analytics for jekyll-seo-tag. I didn't have to do anything for jekyll-sitemap. I was going to pass on jemoji but found myself wanting to use it as I typed this. I will add in jekyll-paginate once I have more than 10 posts. Which at this rate might be a few years:smirk:. I may add avatar, gist and mentions later.

Now it was time to setup GitHub Pages. [Jonathan McGlone](http://jmcglone.com/guides/github-pages/) has a great guide for setting it up along with GitHub's [guide](https://help.github.com/en/github/working-with-github-pages/getting-started-with-github-pages). [Setting up the custom domain](https://help.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site) was easy. It didn't take long for <http://blog.james-dalton.net> to work, but I had to wait until the next day to enable HTTPS. The next step was to set up forwarding for <http://james-dalton.net> and <http://www.james-dalton.net>. After verifying the forwarding, the last step was to stop my WordPress subscription before it renewed.

If you want to check out the source behind the blog, go to <https://github.com/jamesdalton/jamesdalton.github.io>

## Next

At some point I want to add comments, I like the way [Phil Haack](https://haacked.com/archive/2018/06/24/comments-for-jekyll-blogs/) did it. Azure functions are mostly free for low volume and it would be interesting to do.

I'm hoping that having a low friction way to write and publish blogs will encourage me to keep at it. Other than work, my summer is open so I should get some time to blog. I have one ready to go and ideas for a couple more. Once school starts back up, I may blog about my thesis. It's going to be on encryption so the blog will take a heavy math lean for a while. I'll need to figure out how to do LaTex in Jekyll/Markdown.
