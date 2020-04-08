---
title:  "Sync APIs - git patch inspired non-blocking client-server communication pattern"
date:   2020-04-04
---

# Sync APIs - git patch inspired non-blocking client-server communication pattern


Three years ago, we were in a desperate need for a mental model: how to structure the highly asynchronous, non-blocking communication we have between our backend and mobile clients - where Read/Update are the only meaningful operations.

The solution we ended up with was something different than the currently popular CRUD / REST APIs (but closer to GraphQL’s) - and rather close to the concept of git’s patches / diffs.

Linux kernel developers were sending patches to mailing lists, and releases were assembled by applying these patches. While this sounds like a horrendous model for release management today (I guess it works though!) - but as a mental model for reliably communicating changes, it may be a good alternative to the currently popular conventions in certain cases.

CRUD (the basis of REST apis) stands for Create Read Update Delete - which seemed a little bit of an overkill at that time for our use case:

-  We had tons of small records, instead of a couple of big ones. Individual, non-batched updates wouldn’t have been efficient or easy to manage.
- There was no need for “Delete”.
- “Create” was really easy to couple with “Update”. So easy that differentiating between them would have only caused extra headaches.

So, from CRUD, we needed “RU”.

But... the biggest problem wasn’t the unnecessary operations - but managing what happens when the user goes offline for a month, and then comes back online. Our worry was that, then the client could suddenly start sending out-of-date, cached requests to the backend.
The worst case would have been for us to lose that data, or the up-to-date properties being overwritten with previous ones.

## The use case of Sync APIs

Two different communication patterns I see in contemporary software:
- software relying on frequent, blocking, real-time interaction (a social network, for example)
- software relying on infrequent, non-blocking interaction with the server, which mostly stores data that the client needs to time-to-time (potentially, on different devices).
There are two different ways of how we could communicate change: in a batch (as a transaction) or individual changes, one-by-one.
(a presentation tool, a game, a brain training app or a mostly single-player experience)

Example:
You have 3 post-its.

    interface PostIt {
        title: string
        status: “Available” | “Deleted”
        content: string
    }

The actions you’ve done:

1. You changed the 1st, named “birthdays” - PUT request to note/birthdays
2. Then, you deleted the 1st, named “birthdays” - DELETE request to note/birthdays
3. Then, you changed the 2nd, named “friends”. - PUT request to note/<2nd name>

You could batch these, and only communicate the diff, if you added a property that would store the last action’s date/time.

    interface PostIt {
        title: string
        status: “Available” | “Deleted”
        content: string
        lastModified: number
    }

A timestamp (lastModified) here would open up the possibility to compare and merge conflicting data effectively, by comparing what happened last.

    mergePostIts = (lhs, rhs): PostIt => 

Okay, right now it seems like we can only merge whole PostIts - as we only have one lastModified available. This means both status and content will be replaced with the value that was modified later. But… what if on one client, we changed the status, and on another, we changed the content? Well, you’d lose that information.
But there’s a way to prevent that from happening: have a timestamp on a property level, rather than on the larger context.

    Interface PostIt {
        status: {
            Value: “Available” | “Deleted”
            lastModified: number
        }
        content: {
            Value: “Available” | “Deleted”
            lastModified: number
        }
    }


## Advantages of Sync APIs

If you don’t need a separate “Create” operation, and your client-server communication can be made non-blocking for the user, the advantages of Sync APIs over CRUD/REST or GraphQL are:
- Offline support is easy. If the client can’t connect, it can still fall back to the local data it has. As every property has a timestamp, means it will be diffable on the backend later.
- Business logic is concentrated on the backend.
The frontend blindly accepts (with some basic safety checks) what the backend returns.
- Conflict resolution is easy (or, even free) if there are multiple clients editing the same data.
- It’s easier to debug, as the business logic sits in the backend (the thing you’d debug)
- You can reduce the number of requests

We use Sync APIs at [Drops](https://languagedrops.com). Ours is probably the perfect usecase - we’re mobile (lots of offline usage), and there’s no real-time interaction, but it is possible that you play on multiple devices, and modify the same data.

Our frontend is written with React-native, and we’re using redux for state management. To make it easy to store lastModified timestamps, we wrote a higher level reducer that automatically adds a timestamp for each change (when it was initiated by the user).

## Caveats of Sync APIs

- If a blocking “Create” operation is necessary (for example, because it’d generate a unique id), you can not use Sync APIs exclusively.

- Have you noticed that git will create inconsistencies when you set your system’s time to arbitrary values? Probably not. But believe me, it does! Anyway, keeping the client’s clock “real” is a challenge. You may need to delay communication with the backend until you find that the time stamps are correct and are not in the future.

- It may be useful to implement a mechanism on the backend side as well to filter out changes that happened “in the future”. Otherwise these users may not be able to override something they set while “in the future”.
