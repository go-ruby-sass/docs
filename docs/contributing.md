# Contributing

Contributions are welcome. `go-ruby-sass/sass` is built to a small set of
non-negotiable rules — they keep it pure-Go, thin, and honest about what it
does and does not yet do.

## Hard rules

- **Build from source — no vendoring.** Everything compiles from source.
- **This module stays thin.** Sass/SCSS language work and the dart-sass
  differential gate belong in [go-scss/scss](https://github.com/go-scss/scss),
  not here. This module only maps Ruby keyword arguments to the engine's typed
  options.
- **100% test coverage target, enforced in CI.** New code ships with tests,
  and coverage is a CI gate. Fill the error branches, not just the happy path.
- **All GitHub content in English.** Issues, pull requests, commits and
  comments are English-only.
- **Pure Go, cgo disabled.** The whole point is a single static binary with no
  C toolchain, no `libsass`, no Dart VM. Code must build with `CGO_ENABLED=0`.
- **Honest residuals.** Document what does not work yet (e.g. `source_map`,
  `:nested`/`:compact` styles) instead of silently approximating it.

## Workflow

1. Pick or open an issue describing the change.
2. Work test-first: add tests, then make them pass.
3. Run the full suite with coverage and confirm the gate is green:

    ```sh
    COVERPKG=$(go list ./... | paste -sd, -)
    go test -race -coverpkg="$COVERPKG" -coverprofile=cover.out ./...
    go tool cover -func=cover.out | tail -1   # 100.0%
    ```

4. Open a PR in English, referencing the issue.

## Where things live

- The library is in [`github.com/go-ruby-sass/sass`](https://github.com/go-ruby-sass/sass).
- The underlying compiler is [`github.com/go-scss/scss`](https://github.com/go-scss/scss)
  — Sass/SCSS language changes belong there.
- The `rbgo` binding (`require "sass"`) lives in
  [`go-embedded-ruby/ruby`](https://github.com/go-embedded-ruby/ruby).
- This documentation site is in [`github.com/go-ruby-sass/docs`](https://github.com/go-ruby-sass/docs).
