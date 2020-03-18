---
layout: post
title: Big S(tate) notation
date: {}
published: true
---

_This is an early and somewhat funny/incomprehendible post I wrote a long time ago about stateless / stateful components (not in a React, sense, but more in an OOP, object-sense)._


There’s something that bothers me a little bit:

We have this useful thinking tool, [Big 0 notation](https://en.wikipedia.org/wiki/Big_O_notation), to measure the computation complexity of an algorithm / function.

And it's great! We can anticipate the amount of time that a certain operation could take, and the necessary memory we need prepare for it, and it helps us architect our software, and forsee future problems without actually writing a line of code.

But, do we have a good thinking tool for the **understanding (human) complexity of a component**?

I’ll struggle a little bit here, since I don’t have a clear definition of “understanding complexity”.

Maybe it’s easier to just describe what a simple component is in my mind:

- has very little or no state
- mutates its state very rarely (or never)
- it’s easy to reproduce a situation where the component is in a certain kind of state.

I’d consider a string conversion component very simple, since all its functions will be pure: they’ll get an input string and they’ll produce an output string.

In contrast, a complex component:

- has lots of states
- mutates its state very frequently
- it’s hard to reproduce a situation where the component is in a certain kind of state.

I’d consider a database component to be very complex, since none of its public functions will be pure: it’ll have to insert, modify and delete records, write them to the disk, cache them, etc.

It could be very useful to have a thinking tool for this kind of complexity: let’s call it the **big S(tate) notation**.

But… could for example, a linter or a complier classify a component as “complex” or “simple”?
Would it be possible to find that out, just by parsing the code and identifying some keywords?

There are two ideas to play with:

*How many variables are defined in global scope of the component (aka. properties)?*
This will give you a good indication of how many different state it can have.

*What’s the ratio of pure vs. impure functions (methods) the component has?*

---

Sorry if I jumped too far too quickly - let’s define what pure and impure functions are. This is not something I made up, promise. They even have [wikipedia articles](https://en.wikipedia.org/wiki/Pure_function)!

Let’s say:
A **pure function** is where you don’t modify or read anything outside of the scope of the function. You get the parameters, you transform them, and you return a value. It’s completely predictable - call it twice with the same input, and it’s guaranteed that it’ll produce the same result.

An **impure function** is where you take some parameters, but you also access some other properties of the component, change them, and then you return a value. It can be unpredictable - call it twice with the same input, and you can be never sure if you get the same result - since it’s using (and possibly) modifying state, outside of its parameters.

A pure function is like asking your computer to turn down the music it’s playing. It’ll do it as soon as you press the button.

An impure function is like asking a neighbour to turn down the music while it’s having a house party. He/she may say “go to hell” and turn up the volume in response, depending on how drunk he/she is, and various other factors.

---

The million dollar question is:

**How can you reduce the understanding complexity of a component that’s main function is to mutate its state?**

So that you can’t remove any of its defined variables.

Is that possible?
Now, if we have a look at the Big S(tate) notation definition above, we can slightly rephrase this question to:

*Can you increase the ratio of pure vs. impure functions in your component?*

What do you think? Does this make sense?

I’ll continue later, but if you’re curious, have a look at **[redux](https://github.com/rackt/redux)** and the **[explanation of how it works by its author](https://egghead.io/series/getting-started-with-redux)**.


Shout on [twitter](https://twitter.com/itchingpixels) if you find this horrendous / incorrenct or something you can relate to, I'd love to hear about it.
