+++
title = "The Beginning"
date = 2022-08-26T20:22:09-06:00

[taxonomies]
tags = ["Languages"]
+++

I'm a language enthusiast. Natural and constructed, computer and human, I love language in all its forms. I particularly love inventing languages. My childhood obsession with [conlangs](https://conlang.org/) turned into an adult obsession with designing programming languges, and between the two I've spent more hours than I can possibly count tinkering with grammar, syntax, and semantics.

<!-- more -->

Most of these projects have been the kind that I never quite finish. I have an ambitious idea, work on it for a while, and then I lose interest before wrapping things up tidily. This blog is my attempt to change that.

Inspired by [Extra Credits's advice](https://www.youtube.com/watch?v=z06QR-tz1_o) on [starting out in game development](https://www.youtube.com/watch?v=UvCri1tqIxQ) and a blog post on [Minimalism in Programming Language Design](https://pointersgonewild.com/2022/05/23/minimalism-in-programming-language-design/), I am going to build a lot of tiny language MVPs to completion. This blog will document my progress through these languages: what I learned, how they work, what works well, what does not. I hope to both gain experience actually finishing projects and to explore the "tiny language" design space to find out what's possible.

Here are my ground rules for this project:

1. Each language will be implemented in Rust and compiled to WebAssembly so that I can embed the interpreter inline with interactive examples.

2. Each language will be scoped down to the smallest interesting feature set possible to explore specific language concepts. At some point I expect to begin building on these minimal prototypes and combining them in interesting ways, but I'm going to begin by restricting myself to one concept at a time.

Here are some examples, in no particular order, of the kinds of languages I plan on implementing:

* A [Forth](https://www.forth.com/starting-forth/)

* A [concatenative](https://concatenative.org/wiki/view/Front%20Page) language inspired by [Joy](https://hypercubed.github.io/joy/joy.html) and [combinatory logic](https://en.wikipedia.org/wiki/Combinatory_logic)

* The [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus)

* The [join calculus](https://en.wikipedia.org/wiki/Join-calculus)

* A functional language built around [capabilities](https://en.wikipedia.org/wiki/Capability-based_security)

* A [Smalltalk](https://en.wikipedia.org/wiki/Smalltalk)

* A [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language))

For my actual work, I lean heavily towards static typing over dynamic typing, but my initial efforts here will likely all be dynamic. It's easier to implement a dynamic interpreter (a type checker has to be written separately), and it's a space I haven't explored much before. Again, as I get further along I may start including type checkers, or going back to previous languages and adding type safety, but to begin with my goal is to have a tiny scope that I can easily finish.
