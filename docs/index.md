# go-ruby-parser documentation

**A pure-Go (CGO=0) Ruby front-end** — a stateful lexer, a recursive-descent +
Pratt parser, and a typed AST. No cgo, no Prism, no shelling out to Ruby: the
front-end the Go ecosystem lacked for building Ruby tooling (linters, formatters,
analysers, doc generators, LSP servers, transpilers) in pure Go.

It was extracted from the [go-embedded-ruby](https://github.com/go-embedded-ruby/ruby)
interpreter — which now consumes it — and is developed test-first against MRI
Ruby 4.0.5. **100% coverage** is enforced in CI on all three OS lanes (ubuntu,
macOS, windows); the suite additionally runs on all **six 64-bit targets** —
`amd64` and `arm64` natively, `riscv64`/`loong64`/`ppc64le`/`s390x` under QEMU —
and on `js/wasm` and `wasip1/wasm`.

**Current release: [v0.2.0](https://github.com/go-ruby-parser/parser/tree/v0.2.0)**
(2026-09-21), which closed fourteen grammar gaps — see
[Grammar & limitations](grammar.md#what-v020-added).

## Install

```sh
go get github.com/go-ruby-parser/parser@v0.2.0
```

## Usage

```go
import "github.com/go-ruby-parser/parser"

prog, err := parser.Parse(`
def fib(n)
  n < 2 ? n : fib(n - 1) + fib(n - 2)
end
puts fib(20)
`)
if err != nil {
    // err is a parse error carrying a line number
}
// prog is an *ast.Program; walk prog.Body ([]ast.Node).
```

## Packages

| Import | Package | What it is |
| --- | --- | --- |
| `github.com/go-ruby-parser/parser` | `parser` | the entry point: `Parse(src string) (*ast.Program, error)` |
| `github.com/go-ruby-parser/parser/ast` | `ast` | the syntax-tree node types |
| `github.com/go-ruby-parser/parser/lexer` | `lexer` | the stateful lexer (MRI `spaceSeen` semantics) |
| `github.com/go-ruby-parser/parser/token` | `token` | the token kinds |

## Design notes

- **Stateful lexer.** Ruby's grammar is context-sensitive; the lexer tracks
  `SpaceBefore` (MRI's `spaceSeen`) plus a state seed so the same characters lex
  differently by context (`foo -1` is a command call, `foo - 1` a subtraction).
- **Scope-aware parser.** A scope stack resolves Ruby's central ambiguity — a
  bare `x` is a local-variable reference if assigned in scope, otherwise a method
  call on the implicit receiver.

See [Grammar & limitations](grammar.md) for the full surface and the known gaps.
