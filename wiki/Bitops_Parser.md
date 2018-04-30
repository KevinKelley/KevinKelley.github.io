---
layout: default
---

Disclaimer first: this is an example only, to experiment with the [Parser Monad](/wiki/Parser_Monad.html).  It's neat, but I don't know yet how it's going to work out in "real life".  [source code](/wiki/source_code.html)

The setup
===
Hutton and Meijer's (see [theory of monads](/wiki/theory_of_monads.html) ) monadic parser example builds up to a nice, concise expression parser, like the one in your compiler textbook (Aho et al).  As a slight variation on that, I'm going to show a parser that evaluates bitwise ops on literal numbers.  Operator precedence is as normal:
```
  | ^    (or, exclusive-or)      lowest precedence
  &      (and)
  << >>  (left, right shift)
  + -    (add, subtract)
  * /    (multiply, divide)
  - ~    (unary negate, invert)
  ( )    (parenthesis)           highest precedence

```
The parser-combinators we've seen so far are sufficient for defining the grammar, but there's two issues we haven't touched on yet: tokenization, and handling of result values.  Both issues come from the fact that, so far, we're not doing anything with the things we parse except group the atoms (chars) together into nested lists.  Those groupings contain all the information you need to construct a parse tree, but they're not very convenient.

What we need is to add some translation functionality to the parser functions we have, so that they'll do something useful.

```
  **
  ** transformV wraps a parser 'p' and a value-transformer func,
  ** returning a new parser that transforms the value when p succeeds.
  **
  |State->Result| transformV(|State->Result| p, |Obj?->Obj?| transform) {
    bind(p, |Obj? v-> |State->Result| | { unit(transform(v)) })
  }
```
This creates a new monadic-parser function by binding a parser to a transformer.  The 'bind' operation will execute the 'p' parser, and extract its result value into the 'v' variable.  So here we apply the 'transform' function to 'v' and package the result back up with 'unit'.

whitespace
---
Whitespace is always a pain, and adding a separate *tokenizer* step is a common way to simplify things.

Now we can discard extraneous spaces with this:

```
  |State->Result| spaceOpt() { transformV(many(whitespace), |->Obj?|{null}) }
```
'whitespace' recognizes a space, tab, or newline; 'many' groups them into a list of chars.  Then 'transformV' applies a 'null'-returning function to that, replacing the list of stuff we don't want with a null instead.

Another way to get there is to reach into the guts with 'bind' and pull out only what we want:

```
  |State->Result| token(|State->Result| p) {
    bind(               p, |Obj? x-> |State->Result| | {
    bind(many(whitespace), |Obj? _-> |State->Result| | {
      unit(x) // keep p's result, discard space's
    })})
  }
  |State->Result| symbol(Str s) { token(string(s)) }
```
Now 'symbol("monad")' will return the word and discard the spaces.


Values
---
In 'transformV' we saw a parser that applies a function to a parsed value.  Sometimes you need to return a function for later use, instead of applying it immediately:

```
  **
  ** 'semantics' binds a semantic action of some sort to a parser func,
  ** creating a new parser that, when it succeeds, yields the action
  ** as its value. Applying the action to the proper values
  ** is handled for example by the chainX1 parsers, which apply
  ** the action to the values parsed around it; or for ex., by
  ** unary_Exp, which applies the function associated with a unaryOp
  ** to the expression that follows it.
  **
  |State->Result| semantics(|State->Result| p, Func act) {
    bind(p, |Obj? -> |State->Result| | { unit(act) })
  }
```
Using that we can define some operator parsers:

```
  |State->Result| orOp() {
    or(semantics(symbol("|"), |Int a,Int b->Int|{a. or(b)}),
       semantics(symbol("^"), |Int a,Int b->Int|{a.xor(b)}))
  }
  |State->Result| andOp() {
    semantics(symbol("&"), |Int a,Int b->Int|{a.and(b)})
  }
  |State->Result| shiftOp() {
    or(semantics(symbol("<<"), |Int a,Int b->Int|{a.shiftl(b)}),
       semantics(symbol(">>"), |Int a,Int b->Int|{a.shiftr(b)}))
  }
  |State->Result| addOp() {
    or(semantics(symbol("+"), |Int a,Int b->Int|{a.plus(b)}),
       semantics(symbol("-"), |Int a,Int b->Int|{a.minus(b)}))
  }
  |State->Result| mulOp() {
    or(semantics(symbol("*"), |Int a,Int b->Int|{a.mult(b)}),
       semantics(symbol("/"), |Int a,Int b->Int|{a.div(b)}))
  }
  |State->Result| unaryOp() {
    or(semantics(symbol("-"), |Int a->Int|{a.negate}),
       semantics(symbol("~"), |Int a->Int|{a.not}))
  }
```
Each of those will first, parse an operator symbol; then, replace the parsed value (some chars) with a function: a closure that will, when called, perform the proper calculation on its arguments.

Precedence tower
---
With all that infrastructure in place we can write the top level of the parser now:

```
  const |State->Result| expr := |State inp->Result| { or_Exp.call(inp) }
  |State->Result|    or_Exp() { chainl1(  and_Exp,    orOp) }
  |State->Result|   and_Exp() { chainl1(shift_Exp,   andOp) }
  |State->Result| shift_Exp() { chainl1(  add_Exp, shiftOp) }
  |State->Result|   add_Exp() { chainl1(  mul_Exp,   addOp) }
  |State->Result|   mul_Exp() { chainl1( atom_Exp,   mulOp) }
  |State->Result|  atom_Exp() { choice([nat, unary_Exp, paren(expr)]) }
  |State->Result| unary_Exp() {
    bind(unaryOp, |Obj? f -> |State->Result| | {
    bind(atom_Exp, |Obj? a -> |State->Result| | {
      func := f as |Int->Int|
      aint := a as Int
      return unit(func.call(aint))
    })})
  }
```
'expr' is the normal entry point to parse a bitops expression.

One last point I'll gloss over (I'm getting tired of talking about this damn' thing) is, notice that 'expr' is a field, holding a closure; the rest of the functions are methods.  That's to avoid infinite recursion in the declarations: 'expr' won't actually be evaluated until an expression appears in the input.
