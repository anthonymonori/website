---
title: "Dropbox hackathon in London"
layout: post
date: 2016-12-24 16:20
headerImage: true
tag:
- hackathon
blog: true
image: /assets/images/posts/dropbox-hackathon-15/thumb-dbx.png
author: anthonymonori
description: My short summary of what happened at the very first Dropbox London Hackathon in January 2015.
---

_Please note that this blog-post was originally posted on 2015-08-22 11:18:20 on my previous blog._

___

It has been 7 months and I still haven't wrote the promised follow-up about the Dropbox London Hackathon from January 2015. No more excuses!

If you want to look at pictures or tweets circulating around the event, take a look at the following hashtag on Twitter: [#dbxhackathon](https://twitter.com/hashtag/dbxhackathon)

So, how did it start and what happened at this hackathon?

## Intro

First of all, I want to say a big thanks to [Leah Culver](https://www.twitter.com/leahculver), Developer Advocate at Dropbox, for being so helpful over email prior to the event! It was my first time in London, as I was flying out of Denmark to attend the event. I did not regret a moment of it!

I have immediately fell in love with the city, and got to a nice place with a rooftop view in Hackney, London. Hosted by Katy and Ricardo, a really cute couple with whom I had many great conversations over the days. I stayed in London from the 23th until the 26th of January. Check out their place on ~~[Airbnb](https://www.airbnb.com/rooms/265473)~~ for more info.

As the event started, I forgot to bring a UK-specific adapter to my DK-specific power cord! Big mistake! - but again, the Dropbox team and the staff at the hackathon was very helpful to save my ass - literally! :-)

## The idea

We came up with an idea with three other great guys ([Tim](https://twitter.com/tgpc), [Marvin](https://twitter.com/h_marvin) and [Atilla](https://twitter.com/atican)). It was about having a reporting tool integrated with Dropbox's API that we all felt was missing. All the information you can see (back then, at least) on the client application, without visiting their website, was the percentage of used space out of the total available in your Dropbox account, - which to be honest, is not much for the most of us. We are always struggling to clean up **unused, duplicate and huge files**, just to make sure we can save those _really important_ documents when needed :-).

This is where the concept of a better, more visual tool to overview your **Dropbox usage**, - was born.

## The design

![Initial design](/assets/images/posts/dropbox-hackathon-15/dbxhackathon-1.png)

Attila made a good job visualizing **a lot** of information and numbers; I think the above image is self-descriptive enough. We also considered the branding of our hackathon app and created a simple logo for our project that we now codenamed: **Hackathon-Manifest**.

![The logo](/assets/images/posts/dropbox-hackathon-15/dbxhackathon-logo.png)

_Get it? It's a manifest file in a box, uhhm... Dropbox_.

This web-like interface would be lite enough to run as a web-application on your phone or in your local browser, but also detailed enough to give you enough information to know what to delete or clean up.

## The backend

For the backend, we wanted a thin client that can analyse a folder locally, create a .json file out of the results in a set format and send it to a folder (e.g. your private Dropbox account) where the web-app can access it as well.

For this, we choose to create a Java application which, - with the help of some shell hacks and command line tricks, - does the work for us. The source code is available [here](https://github.com/anthonymonori/dropboxhackathon-backend).

## The frontend

The frontend was running on a Node.js server using Angular.js to bind the .json file to the view and to have better abstractions offered by the framework. We hosted it on our private Azure hosting for the duration of the event. After converting the design into an actual prototype using Foundation and some extra libraries for counting up numbers, we have added a login page and tapped into the Dropbox API. We had the following minimal viable product by this time:

![End results](/assets/images/posts/dropbox-hackathon-15/dbxhackathon-web.gif)

That is basically what we have done. To be fair, that demo does not have any integration between the backend and frontend; as it is now, they are working separately. We simply did not had the time to finish the integration due to some bugs, but we made the source code available on GitHub for the web-app too, over [here](https://github.com/anthonymonori/dropboxhackathon-web).

## The hackathon

Clearly, the London hackathon wasn't the first Dropbox-organized event nor hackathon either, but it was their first European hackathon and it did lack behind some of the experiences I have had from previous hackathons and startup weekends.

![End results](/assets/images/posts/dropbox-hackathon-15/dbxhackathon-shoreditch-studios.png)

The hackathon took place at this really cozy studio called Shoreditch Studios in the vibrant quarter of Shoreditch, London, which generally houses many kinds of events: fashion shows, receptions, etc. The food provided all weekend long was great and the staff did take good care of us. Security was up too, ensuring that nobody would enter who was not on the list.

One point I want to emphasize on and felt it was missing from the event, is that they did not make a spot in the schedule, where teams/people could share what they were working on, so nobody would work on the same thing over the weekend. Thankfully, during the hackathon, this did not cause any issues, - but I could easily see that happening. The lenght of the hackathon wasn't really 48 or 36 hours either, but more like 12 hours in total, which gave us a hard time to complete anything, really. If anybody from the staff is reading this: please take this as a constructive feedback. I did had a great time and departed with only good thoughts. Kudos!

___

In overall, I had a fun weekend in London, exploring the city in the first and last days there; meeting some very cool and talented people over at the hackathon and having great discussion with my Airbnb hosts about the demographics of London and the life in the city! Thank for all for making this weekend a **BLAST**!