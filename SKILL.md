---
name: go-1-26
description: Go 1.26 release features and migration guide. Use when writing Go code targeting Go 1.26, upgrading from earlier Go versions, using new Go 1.26 APIs (new() with expressions, self-referential generics, Green Tea GC, errors.AsType, crypto/hpke, simd, reflect iterators, io.ReadAll improvements, slog.NewMultiHandler), or when the user mentions Go 1.26.
metadata:
  author: luoling8192
  version: "2026.03.12"
  source: https://go.dev/doc/go1.26
---

# Go 1.26 Release Features

Quick reference for Go 1.26 new features, API changes, and migration notes.

## Language Changes

### `new()` Accepts Expressions

```go
// Before: needed helper function or temp variable
v := 42
ptr := &v

// Go 1.26: direct expression in new()
ptr := new(42)
Age: new(yearsSince(born))  // useful for optional pointer fields
```

### Self-Referential Generic Types

```go
type Adder[A Adder[A]] interface {
    Add(A) A
}

func algo[A Adder[A]](x, y A) A {
    return x.Add(y)
}
```

## Runtime

### Green Tea GC (Default)

- 10-40% reduction in GC overhead for real-world programs
- +10% on newer amd64 CPUs (Ice Lake+, Zen 4+) via vector instructions
- Disable: `GOEXPERIMENT=nogreenteagc` (removal expected Go 1.27)

### Heap Base Address Randomization

- 64-bit platforms randomize heap base at startup (security)
- Disable: `GOEXPERIMENT=norandomizedheapbase64`

### Faster cgo (~30% overhead reduction)

### Goroutine Leak Profile (Experimental)

- Enable: `GOEXPERIMENT=goroutineleakprofile`
- New `goroutineleak` profile type in `runtime/pprof`
- HTTP endpoint: `/debug/pprof/goroutineleak`
- GC-based detection of goroutines blocked on unreachable primitives

## Tools

### `go fix` Revamped

- Complete overhaul with modernizers framework
- Source-level inliner with `//go:fix inline` directives
- Built on Go analysis framework

### `go mod init` Defaults

- Now defaults to `go 1.(N-1).0` (e.g., Go 1.26 creates `go 1.25.0`)
- Override with `go get go@version`

### Pprof

- Web UI `-http` flag now defaults to flame graph view

## Compiler & Linker

- **Stack allocation**: Slice backing store on stack in more situations
- **windows/arm64**: Internal linking mode for cgo (`-ldflags=-linkmode=internal`)
- `cmd/doc` removed (use `go doc`)

## New Standard Library Packages

### `crypto/hpke`

Hybrid Public Key Encryption (RFC 9180) with post-quantum hybrid KEM support.

### `simd/archsimd` (Experimental)

```go
// Enable: GOEXPERIMENT=simd
// amd64 only: 128/256/512-bit vector types
var a, b archsimd.Int8x16
result := a.Add(b)
```

### `runtime/secret` (Experimental)

Secure erasure of cryptographic temporaries. Enable: `GOEXPERIMENT=runtimesecret`. amd64/arm64 Linux only.

## Key Standard Library Changes

### `errors`

```go
// New generic, type-safe, faster version of As
var target *MyError
if err, ok := errors.AsType[*MyError](err); ok {
    // use err directly - no need for As + target variable
}
```

### `io`

- `ReadAll()` ~2x faster, ~50% less memory allocation, returns minimal-sized slice

### `reflect` (Iterators)

```go
for i, f := range reflect.TypeFor[MyStruct]().Fields() {
    fmt.Println(f.Name)
}
for i, m := range reflect.TypeFor[MyInterface]().Methods() {
    fmt.Println(m.Name)
}
// Also: Type.Ins(), Type.Outs(), Value.Fields(), Value.Methods()
```

### `log/slog`

```go
// New multi-handler
h := slog.NewMultiHandler(jsonHandler, textHandler)
logger := slog.New(h)
```

### `bytes`

```go
// New Peek method - read without advancing
data := buf.Peek(10)
```

### `net`

```go
// New typed Dial methods with context
d := &net.Dialer{}
conn, err := d.DialTCP(ctx, "tcp", addr)
conn, err := d.DialUDP(ctx, "udp", addr)
conn, err := d.DialIP(ctx, "ip4:icmp", addr)
conn, err := d.DialUnix(ctx, "unix", addr)
```

### `net/http`

- `ServeMux` trailing slash redirects: now **307** (was 301)
- New `HTTP2Config.StrictMaxConcurrentRequests`
- New `Transport.NewClientConn()`
- `httptest.Server.Client()` redirects `example.com` to test server

### `net/url`

- `Parse()` rejects malformed `http://::1/` (use `http://[::1]/`)
- Restore: `GODEBUG=urlstrictcolons=0`

### `os`

- New `Process.WithHandle()` (pidfd on Linux 5.4+, Handle on Windows)
- `OpenFile` accepts Windows-specific flags

### `testing`

```go
// New ArtifactDir for test output files
dir := t.ArtifactDir()
os.WriteFile(filepath.Join(dir, "debug.log"), data, 0644)
```

### `runtime/metrics`

- `/sched/goroutines` (by state), `/sched/threads:threads`, `/sched/goroutines-created:goroutines`

### `fmt`

- `Errorf("x")` allocates less (matches `errors.New("x")`)

### `image/jpeg`

- Encoder/decoder replaced with faster, more accurate implementations
- **Breaking**: bit-for-bit output may differ

## Crypto Changes

### Post-Quantum TLS (Default)

- `SecP256r1MLKEM768` and `SecP384r1MLKEM1024` enabled by default
- Disable: `GODEBUG=tlssecpmlkem=0` or `Config.CurvePreferences`

### Random Parameter Changes

Many crypto functions now **ignore** the `random` parameter and always use secure randomness:
- `crypto/dsa.GenerateKey`
- `crypto/ecdh.Curve.GenerateKey`
- `crypto/ecdsa.GenerateKey`, `SignASN1`, `Sign`
- `crypto/rand.Prime`
- `crypto/rsa.GenerateKey`, `EncryptPKCS1v15`

Testing: use `testing/cryptotest.SetGlobalRandom`. Restore: `GODEBUG=cryptocustomrand=1`.

### Deprecations

- `crypto/rsa`: PKCS #1 v1.5 functions (`EncryptPKCS1v15`, `DecryptPKCS1v15`)
- `crypto/ecdsa`: `PublicKey`/`PrivateKey` `big.Int` fields
- `net/http/httputil`: `ReverseProxy.Director` (use `Rewrite`)

## GODEBUG Removals in Go 1.27

These GODEBUG settings are deprecated and will be removed:
- `tlsunsafeekm`, `tlsrsakex`, `tls10server`, `tls3des`
- `x509keypairleaf`, `gotypesalias`, `asynctimerchan`

## Port Changes

- **macOS**: Last release for macOS 12 Monterey (Go 1.27 requires 13+)
- **windows/arm** (32-bit): Removed
- **linux/riscv64**: Race detector now supported
- **WebAssembly**: Significantly reduced memory for heaps <16 MiB
- **linux/ppc64**: Last release for ELFv1 ABI

## Bootstrap

- Requires Go 1.24.6+ to build
- Go 1.28 will require Go 1.26+
