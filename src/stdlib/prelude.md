# Prelude

The prelude is bound automatically before any script runs. It's small
and entirely written in ff on top of the native built-ins — you can
read the source in `src/prelude/stdlib.ff`.

## Pipe and composition

```ff
(|>) = (x, f) => f(x)               # x |> f         ==  f(x)
(<<) = (f, g) => x => f(g(x))       # (f << g)(x)    ==  f(g(x))
(>>) = (f, g) => x => g(f(x))       # (f >> g)(x)    ==  g(f(x))
```

```ff
[1, 2, 3] |> map(x => x * x) |> reduce((a, b) => a + b, 0)   # 14

double_then_inc = inc << double
inc_then_double = inc >> double
```

## Collection combinators

All operate on lists, cons-spines, ranges, and strings (where it makes
sense — `map` over a string maps per-character cons).

### `map(f, xs)`

```ff
map(x => x * 2, [1, 2, 3])         # [2, 4, 6]
```

### `filter(p, xs)`

```ff
filter(x => x % 2 == 0, [1..=10])  # [2, 4, 6, 8, 10]
```

### `reduce(f, init, xs)`

Left fold:

```ff
reduce((a, b) => a + b, 0, [1, 2, 3])    # 6
```

Pipe-friendly: `xs |> reduce(f, 0)`.

### `fold(f, init, xs)`

Right fold (not tail-recursive — use `reduce` for large inputs unless
right-associativity matters):

```ff
fold((x, acc) => x :: acc, [], [1, 2, 3])  # [1, 2, 3]
```

### `take(n, xs)`

Returns the first `n` elements as a list. Works on lazy ranges and
infinite sequences:

```ff
[0..] |> filter(x => x % 2 == 0) |> take 5      # [0, 2, 4, 6, 8]
```

### `take_while(p, xs)`

Same as `take` but instead of returning the first `n` elements,
we return elements until the predicate `p` evaluates to `false`.

```ff
[0..] |> take_while (n => n < 5)      # [0, 1, 2, 3, 4]
```

### `rev(xs)`

Reverses a sequence into a list:

```ff
rev([1, 2, 3])                     # [3, 2, 1]
```

## `assert`

```ff
assert(1 == 1)                     # 1
assert([1 == 2, "msg"])            # panic: msg
```

Takes either a bare value or a `[val, msg]` pair. Panics with `msg` (or
`"assert failed"`) when `val` equals its `default`.

## `panic`

```ff
panic "boom"
```

Aborts the current evaluation with the given message. There's no
recovery — see [Error Handling](../language/errors.md) for the
convention you should normally use instead.

## `typeof` and `default`

```ff
typeof 42                          # :number
typeof "hi"                        # :string
typeof [1, 2]                      # :list
typeof :ok                         # :atom

default(42)                        # 0
default("hi")                      # ""
default([1, 2])                    # []
default({:k: 1})                   # {}
```

`default` is unsupported for atoms, functions, and pids — those don't
have a meaningful zero.

## `print` and `println`

```ff
print("no newline")
println("with newline")
```

Both write to stdout. In the REPL, `print` also adds a trailing newline
for readability; in script mode it doesn't.

## Operator natives

Built-in infix operators are bound in the prelude as regular functions
named with their operator characters. `1 + 2` and `(+)(1)(2)` are the
same call. The set:

| Name | Arity | Notes                                                        |
|------|-------|--------------------------------------------------------------|
| `+`  | 2     | number-only addition. Use `::` to concatenate strings.       |
| `-`  | 2     | number subtraction.                                          |
| `*`  | 2     | number multiplication.                                       |
| `/`  | 2     | rational division. Division by `0` errors.                   |
| `%`  | 2     | modulo. Modulo by `0` errors.                                |
| `**` | 2     | exponentiation. Integer exponents only.                      |
| `==` | 2     | structural equality across all values.                       |
| `!=` | 2     | inverse of `==`.                                             |
| `<`, `<=`, `>`, `>=` | 2 | ordering on numbers or strings.                    |
| `~`  | 2     | substring containment for strings.                           |
| `!~` | 2     | inverse of `~`.                                              |
| `::` | 2     | strict cons. Use the `a :: b` syntax for the lazy form.      |

Because they're regular bindings, you can pass them around, partially
apply them, or shadow them:

```ff
plus = (+)
[1, 2, 3] |> reduce(plus, 0)        # 6

inc = (+)(1)                        # partial application
inc(10)                             # 11
```

The three operators with non-strict semantics — `&&`, `||`, and the
infix-syntax `::` — are **not** ordinary bindings: they're special
forms so they can short-circuit (`&&`/`||`) or capture the rhs lazily
(`::`). Their function counterparts (`(::)`) are strict.

## Pre-imported modules

`Object` and `Actor` are bound by the prelude — you can use them
without an explicit `import`.
