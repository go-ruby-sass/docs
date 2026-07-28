# Getting started in rbgo

[`rbgo`](https://github.com/go-embedded-ruby) is the pure-Go, CGO=0 embedded
Ruby (mruby/YARV-style VM). It binds `go-ruby-sass/sass` as a native module —
`require "sass"` — so pure-Go Ruby can compile Sass/SCSS with no `libsass`, no
Dart VM, and no external `sass` binary anywhere in the process.

## `require "sass"`

```ruby
require "sass"

css = Sass.compile_string(<<~SCSS, style: :compressed)
  $accent: #DC2626;
  .card { color: $accent; .title { font-weight: bold; } }
SCSS

puts css
# => .card{color:#dc2626}.card .title{font-weight:bold}
```

`Sass.compile_string` returns the compiled CSS **as a `String` directly** — this
is the binding's one deliberate simplification versus the modern `sass-embedded`
gem, which returns a `Sass::CompileResult` object with `.css` / `.loaded_urls` /
`.source_map`. `loaded_urls` and `source_map` are not surfaced by the binding
today; see [Ruby API](api.md#residuals) for the underlying engine residuals.

## Module methods

| Method | What it does |
| --- | --- |
| `Sass.compile_string(source, **opts)` | compiles Sass/SCSS source text, returns CSS `String` |
| `Sass.compile(source, **opts)` | alias of `compile_string` in this binding — **it also takes source text, not a path** (see note below) |
| `Sass.compile_file(path, **opts)` | compiles the file at `path`, returns CSS `String` |

!!! warning "`Sass.compile` takes source text here, not a path"
    In the reference `sass-embedded` gem, `Sass.compile(path)` compiles a file
    and `Sass.compile_string(source)` compiles text. The `rbgo` binding wires
    **both** `Sass.compile` and `Sass.compile_string` to the same source-text
    entry point; use `Sass.compile_file(path, **opts)` to compile a file from
    Ruby. This is a binding-level naming residual, not a language limitation —
    the underlying Go adapter's `sass.Compile(path, opts)` is exactly the
    file-compiling call `Sass.compile_file` wraps.

## Keyword options

All three module methods accept the same trailing keyword-argument `Hash`:

| Keyword | Values | Default |
| --- | --- | --- |
| `syntax:` | `:scss`, `:indented` (or `:sass`), `:css` | `:scss` |
| `style:` | `:expanded`, `:compressed` | `:expanded` |
| `load_paths:` | an `Array` of directory path `String`s for `@use`/`@forward`/`@import` | none |

```ruby
Sass.compile_string(src, syntax: :indented, style: :compressed, load_paths: ["_sass"])
```

A compile failure raises **`Sass::CompileError`**, matching the gem's error
class.

## Legacy `sassc` surface

Real applications still target the older `sassc` gem's `SassC::Engine` API;
`rbgo` layers it over the same pure-Go engine:

```ruby
require "sass"

engine = SassC::Engine.new(".card { .title { font-weight: bold; } }", style: :compressed)
puts engine.render
# => .card .title{font-weight:bold}
```

`SassC::Engine.new(template, style:, syntax:)` accepts `style:` (`:expanded` /
`:compressed`; `:nested` and `:compact` fall back to `:expanded`) and `syntax:`
(`"scss"` default, or `"sass"`/`"indented"`). `#render` raises
**`SassC::SyntaxError`** on a compile failure, matching the gem's error class.

## Why this matters

Because both `go-ruby-sass/sass` and `rbgo` itself are pure Go with cgo
disabled, this whole path — parsing Ruby, running the VM, compiling Sass/SCSS —
builds into a single static binary with no C toolchain, and cross-compiles (and
compiles to WebAssembly) like any other pure-Go program.

See [Ruby API](api.md) for the full Go-level adapter surface this binding wraps.
