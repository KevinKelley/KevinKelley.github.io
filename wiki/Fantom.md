---
layout: default
---

reFlux[reFlux]
******


*reFlux* is an IDE for the [Fan]`http://www.fantom.org/`
programming language, based on the Flux editor.

Pod List pane
=====================

Displays all pods in the system,
showing popup documentation for all defined types, including enum values,
fields, and method declarations. An easy way to see available methods
and signatures without needing to go out to the Fandoc html.

 PodList is a drop-in extension to Flux.

To install PodList, just get the [PodList]`/fan/podlist-0.2.1.zip` source and pod.
-  Extract podlist.pod from the zip file to your $fanhome$/lib/fan pods directory.
-  Then start Flux, and you should have a new option on the View menu: **PodList  Alt-P**.
-  Select that and the PodList sidebar will open; now you've got a quick view at the Fan system!

FanFold source editor
=====================

 Integrated parsing of Fan language allows hierarchical
'folding' of source code, so you can keep your field and method definitions on-screen, with
the body of the method folded away until you need to see it.

Coming soon:
=====================

- Code completion with semantic parsing
- Hover Help in the editor
- Type hierarchy browser

 Comments are welcome!



PEP is an Earley Parser[PEP]
***********************


 [PEP]`/fan/pep-0.2.zip`
is my translation of the [Java Pep parser]`http://www.ling.ohio-state.edu/~scott/#projects-pep`
to [Fan]`http://www.fandev.org/`.

Pep is a chart-based parser using the
[Earley algorithm]`http://en.wikipedia.org/wiki/Earley_parser`.

It has the
great benefit that it gives all possible parse node matches of an input source string.
 This means, for instance, that it's a great choice for an editor; it can be used
to give parse information even on mal-formed code.


Earley parsers are used to parse strings for conformance with a given
Context-Free Grammar. Once instantiated with a grammar,
an instance of this class can be used to
*parse* (or just *recognize*) strings
(represented as a sequence of tokens).

This parser fills out a Chart based on the specified tokens
for a specified seed Category. Because of this, it can
be used to recognize strings that represent any rule in the grammar. The
parse returns a Parse object
that encapsulates the completed chart, the tokens given and the seed
category for that parse.

For example, if a grammar contains the following rules:

pre>
S   -> NP VP
NP  -> Det N
Det -> the
N   -> boy
VP  -> left
<pre


Parses can be requested for category <code>S</code>
("<code>the boy left</code>") but also for category <code>NP</code>
("<code>the boy</code>"). For convenience, this class provides
the recognize method that just returns
the status for a given parse (but not its completed chart,
tokens, and seed category).


Unger parser
***************************


Another useful sub-string parser is Unger's algorithm.
It can be used to generate a 'forest' of parse trees that each represent
a recognition of a subset of the input string.

My Fan implementation of the Unger algorithm, from the Grune and Jacobs book
<u>Parsing Techniques</u>,
is [here]`/fan/Unger.zip`.
