- Start Date: 2014-10-24
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

"ask the compiler" - Parse "_" in any ident position as a placeholder to query possibilities, inspired by the haskell "holes" workflow, allowing typechecking of incomplete code. Add such a concept to the Rust AST.

# Motivation

Popular OO languages have a strong draw with mature IDEs, where "dot" autocompletion discovers available methods & fields rapidly as a user enters code, without having to manually consult documentation. 
This is invaluable for navigating complex APIs such as GUIs or 3d package SDKs- for many purposes, the tooling is more significant than any advantages in the language.

This proposal aims to close the productivity gap another way, leveraging Rusts 2-way type inference, within the traditional edit-compile cycle.

The compiler itself is a complex project that would benefit from better tooling.

# Detailed design

When the compiler encounters a placeholder ident, it should compile&type check as much as possible , satisfying surrounding type demands.

Then it can examine the placeholders report suitable candidate functions or fields, in error messages or warnings.

The most basic use would be:
 `x._ `  - list available fields & methods (and report the type of x)

but going further increasingly elaborate queries could be made, eg

 `fn foo(x:X)->Y { _(x) }`  - find any functions satisfying  (X)->Y
 
 
 `x._.bar()`  - search the fields to find which one in turn has a 'bar()' method; also try `x._._.bar()` etc, this would be a great help for navigating deep structures.
 
 `fn foo(x:X)->Y {a._()._()}`  - find pairs of functions such that (X)->Z,  (Z)->Y
 
 `fn foo(x:X,y:T)->_{ ... code..}` - report the output type
 
 `fn foo(x,y)->_ {...code..}` - examine 'foo' callsites, and report a valid signature for foo (refactoring scenario, when extracting code as a new function)

 `_{ foo: expr, bar: expr,.. }`  - find any structs with these named fields

## Other possibilities
In future tooling - an interactive environment could provide GUI dropboxes to fill in the holes, with continuous compilation in the background, gathering suggestions.

# Drawbacks

Additional compiler complexity.
possibly takes syntax space from other uses, eg some languages use _ as a lambda shortcut, or you might use it for positional default-parameters.

# Alternatives

-Experiment with similar functionality using an obscure unicode character for the same purpose, without changing the parser, just changing error messages.

-Code with placeholders could be treated as "unimplemented", or it could fail to compile.

-The placeholder could merely be placed in the AST, and left to external tools using the Rust API to make suggestions.

-just wait for a traditional IDE.

# Unresolved questions

