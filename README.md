tree-sitter-java
================

[![Build Status](https://travis-ci.org/tree-sitter/tree-sitter-java.svg?branch=master)](https://travis-ci.org/tree-sitter/tree-sitter-java)
[![Build status](https://ci.appveyor.com/api/projects/status/12nl5pyqvl75g2ws/branch/master?svg=true)](https://ci.appveyor.com/project/maxbrunsfeld/tree-sitter-java/branch/master)

Java grammar for [tree-sitter](https://github.com/tree-sitter/tree-sitter).

#### :construction: Currently WIP — this is still in development. :construction:

### General grammar structure

Language constructs are grouped into the top-level categories denoting declarations, statements within methods and expressions. All granular constructs feed into those through the defined grammar hierarchy.

```
rules: {
    program: $ => repeat($._statement),

    _statement: $ => prec(1, choice(
      $._expression_statement,
      $._declaration,
      $._method_statement
    )),
```

### Deviating from the language spec

The `grammar.js` file follows the BNF grammar outlined in the [Java Language Specification](https://docs.oracle.com/javase/specs/jls/se9/html/jls-19.html). 

There are situations where we've deviated from the spec:

- **Prefered naming:** if common developer parlance prefers a naming convention other than the spec, we tend to deviate. An example of this is for `generic_type` as the outer wrapper for `type_arguments`, since generics are a familiar Java programming concept.
- **Simplicity:** The spec is convoluted and not conducive to compact, readable code. In this situation, we've preferred structuring things in a way that are more reusable throughout the grammar and also read clearly. An example of this is our preference to use `binary_` and `unary_` expressions to model relationships between operators, as opposed to supporting the spec's [`ConditionalExpression`](https://docs.oracle.com/javase/specs/jls/se9/html/jls-15.html#jls-ConditionalExpression) hierarchy.

### When it's okay to parse invalid Java

There are situations in which we parse invalid code to support end-user experiences. For example, it's important to ensure syntax-highlighting doesn't break down for a snippet of Java code in a markdown file. For this reason, we currently allow expressions to be parsed outside of methods, even though that is not valid Java.

To know what is "valid enough", consider what good documentation would look like:

- ✅ `int x = (1 + 2);` = This is invalid since it is not within a method, but still comprehensible. Parse this.
- ❌ `int x = (1 + ) =;` This is not only invalid Java, but it is invalid logic. It wouldn't make sense in documentation. Don't parse this.

### Adding unit tests

The recommendation is to be comprehensive in adding tests. If it's a visible node, add it to a `/corpus/` test file. It's typically a good idea to test as many permutations of a particular language construct as possible. This increases test coverage, but doubly acquaints readers with a way to examine expected outputs and understand the "edges" of a language.

### Testing on external repos

Three of the "most popular"[1] Java repositories have been cloned into the project under the `/test-repos` directory. Parsing these repos allows us to gauge how well our grammar performs at parsing "real world" Java.

_To test:_
- `./script/parse-examples.rb` runs the tests and outputs them to `known-errors.txt`, representing the files that have any errors or `MISSING ;` flags.
- The goal is to drive down the errors in `known-errors.txt` to 0.
- `known-errors.txt` allows you to find erroring files and parse them individually to diagnose and debug errors.

[1] Popularity defined by most starred + most amount of active contributors in the last month.
