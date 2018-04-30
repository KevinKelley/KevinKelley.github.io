---
layout: default
---

Continuation-Passing Style
****
CPS transformation (Steele, Lambda the Ultimate xxx, Rabbit compiler) is a basic and powerful idea.  With closures and CPS you can for ex. implement procedure calls, tail recursion, actors...

Converting a function call to CPS: consider: '|Int i->Float| fn { return i*11f }'.  Normally: 'f(2) => 22'.  Call the fn with an int, and it executes and returns a float to the point of call.  In CPS there's no return: instead you pass an extra parameter, the "continuation"; and the fn instead of "return"ing its value, will call the continuation with it.  So, instead of a stack model, push args and pop return, there's no going back: you just keep moving forward to the next continuation.

So 'fn' becomes '|Int i, |Float| c| { c(i*11f) }'; and with a simple "echo" continuation like 'c := |Float f| {echo(f)}', the call to 'fn' becomes 'fn(2,c)  => 22'.  No big deal, it's a mechanical transformation; it's just that it's pervasive -- now, everything that touches 'fn' has also to be converted to CPS.  Surely we don't want to be doing such transformations by hand.

Enter the monad.  Monads are about combining computations according to rules; maybe we can make a Continuation Monad that will handle the connectivity, so we can continue to write functions the normal way, and use the monad to connect them together.

For an example, think about a simple web-app, as compared to a console app.  In the simplest kind of console app, you'd maybe start, print some options, wait for input, execute a selected action, and repeat.

In a web-app, it's not like that.  Http is stateless, so you'll have to handle your own session tracking; and you can't just have the server sitting waiting at the prompt for the user to respond.
