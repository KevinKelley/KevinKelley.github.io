---
layout: default
---

It's been said that parsers are the classic example of the power of monads.  Since I'm always interested in parsing techniques, developing a *parser monad* in Fantom, along the lines laid down by Hutton and Meijer in their Haskell work (see [[theory of monads]] for references), will be an interesting learning experiment: some neat code will be seen, some old neurons will be burned, and at the end of the day maybe we'll know something we didn't before.  Hi ho.

The Setup
====
Briefly, we're developing a top-down, recursive-descent, deterministic parser, with arbitrary lookahead.  (Minor changes to the structure of the monad can give different characteristics).

The type of the monad is '|State -> Result|': that is, functions that take a current state as input, and return a result.  'Result' is a container that holds an arbitrary parsed-value, and a new state representing the remaining input to be parsed.  So, before we get to the monad itself, here's the little infrastructure classes.

State
----
The parser monad is a lot like the [[State Monad]], where the state being processed is the input string to be parsed.  Since functional style treats values as constant and handles transformation of values explicitly, we'll represent the state at any point in the parse with a 'State' class like this:

  const class State {
    const Int pos
    const Str remaining
    Bool empty() { remaining.isEmpty }
    new make(Str src, Int pos := 0) { this.pos = pos; remaining = src }
    This advance(Int n) { make((n&lt;remaining.size)? remaining.slice(n..-1): "", pos+n) }
    override Str toStr() { "@$pos:'$remaining'" }
  }

It's 'const' so individual states don't mutate; it holds an int 'pos' representing current position in the string being parsed, and a string 'remaining' which is the unparsed input.  Advancing the state is done by returning a new state, modified appropriately.  'empty' is true if there's no more input; and 'toStr' shows the current position and the remaining input.

Result
----

  **
  ** Result is the result of a parse, combining a parsed value and
  ** the remaining state
  **
  abstract const class Result
  {
    Obj?    val() { (this as Ok)?.v }
    State? rest() { (this as Ok)?.s }

    Bool okay()     { this is Ok }
    Bool error()    { this is Error }

    **
    ** construct an 'Ok' result: a result value, and a snapshot of state
    **
    static Result Ok(Obj? a, State s) { Ok(a,s) }

    **
    ** construct an 'Error' result with current state.
    **
    static Result Error(State at) { Error(at) }
  }

**Ok** and **Error** are the obvious value-holder classes:

  const class Ok : Result {
    const Obj? v;
    const State s;
    new make(Obj? v, State s) {this.v=v; this.s=s}
    override Str toStr() { "Ok [$v, $s]" }
  }
  const class Error : Result {
    const State at
    new make(State at) {this.at=at}
    override Str toStr() { "Error at $at" }
  }

Notice that we're not actually saying anything yet about the result value, other than that it's an 'Obj?'.  The parser monad can equally well be used as an evaluator, immediately executing its function, or as an AST-tree generator, creating a structure of the parsed program for later processing.

The Monad
====
First, a word about design choices: the parser monads described by Wadler, and by Hutton and Meijer (see [[theory of monads]] for references) are often defined to return an array of results, rather than a single result.  This makes powerful parsers, but it brings up the problem of what to do with ambiguity: ambiguous grammars will then yield multiple results for a given input.  Good, if you need it; but typically for programming languages you want to be unambiguous, and either yield a single successful parse or a failure.

For my examples here, therefore, I'm limiting myself to single results.  The parser monad isn't much different either way, but using it is simpler when you don't need to deal with selecting among ambiguity.

Unit and Bind
----
So!  Again, our parser monad will be defined for functions of type |State->Result|.  First we need a *unit*:

  **
  ** monadic unit: convert a basic value into a parser-monadic-function
  ** by wrapping it into a Result along with the then-current state.
  **
  |State->Result| unit(Obj v) { |State st->Result|{Result.Ok(v, st)} }

then a *bind*.

  **
  ** monadic bind: bind a parser monad to a combining-function.
  ** return a bound computation which will run the monad in a
  ** current state; if the parser ('m') fails, immediately return
  ** the error result; otherwise, pass the resulting value to the
  ** combining-function to get a new monadic function, then call
  ** that second function with the new state.
  **
  |State->Result| bind(|State->Result| p, |Obj?-> |State->Result| | f)
  {
    |State inp->Result|
    {
      r1  := p.call(inp)        // do computation m, get a result
      if (r1.error) return r1
      m2 := f.call(r1.val)      // feed the computed val to f,
                                // and get another parser, m2.
      return m2.call(r1.rest)   // do the new parse in the new state,
                                // and return its result
    }
  }

a **zero** is going to be useful sometimes, to represent a parser-monad that always fails:

  **
  ** return the parser monad's "zero" function: a parser that
  ** always immediately fails, capturing the state at the fail point.
  **
  |State->Result| zero() { |State inp->Result| {Result.Error(inp)} }

Quoting Wadler in *Monads for FP*:
> The empty parser and sequencing correspond directly to *unit* and *bind*, and the monads laws reflect that sequencing is associative and has the empty parser as a unit. The failing parser and alternation correspond to *zero* and &oplus;, which satisfy laws reflecting that alternation is associative and has the failing parser as a unit, and that sequencing distributes through alternation.

(The &oplus; operator from Wadler, represents **alternation**:  'm &oplus; n' is '&lambda;inp. [m(inp), n(inp)]' which means, apply 'm' and 'n' to the same input and combine the results.  This can be used to generate all possible parses of an input; usually though we want just the longest possible parse instead.  For example to parse an identifier from the input "monad", we'll want 'monad', not '[monad, mona, mon, mo, m, ""]'.  For this reason the **biased choice** operator which follows is more useful to us.)


Combinators
----
Now we can start building some parser functionality on top of this.  Let's have a function **or**, representing biased choice: 'p := or(t,u)' creates a new  parser 'p' which chooses between parsers 't' and 'u': if 't' succeeds, return its result; if not, try 'u'.

  **
  ** combinator that tries each parser in turn,
  ** returning the first to succeed.
  **
  |State->Result| or(|State->Result| p, |State->Result| q)
  {
    |State inp->Result| {
      r1 := p(inp); if (r1.okay) return r1
      r2 := q(inp); if (r2.okay) return r2
      return Result.Error(inp)
    }
  }

Sequencing of parsers:

  **
  ** return a parser that executes 'p', followed by 'q', and
  ** returns their results in a list.  Parse succeeds only if
  ** both 'p' and 'q' succeed.
  **
  |State->Result| seq(|State->Result| p, |State->Result| q) {
    bind(p, |Obj? x-> |State->Result| | {
    bind(q, |Obj? y-> |State->Result| | {
    unit([x,y])
    })})
  }

  **
  ** return a parser that executes 'p' as long as it succeeds,
  ** concatenating the results.
  **
  |State->Result| many(|State->Result| p) { or(many1(p), unit(null)) }

  **
  ** execute 'p' one or more times, concatenating the results,
  ** or fail if there is not at least one success.
  **
  |State->Result| many1(|State->Result| p) {
    bind(      p , |Obj? x-> |State->Result| | {
    bind( many(p), |Obj? y-> |State->Result| | {
      unit([x,y])
    })})
  }

Slightly more complicated are 'chainl1' and 'chainr1', for left- or right-associative lists of operator-separated expressions; and 'sepby' and 'bracket', for things like comma-separated lists, or bracketed expressions: see the [source code]``.

Parsers
----
With those, we have a pretty complete set of parser-constructs.  It should be easy enough to use them to build actual, useful parsers; and in fact it is -- actually, we need one more thing.

  **
  ** 'item' parses a single item from the input stream: this could be
  ** a token-grabber, if we want to use a lexical stage; but for many
  ** purposes we can build parsers all the way down to the individual
  ** character.
  **
  |State->Result| item() {
    |State in->Result| {
      if (in.empty) return Result.Error(in)
      char := in.remaining[0] // grab any char
      state := in.advance(1)  // move state past it
      return Result.Ok(char.toChar, state) // return char (as Str) and state
    }
  }

'item' parses a single character (any character) from the input, updates the state, and returns a result consisting of the one-character string and remaining input.

  **
  ** parse any single character that satisfies the test predicate
  **
  |State->Result| satisfy(|Int->Bool| test) {
    bind(item, |Str x-> |State->Result| | {
      (test(x[0])) ? unit(x) : zero
    })
  }

'satisfy' parses a single character, but only if it passes the 'test' predicate: otherwise this parser returns a fail result.  With those, we can have these:

  **
  ** character parsers: match a single char, according to some predicate.
  **
  |State->Result|   char(Int ch) { satisfy(|Int c->Bool|{ch == c}) }
  |State->Result|  oneOf(Str cs) { satisfy(|Int c->Bool|{ cs.any|Int ch->Bool|{c==ch}}) }
  |State->Result| noneOf(Str cs) { satisfy(|Int c->Bool|{!cs.any|Int ch->Bool|{c==ch}}) }
  |State->Result|  digit()       { satisfy(|Int c->Bool|{('0'..'9').contains(c)}) }
  |State->Result|  lower()       { satisfy(|Int c->Bool|{('a'..'z').contains(c)}) }
  |State->Result|  upper()       { satisfy(|Int c->Bool|{('A'..'Z').contains(c)}) }
  |State->Result| letter()       { or(lower,upper) }

Neat!

Okay, that pretty well covers the structure and layout of the parser monad.  Getting this far, coming to grips with the concepts, trying to understand what the heck's really going on in those Haskell examples, has been a bit of a mind-blow.  There's something pretty powerful going on here, somewhere.

Of course it's no good if it doesn't do some good!  So with that thought in mind, for a couple simple test cases, have a look at the [[Bitops Parser]] and the [[Json Parser]].
