# book for the ff language

[live](https://zjom.github.io/ff)

[ff repo](https://github.com/zjom/ff-book)


## running locally

this book is built using [mdbook](https://rust-lang.github.io/mdBook/).

[mdbook installation guide](https://rust-lang.github.io/mdBook/guide/installation.html)


```sh
# for live reload development
mdbook serve 

# to build the book
mdbook build


# to test the snippets
cargo clean
cargo build
mdbook test -L target/debug/deps
```
