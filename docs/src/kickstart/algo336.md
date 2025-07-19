# Algorithm 3.36

Finite automata come in two types: deterministic (DFA) and non-deterministic (NFA). While DFAs are generally faster, NFAs allow more flexible transitions and is more extensive. However, our main parsing strategy is based on PEG, which supports infinite backtracking. Since the lexer only needs to recognize simple lexemes and efficiency is important, DFAs are preferred for lexical analysis. Algorithm 3.36 introduced by the Dragon Book can be used to construct DFAs directly.

## Introduction to Language

This is a highly academic topic and too lengthy for this book, so I will briefly explain it in plain language. In short, a formal language is a set of strings over a given alphabet, and a regular expression is a syntactic tool used to describe certain types of formal languages, specifically regular languages. If `s` is a symbol from an alphabet, then `L(s) = {s}` is a language consisting of a single string. Let `c1` and `c2` be languages. The core operations used in regular expressions are:

1. `(c1)|(c2)`: denotes the union of `c1` and `c2`, i.e., strings in either `c1` or `c2`
2. `(c1)(c2)`: denotes concatenation, i.e., strings formed by `c1` followed by `c2`
3. `(c)*`: denotes the Kleene star, i.e., zero or more repetitions of strings from `c`
4. `(c)`: parentheses are used for grouping and are semantically neutral

**Note**: All these are still languages, so that induction works.

## Preparator

Before generating them, we need to compute `followpos` using `nullable`, `firstpos`, and `lastpos`. To make this table look nicer, I will remove all `pos` postfixes. Note that parenthesis is not mentioned, because it only change precedence, so we can just recursively call into it.

| Node: `n`       | `nullable(n)`                    | `first(n)`                                              | `last(n)`                                            |
| --------------- | -------------------------------- | ------------------------------------------------------- | ---------------------------------------------------- |
| Position: `i`   | `false`                          | `{i}`                                                   | `{i}`                                                |
| Union: `c1\|c2` | `nullable(c1) \|\| nullable(c2)` | `first(c1) \| first(c2)`                                | `last(c2) \| last(c1)`                               |
| Concat: `c1c2`  | `nullable(c1) && nullable(c2)`   | `first(c1) \| first(c2) if nullable(c1) else first(c1)` | `last(c2) \| last(c1) if nullable(c2) else last(c2)` |
| Kleene: `c*`    | `true`                           | `first(c)`                                              | `last(c)`                                            |

Once we have these formulae, we can compute `followpos`, which can be represented as a graph `G(V, E)`, where `V` is the set of positions, and `E` is the set of edges corresponding to `followpos` relationships. To compute this, we need to traverse the syntax tree. There are only two cases in which one position can be made to follow another:

1. Concat(`c1c2`): all positions in `first(c2)` are in `follow(i)` for `i` in `last(c1)`
2. Kleene(`c*`): all positions in `first(c)` are in `follow(i)` for `i` in `last(c)`

If your usecase involves constructing very large transition table, it would be helpful to cache all the reasults of `nullable`, `firstpos`, and `lastpos`. For Felys, the transition table are usually small, so there is no need to bring extra overhead.

## Construction

Before building everything, we need to append a special pound symbol to the end of the language so that we can identify the terminal state. It is recommended to do this at the syntax tree level, rather than appending it directly to the regular expression.
