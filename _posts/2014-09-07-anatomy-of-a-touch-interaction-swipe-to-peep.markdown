---
layout: post
title:  "Anatomy of a touch interaction: Swipe-to-peep"
date:   2014-09-07
categories: iOS design
---
You open up your messaging app.
Question: Why can’t you just "peep" into the thread by swiping on a conversation cell like this?

![](../content/images/swipe-to-peep/1.gif)



Today’s state of interaction design seems a bit swipe-obsessive, right?
But really, why do people love to cuddle their phone?
Does swiping really make touch interactions seem more effortless?

Mailbox, the iOS app I’m using stopped working for a few days for me, so I went back to the Gmail app, and I was like, how can people use this? It felt like flicking through my emails took me twice as long.
Which was clearly not true. Otherwise I would have just abandoned emails. Mailbox helps with dealing with obvious spam, or emails that you expect to receive, but not the ones that require any examination - then you’re still tapping on the email, it slides to another view, then you see that ohh it’s just your aunt asking you about whether you can make it next month to a large family gathering, which you promptly delete so you can pretend that you’ve never received it.

Okay now I’m curious, let’s have a look at some chat apps.
You’ve got some arbitrary actions if you swipe on a conversation:

![](../content/images/swipe-to-peep/2.png)

Seems kind of useless, there are other ways to delete a thread if you want to.


Is this time to do a prototype?
Hell yeah!

For the ones who’re interested in the technical details, we’re creating a UITableViewCell that if you start dragging (just add a UIPanGestureRecognizer), communicates with the main ViewController through a delegate.
We'll trigger the final transition (full-screen content view) if the drag movement ended on the left side of the screen.


```
if (gestureRecognizer.state == UIGestureRecognizerStateBegan) {
  [self.delegate swipeableCellDidStartSwiping:self];
} else if (gestureRecognizer.state == UIGestureRecognizerStateChanged) {
  [self.delegate swipeableCell:self didSwipeWithHorizontalPosition:touchLocation.x progress:progress];
} else if (gestureRecognizer.state == UIGestureRecognizerStateEnded) {
  if (progress >= 0.7) {
    [self.delegate swipeableCellCompletedSwiping:self];
  } else {
    [self.delegate swipeableCellCancelledSwiping:self];
  }
}
```

Now let’s just create a a second view and move it (its left side) along the path you leave with your finger.

```
- (void)adjustViewBasedOnSwipeProgress:(float)progress {
    self.postWebView.center = CGPointMake(self.view.center.x+(self.view.bounds.size.width*progress), self.view.center.y);
}
```

![](../content/images/swipe-to-peep/3.gif)


Easy.

This is not exactly how the Mailbox swipe interaction works, but if you just grab the tile from the current position, you’ll never see enough of the content to make "peeping" work.

Okay, I found a problem though:
If you just swipe on a TableView Cell, it just jumps back, we don't trigger the new screen.

![](../content/images/swipe-to-peep/4.gif)


Solution: Let's check the horizontal velocity (the speed) of the swipe and base the decision also on that:

```
 else if (gestureRecognizer.state == UIGestureRecognizerStateEnded) {
        if (progress >= 0.7 || touchVelocity.x < -300) {
            [self.delegate swipeableCellCompletedSwiping:self];
        } else {
            [self.delegate swipeableCellCancelledSwiping:self];
        }
    }
```


Okay, let's come back to this a bit later. What happens with the visual transition? It feels a clunky.
What if we move the original screen to the left at the same time?

![](../content/images/swipe-to-peep/5.gif)

```
- (void)adjustViewBasedOnSwipeProgress:(float)progress {
    self.tableView.center = CGPointMake(self.view.center.x-(self.view.bounds.size.width*(1-progress)), self.view.center.y);
    self.postWebView.center = CGPointMake(self.view.center.x+(self.view.bounds.size.width*progress), self.view.center.y);
}
```

Cool, makes a bit more sense.
What if we do a bit of a “parallax” movement with the original screen? Also, fade out the original?

![](../content/images/swipe-to-peep/6.gif)

```
- (void)adjustViewBasedOnSwipeProgress:(float)progress {
    self.tableView.alpha = progress;
    self.tableView.center = CGPointMake(self.view.center.x*progress, self.view.center.y);
    self.postWebView.center = CGPointMake(self.view.center.x+(self.view.bounds.size.width*progress), self.view.center.y);
}
```


Okay, fine. But, in general, scrolling on the conversation list is crap. It's a bit hard to illustrate with only a few;), but the "content screen" will suddenly start jumping in when you slightly move to the right:

![](../content/images/swipe-to-peep/7.gif)

Let’s disable the “swipability” on the cell when the vertical acceleration (scrolling on the table) is larger than the horizontal acceleration (swiping to reveal the content)

![](../content/images/swipe-to-peep/8.gif)

```
- (BOOL)gestureRecognizerShouldBegin:(UIPanGestureRecognizer *)gestureRecognizer {
    if ([gestureRecognizer class] == [UIPanGestureRecognizer class]) {
        CGPoint velocity = [gestureRecognizer velocityInView:nil];
        if (fabsf(velocity.x) > fabsf(velocity.y) ) {
            return YES;
        }
        return NO;
    }
    return YES;
}
```

But now, we have to disable scrolling on the main screen’s tableview, otherwise you’ll get this weird effect:



![](../content/images/swipe-to-peep/9.gif)


```
- (void)swipeableCell:(SwipeToPeepCell *)cell didSwipeWithHorizontalPosition:(CGFloat)horizontalPosition progress:(float)progress {
    self.tableView.scrollEnabled = NO;
    [self adjustViewBasedOnSwipeProgress:(1-progress)];
}
```


Okay, but you can still scroll to the other direction (right), and have a view popping where your finger is. Really confusing.
Let’s not do that.

![](../content/images/swipe-to-peep/10.gif)


```
if (velocity.x > 0) {
 	return NO;
 }
```


Now that we’re here, we could try out pop by Facebook (an awesome animation framework). It’s awesome, and with [MCAnimate+POP](https://github.com/matthewcheok/POP-MCAnimate), the syntax is as concise as it can get. With one extra keypath addition, you can do fancy stuff!
Just type in ".spring" before whatever you want to animate.

![](../content/images/swipe-to-peep/11.gif)


```
- (void)adjustViewBasedOnSwipeProgress:(float)progress {
    self.tableView.spring.alpha = progress;
    self.tableView.spring.center = CGPointMake(self.view.center.x*progress, self.view.center.y);
    self.postWebView.spring.center = CGPointMake(self.view.center.x+(self.view.bounds.size.width*progress), self.view.center.y);
}
```

Smooth!

The problem now is that you have no feedback on the cell what you’ve selected...
Let’s animate the background color of cell you're swiping on in a hacky way!

![](../content/images/swipe-to-peep/12.gif)


```
- (void)changeBackgroundColorBasedOnProgress:(float)progress {
    self.interactiveBackground.alpha = progress;
}


- (void)didMoveToSuperview {
    self.interactiveBackground = [[UIView alloc] initWithFrame:CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width,  self.bounds.size.height)];
    self.interactiveBackground.backgroundColor = [UIColor flatGrayColor];
    self.interactiveBackground.alpha = 0;
    [self insertSubview:self.interactiveBackground atIndex:0];
}
```


This probably makes more sense without the other animation, that fades out the original view completely. But it's something!

Ready for primetime?

Probably not.

If you're intrigued, download the prototype from [github](https://github.com/itchingpixels/swipe-to-peep) and try it out yourself!


*Don’t ever use any of the hacky solutions I’ve used here in your production code, or Jonathan Ive will come around and redesign your face into an aluminium ball with a blinking light.
You need the UIViewControllerTransition API and all that fancy stuff.
It’s fine, you can do it. I know it’s confusing, but good people wrote [wrappers](https://github.com/stepanhruda/STPTransitions) [around](https://github.com/Raizlabs/RZTransitions) [it](https://github.com/itsmeichigo/ICGTransitionAnimation). Minus one horrible API you need to learn!*


There's a [branch](https://github.com/itchingpixels/SwipeToPeep/tree/hackernews) in the repository that has a working HackerNews client with the same interaction - the only problem is that downloading & rendering a web page takes around 5 seconds on your phone, so "peeping" is not that instant, which makes it kind of useless. Tip from [Peter](https://twitter.com/spacetimetours): render it on the server side!

What else would you use this for?
I have nothing in the pipeline, so please, steal if if you want. And ping me on [twitter](http://twitter.com/itchingpixels)!

(Yeah I have no intention writing another messaging app. Please publicly execute me if I do.)