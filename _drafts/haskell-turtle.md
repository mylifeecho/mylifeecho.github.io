---
layout: post
title: "Command-line tool in Haskell step-by-step"
description: ""
category: dev
tags: [ Haskell, Turtle ]
---
{% include JB/setup %}

After finishing [Introduction to Functional Programming][fp-edx] course on *edx* which was mainly focused on Haskell I was very excited about this language. Concise, powerful type inference system, desined to make functional programming easy to write and read. I have even found out that it has a few concepts that corresponds with my univercity background which is quantum physics. And actually I'm not the only one. There is even [library called quipper][quipper] to program quantum computer (which does not exists yet) based on Haskell... but it's a topic for another article. Then I started to fill that Haskell is awesome but it's most likely too complex and too advanced to be applied for regular tasks. You remember "There is no bad or good languages, tools, techniligies, etc. You should choose appropriate technology to solve your problem in most efficient way." You know that effect described perfectly in [this][hhh] blog post. I started to look for proofs and opinions to confirm that, but instead I found out that Haskell is mature enough to become technology of chose for many applications. And this time we are going to build simple command-line tool on Haskell and see how easy it is.

1. Use Stack as package management tool
2. Create project using stack
3. Add Turtle library for command-line tools
4. Write first command
5. Add flag to print version
6. Add subcommand to print version information
7. Add subcommand with arguments and options
8. Extend your tool following Turtle tutorial


[fp-edx]: https://www.edx.org/course/introduction-functional-programming-delftx-fp101x-0
[quipper]: http://www.mathstat.dal.ca/~selinger/quipper/
