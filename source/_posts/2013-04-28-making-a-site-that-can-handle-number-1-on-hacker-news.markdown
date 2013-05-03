---
layout: post
title: "Making a site that can handle #1 on Hacker News"
date: 2013-04-28 14:16
comments: true
categories: 
---

I recently launched my new consulting site, [AppRaptor](http://www.appraptor.com). I was completely surprised to see it vault all the way to #1 on Hacker News and stay there for a couple hours. 

The unfortunate thing was my server couldn't handle all of that traffic. At the height of the surge, I was seeing 287 concurrent users hitting my site. 

I learned a lot about what not to do when developing a site which doesn't require dynamic content.

My big mistake was I wasn't expecting more than a handful of upvotes on HN so I just put the site on my Linode 512 (512 Mb of RAM). I usually do most of my development in Ruby and Rails and deploy to Heroku but I was lazy and already had a linode setup and configured for PHP. What made the performance infinitely worse was I was using PHP to factor out common code to ease the development process. This site didn't have any dynamic content and shouldn't have used a server side language at all. A massive performance hit is an unacceptable trade off for ease of development (turns out you can have your cake and eat it too with middleman app. I talk about this below).

During the surge I should have immediately converted the site to static HTML. Instead, I wasted my time trying to optimize the max_clients and the timeout variables in apache.conf. Messing with apache didn't help with 287 concurrents.

There were a lot of good suggestions on HN for improving the performance during the surge. Thanks to HN users [krallin](https://news.ycombinator.com/user?id=krallin) and [bradgessler](https://news.ycombinator.com/user?id=bradgessler) for pointing me to S3, Cloudfront, and Middleman for static site hosting and development workflow. 

I'm now hosting [AppRaptor](http://www.appraptor) with an Amazon S3 Bucket and Cloudfront as a CDN. Just for the sake of amusement I did a brief loadtest with blitz.io to see what the performance would be like with this new stack. I did a rush from 1-250 users in 60 seconds and it had 0 errors and 0 timeouts. Screenshot of the test is below:

![blitz test](/images/blitz.png "blitz load test")

Setting up S3 for static site hosting with Cloudfront was pretty simple. The only gotcha you should be aware of is the bucket will initially interpret your css style sheets as an octet/stream data type, and consequently your design won't show up. You just need to manually go into the properties of those css files and change them to "text/css".

The biggest pain about developing a static site, however, is having to repeat yourself with shared code (headers, footer, etc...). Turns out a great solution for this problem is [Middleman](http://middlemanapp.com/). With Middleman, you can use all of your favorite dsl's, like HAML and SASS. It even allows you to build your site with embedded ruby. When you're ready to deploy your site to your S3 bucket, just type "middleman build" and it creates static html for your project. Needless to say, it's pretty sweet for optimizing your workflow.

Of course, this stack will not work if you have a site that requires dynamic content. But if you want to survive a massive HN surge with a static site, I highly recommend this approach.

Thanks to HN for all of the great comments and critique on my new consulting site, [AppRaptor](http://www.appraptor.com)!  

