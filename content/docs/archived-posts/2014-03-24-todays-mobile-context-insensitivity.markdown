---
layout: post
title:  "2014 - Today’s mobile context-insensitivity"
date:   2014-03-25
---

# Today’s mobile context-insensitivity

> The supercomputers we’re carrying around don’t know what we are doing and what we are up to — and that’s a problem. 


I have an iPhone.
The technology inside is so advanced that I can’t even imagine — and there’s no chance I can actually understand it.
Its operating system has a built in physics engine, a database management system, OpenGL, and many awesome features and it would take me half an hour to list them all.
But… it can’t turn on the wi-fi automatically when I’m at home.
(There are a few reasons why you don’t want to have your wi-fi always on.)

Actually, this is a bad example. It could do so much more.
It could find me a flower shop nearby before my date tonight, it could 

Well, at least on Android, I have apps that could turn on my wi-fi when I get home.

How do they work?
You install “WiFi Auto-switch”, which subscribes to location monitoring.
Then you realize that you also want to turn on your smart thermostat when you get home. You install “Smart Thermostat Control”.

Ohh, you forget another one: turn off the lights. So you download “Smart Light Control”…

But, location monitoring in itself is meaningless — they also have to know where “home” is.
And yes, they'll ask you three times, because there's no central place where you can store this information.

But.. this why doesn’t your phone have an event API on the OS level that triggers specific apps when you leave the office, when you get on a train, before you leave for a holiday and when you’re about to have a date with your girlfriend? It could have access to your calendar, your location, your emails. So before your date, Yelp could check whether there’s a flower shop nearby and send you a notification if it found any, Gmail can automatically send an email to your Mum if you arrived safely to your destination, or whatever.

There are many apps that are doing these individually anyway, but there’s a gap — they can’t really interact with each other and therefore become really useful.
On Android, this could happen tomorrow — as Google Now already knows when you are near to a bus station, have a flight to catch, etc. You just need to access that information as an app and build services upon it. And Apple bought the personal assitant/smart calendar startup Cue in 2013...

We also have IFTTT, which could easily fill the gap — if the design of iOS would let it. Unfortunately, the way of it deals with the problem feels more like a workaround than a solution, and I’m not blaming them, that’s all they can do right now on mobile. Also, the fact that you have to find and set up individual rules one by one, like “leaving home” is also greatly limiting the possibilities of how app developers can make your phone a lot more useful.
With the coming of wearables, this is gonna be even more relevant — they’re all about context-sensitivity, aren’t they?

These devices will be all about displaying the content you need, at the right time, by detecting where you are and what you’re up to.

Without an API like this, you’ll see only a[ really few apps](http://recode.net/2014/03/17/three-ways-to-maximize-your-possibly-disappointing-wearables/) that are actually useful. 

Do you have privacy concerns? Fair enough. I have privacy concerns. You need annoying popups to ask you whether you want to give this and that kind of access to the app. Which is fine.
Or! We can have an AI that would predict which apps you want to give full access to. That would be awesome!

*I wrote this 6 months ago, and it does seem like this is getting fixed by a lot of companies. For example, [Moves](https://dev.moves-app.com/) now has an API that does something similar. iOS8 clearly doesn't do much about it, except on the fitness/health side. [Sunrise](https://dev.moves-app.com/), [Gmail](https://developers.google.com/gmail/api/) also started working on an API that would enable devs to build context-sensitive apps. It's still not one brain that has all the information, but it's getting easier to put together the pieces. Hurray!*