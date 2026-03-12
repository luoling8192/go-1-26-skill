# go-1-26-skill

Go 1.26 release features skill for [Claude Code](https://claude.com/claude-code) / [skills.sh](https://skills.sh).

## Install

```bash
npx skills install luoling8192/go-1-26-skill
```

## What's Included

A comprehensive quick reference for Go 1.26 covering:

- **Language**: `new()` with expressions, self-referential generic types
- **Runtime**: Green Tea GC (default), heap randomization, cgo 30% faster, goroutine leak profiling
- **New packages**: `crypto/hpke`, `simd/archsimd`, `runtime/secret`
- **Stdlib highlights**: `errors.AsType[T]`, `io.ReadAll` 2x faster, `reflect` iterators, `slog.NewMultiHandler`, `net.Dialer` typed methods, `testing.ArtifactDir`
- **Crypto**: Post-quantum TLS by default, random parameter changes
- **Migration**: GODEBUG deprecations, `net/url` strict parsing, `image/jpeg` output changes, port removals

## Usage

Once installed, the skill triggers automatically when you:

- Write Go code targeting Go 1.26
- Upgrade a project from an earlier Go version
- Use new Go 1.26 APIs
- Ask about Go 1.26 features

## Source

Based on the official [Go 1.26 Release Notes](https://go.dev/doc/go1.26).

## License

[MIT](LICENSE)
