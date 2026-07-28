# Ruby API

`go-ruby-sass/sass` exposes two Ruby gem surfaces over the same pure-Go
[go-scss/scss](https://github.com/go-scss/scss) engine.

## `sass-embedded` (`Sass`)

The modern surface — `Sass.compile_string` / `Sass.compile` — used by current
Dart-Sass-based tooling.

```go
import sass "github.com/go-ruby-sass/sass"

// Sass.compile_string(source, syntax:, style:, load_paths:, importer:)
res, err := sass.CompileString(src, &sass.Options{
    Syntax:    sass.SyntaxSCSS,      // scss | indented | css
    Style:     sass.StyleCompressed, // expanded | compressed
    LoadPaths: []string{"_sass"},
})
_ = res.CSS        // => css
_ = res.LoadedURLs // => loaded_urls
_ = res.SourceMap  // => source_map (empty; see Residuals)

// Sass.compile(path, ...)
res, err = sass.Compile("styles/main.scss", nil)
```

`CompileString` is the exact entry point **Jekyll's `jekyll-sass-converter`**
calls — that path is covered by a dedicated test in the library.

## `sassc` (`SassC::Engine`)

The legacy surface, layered over the same engine, since real applications
still use both the modern `sass-embedded` and the older `sassc` API.

```go
import "github.com/go-ruby-sass/sass/sassc"

// SassC::Engine.new(template, style: :compressed).render
css, err := sassc.NewEngine(template, sassc.EngineOptions{
    Style: sassc.StyleCompressed,
}).Render()
```

## Ruby-to-Go mapping

This is the table the `rbgo` binding uses to wire `require "sass"` onto this
adapter:

| Ruby | Go |
| --- | --- |
| `Sass.compile_string(src, **kw)` | `sass.CompileString(src, opts)` |
| `Sass.compile(path, **kw)` | `sass.Compile(path, opts)` |
| result `.css` / `.loaded_urls` / `.source_map` | `CompileResult.CSS` / `.LoadedURLs` / `.SourceMap` |
| `syntax:` / `style:` / `load_paths:` | `Options.Syntax` / `.Style` / `.LoadPaths` |
| custom importer | `Options.Importer func(url) (src, canonical, ok)` |
| `SassC::Engine.new(t, style:).render` | `sassc.NewEngine(t, opts).Render()` |

### Types

- `Syntax` — `SyntaxSCSS` (`"scss"`), `SyntaxIndented` (`"indented"`),
  `SyntaxCSS` (`"css"`). `CompileString` honors it directly; `Compile` infers
  it from the file extension unless overridden.
- `Style` — `StyleExpanded` (`"expanded"`), `StyleCompressed` (`"compressed"`).
- `Options.Importer` — resolves `@use` / `@forward` / `@import` URLs, mirroring
  a custom Ruby importer: given a URL it returns the source, a canonical URL,
  and whether it handled the request.
- `sassc.Style` additionally accepts `StyleNested` and `StyleCompact` for
  source compatibility with `SassC::Engine`; both fall back to `expanded`
  output (see Residuals below).

## Residuals

Honest gaps versus a full `sass-embedded` / `sassc` gem, inherited from the
current state of the underlying engine:

- `source_map` is not yet emitted (the engine does not produce source maps
  yet).
- `sassc` `:nested` / `:compact` styles fall back to `expanded` (the engine
  emits `expanded` and `compressed` only).
- Output fidelity is exactly that of the underlying engine — see the
  [go-scss/scss residuals](https://github.com/go-scss/scss#honest-residuals)
  (notably CSS Color 4 percentage serialization for some computed colors).

See [Getting started in rbgo](rbgo.md) for how this surfaces as `require "sass"`.
