---
title: "The \"Practical Haskell Bits\" initiative"
date: "2022-09-10"
description: "An attempt to collect and organize various real world Haskell patterns, snippets and popular library examples."
---

> Go [straight to the repository](https://github.com/dnikolovv/practical-haskell)

It happens to all of us.

Once in a while you stumble upon a problem that seems way too familiar. You remember that you've either solved it before or saw it in a blog post somewhere.

You start browsing through Reddit, old projects and bookmarks, searching aggressively for that single line of code or function that you just need to copy-paste or at least take a glimpse at to remember what it was about. It's just one "bit" that you need.

Practical Haskell Bits is an initiative to contain as many of these as possible and become the go-to place for real-world patterns, snippets and popular library examples.

Bits should be aimed towards **everyone** in the community, regardless of their level of experience or understanding.

> No example is too trivial!

The Haskell community is often criticized for focusing on things that are way too complicated to outsiders (and frankly, most "intermediate" Haskellers as well) and pushing away beginners. This is true and in order to attract more people and have them become productive, we need to address it.

Not everyone can figure out how to use a library by reading the source code.

Some people will struggle with "basic" things such as parsing JSON or setting up a web server. It's exactly these people that will not have the confidence to ask a question because seemingly everyone is busy looking at [`examples of compiler optimizations changing asymptotic complexity`](https://www.reddit.com/r/haskell/comments/xah8v1/examples_of_compiler_optimizations_changing/).

# What defines a Practical Haskell Bit?

#### A Practical Haskell Bit is a **mini-project** that:


* Looks at a single, well-defined scenario
* Is of production quality
* Contains as little code and dependencies as possible
* Is self-contained and buildable on its own
* Reflects the current (or at least some) best practices
* Is suitable for copy-pasting or just refreshing your memory
* Has a sufficient, but not necessarily detailed explanation
* Aims to use terminology and examples as close to the real world as possible

#### A Practical Haskell Bit is **not**:

* [Necessarily] a detailed tutorial
* An incomplete code snippet
* A full blown project example (e.g. [realworld.io](realworld.io))
* A blog post that you need to follow to put the code together

# Examples and non-examples:

#### Good Practical Haskell Bit candidates:

* Streaming `persistent` queries using `conduit`
* Integrating with an external API via `servant-client`
* Making your `wai` app AWS Lambda compatible
* Setting up your application monad and business logic with `mtl`
* Protecting `servant` routes with a JWT token
* Using smart constructors
* Record update scenarios via `lens`
* Logging to multiple destinations (e.g. file + stdout) via `katip`
* Property testing with `QuickCheck`
* etc.

One might argue that for examples using specific libraries, we should just focus on updating the documentation. I support that 100%, but there are multiple points of view.

Better documentation only helps only if you're **already aware** of the library in question.

It's easy to underestimate how many people haven't heard about `QuickCheck`, `katip` or even `servant`. Containing and organizing all examples in a single repository makes them "contagious". You come for 1 thing, but find out a few others.

#### Bad Practical Haskell Bit candidates:

* A CRUD app
* A console game
* A snippet containing some part of a solution
* A snippet without sufficient context
* etc.

# Reasoning

Having isolated and focused examples makes them a lot easier to search, develop and maintain. Full blown projects can quickly grow to gigantic proportions and make it confusing what the whole thing is about.

While some of the aforementioned points might seem "too simple", this will not be the case for everyone. It's easy to remember how hard and confusing the Haskell space can be in the early days, especially with there usually being multiple "correct" ways of reaching the same destination.

#### Don't we already have a ton of blog posts?

The Haskell space has been blessed with some pretty intelligent people and we're lucky to have many of them share their thoughts and research via blog posts.

While this is undisputably great, often times the level of abstraction is just too far from the "real world" for most people.

As an example, contrast the practical `DerivingVia` trick that I've shown [here](/practical-haskell-deriving-via) that uses concepts familiar to everyone such as calling an external API, versus the [documentation](https://ghc.gitlab.haskell.org/ghc/doc/users_guide/exts/deriving_via.html) that talks about Kleisli arrows, `Applicative` and `Semigroup`.

# Contributing

I have set up a repository at [dnikolovv/practical-haskell](https://github.com/dnikolovv/practical-haskell) and added a couple of bits, with more coming in the following days.

It would be great if you can provide feedback or submit your own examples. The hope is that eventually this initiative can grow into its own organization and become a cherished resource in the Haskell space.