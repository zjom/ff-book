# Operators

ff has a small set of built-in operators and lets you define your own.
Most built-ins are just **regular prelude bindings** named with operator
characters — `(+)`, `(*)`, `(==)`, etc. The parser desugars `a + b` to
`(+)(a)(b)`, so the operator forms and the function forms are
interchangeable. The only operators with bespoke evaluation are `&&`,
`||`, and `::`, which have non-strict semantics.

## Built-in operators

### Arithmetic

| Op       | Meaning                | Precedence (tightest first) |
|----------|------------------------|-----------------------------|
| `**`     | exponent (right-assoc) | 1                           |
| `*`, `/`, `%` | mul, div, mod     | 2                           |
| `+`, `-` | add, sub               | 3                           |

All arithmetic is on exact rationals. `/` never truncates. `+` is
number-only — use `::` to concatenate strings.

### Comparison

`==`, `!=`, `<`, `<=`, `>`, `>=` work on numbers, strings, atoms, and
structurally on collections (`[1, 2] == [1, 2]`, `{1, 2, 3} == {3, 2, 1}`).

`~` tests substring containment for strings; `!~` is its negation.

### Logical

`&&`, `||`, `!`. `&&` and `||` short-circuit — they are special forms,
not regular functions.

### Cons

`::` prepends an element to a sequence:

| Left           | Right          | Result                                   |
|----------------|----------------|------------------------------------------|
| any value `v`  | list `xs`      | list with `v` at the head                |
| string `a`     | string `b`     | concatenation                            |
| any value `v`  | set `xs`       | set with `v` inserted                    |
| `[k, v]`       | object `o`     | object with `k → v` inserted             |

Cons is the canonical way to build sequences incrementally; combined
with [pattern matching](./patterns.md), it forms the basic shape of
list-processing recursion. The right operand is captured lazily, which
is what lets recursive stream builders like `f(x) :: map(f, rest)`
terminate.

### Prefix

`-` (numeric negation) and `!` (logical not). Both are single-character
so they stack: `--5`, `!!true`.

## Operator precedence at a glance

From tightest to loosest:

1. function application (`f x`, `f(x)`), `.access`
2. prefix `-`, `!`
3. `**`
4. `*`, `/`, `%`
5. `+`, `-`
6. `^…`, `@…`  (custom "concat" ops)
7. `==`, `!=`, `<`, `<=`, `>`, `>=`, `~`, `!~` (and custom `<…`/`>…`/`=…`/`!…`/`$…`)
8. `&&` (and custom `&…`)
9. `||` (and custom `|…`)
10. `::` and the prelude pipe `|>`

Inside each level, associativity follows the leading character — `**`
right-associates; the rest left-associate.

## Custom operators

Any sequence of operator characters can be a user-defined infix.
Precedence is **OCaml-style**, picked from the first character: ops
starting with `*`/`/`/`%` bind tighter than ones starting with `+`/`-`,
which bind tighter than ones starting with `=`/`<`/`>`, and so on.

```ff
(<|>) = (x, y) => if x != default(x) then x else y
"" <|> "fallback"             # "fallback"
0  <|> 42                     # 42
```

Prefix operators start with `?` or `~` and must be at least two characters:

```ff
(~?) = x => default(x)
~?[1, 2, 3]                   # []
```

Operator characters: `! $ % & * + - / < = > ? @ ^ | ~`. The `:` character
is reserved (atoms, `::`), and `=>` / `->` are reserved (function and
match arrows).

### Using operators as values

Wrap any operator name in parens to use it like an ordinary identifier.
This works uniformly for built-ins and user-defined operators — they're
the same kind of thing:

```ff
plus = (+)
[1, 2, 3] |> reduce(plus, 0)         # 6

inc = (+)(1)                         # partial application
inc(10)                              # 11

cons = (::)
cons(0, [1, 2, 3])                   # [0, 1, 2, 3]
```

You can also shadow a built-in the same way you'd shadow any binding:

```ff
(+) = (a, b) => a - b
2 + 3                                # -1
```

This works in both expressions and patterns. To export a custom
operator from a module, use the parenthesized form: `export (<|>)`.

## Pipes and composition (prelude)

Defined in the prelude as ordinary functions, not built-ins:

```ff
(|>) = (x, f) => f(x)
(<<) = (f, g) => x => f(g(x))
(>>) = (f, g) => x => g(f(x))
```

`|>` is the workhorse for chained transformations:

```ff
[1..=10]
  |> filter(x => x % 2 == 0)
  |> map(x => x * x)
  |> reduce((a, b) => a + b, 0)
```
