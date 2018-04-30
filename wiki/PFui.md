---
layout: default
---


Pfui - PEP (Earley Parser) for Fan, with a UI
***************************

yacc, occs, llama, bison, ANTLR... themed names for parsers seems to be a ...well, theme.
PEP is another good theme-name, since it stands for **PEP is an Earley Parser**, and
the Earley parsing algorithm is useful in part because it has no problem dealing with
left-recursive grammars; since PEP is a left-recursive name, that's appropriate.

This program is built on a translation from Java of PEP; it's an Earley parser written
in the Fan language, with a User-Interface.  So the best I got for names is **PFUI**
-- **PEP for Fan, with a UI**.  Maybe it describes the exasperation that usually comes
from writing a parser.


Overview
=====================

There are two main parts to this: the **PEP** Earley parser library, and **Pfui**, the grammar-builder application.  PEP contains the core parser classes, a lexer, and a default BNF grammar to bootstrap other languages, and is intended to be linked and used by any application that needs a parser.  Pfui uses PEP as its parser, and provides a user interface for creating, editing, and testing language grammars.

The typical use case is to build a grammar in Pfui, and when you've got it correctly generating parse trees for some sample input, generate a **Grammar Initializer** which you then include in your application.

The Pfui package (including source for both PEP and Pfui) is [here]`/fan/pfui-1.0.1.zip`.

Here's a rough start at a [Pfui User's Guide]`/fan/PfuiDoc.html`.


Installation
=====================

[Download]`/fan/pfui-1.0.1.zip` the source distribution.
  - Unzip it to a convenient location; you'll get a folder called **pfui-1.0.1**.
  - Open a command window in the pfui-1.0 folder
  - Build the pods:
    'fan buildPfui.fan'
    (should build with no errors)
  - Start the Pfui app:
    'fan pfui'
  - Open one of the sample grammars:
    'File->Open pfui-1.0.1/pfui/grammars/number.grammar'

  - Parse it (toolbar button), and explore the generated parse tree
  - Transform it (toolbar button) into Grammar form
  - Run it (toolbar again) -- this opens a new View, with **number** as the language.
    Type a number into the text editor area:
    '123.4e-5'
    and click the Parse button.  You'll get a parse tree with nodes for each sub-element of the parsed number.
