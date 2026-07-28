# go-ruby-sass documentation

**Pure-Go, CGO=0 Sass/SCSS compiler exposed as a Ruby-facing adapter — Dart-Sass-compatible CSS output.**

`go-ruby-sass/sass` is a **thin Ruby-facing adapter** over the pure-Go, CGO-free
Sass/SCSS compiler [`github.com/go-scss/scss`](https://github.com/go-scss/scss).
It exposes the modern **`sass-embedded`** gem surface (and the legacy **`sassc`**
surface) in a shape the [rbgo](https://github.com/go-embedded-ruby) binding wraps
directly, so pure-Go Ruby can compile Sass with **no cgo and no external `sass`
binary**. The module path is `github.com/go-ruby-sass/sass`.

All language work and the dart-sass differential gate live in
[go-scss/scss](https://github.com/go-scss/scss); this module only maps Ruby
keyword arguments to the engine's typed options and returns a result mirroring
the gem's `Sass::CompileResult`.

!!! success "Status: pure-Go, CGO=0, adapter over go-scss/scss"
    Two gem surfaces (`sass-embedded` and `sassc`) over one pure-Go engine, no
    C toolchain, `gofmt` + `go vet` clean.

## Install

```sh
go get github.com/go-ruby-sass/sass
```

## Quick example

```go
import sass "github.com/go-ruby-sass/sass"

res, err := sass.CompileString(`
$accent: #DC2626;
.card { color: $accent; .title { font-weight: bold; } }
`, &sass.Options{Style: sass.StyleCompressed})
// res.CSS == ".card{color:#dc2626}.card .title{font-weight:bold}\n"
```

## Repositories

| Repo | What it is |
| --- | --- |
| [`sass`](https://github.com/go-ruby-sass/sass) | the library — the Ruby-facing Sass/SCSS adapter in pure Go |
| [`docs`](https://github.com/go-ruby-sass/docs) | this documentation site (MkDocs Material, versioned with mike) |
| [`go-ruby-sass.github.io`](https://github.com/go-ruby-sass/go-ruby-sass.github.io) | the organization landing page (Hugo) |
| [`brand`](https://github.com/go-ruby-sass/brand) | logo and brand assets |

## Principles

- **Pure Go, `CGO_ENABLED=0`** — no `libsass`, no Dart VM, no external `sass`
  binary; a single static Go binary compiles Sass/SCSS.
- **Two gem surfaces, one engine.** `sass-embedded` (`Sass.compile_string` /
  `Sass.compile`) and the legacy `sassc` (`SassC::Engine#render`) both compile
  through the same [go-scss/scss](https://github.com/go-scss/scss) engine.
- **Standalone & reusable.** No dependency on the Ruby runtime — the dependency
  runs the other way; `rbgo` binds this module as `require "sass"`.
- **Thin by design.** This module maps Ruby keyword arguments to the engine's
  typed options; all Sass/SCSS language semantics live in `go-scss/scss`.

## Where to go next

- [Ruby API](api.md) — the `sass-embedded` and `sassc` surfaces, and how Ruby
  keyword arguments map onto the Go options.
- [Getting started in rbgo](rbgo.md) — using `require "sass"` from pure-Go Ruby.

Source lives at [github.com/go-ruby-sass/sass](https://github.com/go-ruby-sass/sass).
