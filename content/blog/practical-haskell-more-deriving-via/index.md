---
title: "More real-world examples of DerivingVia"
date: "2022-09-16T09:52:55.684205549Z"
description: "Cut off even more boilerplate using DerivingVia."
---

> This short post is part of the [Practical Haskell Bits initiative](/practical-haskell-bits-initiative/). Visit the [repository](https://github.com/dnikolovv/practical-haskell) to find out more real-world examples like this.

I received feedback from [this post](/practical-haskell-deriving-via/) that the given example is way too trivial to justify the abstraction, which could potentially leave some readers confused.

I actually tend to agree with that, so this post is a follow-up to explore some more (and more relevant) ways that you can cut off boilerplate and make your code more expressive using `DerivingVia`.

# Using `Read` and `Show` to get instances for free

Often times you'll run into 

# Parameterize `FromJSON` and `ToJSON` instances

> This short post is part of the [Practical Haskell Bits initiative](/practical-haskell-bits-initiative/). Visit the [repository](https://github.com/dnikolovv/practical-haskell) to find out more real-world examples like this.