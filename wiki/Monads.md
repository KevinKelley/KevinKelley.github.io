---
layout: default
---

**Monads** are a construct from Category Theory and functional programming (notably Haskell) that offer a means of abstracting over types of computations to compose and manipulate them in interesting ways.  **Eugenio Moggi** combined monads from category theory with lambda calculus to describe state, exceptions and continuations, giving us a powerful way of describing program structure.

Briefly: a type is a Monad if it provides *unit* and *bind* operations that obey three *monad laws*.  By analogy, *unit* is like the mathematical one:  '1 . x  =  x . 1  =  x' expresses the left and right unit; and *bind* is associative  (see [monad laws](/wiki/monad_laws.html) for more).

Basic Monads
=====
Wadler's paper *Monads for Functional Programming* is a clear, concise intro to Monads, using as an example a few lines of code from an expression evaluator, and showing how to extend it in three different ways with three monads.  [Three Basic Monads](/wiki/Three_Basic_Monads.html) shows a Fantom implementation of those examples, with a little more commentary.

Comprehensions
=====
Working with monads quickly leads to a desire for *comprehensions*, and in fact monad comprehensions are a generalized form of the list comprehensions available in Python and other languages.  See **Wadler, Comprehending Monads**.  An interesting example of pretty code using list comprehensions is the [Norvigs Spelling Corrector](/wiki/Norvigs_Spelling_Corrector.html), implemented in Python, Clojure, and Fantom.  Comprehensions are a very handy concept, and make certain kinds of code much neater.

Sequences form a Monad, and in fact the formulation of a monad from *unit* and *bind* operators has been shown to be equivalent to a formulation in terms of *map* and *join*.  Either can be expressed in the other; Haskell sources seem to prefer the first as being more concise, but the *map* and *join* operations can sometimes be easier to understand.  [Map and Join](/wiki/Map_and_Join.html) explores Fantom Lists as monads.

Parsers
=====
Parsers make a good example of the power of Monads, and in fact Haskell's GHC is fundamentally based on monads.   Hutton and Meijer in **Monadic Parser Combinators** give a thorough overview of how that works.  Meijer in **Parsec: Direct Style Monadic Parser Combinators For The Real World** gives further detail on proper implementation; and the Functional Pearl **Monadic Parsing in Haskell** (Hutton and Meijer) is a concise description of some beautiful code.  In [Parser Monad](/wiki/Parser_Monad.html) I implement their parser combinators in Fantom.

Continuations
=====
An interesting (and extremely mind-mush-inducing) application of Monads is in the [Continuations](/wiki/Continuations.html) monad.  Continuations are well-known in Lisp and Scheme, as a means of implementing low-level control structures like exceptions, co-routines, and really any sort of program-flow control.  Continuations are handy when you're talking about tail-call optimization, for example, and the can be useful in implementing a web-app framework, as a means of threading state through a series of stateless page-requests.

Monad Transformers
=====
In some of these examples, the monad is actually a combination of two or several monads.  For now I'm not using that fact; but ideally the monads would be implemented separately and combined as needed.  How to express such [Monad Transformers](/wiki/Monad_Transformers.html) isn't yet clear to me, though.  **Clojure** in its *contrib/monads* library makes a beginning at dealing with it.  The **Maybe Monad** which expresses a nullable type, for instance, can be applied as a Monad Transformer to another monad.

Credits
=====
I should mention that much of my (limited) understanding of Monads comes from two sets of tutorials on Monads in Clojure.  Konrad Hinson wrote the Clojure monads and a nice [tutorial]`http://onclojure.com/2009/03/05/a-monad-tutorial-for-clojure-programmers-part-1/` on them; and Jim Duey wrote some [good clojure stuff]`http://intensivesystems.net/tutorials/cont_m.html` on using monads as well.  Much of my code on these pages is a straightforward attempt to steal their ideas for Fantom.  Much thanks to those guys!
