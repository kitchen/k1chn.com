---
title: "Web Hosting"
date: 2019-07-11T22:16:44-07:00
---

I've had a few people ask me about my hosting setup. How much it costs, what services I use, etc. So I thought I'd talk about that here.

My case is pretty special for a lot of reasons. I do this stuff for a living, so I am much more willing to DIY and play around with complex setups and such than some people might be. I'm also very picky about how things are set up, so I do some things that might be more difficult than they need to be, just because I want it done a very specific way.

## TL;DR
For the very very very short answer, the services I use are:

* DreamHost
* AWS
* G Suite

## Details

For this site in particular, k1chn.com, I actually don't use DreamHost, but I do use it for [my personal blog](https://words.kitchen.io/) and some other random things. The main reason I use DreamHost for anything is that I used to work there, and one of the perks of the job is "free hosting for life", so it makes sense to use it for a lot of stuff simply because it doesn't cost me anything :)

Additionally, I use G Suite for managing my email, in part because Google seems to have pretty much solved spam, and also because it doesn't cost me anything. At DreamHost we were an early integration partner for "Google Apps for your Domain", so of course I set it up on mine, and they've kept on those legacy accounts as free. It's pretty great. I would absolutely pay for it though if I needed to, the fact that I don't have to even think about email at all is pretty great.

As far as web hosting is concerned, I use DreamHost for my personal blog and it's running a WordPress installation. Pretty standard stuff. I have it set up on its own VPS and user account and have a dev version of the site also on its own VPN and user account (to limit splash damage if one of the sites gets compromised).

For this site I'm running everything out of an S3 bucket with web hosting enabled, and fronting it using CloudFront. CloudFront in front of an S3 bucket makes hosting a static website super easy and incredibly cheap, I think my estimated non-free-tier bill for the site is about $.50/mo, with another $.50/mo for Route53. Of course, in order to do that it needs to be all static website, which is fine, I'm using [hugo](https://gohugo.io/) as a static site generator and uploading the site to S3. It's pretty great. WordPress is awesome and I have used it for a long time, but I really love that my hugo site is entirely static and therefore very cheap to host and I don't have to worry about it getting compromised.

The DNS for my primary domains is hosted at Route53, in large part because it's cheap, easy, reliable, I can manage it programatically, etc, but also because it integrates nicely with some of the other AWS services I'm using.

As part of migrating k1chn.com from DreamHost over to AWS I decided to get everything set up in [terraform](https://www.terraform.io/). This is another tool I use a lot in my work, and it makes managing all of this stuff much easier. And, since [it's all in terraform](https://github.com/kitchen/personal-terraform) (or at least, I'm working on getting it all into terraform) I have a constant inventory of everything I have configured, which can make managing things easier if I need to remove some things. I can also group things logically rather than just by service, e.g. my k1chn.com terraform code is all in one file, the S3 bucket, CloudFront distribution, ACM Certificate, Route53 stuff. It's also good practice for work :)

So, given that I have free DreamHost and G Suite accounts, my monthly cost for all of this stuff is *incredibly* cheap. My AWS bill up to this point has been on the order of $1.05/mo, and with the extra Route53 zone for k1chn.com and S3 bucket and CloudFront stuff (though I think I'm on Free Tier for a good while anyways), it'll probably go up to around $2, maybe $3/mo. Plus the costs of registering domains, which run me about $100/year all told across all of the domains I own. I've pared down quite a few (I used to own ktc.hn, which was $100/year by itself), and I get some free via my DreamHost account, but I still have a good handful I'm holding on to for various reasons. I even own (my full name)[http://jeremy.kitchen/] which is pretty fun :)

## What should you use?

It really comes down to what you want. But I would start off with something easy and hosted first, like SquareSpace\*, Weebly, DreamHost\*, WordPress.com\*, or ghost.org and go from there. It may cost you more than my setup, but if you don't happen to work in the webhosting industry or feel like getting deep into the weeds on implementation details, it's worth the cost for convenience.

If you *do* want to implement any of these things, let me know and I'm willing to help you get started!

\*Full disclosure: I have friends who work for SquareSpace, Automattic (parent company of WordPress.com), DreamHost, and I've interviewed with, worked at, or will be interviewing with all 3.
