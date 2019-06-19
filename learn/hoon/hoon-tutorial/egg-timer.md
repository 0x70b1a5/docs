+++
title = "2.7.1 Walkthrough: Egg Timer"
weight = 36
template = "doc.html"
+++

The Arvo operating system is divided up into modules called vanes. Gall is the vane responsible for providing an API for building stateful applications in Urbit. Gall is a complex system that takes a lot of getting used to, so let's dip our toes in with a simple egg-timer app.


```
|%
+$  effect   (pair bone syscall)
+$  syscall  [%wait path @da]
--
|_  [bowl:gall ~]
++  poke-noun
  |=  t=@dr
  ^+  [*(list effect) +>.$]
  :_  +>.$  :_  ~
  [ost %wait /(scot %da now) (add now t)]
++  wake
  |=  [=wire error=(unit tang)]
  ^+  [*(list effect) +>.$]
  ~&  "Timer went off!"
  [~ +>.$]
--
```

The first thing to notice is that we are creating a `core` (`|%`) and a `door` (`|_`). This is a typical style of Gall programming where your types are defined in the first `core` and your application is defined in the following `door`.

```
|%
+$  effect   (pair bone syscall)
+$  syscall  [%wait path @da]
--
```

The `core` here defines two types: `effect` and `syscall`. An `effect` is a `pair` of `bone` and `syscall`. A `bone` is a Gall-only type that identifies app event chains by mapping atoms to them. We define `syscall` to be a request to Arvo to do something for us. In this case, the only valid `syscall` will be `%wait` which we'll discuss in a bit.

It's important to note that the names `effect` and `syscall` are arbitrary; you can call them whatever you'd like. Commonly you will see them called `move` and `card`, respectively, but we've changed the names here to try to introduce some clarity that that older convention lacks.

The sample of the `door` is:

```
[bowl:gall ~]
```

You can find the full definition of `bowl` in `sys/zuse.hoon`, but for now it's enough to know that this is the default app state and includes various faces for information. Below are some important such faces:

`our`  The current ship
`src`  The ship the request originated with
`ost`  A reference to a chain of events
`eny`  Entropy
`now`  The current time of the event

The `door` we've made has two arms `poke-noun` and `wake`. Gall is capable of dispatching `pokes`, or requests, to an app based on the mark of the data given along with that poke. These are sent to the arm with the name that matches the mark of the data. Here we use the generic `noun` mark

```
++  poke-noun
  |=  t=@dr
  ^+  [*(list effect) +>.$]
  :_  +>.$  :_  ~
  [ost %wait /(scot %da now) (add now t)]
```

In the above code, we create a gate that takes a single `@dr` argument. `@dr` is an aura for a 128-bit relative date. Here are a few examples.

`~s17`  17 seconds
`~m20`  20 minutes
`~d42`  42 days

As a matter of good type hygiene, we explicitly cast the output of this gate with `^+` to ensure we are producing the correct thing for Gall to handle. `^+` is the rune for casting by example. Our example is a cell: `list` of `effect`, which we `bunt` with `*`, is the head; `+>.$`, the enclosing core which is our `door`, is the tail.

Next we're going to use the `:_` rune which is just the inverted form a `:-` the cell construction rune. We use it twice so the actual data will end up looking something like:

```
[[[ost %wait /(scot %da now) (add now t)] ~] +>.$]
```

`ost` is the `bone`, the opaque reference to a chain of events, that comes from `bowl`, so we're going to continue to use it here. `%wait` is the name of the `syscall`, in this case a request to the Behn vain to start a timer for us.

After `%wait`, we have the `path`, a `list` of `cords` that serves as the unique identifier for this `syscall`:

```
/(scot %da now)
```

The `/` here is an irregular syntax to make a `path` and `scot` is going to transform `now`, the current time as given to us in the `bowl` from a `@da` to a `cord`.

The final part of this `syscall` is:

```
(add now t)
```

`now` is again the current time, as a `@da` and `t` was our `@dr`. Through the magic of mathematics, we can add these two to get a `@da` that is `t` time in the future from now.

That's all for our `poke-noun` arm. What about when the timer goes off? `behn` will create a `move` similar to how we created one only this time it will end up being dispatched back to us in the `++wake` arm.

```
++  wake
  |=  [=wire error=(unit tang)]
  ^+  [*(list effect) +>.$]
  ~&  "Timer went off!"
  [~ +>.$]
```

`wake` is a `gate` that has two arguments, a `wire` and a `(unit tang)` with the face `error`. The syntax used here `=wire` is a shortcut for `wire=wire`. A `wire` is a specialized kind of `path`. This will be the `path` we gave in our original `syscall` so that we can use to dispatch on if we wanted to do different things based on which event this is. In this case, we don't.

Next we have the same cast we used in `++poke-noun` to make sure we are producing the correct thing for Gall. These casts are not strictly speaking necessary as the type system can infer what the type will be but they can be very useful both for debugging our own code and for someone coming behind us trying to determine what our code should be producing.

```
~&  "Timer went off!"
```

`~&` is the debugging printf rune. Here we're simply going to output a message. We could however do any other computation here we wanted to happen when `++wait` gets called.

Finally we need to produce our `effect`, which here is simply `[~ +>.$]` since we have no `syscall` or `cards` we want to make and we are not changing the state of our app, we just use the `door` without changing its sample.

### [Next Up: Reading -- Ford](../ford)
