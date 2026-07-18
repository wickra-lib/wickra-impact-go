# Wickra Impact — Go

[![CI](https://github.com/wickra-lib/wickra-impact/actions/workflows/ci.yml/badge.svg)](https://github.com/wickra-lib/wickra-impact/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/wickra-lib/wickra-impact/branch/main/graph/badge.svg)](https://codecov.io/gh/wickra-lib/wickra-impact)
[![Go module](https://raw.githubusercontent.com/wickra-lib/.github/main/profile/badges/go.svg)](https://pkg.go.dev/github.com/wickra-lib/wickra-impact-go)
[![License: MIT OR Apache-2.0](https://img.shields.io/badge/license-MIT_OR_Apache--2.0-blue)](https://github.com/wickra-lib/wickra-impact#license)

**Go bindings for the Wickra Impact market-impact backtester over its C ABI hub via cgo. An `Impact` is built from a spec JSON and driven over a JSON boundary, so the result is byte-identical to every other Wickra Impact binding.**

## Install

Use the published **`wickra-impact-go`** module, which bundles the prebuilt C ABI
library for every platform, so `go get` + `go build` works with no extra steps
(a C compiler is still required, as the binding uses cgo):

```bash
go get github.com/wickra-lib/wickra-impact-go
```

```go
import wickra "github.com/wickra-lib/wickra-impact-go"
```

`wickra-impact-go` is generated from the [`bindings/go`](https://github.com/wickra-lib/wickra-impact/tree/main/bindings/go)
directory of [wickra-impact](https://github.com/wickra-lib/wickra-impact) by the release
pipeline: it mirrors the Go sources, the vendored C ABI header (`include/wickra_impact.h`)
and the prebuilt libraries under `lib/<goos>_<goarch>/`. On Linux/macOS the
library path is baked in via rpath; on Windows the DLL must be discoverable at
run time (next to the executable or on `PATH`).

### Building from this repository (contributors)

The `bindings/go` directory in the main repository is the development source. To
build against a locally compiled C ABI, build the hub and stage the library into
the per-platform directory cgo links against:

```bash
cargo build -p wickra-impact-c --release
mkdir -p lib/linux_amd64                          # match your GOOS_GOARCH
cp target/release/libwickra_impact.so    lib/linux_amd64/    # Linux
cp target/release/libwickra_impact.dylib lib/darwin_arm64/   # macOS (arm64)
cp target/release/wickra_impact.dll      lib/windows_amd64/  # Windows
```

## Quick start

```go
package main

import (
	"fmt"

	wickra "github.com/wickra-lib/wickra-impact-go"
)

func main() {
	spec := `{"strategy":{"spec_version":1,"symbol":"IMPACT","timeframe":"1h",` +
		`"indicators":{},"entry":{"ge":[{"price":"close"},0]},"exit":{"in_position":true},` +
		`"sizing":{"type":"fixed_qty","qty":10.0},` +
		`"execution":{"order_type":"market","fill_timing":"next_open"}},` +
		`"book_model":{"kind":"orderbook_walk"},"participation_cap":1.0,"latency_ms":0}`

	impact, err := wickra.New(spec)
	if err != nil {
		panic(err)
	}
	defer impact.Close()

	resp, err := impact.Command(`{"cmd":"run","data":` + data + `}`)
	if err != nil {
		panic(err)
	}
	fmt.Println(resp) // the report carries the market impact a naive backtest hides
}
```

## Documentation

The full guides, quickstarts and API reference live in the main repository and
documentation site:

- **Repository:** <https://github.com/wickra-lib/wickra-impact>
- **Docs:** <https://docs.wickra.org>

Wickra ships native bindings for Python, Node.js, WASM and Rust, plus a C ABI hub
that any C-capable language (C, C++, C#, Go, Java, R) links against — all exposing
the same core from the shared, `unsafe`-forbidden Rust core.

## Security

Found a security issue? **Please don't open a public issue.** Report it privately
via the affected repository's *Security* tab (*"Report a vulnerability"*) or email
**support@wickra.org** with a subject line starting `[wickra security]`. Full
policy: <https://github.com/wickra-lib/wickra-impact/blob/main/SECURITY.md>.

## Disclaimer

Wickra Impact is research and analytics software. Its outputs are
deterministic transforms of the input data — they are not financial advice and do
not predict the market. Any use in a live trading context is at your own risk. The
software is provided **as is**, without warranty of any kind.

## License

Licensed under either of [Apache-2.0](https://github.com/wickra-lib/wickra-impact/blob/main/LICENSE-APACHE)
or [MIT](https://github.com/wickra-lib/wickra-impact/blob/main/LICENSE-MIT) at your option.
