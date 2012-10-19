# Fast Code Nation

Bryan O'Sullivan

Facebook


# A little bit about me

My background is a little bit all over the place:

* Distributed systems

* High performance computing

* Linux kernel hacking

* Compilers and language runtimes


# Why performance?

Call it a combination of outlook and personality quirk.

* I like to develop my software quickly.

* But I also like that software to be *fast*.


# Performance---but when?

I often work on software where speed really matters.

Interactive tools

* *Don't make your users wait!*

Components that other people depend on

* *Don't constrain how people use your libraries*


# Interactive tools

Back in 2005, I started working on a DVCS:

* Mercurial

Although written in Python:

* It's *fast*---competitive with git


# An example from the early days of Mercurial

The `log` command displays history in reverse.

Two common use cases:

* I just want to look at the last few changes

* I'm searching through history for something


# Latency vs throughput

Low latency:

* I want (parts of) an answer quickly

* "I just want to look at the last few changes"

High throughput:

* I want as much data per unit time as possible

* "I'm searching through history for something"

Problem?


# Where we started

The original version of `hg log`:

* Walked history backwards

* One rev at a time

How did this behave?

* Displayed the first few revs quickly

* Was *incredibly slow* at retrieving all revs


# Why?

Filesystems optimize for one access pattern:

* *Forwards*, from beginning to end

Seeking backwards a little at a time?

* It works, but is *terribly* slow


# One "fix"

Force the poor user to tell us what they want!

~~~~
hg log --limit 5
~~~~

Problem:

* I often don't know how many revs I want to look at!

Very common usage:

~~~~
hg log | less
~~~~


# Perverting the usage pattern

These characteristics forced users to plan for how to get best
performance

For "I just need a few recent revs":

* View in reverse order, as usual

For "I need to bulk search":

* Ask for *forward* order

**Yuck!**



# A step on the road

Instead of reading one rev at a time, backwards:

Use a *windowed* approach!

* Jump back $n$ revs

* Read a window of $n$ revs *forwards*

* Present these in reverse order

* Repeat as necessary


# How did this fare?

Reduced frequency of backwards seeks by $n$ times

* Bulk throughput got better

* Latency for retrieving the first few revs got *worse*

Not good enough!


# A refinement

Start with a tiny window

Repeatedly double the window size as more revs requested


# Results?

Instant response for small requests

What about bulk throughput?

* Iterating backwards and forwards became *indistinguishable*

Hooray!


# And my point is?

Much (*but not all*) of the time:

* The big-picture details are what count

My `hg log` story would play out the same way in any programming
language


# How I spend my time

I spend way too much of my time paying attention to performance
issues!

Current job (Facebook):

* Make a big interactive Python app fast on huge data sets

Previous job (company I founded):

* Make a big interactive Haskell app fast on huge data sets


# Measure once

CPython has so-so performance analysis support

Its `cProfile` module measures every function call

* Measurements are expensive enough to distort performance

Even worse:

* We find out nothing about what code *inside* a function is costly


# Measure twice

The `statprof` profiler captures a stack trace periodically

* Very low overhead

* Tells us about hot spots *inside* functions

* Not accurate on short lived programs

* Call-outs to C code delay sample collection


# Measure ... thrice?

"Poor man's profiling"

* Attach `gdb` to a Python process

* Interrupt and capture a stack periodically

* Modern `gdb` can decode both C and Python stacks

Pain in the ass, high overhead, *but* greatest insight


# If performance matters ...

If performance is important to you, *build support into your app*

Mercurial makes measuring really easy:

* Run with `--time` to measure wall clock execution time

* Use `--profile` to get a `cProfile` breakdown

* Set `HGPROF=stat` before `--profile` for a `statprof` profile


# What measurements tell us

Before we pull out the big guns of "rewrite it in C!" ...

How can we improve the performance of code under CPython?

* Obviously, we measure before and after!


# What's in a name?

In CPython, most name lookups are expensive

For example, even in a process that is mostly network- and IO-bound,
this transformation makes for a 3% speedup:

~~~~ {.diff}
     def sendstream(self, source):
+        write = self.fout.write
         for chunk in source.gen:
-            self.fout.write(chunk)
+            write(chunk)
         self.fout.flush()
~~~~


# Wait, what?

What's this even *doing*?

~~~~ {.diff}
     def sendstream(self, source):
+        write = self.fout.write
         for chunk in source.gen:
-            self.fout.write(chunk)
+            write(chunk)
         self.fout.flush()
~~~~

* Look up the name `self.fout.write` just once

* Hoist this expensive lookup out of the loop

* Replace it with a local variable lookup

Even shortish chains of dots are expensive!


# Scary features

In fact, if you really need top performance out of CPython:

* There are quite a few language features you can avoid


# Nested functions

Why are nested functions expensive?

~~~~ {.python}
def foo():
    def bar():
	    wat()
~~~~

When resolving a name, a normal function:

* Searches its own environment and its module

A nested function:

* Must search *all* of its enclosing environments


# Classes with deep hierarchies

Why are classes with deep hierarchies expensive?

* They build up long chains of `dict`s to search

* If you use it, the `super` keyword is costly (!)


# Precompute stuff where possible

Here, we reduce the number of string concatenations in a heavily used
method:

~~~~ {.diff}
     def __init__(self, path, openertype, encode):
         self.encode = encode
         self.path = path + '/store'
+        self.pathsep = self.path + '/'

     def join(self, f):
-        return self.path + '/' + self.encode(f)
+        return self.pathsep + self.encode(f)
~~~~


# Generators are good!

A lovely pattern for reading a file in manageable chunks:

~~~~ {.python}
for chunk in util.filechunkiter(open(name), limit=size):
    yield chunk
~~~~


# Generators are bad!

This code is 10% faster by avoiding the `filechunkiter` in the common
case of small files:

~~~~ {.python}
if size <= 65536:
    yield open(name).read(size)
else:
    for chunk in util.filechunkiter(open(name), limit=size):
        yield chunk
~~~~


# The bottom line

In Python, almost *every* abstraction you use to make your code more
readable will hurt its performance.

* Function calls

* Classes of any sort

* Generators

* etc, etc


# So what to do?

Ultimately, we get the best performance out of Python by relying on C

* Know which functions/methods are written in C

Find ways to make use of this

* Cheap: transform a string in one pass, then split

* Costly: split string, then transform parts


# Even scarier

Dropping down into C is hard

* Manual reference counting is painful

* The usual C risks: bugs turn into corruption or crashes

* 5x longer development cycles, code much harder to review

Alternatives (Cython, boost::python) are heavyweight and meh


# Performance in Haskell

The very first thing people find in Haskell when they care about
performance:

* "I'm trying to process a text file, and it's *really slow*!"

Uh oh.


# A little history

The built-in `String` datatype in Haskell is really just a singly
linked list

Why?

Lists are famously easy to *reason* about

A list is either:

* Empty

* A single item, followed by the rest of the list


# Trouble in paradise

Our problem is *pointer chasing*:

* Every link in a list is allocated separately

* Each value pointed to by a link is allocated separately

There's a ton of costly overhead associated with lists

* And hence with strings


# What did we do about this?

The poor performance of strings was recognised early on

* I developed a `PackedString` type in 1993

* A packed array of bytes, nice and compact

The `String` type stays on, as a great teaching tool


# Was that it?

The `PackedString` type got a huge overhaul in the mid-2000s

* New `ByteString` type still a packed array of bytes

* Great for binary data, but ...

* ... useless for modern text processing


# Fast forward some more

The modern Haskell type for working with Unicode text is named `Text`

Uses some of Haskell's key features:

* Data is immutable

* Functions never change behaviour

Also has a really nice API


# Programming with pipelines

In Haskell, we often manipulate text using right-to-left function
pipelines:

~~~~ {.haskell}
length . toUpper . dropWhile isSpace
~~~~

Naively, this has quite a cost


# Wait, what cost?

~~~~ {.haskell}
length . toUpper . dropWhile isSpace
~~~~

Most stages of our pipeline create new `Text` strings:

~~~~ {.haskell}
dropWhile isSpace
~~~~

~~~~ {.haskell}
toUpper
~~~~

Yuck!


# What is to be done?

GHC is a powerful compiler

* Really good at inlining and program transformation

Its job is made easier by those key features:

* Data is immutable

* Functions never change behaviour


# But ...

There's a problem:

* In Haskell, instead of loops, we write recursive functions

* The building blocks of our pipeline are recursive functions

* A compiler cannot safely inline recursive functions!

Very few options for improvement here


# Let's change the game

Instead of processing text directly, let's process *streams*

A stream is:

* A generic piece of state

* A state transformer ("step") function


# What's a state transformer?

A state transformer:

* Consumes a state

* Returns a new state and a value

Importantly:

* State transformers are *not* recursive

* Candidates for inlining


# Pipeline transformation

Our original pipeline:

~~~~ {.haskell}
length . toUpper . dropWhile isSpace
~~~~

Here's how each function is implemented in the `text` library:

~~~~ {.haskell}
Stream.toText . Stream.dropWhile isSpace . Stream.fromText
~~~~

~~~~ {.haskell}
Stream.toText . Stream.toUpper . Stream.fromText
~~~~

~~~~ {.haskell}
Stream.length . Stream.fromText
~~~~


# Glue them together

Our original pipeline:

~~~~ {.haskell}
length . toUpper . dropWhile isSpace
~~~~

Our pipeline with a tiny bit of inlining:

~~~~ {.haskell}
Stream.length .
Stream.fromText . Stream.toText {- <<-- -} .
Stream.toUpper .
Stream.fromText . Stream.toText {- <<-- -} .
Stream.dropWhile isSpace .
Stream.fromText
~~~~

I've highlighted a few "do-nothing" transforms above. Why?


# A nice item in GHC's toolbox

Making use of these language features:

* Data is immutable

* Functions never change behaviour

GHC exposes a *rule-driven rewrite system* to programmers:

~~~~ {.haskell}
{-# RULES "drop fromText/toText"

Stream.fromText (Stream.toText t) = t

#-}
~~~~

"If you see the code on the left, replace it with the code on the
right."


# Ooh!

Our rewrite rule causes GHC to transform our pipeline into something
simpler:

~~~~ {.haskell}
Stream.length .
Stream.toUpper .
Stream.dropWhile isSpace .
Stream.fromText
~~~~

What's important about this?

* These functions are all written as state transformers

* None of them is recursive

* They can *all* be inlined


# Stream fusion

This approach is called *stream fusion*.

Starting from an expensive looking pipeline of functions:

* GHC will fuse our code into a single loop

* That loop will allocate no memory

* Runs about as fast as hand-written C


# How to use stream fusion

Built into in the `text` and `vector` libraries

* Users of these libraries benefit "for free"

The only "special" tricks required of users?

* Compile with `-O2`

* For an extra boost, use the LLVM back end (`-fllvm`)

Easy, right?


# Why does stream fusion work?

Haskell's powerful type system

* Lets GHC (and us!) tell when code can be aggressively transformed

* Segregates impure, unsafe code that deals with the outside world

Immutable data

* Doesn't change, easy to reason about

Predictable functions

* Not affected by hidden mutable state


# Compare and contrast

Need some speed?

Python:

* Use *fewer* abstractions

* Take more risks in dangerous languages

Haskell:

* Choose *smart* abstractions

* Make the compiler do the hard work
