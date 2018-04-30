---
layout: default
---

These [Monads](/wiki/Monads.html) pages derive pretty directly from one or another of the resources listed below.  There are a lot more out there; these are what I found to be the most important, or most relevant, to what I am trying to do.  This has been a learning experience for me; I've attempted to survey the history and current state of the art as relates to monads, and to codify the many known variations into Fantom language examples.

A Fantom **monad library** would hopefully be able to address the various examples shown here, but would need to address some tradeoffs between genericity and closure type safety; the **comprehensions syntax** issue comes up too, when you think about making monads generally useful in Fantom.

Anyway, some references:

- Eugenio Moggi
  - Computational lambda-calculus and monads. In Symposium on Logic in Computer Science, Asilomar, California; IEEE, June 1989.

- Philip Wadler:
  - Monads for functional programming
  - Comprehending Monads

- Hutton and Meijer:
  - Functional Pearls: Monadic Parsing in Haskell
  - Monadic Parser Combinators
  - Parsec: Direct Style Monadic Parser Combinators For The Real World

Clojure monad library and tutorials
  - Konrad Hinson (monad library and [tutorials]`http://onclojure.com/2009/03/05/a-monad-tutorial-for-clojure-programmers-part-1/`)
  - Jim Duey ( [tutorials]`http://intensivesystems.net/tutorials/cont_m.html`)


In the Clojure email list, Michal Marczyk posted the following:

Monads: A Bibliography
=================

First, the above mentioned tutorial by Konrad Hinsen:

[A Monad Tutorial For Clojure Programmers][1]

The following articles by Jim Duey are also very good (and they do take a somewhat different approach to Konrad's tutorial, so it's worth while to read both series):

[Monads in Clojure][2]
[Higher Level Monads][3]
[Why Use Monads][4]
[The Continuation Monad in Clojure][5]
[Sessions for Compojure][6]

There's also a very good Haskell resource on the most frequently used monads:

[All About Monads][7]

The example code is in Haskell, but each monad's section includes some motivating discussion.

Then there are [Philip Wadler's monad-related papers][8]. IIRC, "Monads for functional programming" is a bit of a tutorial paper, so that may be worth skimming.

And just for the pleasure of reading through the numerous displays of breathtaking FP brilliance collected therein, take a look at [Oleg Kiselyov's site][9]. The section labelled "Computation" has a subsection devoted to monads.

- [1] `http://onclojure.com/2009/03/05/a-monad-tutorial-for-clojure-programmers-part-1/`
- [2] `http://intensivesystems.net/tutorials/monads_101.html`
- [3] `http://intensivesystems.net/tutorials/monads_201.html`
- [4] `http://intensivesystems.net/tutorials/why_monads.html`
- [5] `http://intensivesystems.net/tutorials/cont_m.html`
- [6] `http://intensivesystems.net/tutorials/web_sessions.html`
- [7] `http://www.haskell.org/all_about_monads/html/index.html`
- [8] `http://homepages.inf.ed.ac.uk/wadler/topics/monads.html`
- [9] `http://okmij.org/ftp/`
