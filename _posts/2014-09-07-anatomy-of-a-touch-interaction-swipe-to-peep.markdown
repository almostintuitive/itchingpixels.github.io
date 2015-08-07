---
layout: post
title:  "Anatomy of a touch interaction: Swipe-to-peep"
date:   2014-09-07
categories: iOS design
---
You open up your messaging app.
Question: Why can’t you just "peep" into the thread by swiping on a conversation cell like this?

![](/_images/swipe-to-peep/1.gif)



Today’s state of interaction design seems a bit swipe-obsessive, right?
But really, why do people love to cuddle their phone?
Does swiping really make touch interactions seem more effortless?

Mailbox, the iOS app I’m using stopped working for a few days for me, so I went back to the Gmail app, and I was like, how can people use this? It felt like flicking through my emails took me twice as long.
Which was clearly not true. Otherwise I would have just abandoned emails. Mailbox helps with dealing with obvious spam, or emails that you expect to receive, but not the ones that require any examination - then you’re still tapping on the email, it slides to another view, then you see that ohh it’s just your aunt asking you about whether you can make it next month to a large family gathering, which you promptly delete so you can pretend that you’ve never received it.

Okay now I’m curious, let’s have a look at some chat apps.
You’ve got some arbitrary actions if you swipe on a conversation:

![](/_images/swipe-to-peep/2.png)

Seems kind of useless, there are other ways to delete a thread if you want to.


Is this time to do a prototype?
Hell yeah!

For the ones who’re interested in the technical details, we’re creating a UITableViewCell that if you start dragging (just add a UIPanGestureRecognizer), communicates with the main ViewController through a delegate.
We'll trigger the final transition (full-screen content view) if the drag movement ended on the left side of the screen.

<script src="https://gist.github.com/itchingpixels/0091c5e5d36aff914384.js"></script>
    
Now let’s just create a a second view and move it (its left side) along the path you leave with your finger.

<script src="https://gist.github.com/itchingpixels/5a6bbcc9849ae7fce7b4.js"></script>

![](/content/images/2014/Aug/swipetopeep-3.gif)


Easy.

This is not exactly how the Mailbox swipe interaction works, but if you just grab the tile from the current position, you’ll never see enough of the content to make "peeping" work.

Okay, I found a problem though:
If you just swipe on a TableView Cell, it just jumps back, we don't trigger the new screen.

![](/content/images/2014/Aug/swipetopeep-4.gif)


Solution: Let's check the horizontal velocity (the speed) of the swipe and base the decision also on that:

<script src="https://gist.github.com/itchingpixels/8237d8425fe9e035c928.js"></script>


Okay, let's come back to this a bit later. What happens with the visual transition? It feels a clunky.
What if we move the original screen to the left at the same time?

{<5>}![](/content/images/2014/Aug/swipetopeep-5.gif)

<script src="https://gist.github.com/itchingpixels/ed89a0e52410da8ea4ae.js"></script>

Cool, makes a bit more sense.
What if we do a bit of a “parallax” movement with the original screen? Also, fade out the original?

![](/content/images/2014/Aug/swipetopeep-6.gif)

<script src="https://gist.github.com/itchingpixels/b55acae4f1aa0bb92bf1.js"></script>

Okay, fine. But, in general, scrolling on the conversation list is crap. It's a bit hard to illustrate with only a few;), but the "content screen" will suddenly start jumping in when you slightly move to the right:

![](/content/images/2014/Aug/swipetopeep-7.gif)

Let’s disable the “swipability” on the cell when the vertical acceleration (scrolling on the table) is larger than the horizontal acceleration (swiping to reveal the content)

![](/content/images/2014/Aug/swipetopeep-8.gif)

<script src="https://gist.github.com/itchingpixels/312355bcb6aada3475ae.js"></script>


But now, we have to disable scrolling on the main screen’s tableview, otherwise you’ll get this weird effect:



![](/content/images/2014/Aug/swipetopeep-9-1.gif)

<script src="https://gist.github.com/itchingpixels/06a6a82a4a225748c1a2.js"></script>


Okay, but you can still scroll to the other direction (right), and have a view popping where your finger is. Really confusing.
Let’s not do that.

![](/content/images/2014/Aug/swipetopeep-10.gif)

<script src="https://gist.github.com/itchingpixels/75685ce5a7cc9f5984d9.js"></script>

Now that we’re here, we could try out pop by Facebook (an awesome animation framework). It’s awesome, and with [MCAnimate+POP](https://github.com/matthewcheok/POP-MCAnimate), the syntax is as concise as it can get. With one extra keypath addition, you can do fancy stuff!
Just type in ".spring" before whatever you want to animate.

![](/content/images/2014/Aug/swipetopeep-11.gif)

<script src="https://gist.github.com/itchingpixels/579c37decd0e95e54247.js"></script>

Smooth!

The problem now is that you have no feedback on the cell what you’ve selected...
Let’s animate the background color of cell you're swiping on in a hacky way!

![](/content/images/2014/Aug/swipetopeep-12.gif)

<script src="https://gist.github.com/itchingpixels/8ab9a3fcbc2531803fa3.js"></script>

This probably makes more sense without the other animation, that fades out the original view completely. But it's something!

Ready for primetime?

Probably not.

If you're intrigued, download the prototype from [github](http://github.com/itchingpixels/SwipeToPeep) and try it out yourself!


*Don’t ever use any of the hacky solutions I’ve used here in your production code, or Jonathan Ive will come around and redesign your face into an aluminium ball with a blinking light.
You need the UIViewControllerTransition API and all that fancy stuff.
It’s fine, you can do it. I know it’s confusing, but good people wrote [wrappers](https://github.com/stepanhruda/STPTransitions) [around](https://github.com/Raizlabs/RZTransitions) [it](https://github.com/itsmeichigo/ICGTransitionAnimation). Minus one horrible API you need to learn!*


There's a [branch](https://github.com/itchingpixels/SwipeToPeep/tree/hackernews) in the repository that has a working HackerNews client with the same interaction - the only problem is that downloading & rendering a web page takes around 5 seconds on your phone, so "peeping" is not that instant, which makes it kind of useless. Tip from [Peter](https://twitter.com/spacetimetours): render it on the server side!

What else would you use this for?
I have nothing in the pipeline, so please, steal if if you want. And ping me on [twitter](http://twitter.com/itchingpixels)!

(Yeah I have no intention writing another messaging app. Please publicly execute me if I do.)