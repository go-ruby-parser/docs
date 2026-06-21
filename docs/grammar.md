# Grammar & limitations

go-ruby-parser accepts a broad, practical subset of Ruby 4.0, all
differential-tested against MRI 4.0.5.

## What it parses

- **Literals:** integers (`Bignum`/arbitrary precision, radix `0x`/`0o`/`0b`/`0d`,
  underscores), floats (incl. **scientific `1.5e3`**), strings (double- **and
  single-quoted**, interpolation, heredocs `<<`/`<<-`/`<<~`), symbols (incl.
  quoted/operator), `%w`/`%i`/`%W`/`%I` arrays, arrays, hashes, ranges (incl.
  beginless/endless), regexps (`/re/imx`), `true`/`false`/`nil`.
- **Operators:** arithmetic, comparison/`<=>`, `==`/`===`, bitwise/shift,
  `&&`/`||`/`and`/`or`/`not`, ternary, `::` scope, safe navigation `&.`,
  compound assignment (`+=`, `-=`, `*=`, `/=`, `%=`, `<<=`, `||=`, `&&=`).
- **Control flow:** `if`/`unless`/`while`/`until` (block and modifier),
  `case`/`when`, `case`/`in` **pattern matching** (array/find/hash/pin/
  alternative/range patterns, guards, one-line `=>`/`in`), `begin`/`rescue`/
  `else`/`ensure`/`retry`, `break`/`next`/`return`, `loop`.
- **Methods/blocks:** required/optional/`*splat`/keyword/`**rest`/`&block`
  params, endless methods (`def f = expr`), setters, operator/`[]`/`[]=` method
  names, `{ }` / `do…end` blocks, `(a, b)` destructuring group params, stabby
  lambdas `->(){}`, numbered params (`_1`) and `it`, `yield`, `super`.
- **Classes/modules/metaprogramming:** `class`/`module`, inheritance, `@ivars`,
  constants, `def self.`, multiple assignment / destructuring.

## Known limitations

These are not yet parsed (they remain on go-embedded-ruby's roadmap;
contributions welcome):

- multiple-value `return a, b` (a single expression only)
- class variables `@@cvar`, and global-variable **assignment** (`$g = …`)
- operator-method **calls** `1.+(2)`, hash-shorthand `{x:}`
- paren-less command calls with keyword/splat/block args (`foo a: 1`)
- adjacent string-literal concatenation (`"a" "b"`)
- splat/default **block** parameters (`{ |*a| }`, `{ |a = 1| }`), empty `||`
  block params, and the positional `Class(a)` find-pattern (`Class[a]` is
  supported)

## Errors

`Parse` returns a parse error carrying the line number; it never panics on
malformed input. Unterminated literals (strings, regexps, heredocs, `%`-arrays)
and structural mistakes are reported as parse errors.
