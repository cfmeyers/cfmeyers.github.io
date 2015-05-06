---
layout: post
title: "Consider Static Typing Article"
date: 2015-05-06T14:43:27-04:00
---

Just came across [this](http://codon.com/consider-static-typing) post by Tom Stuart (of [Understanding Computation](http://computationbook.com/) fame).  

Tom, where were you when I was studying Principles of Programming Languages?  Apparently I've misunderstood static and dynamic types all these years.  It's a long article, but worth reading both entirely and closely.

###My Takeaways

-  Dynamic and Static typed languages both have metadata about "types"

-  The difference lies in *when* the metadata is checked; dynamic languages do the checking at execution time, whereas statically typed languages do the checking at compile time

Maybe my confusion was pardonable; my only statically typed language was Java, and the most obvious difference between Java and dynamic languages like Ruby, Python, or Javascript is having to endlessly write out type declarations.  I came to associate types with these tedious declarations (lots of typing in Java!).  

What really cleared it up for me was about a third of the way into the article where he laid out *how* MRI Ruby actually does type checking (using a series of boolean helper functions).  

Programming strikes me as one of the few disciplines where you  can create something you can't understand.  In fact it's even easier to create something you don't understand than to create something you do understand.  Carpenters understand the furniture they're building, mechanics understand the engines they work on, and tailors the clothes they make in ways that would make a programmer envious.  

Testing, like static types, struck me as a tedious, bureaucratic imposition when I first heard about it.  It was several years before I saw testing for what it really is: a tool for augmenting our understanding of our own programs.  I'm starting to feel that way about static types.  Next language, Go.
