# Grammar & limitations

go-ruby-parser accepts a broad, practical subset of Ruby 4.0, all
differential-tested against MRI 4.0.5. This page describes **v0.4.0**
(2026-09-26); the grammar gaps closed in v0.2.0 are listed under *What v0.2.0 added*
below.

## What it parses

- **Literals:** integers (`Bignum`/arbitrary precision, radix `0x`/`0o`/`0b`/`0d`,
  underscores), floats (incl. **scientific `1.5e3`**), strings (double- **and
  single-quoted**, interpolation, heredocs `<<`/`<<-`/`<<~`, `%q`/`%Q` literals,
  the `\a`/`\b`/`\v`/`\f`/`\s`/`\n`/`\t`/`\r`/`\e`/`\0` escapes), symbols (incl.
  quoted/operator), `%w`/`%i`/`%W`/`%I` arrays, arrays, hashes (incl. the `{x:}`
  value-shorthand), ranges (incl. beginless/endless), regexps (`/re/imx`),
  `true`/`false`/`nil`.
- **Operators:** arithmetic, comparison/`<=>`, `==`/`===`, bitwise/shift,
  `&&`/`||`/`and`/`or`/`not`, ternary, `::` scope, safe navigation `&.`,
  compound assignment (`+=`, `-=`, `*=`, `/=`, `%=`, `<<=`, `||=`, `&&=`).
- **Control flow:** `if`/`unless`/`while`/`until` (block and modifier),
  `case`/`when`, `case`/`in` **pattern matching** (array/find/hash/pin/
  alternative/range patterns, guards, one-line `=>`/`in`), `begin`/`rescue`/
  `else`/`ensure`/`retry`, `break`/`next`/`return`, `loop`.
- **Methods/blocks:** required/optional/`*splat`/keyword/`**rest`/`&block`
  params, endless methods (`def f = expr`), setters, operator/`[]`/`[]=` method
  names, **operator-method calls** (`1.+(2)`), `{ }` / `do…end` blocks,
  `(a, b)` destructuring group params, stabby lambdas `->(){}`, numbered params
  (`_1`) and `it`, `yield`, `super`, **multiple-value `return a, b`**.
- **Classes/modules/metaprogramming:** `class`/`module`, inheritance, `@ivars`,
  **`@@class variables`**, constants, singleton method defs
  (`def self.foo`/`def obj.foo`/`def Const.foo`), **global-variable assignment**
  (`$g = …`), multiple assignment / destructuring, **adjacent string-literal
  concatenation** (`"a" "b"`).

## What v0.2.0 added

v0.2.0 (2026-09-21) closed fourteen gaps. Its own measure: **every one of the 68
files in the pinned ruby/spec `language/` corpus now parses** — thirteen of them
had not parsed at all before, so they had been contributing nothing to either
column of any conformance measurement.

- `%`-literal delimiters that are themselves significant — interpolation, `=`,
  `\` inside `%w`/`%W`/`%i`/`%I`/`%q`/`%Q`
- `class`, `module` and `def` as **primaries**, so they can be an operand
  (`class C; end.foo`)
- **dynamic-symbol** names in `alias` and `undef` (`alias :"b" :"a"`)
- `$=` as a global variable name
- `defined?(expr)` over a full expression, and the jump keywords
  (`break`/`next`/`return`/`redo`/`retry`) in expression position
- `:a=` immediately before `=>`
- `{` nesting inside a lambda body
- a **spaced argument list** distinguished from a brace block, and **nested
  destructuring** block params (`|a, (b, (c, d))|`)
- **block-local variables** (`{ |x; y| }`), carried on `ast.Block`
- four pattern-matching gaps: a trailing comma in an array pattern, the
  parenthesised `Const(…)` pattern, `in {"a": 0}`, and `in ^@a`
- a **parenthesised receiver** as a multiple-assignment target
- `for` loop variables beyond a bare name (`for @v in …`, `for (i, j), k in …`)

Two of the fourteen were **silent mis-parses** rather than refusals, which is
worse, because nothing diagnoses them:

```ruby
o.s (:a){ 1 }          # the brace block bound to :a, not to the call
case [0,1,2,3]
in [0, 1, ]            # the trailing comma was dropped, so a partial pattern
  :partial             # became an exact one and this fell through to else
else
  :exact
end
```

## Known limitations

The three limitations this page listed before v0.2.0 — paren-less command calls
with keyword/splat/block args, splat and default **block** parameters, and the
positional `Class(a)` find-pattern — **all parse now**. Re-verified against
**v0.4.0** with `parser.Parse`, alongside a control that must fail (`BEGIN { }`),
which did.

What is left, from a 41-construct differential sweep against MRI 4.0.5 in which
this was the only disagreement, and **0 over-permissive**:

- **`BEGIN { }` / `END { }` blocks** do not parse. MRI accepts them.

An earlier revision of this page named four files in the pinned ruby/spec
`language/` corpus — `for_spec`, `block_spec`, `defined_spec` and `variables_spec`
— as parsing here but stopping later in go-embedded-ruby's compiler. **That no
longer reproduces:** re-run on 2026-09-26 all four execute and report results
(29, 163, 256 and 114 passing examples respectively).

## Errors

`Parse` returns a parse error carrying the line number; it never panics on
malformed input. Unterminated literals (strings, regexps, heredocs, `%`-arrays)
and structural mistakes are reported as parse errors.
