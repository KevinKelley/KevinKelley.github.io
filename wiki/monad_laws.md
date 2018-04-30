---
layout: default
---

The three laws of Monadics:
====
To "be" a monad, a type must obey three relations: left and right unit, and associative bind.

*Left unit*
----
From Wadler:  'unit a &lowast; &lambda;b. n &equiv; n[a/b]'

 that is, compute value 'a', bind 'b' to the result, and then compute 'n'.  The result is the same as computing 'n' with value 'a' substituted for variable 'b' (note that this definition is assuming Lisp-like dynamic variable binding).

Expressed in terms of the 'unit' and 'bind' functions as developed in these [[Monads]] pages, that definition is equivalent to this:
  bind(unit(a), |b -> Func| { n(b) })  ==  n(a)
or in words: for a value 'a' and a function 'n', binding 'unit(a)' to 'n(b)' must be the same as directly computing 'n(a)'.

*Right unit*
----
Wadler again:  'm &lowast; &lambda;a. unit a &equiv; m'

Compute 'm', bind result to 'a', and return 'a'; result must be the same as 'm'.

In our terms:

  bind(m, |a->Func| { unit(a) })  ==  m

that is, binding 'm' to a function that returns the *monadic unit* of its value must yield a function that is equivalent to the function 'm'.


*Associative bind*
----
 m &lowast; (&lambda;a. n &lowast; &lambda;b. o) &equiv; (m &lowast; &lambda;a. n) &lowast; &lambda;b. o

Compute 'm', bind result to 'a', compute 'n', bind result to 'b', compute 'o'.  The order of parentheses in such a computation is irrelevant.

Using our 'unit' and 'bind':

  bind(m, |a->Func| {bind(n, |b->Func| {o.call})})

must be equal to:

  bind(bind(m, |a->Func| {n}), |b->Func| {o.call})


Proving these three relations in our monads, exceeds the scope of what I'm able to accomplish here.  In most cases, inspection makes it clear that they hold; if a "monad" applied to a new type doesn't seem to behave as monads should, then verify that these three laws do actually hold in that context.

Remember that these "monad laws" are in the nature of mathematical proofs: if your assumptions are valid, and the laws are obeyed, then the further computations that build on them are solid.

More Laws?
====

If the 'unit' and 'bind' laws are analogous to the mathematical *one* and *multiplication*, then probably there's something analogous to mathematical *zero* and *addition* too... and indeed, monadic 'zero' and 'plus' do make an appearance.

*Zero*
----
Zero is a useful concept sometimes.  The Haskell **Maybe** monad is a concise way of representing computations that may fail: for example, take integer division, where the result may be undefined.  Then in Haskell 'Maybe m' is 'Just m' or 'Nothing', and division is 'a / b &equiv; if b = 0 then Nothing else Just m'.

*Plus*
----
Monads with a 'plus' operator are things like *Sequence* or *List*, *Set*, *Map*...

The [[Parser Monad]] makes use of 'zero' and 'plus'; also in [[Map and Join]] I've started to explore the relation between the monadic formulation ( 'unit' and 'bind'; 'zero' and 'plus') and the more well-known **map** and **join** dual.
