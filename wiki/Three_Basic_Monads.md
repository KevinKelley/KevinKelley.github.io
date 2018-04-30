---
layout: default
---

This is part one of the [[Monads]] pages.

Start with a snippet to explicate: I'm working from the Philip Wadler paper **Monads for functional programming**, which is an excellent intro into the what and why of Monads, by the man who brought the concept to life.  That paper expresses its examples in a Haskell-like syntax, but it's kept small and clear.

From the paper, we start with

  data   Term = Con Int | Div Term Term

and

  eval         :: Term -> Int
  eval(Con a)   = a
  eval(Div t u) = eval t &divide; eval u

which first declares a data type 'Term', which is made by either the 'Con Int'  or the 'Div Term Term' constructor.

Then: 'eval' is declared to be a function that accepts a 'Term' and returns an 'Int'.  When called with a 'Con a' term it returns 'a'; when called with the 'Div t u' form, it evaluates its arguments and divides them.

(In the above, 'a' is a *Type Variable*, and the Haskell type inferencing system will figure out from context that 'Con a' must be called with an 'Int' so therefore 'eval(Con a)' is returning an Int.)

In a Java-like language, that would probably be modeled as

  interface Term {
    int eval()
  }
  class Con extends Term {
    private int a;
    public Con(int a) { this.a = a; }
    public int eval() { return a; }
  }
  class Div extends Term {
    private Term t, u;
    public Div(Term t, Term u) { this.t=t; this.u=u; }
    public int eval() { return t.eval / u.eval; }
  }

where the expression tree is built in the constructors, and evaluated by a call to 'eval'.  All very fine and handy.

I'm going to do it a bit different, though.  The point of this exercise is to follow along, in Fantom, with the various examples and demonstrations of monads that I've found, for the purpose of understanding what monads are, and to try to come to grips with turning theory into practice.  Since the [[theory of monads]] is mostly coming from the functional side of the world, the examples too will nearly always be functional, not so much object-oriented.  Monad theory is complicated enough that I'm not ready to try to translate in my head from functional to OO style, while I'm learning it.  Luckily, Fantom has good support for functional styles: closures are easy to use and powerful, and come with type inference capabilities too, which might be helpful.

So anyway.  Here's how I'm going to start, translating that Haskell-style example above into Fantom.  Instead of defining object classes, I'm going to define **computations**: functions that produce a value, implemented as Fantom closures.  So that data type definition in Haskell:

  data   Term = Con Int | Div Term Term

gets turned into two constructor functions:

  Func Con(Int v) { |->Int| { v } }

  Func Div(Func t, Func u) { |->Int| { t.call / u.call } }

--construct a **constant int** computation by returning a closure that evaluates to that int; and construct a **division** expression that evaluates its arguments and returns the result of dividing them.  Fun!  Testing such a thing looks like this:

  echo( Div(Div(Con(1972),Con(2)), Con(23)) .call)

which prints **42**.

Defining the monad
------------------
That's dandy; we have a way of doing functional composition using closures that can mimic our Java-like object-composition example.  But where's the monad?  For that matter, what is a monad?

Coming from the work of **Moggi** and then **Wadler**, a type is a monad if it obeys three [[monad laws]]:
- left unit
- right unit
- bind

refer to [[monad laws]] for the definitions of how those three things have to act; a quick glossing-over summary is that, for a type to be a monad, it has to have a **unit** (function that brings a basic value into the monad, without changing it) and a **bind** (function that connects two monadic values together).

Since I've chosen, for these example pages, to deal in *computations* -- that is, closures that can be composed together, and which compute some value -- then, I'll need a monadic type that can create, and combine, those computations.  So:

  class IdentityMonad
  {
    Func unit(Int a) { |->Int| {a} }

    Func bind(Func m, |Int->Func| f) { |->Int| { f(m()).call } }
  }

That is, the identity monad can turn a basic Int into a computation that produces the int; and can bind a monadic value (an int-producing computation) to an int-consumer by extracting the computed Int and calling the consumer function with it.

Given that identity monad, our example code looks like this:

  class Expr : IdentityMonad
  {
    Func Con(Int v) { unit(v) }
    Func Div(Func t, Func u) {
      bind(t, |Int a -> Func| {
      bind(u, |Int b -> Func| {
        unit(a/b)
      })})
    }
    Void test() {
      echo( Div(Div(Con(1972),Con(2)), Con(23)) .call)
    }
  }

So.  Weird, maybe, but almost pretty.  It works, and again prints **42**.

**unit** is obvious enough; convert a "normal" value into a "monadic" value.  **bind** is where the magic happens.  Don't worry; we'll come back to it, over and over.  (Or maybe you should worry, for that reason, :-)  **bind** is the heart of monads, and at least in some sense monads are the heart of everything computational, so it doesn't surprise me that there's a lot to understand about those few lines of code.

Exceptions Monad
----
This is just the first simple intro page and it's already running long, so I'll leave off justifying all this and jump into the examples.  First consider, the above example doesn't consider errors, such as divide-by-zero.  Building error-handling into the non-monadic example requires review and modification to the whole code; in the monad we can do this:

  class ExceptionMonad : IdentityMonad
  {
    Func raise(Str e) { |->Obj?| {e} }

    override Func bind(Func m, |Obj?->Func| f) {
      |->Obj?| { v := m(); if (v is Str) return v; return f(v).call }
    }

    Func Con(Int v) { unit(v) }
    Func Div(Func t, Func u) {
      bind(t, |Int a -> Func| {
      bind(u, |Int b -> Func| {
        if (b == 0) return raise("divide by zero"); return unit(a/b)
      })})
    }
    Void test() {
      echo( Div(Div(Con(1972),Con(2)), Con(23)) .call)
      echo( Div(Con(1), Con(0)) .call)
    }
  }

--that is, change the **bind** function to check for error and if there is one, halt and return it; only if the computation succeeds does the bind operation continue to the consumer-function.  Then functions like 'Div' can raise an error if necessary, and functions that don't need to know about it don't have to deal with it.  We've just implemented a form of Exceptions, using a monad, in a few lines of code.

State Monad
----
Dealing with mutable state is a big deal in the **FP** style; earlier functional languages tended to sweep it under the rug or out the front door, more or less saying "all bets are off" if you let state-change into your functional code.  The **State Monad** offers a way to bring changing state into a functional program.

Let's define **State** as a const value representing a program's state at some instant in time; the **const** satisfies the functional requirement that values not mutate.  Changing state as the computation moves forward is handled explicitly by creating a new state.

  const class State
  {
    const Int count
    private new make(Int c) { count=c }
    new def() : this.make(0) {}    // create a new State with default value
    State inc() { make(count+1) }  // create a new State with value 1 greater than this's
    override Str toStr() { "#$count" }
  }

Now the state-transforming expression-monad.  For this we complicate things by having the monadic functions accept a State as input, and return not just the computed value, but instead the computed value paired with a new State (in a 2-element array).

  class StateMonad
  {
    **
    ** 'tick' increments the state: return a function that takes a 'current'
    ** state, and returns a pair containing 'null' value and ticked state.
    **
    Func tick() { |State s->Obj?| {[null, s.inc]} }

    **
    ** 'unit' wraps a basic value (a) into a monadic function that pairs
    ** the value with a state, without changing the state.
    **
    override Func unit(Obj? a) { |State s->Obj?| {[a,s]} } // return a and unchanged state

    **
    ** 'bind' a monadic function to a consumer-function, by creating a
    ** monadic function ( |State -> [val, State]| ) that will:
    **   apply the m.f. to a state, yielding a value 'a' and new state 'y';
    **   thread the value and the new state thru the consumer-function,
    **   yielding a new value and a final state; return that as a pair.
    **
    override Func bind(Func m, |Obj? -> Func| f) {
      |State x->Obj?| {
        mx := m(x) as Obj?[]
        a := mx[0]; y := mx[1]
        m2 := f(a)(y) as Obj?[]
        b := m2[0]; z := m2[1]
        return [b,z]
      }
    }
    ** create a Con that ticks the state
    Func Con(Int v) {
      bind(tick, | Int? -> |State->Obj?| | {
        unit(v)
      })
    }
    ** create a Div that divides t and u, and ticks the state
    Func Div(Func t, Func u) {
      bind(   t, |Int a -> |State->Obj?| | { // val from computation t is bound to a
      bind(   u, |Int b -> |State->Obj?| | { // "                  " u "         " b
      bind(tick, |    _ -> |State->Obj?| | { // tick modifies state, doesn't produce a val
        unit(a/b)
      })})})
    }
    Void test() {
      echo( Div(Div(Con(1972),Con(2)), Con(23)) .call(State.def))
    }
  }
yields
  [42, #5]
--the answer's '42' and it took 5 ticks.

Okay!  That was fun, in a masochistic kind of way.  I think it makes explicit how much we take for granted in declarative (non-functional) languages.  This seems like a lot of contorting to get something we already have, But!  There's at least one reason to be aware of this kind of thing:  threads, multiprocessing, Actors, all that.  What this StateMonad does is simulate mutable state without actually having any.  Everything here is pure, functional, thread-safe, suitable for safely being passed among processors without worrying about conflicts and locks.

The *bind* contortions here, to make this work, are fairly mind-bending.  But once a monad is defined, the 'bind' function doesn't change.  You use it as in 'Div', with some boilerplate that sequences the binds, extracting the basic values for you to work with and return as a new *unit*.

If you've ran into List Comprehensions somewhere, or more generally *monad comprehensions*, that "successive bind" syntax will suggest itself as being what comprehensions are for.

Output Monad
------
One more example to complete our triad of basic monads.  Consider output: suppose we want to log some output at every step along the way, instead of just calculating and returning the result of the computation.  Again without monads the change pervades the source, but with a proper monad it's localized to a few places.

  **
  ** simulate output by returning a string along with the computation's result.
  **
  class OutputMonad : Monad
  {
    **
    ** "output" a string by wrapping it into a pair with an ignored value
    **
    Func out(Str x) { |->Obj?| {[x, null]} }

    **
    ** format an informative message from a description of a computation, and its result value
    **
    Str line(Str t, Int n) { "eval($t) ==> $n\n" }

    **
    ** 'unit' packages a result value into a function that pairs it with an empty message
    **
    override Func unit(Obj? a) { |->Obj?| {["", a]} }

    **
    ** 'bind' returns a monadic function that executes a computation and threads its value
    ** through a consumer-function, along with a concatenation of any messages generated.
    **
    override Func bind(Func m, |Obj? -> Func| f) {
      |->Obj?| {
        mx := m() as Obj?[]
        y := mx[0] as Str; a := mx[1] as Int  // extract msg and rslt from pair
        m2 := f(a)() as Obj?[]                // feed prev result to consumer fn
        z := m2[0] as Str; b := m2[1] as Int  // extract msg and final result...
        return ["$y$z",b]                     // ...and package them into a pair to return.
      }
    }

    **
    ** create a monadic func that returns a constant and talks about it.
    **
    Func Con(Int v) {
      bind(out(line("Con $v", v)), |Int -> |->Obj?| | {
        unit(v)
      })
    }
    **
    ** create a monadic func that divides the results of two other m.f.'s,
    ** and bind a message about it in, too.
    **
    Func Div(Func t, Func u) {
      bind( t, |Int a -> |->Obj?| | {
      bind( u, |Int b -> |->Obj?| | {
      bind(out(line("Div $a $b", a/b)),
               |Int _ -> |->Obj?| | {
        unit(a/b)
      })})})
    }
    Void test() {
      echo( Div(Div(Con(1972),Con(2)), Con(23)) .call)
    }
  }
--which again follows the pattern; most of the magic happens in the sequencing of 'bind's.  Running this yields
  [eval(Con 1972) ==> 1972
  eval(Con 2) ==> 2
  eval(Div 1972 2) ==> 986
  eval(Con 23) ==> 23
  eval(Div 986 23) ==> 42
  , 42]
-- the steps messages, accumulated and paired with the result value.









And that's that.  Next I'll look at the relation of monads to [[Map and Join]]. (Or return to the [[Monads]] page)
