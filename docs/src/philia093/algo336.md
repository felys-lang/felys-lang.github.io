# Algorithm 3.36

Finite automata come in two types: deterministic (DFA) and non-deterministic (NFA). While DFAs are generally faster, NFAs allow more flexible transitions and is more extensive. However, our main parsing strategy is based on PEG, which already supports infinite backtracking. Therefore, the lexer only needs to recognize simple lexemes and efficiency matters, so DFAs are preferred for lexical analysis. Algorithm 3.36 introduced by the Dragon Book can be used to construct DFAs directly.

## Introduction to Language

This is a highly academic topic and too lengthy for this book, so I will briefly explain it in plain language. In short, a formal language is a set of strings over a given alphabet, and a regular expression is a syntactic tool used to describe certain types of formal languages, specifically regular languages. If `s` is a symbol from an alphabet, then `L(s) = {s}` is a language consisting of a single string. Let `c1` and `c2` be languages. The core operations used in regular expressions are:

1. `(c1)|(c2)`: denotes the union of `c1` and `c2`, i.e., strings in either `c1` or `c2`
2. `(c1)(c2)`: denotes concatenation, i.e., strings formed by `c1` followed by `c2`
3. `(c)*`: denotes the Kleene star, i.e., zero or more repetitions of strings from `c`
4. `(c)`: parentheses are used for grouping and are semantically neutral

**Note**: All these are still languages, so that induction works.

## Prerequisite

Before generating them, we need to compute `followpos` using `nullable`, `firstpos`, and `lastpos`. To make this table look nicer, I will omit all `pos` postfixes. Note that parenthesis is not mentioned, because it only change precedence, so we can just recursively call into it.

| Node: `n`       | `nullable(n)`                    | `first(n)`                                              | `last(n)`                                            |
| --------------- | -------------------------------- | ------------------------------------------------------- | ---------------------------------------------------- |
| Position: `i`   | `false`                          | `{i}`                                                   | `{i}`                                                |
| Union: `c1\|c2` | `nullable(c1) \|\| nullable(c2)` | `first(c1) \| first(c2)`                                | `last(c2) \| last(c1)`                               |
| Concat: `c1c2`  | `nullable(c1) && nullable(c2)`   | `first(c1) \| first(c2) if nullable(c1) else first(c1)` | `last(c2) \| last(c1) if nullable(c2) else last(c2)` |
| Kleene: `c*`    | `true`                           | `first(c)`                                              | `last(c)`                                            |

Once we have these formulae, we can compute `followpos`, which can be represented as a graph `G(V, E)`, where `V` is the set of positions, and `E` is the set of edges corresponding to `followpos` relationships. To compute this, we need to traverse the syntax tree. There are only two cases in which one position can be made to follow another:

1. Concat(`c1c2`): all positions in `first(c2)` are in `follow(i)` for `i` in `last(c1)`
2. Kleene(`c*`): all positions in `first(c)` are in `follow(i)` for `i` in `last(c)`

If your use case involves constructing very large transition table, it would be helpful to cache all the results of `nullable`, `firstpos`, and `lastpos`. For Felys, the transition tables are usually small, so there is no need to bring extra overhead.

## Transition Table Construction

Here are the syntax tree nodes:

```rust,noplayground
enum Language {
    Union(Box<Language>, Box<Language>),
    Concat(Box<Language>, Box<Language>),
    Kleene(Box<Language>),
    Nested(Box<Language>),
    Position(Position, usize),
}

enum Position {
    Set(Vec<(usize, usize)>),
    Pound,
}
```

This is pretty straight forward, but I do want clarify two designs here. The `usize` in `Language::Position` is its label or `i` mentioned in the table. Every position must have a unique `i`. Secondly, instead of a single character, we want to use `Position::Set` that represent all acceptable characters for this position. This is a necessary modification to make this algorithm practical, because we can treat a range of characters as a whole. For instances (pseudocode only):

- single character `'0'` is equivalent to `[('0', '0')]`
- inclusive set `[0-9]` is equivalent to `[('0', '9')]`
- exclusive set `[^0-9]` is equivalent to `[(MIN, '/'), (':', MAX)]`

### Core Algorithm

Before building the transition table, we need to append a special pound symbol at the end to identify the terminal state. It's recommended to do this at the syntax tree level rather than directly appending it to the regular expression. Once the syntax tree is built and the pound is appended, we can compute the `followpos` graph, and also collect all the position indices.

Then we can use apply the following algorithm (the original pseudocode):

```text
initialize Dstates to contain only the unmarked state firstpos(n0),
    where n0 is the root of syntax tree T for (r)#;
while ( there is an unmarked state S in Dstates ) {
    mark S;
    for ( each input symbol a ) {
        let U be the union of followpos(p) for all p
            in S that correspond to a;
        if ( U is not in Dstates )
            add U as an unmarked state to Dstates;
        Dtran[S, a] = U;
    }
}
```

The accepting states are those that contain the pound symbol.

### Improve the Algorithm

However, when performing `each input symbol a`, there are millions of Unicode characters, so it's not practical to iterate through all of them. The previously mentioned modification, `Position::Set`, is designed to address this issue. Nevertheless, it introduces another problem: to compute `U`, we cannot simply use a set to store the data, because the ranges may overlap. Therefore, additional splitting and merging are required to produce the minimal number of atomic ranges. There are three steps involved: compute boundaries, create atomic ranges, and select valid ranges.

Here's an implementation in Rust:

```rust,noplayground
let mut saturated = false;
let mut boundaries = Vec::with_capacity(symbols.len() * 2);
for &(start, end) in &symbols {
    if end == usize::MAX {
        saturated = true;
    }
    boundaries.push(start);
    boundaries.push(end.saturating_add(1));
}
boundaries.sort_unstable();
boundaries.dedup();

let mut atomic = boundaries
    .windows(2)
    .map(|x| (x[0], x[1].saturating_sub(1)))
    .collect::<Vec<_>>();
if saturated {
    atomic.last_mut().unwrap().1 = usize::MAX;
}

let ranges = atomic
    .into_iter()
    .filter(|x| {
        symbols
            .iter()
            .any(|&(start, end)| start <= x.0 && x.1 <= end)
    })
    .collect::<Vec<_>>();
```

Then we can safely iterate through `ranges` instead of all unicode characters.

### Representation

Unlike standard transition table represented in matrix, our version takes range and transfer into another state. There are many ways to do it, but for Rust we can use a `match` expression with multiple `(state, start..=end) => state` ended with a `_ => break` inside a loop. Once the loop breaks, we can check for the acceptance of the state.

## More Information

Finite automata is a well-studied topic in computer science, and I have only covered the tip of the iceberg. If you're interested, please refer to the Dragon Book. It contains an entire section on lexical analysis techniques, presented in highly academic languages.
