---
layout: default
---

Disclaimer again; this is meant to be a reasonably-sized example, not a working parser.  Use it to understand the [[Parser Monad]], not to actually parse Json. [[source code]]

Overview
===
We've got infrastructure out the wazoo by now, and I don't want to belabor the discussion of it.  The point of it is to be able to take a grammar, like the one at `http://www.json.org` , and translate it directly into executable code.

So, without spending much time at it, here's how I write that grammar using the monad:


  **
  ** Parser monad that recognizes Json (json.org) grammar (assuming no extraneous whitespace)
  **
  const class JsonParser : ParserMonad
  {
    // parser-func generator methods for Json grammar
    |State->Result| object  () { bracket(char('{'), members, char('}')) }
    |State->Result| members () { sepby(pair, char(',')) }
    |State->Result| pair    () { seq(str, seq(char(':'), value)) }
    |State->Result| array   () { bracket(char('['), elements, char(']')) }
    |State->Result| elements() { sepby(value, char(',')) }
    |State->Result| str     () { bracket(char('"'), many(strchar), char('"')) }
    |State->Result| strchar () { or(nonCtrl, escaped) }
    |State->Result| nonCtrl () { satisfy(|Int c->Bool|{!(c=='"' || c=='\\' || c<' ')}) }
    |State->Result| escaped () { seq(char('\\'), escChar) }
    |State->Result| escChar () { or(oneOf("\"\\/bfnrt"), seq(char('u'), hex4)) }
    |State->Result| number  () { seq(int, seq(fracOpt, expOpt)) }
    |State->Result| int     () { seq( signOpt, or(char('0'), seq(nonzero,digits)) ) }
    |State->Result| fracOpt () { or(seq(char('.'), digits), nothing) }
    |State->Result| expOpt  () { or(seq(e, digits) ,nothing) }
    |State->Result| digits  () { many(digit) }
    |State->Result| hex4    () { seq(seq(seq(hexdigit,hexdigit),hexdigit),hexdigit) }
    |State->Result| hexdigit() { oneOf("0123456789abcdefABCDEF") }
    |State->Result| digit   () { satisfy(|Int c->Bool|{('0'..'9').contains(c)}) }
    |State->Result| nonzero () { satisfy(|Int c->Bool|{('1'..'9').contains(c)}) }
    |State->Result| signOpt () { or(char('-'), nothing) }
    |State->Result| nothing () { unit("") }
    ** eN, e+N, e-N, EN, E+N, E-N all good; this parses the "e" and optional sign
    |State->Result| e() { seq(oneOf("eE"), or(oneOf("+-"), nothing)) }

    // generator-methods syntax (above) is convenient, but you have to break the
    // mutual recursion somewhere.  So 'value' will, instead of immediately creating
    // a parser, will instead create a closure that when evaluated during
    // parsing, will create a parser and then call it.
    //
    // Mutual recursion is fine, during a parse, because we're pretty sure that
    // the input will terminate eventually, no matter how deep the nested objects
    // go.  The problem is just in trying to pre-declare the parser structure:
    // without this change, it's trying to build an infinite-capacity parser
    // before even looking at the input.
    const |State->Result| value := |State inp->Result| {
      or(str,
      or(number,
      or(object,
      or(array,
      or(keyword("true"),
      or(keyword("false"),
         keyword("null")
      ))))))
      .call(inp)
    }

    |State->Result| item() {
      |State in->Result| {
        if (in.empty) return Result.Error(in)
        char := in.remaining[0] // grab any char
        state := in.advance(1)  // move state past it
        return Result.Ok(char.toChar, state) // return char and state
      }
    }

    ** parse any single character that satisfies the test predicate
    |State->Result| satisfy(|Int->Bool| test) {
      bind(item, |Str x-> |State->Result| | {
        (test(x[0])) ? unit(x) : zero
      })
    }
    |State->Result|  oneOf(Str cs) { satisfy(|Int c->Bool|{ cs.any|Int ch->Bool|{c==ch}}) }
    |State->Result| noneOf(Str cs) { satisfy(|Int c->Bool|{!cs.any|Int ch->Bool|{c==ch}}) }
    |State->Result|   char(Int ch) { satisfy(|Int c->Bool|{ch == c}) }

    |State->Result| keyword(Str s) {
      (s.isEmpty)
        ? unit("")
        : seq(char(s[0]), keyword(s.slice(1..-1)))
    }


    Void main() {
      echo(str.call(State("\"a\"")))
      echo(number.call(State("123")))
      echo(escaped.call(State("\\uabcd123"))) // parse the unicode, ignore the 123
      echo(pair.call(State("\"a\":123")))
      echo(sepby(number, char(',')).call(State("1,2,3")))
      echo(many(bracket(char('['), sepby(number, char(',')), char(']'))).call(State("[1,2,3]")))
      echo(array.call(State("[1,2,3]")))
      echo(object.call(State("""{"a":[1,2,3]}""")))
      echo(object.call(State("""{"id":12.3,"ints":[1,2,3],"value":456e8}""")))
      echo(object.call(State("""{"ident":123,"value":-456e8}""")))
    }
  }

I tried to be complete and correct in the grammar rules; and in fact the tests here do succeed on these simple inputs.  I haven't bothered to verify it any further, and I didn't handle the two issues I mentioned in [[Bitops Parser]] -- extraneous whitespace, and generating values from the parse.

But hey, neat toy!  Getting through the setup of the parser monad took a while, but once it's in place, we can generate a fairly complex parser, fairly concisely.  No YACC, no ANTLR, no external tools at all: just one small set of functions, available as a mixin.
