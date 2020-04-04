---
layout: post
title:  "2016 - Functional Reactive Intuition - Swift edition"
date:   2016-01-30
---

# Functional Reactive Intuition - Swift edition

So, you’ve heard about reactive programming.
Then you got discouraged, almost immediately.

Like: is this guy really talking about switching on the observable of observables? I have no idea what's going on.

That’s all fine, we agree on one thing: people usually try to explain functional programming in a way that only makes sense to people who *already know functional* programming.

Let me show you something, maybe it can help with your appetite.

Have you heard of pipes? streams? signals? observables? eventemitters?
Let’s imagine they’re all the same: channels that emit events, over time.

And just in this moment, your boss interrupts you.
She asks you to implement a new feature in the app. She gives you this brief:

### *When the user starts simultaneously panning and rotating an object, start a countdown from 3. Stop the timer either when the countdown ends or when the user stops the gestures.*



This is gonna be a mission critical part of the app. There’s no way you can make a mistake here.

So, as an imperative programmer, you start thinking about what do you need to keep track of in order to implement this:
Let’s see:

*"user starts simultaneously panning and rotating an object"* - okay, I need to store this in a variable.
I mean, otherwise how else can I check if both of them happening at the same time?

*“Stop the timer either when the countdown ends”* - means I’ll need to keep track of the timer. And also, the number of seconds left.

Etc, etc.

So, you get the idea: we’re building a kind-of data structure describing what’s exactly happening at the screen, in this specific moment, so we can compare the old state with the new state - this is how we're building our control-flow.
With *ifs*.


{% highlight swift %}

var panPresent = false
var pinchPresent = false
var gestureTimer: NSTimer?
var secondsLeft = 3

func handlePan(panGesture: UIPanGestureRecognizer) {
  if panGesture.state == .Began && self.panPresent == false {
    self.panPresent = true
    self.checkIfBothGesturesPresent()
  } else if panGesture.state == .Ended {
    self.panPresent = false
    self.stopTimerIfNeeded()
  }
}

func handlePinch(pinchGesture: UIPinchGestureRecognizer) {
  if pinchGesture.state == .Began && self.pinchPresent == false {
    self.pinchPresent = true
    self.checkIfBothGesturesPresent()
  } else if pinchGesture.state == .Ended {
    self.pinchPresent = false
    self.stopTimerIfNeeded()
  }
}

func checkIfBothGesturesPresent() {
  if self.pinchPresent == true && self.panPresent == true && self.gestureTimer == nil {
    self.secondsLeft = 3
    self.gestureTimer = NSTimer.scheduledTimerWithTimeInterval(1, target: self, selector: “tick:”...
    print("started")
  }
}

func stopTimerIfNeeded() {
  if let gestureTimer = gestureTimer {
    gestureTimer.invalidate()
    self.gestureTimer = nil
    print("completed")
  }
}

func tick(timer: NSTimer) {
  if self.secondsLeft <= 0 {
    self.stopTimerIfNeeded()
    return
  }
  self.secondsLeft--
  print("tick")
}
{% endhighlight %}


So, we turned our original sentence into this:

{% highlight swift %}

- When the user touches the screen
  - Check if a user is panning an object, store that information
    - Check if both gestures are running simultaneously
      - start a timer, from 3, counting down.
  - Check if the user is rotating an object, store that information
    - Check if both gestures are running simultaneously
      - start a timer, from 3, counting down.
  - Check if the user stopped panning
    - Stop the timer if needed
  - Check if the user stopped rotating
    - Stop the timer if needed

- When the timer ticks, decrease the number of seconds left
	- If the number of seconds left is zero, stop the timer*

{% endhighlight %}


Ohh my god, it involves having 4 temporarily variables expressing the state of the UI!

Also, it is *nothing* like the original English sentence. The logic is buried between if statements, where we set random booleans.
We write functions to eliminate duplicated code. And the we jump around like an inconsistent bugs bunny - there are 5 of them, all calling each other.

How can we improve upon this?

So, the question that reactive programming can help you with is:

### How can you replace all those "if" instances with "when"?


Let's have a look!

{% highlight swift %}

let pan = UIPanGestureRecognizer()
let pinch = UIPinchGestureRecognizer()

// Here, we want to create a signal that emits an event when the
// gesture has started.

// We can call .rx_event on UIPangestureRecognizer, and
// transform it's output into a signal, which then
// We can use filter to discard all other events!

let panStarted = pan.rx_event.filter { gesture in gesture.state == .Began }

// We do the same here, but this time we're only interested
// when the gestureRecognizer is in the .Ended state.

let panEnded = pan.rx_event.filter { gesture in gesture.state == .Ended }

let pinchStarted = pinch.rx_event.filter { gesture in gesture.state == .Began }
let pinchEnded = pinch.rx_event.filter { gesture in gesture.state == .Ended }

// Okay, let's think. Our aim is to only trigger the timer when both
// gestures began. We originally had to keep track of these states ourselves,
// but now that we transformed them into signals, we can also merge them, same
// way as you'd merge two arrays into one!
// Well, not exactly, we need a special combination, where the new signal will only
// start when both of its sub-signals have emitted an event: that's combineLatest().

let bothGesturesStarted = Observable.combineLatest(panStarted, pinchStarted) { (_, _) -> Bool in return true }

// For gestures ended, we need merge(), since we don't need to wait until both of
// them has started, we can immediately forward events from both channels into
// the combined one.

let bothGesturesEnded = Observable.of(panEnded, pinchEnded).merge()

// Now that we have our starting point, what should we do?
// Signals are special beasts: you can subscribe to them. Same way as
// you'd subscribe to a notification through NSNotificationCenter.
// subscribeNext will take a closure (block), and every time the signal
// fires, it'll run the signal's emitted value through the block!

bothGesturesStarted.subscribeNext { _ in

  // So, when bothGesturesStarted started, do this:
  print("started")

  // Now we need to create a timer. Rx is a quite comprehensive library,
  // it has some really handy structures and extensions - one of them is
  // exactly a timer.
  // What do we want? If we don't want to keep track of any more state,
  // we can just create a signal that emits a number (Int), and increase it
  // on every 'tick'.

  let timer = Observable<Int>.timer(1, period: 1, scheduler: MainScheduler.instance)

  // But wait, we don't want this timer to go on indefinitely!
  // Hmm, actually this is quite easy. Rx will also provide you operations
  // where you can limit the lifetime of a signal.
  // Same way as you'd take only the first 3 items in the array, you can
  // use take(x) to only take the first x events (values) from a signal.

  let timerThatTicksThree = timer.take(3)

  // And finally, we can also combine signals in other ways: this time,
  // we need to stop the timer signal when another signal is fired.
  // Here, we just use takeUntil! It's like the "while" loop of signals.

  let timerThatTicksThreeAndStops = timerThatTicksThree.takeUntil(bothGesturesEnded)

  timerThatTicksThreeAndStops.subscribe(onNext: { count in
    // when a tick happens, do this:
    print("tick: \(count)")
  }, onError: nil, onCompleted: {
    // when the timer completes, do this:
    print("completed")
  }, onDisposed: nil)
}

{% endhighlight %}

And now, voila, all four of the temporary variables are gone!
Also, did you notice, that the code looks like this:

{% highlight swift %}

define condition “simultaneously panning and rotating”
define condition “start a countdown from 3”
define condition “when the user stops the gestures”
define what a timer is

now do this: “When the user starts simultaneously panning and rotating an object, start a countdown from 3.
Stop the timer either when the countdown ends or when the user stops the gestures.”

{% endhighlight %}

You can also compress the syntax quite a bit, if you want to see how, [here's the link](https://github.com/itchingpixels/talks/blob/master/functional-reactive-intuition/Project/RFP/ReactiveShortViewController.swift) to the file!


Have a look at the example project [here](https://github.com/itchingpixels/talks/tree/master/functional-reactive-intuition/Project) and feel free to play around.

Does this makes sense? [Let me know](https://twitter.com/itchingpixels) what you think!
