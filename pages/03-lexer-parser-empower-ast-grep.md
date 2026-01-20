---
layout: section
background: https://source.unsplash.com/1600x900/?circuit,blueprint
---

# Part 3

Lexer + Parser empower Ast-Grep

<!--
Motivation: Ast-Grep is only as good as the tree it matches on.
-->

---
layout: default
class: text-left
---

# The DFA

<div class="mt-3">

```typst
#import "@preview/fletcher:0.5.8" as fletcher: diagram, node, edge

#html.frame(diagram(
  spacing: 6.4em,
  node-stroke: 1pt,
  node((0, 0), [S0], shape: circle, radius: 1.25em, extrude: (0, 2)),
  node((1, 0), [S1], shape: circle, radius: 1.25em),
  node((2, 0), [S2], shape: circle, radius: 1.25em),

  node((2, 1), [S3], shape: circle, radius: 1.25em),
  node((3, 1), [S4], shape: circle, radius: 1.25em),
  node((4, 1), [S5 (accept)], shape: circle, radius: 1.25em),

  edge((0, 0), (1, 0), [`[A-Za-z0-9]`], "-|>"),
  edge((1, 0), (1, 0), [`[A-Za-z0-9]`], "--|>", bend: 120deg),

  edge((1, 0), (2, 0), [`@`], "-|>"),

  edge((2, 0), (2, 1), [`[A-Za-z0-9]`], "-|>"),
  edge((2, 1), (2, 1), [`[A-Za-z0-9]`], "--|>", bend: 120deg),

  edge((2, 1), (3, 1), [`.`], "-|>"),

  edge((3, 1), (4, 1), [`[A-Za-z0-9]`], "-|>"),
  edge((4, 1), (4, 1), [`[A-Za-z0-9]`], "--|>", bend: -120deg),
))
```

</div>

<div class="mt-4 text-xs">
  <div class="flex items-center gap-2">
    <span class="opacity-70 w-14">state</span>
    <code class="px-2 py-1 rounded bg-gray-100">S0</code>
    <span class="opacity-70 ml-4">capture</span>
    <code class="px-2 py-1 rounded bg-gray-100">()</code>
  </div>

  <div v-click="1" class="flex items-center gap-2 mt-2">
    <span class="opacity-70 w-14">read</span>
    <code class="px-2 py-1 rounded bg-gray-100">a</code>
    <span class="opacity-70">→ state</span>
    <code class="px-2 py-1 rounded bg-gray-100">S1</code>
    <span class="opacity-70 ml-4">capture</span>
    <code class="px-2 py-1 rounded bg-gray-100">("a")</code>
  </div>

  <div v-click="2" class="flex items-center gap-2 mt-2">
    <span class="opacity-70 w-14">read</span>
    <code class="px-2 py-1 rounded bg-gray-100">@</code>
    <span class="opacity-70">→ state</span>
    <code class="px-2 py-1 rounded bg-gray-100">S2</code>
    <span class="opacity-70 ml-4">capture</span>
    <code class="px-2 py-1 rounded bg-gray-100">("a", "@")</code>
  </div>

  <div v-click="3" class="flex items-center gap-2 mt-2">
    <span class="opacity-70 w-14">read</span>
    <code class="px-2 py-1 rounded bg-gray-100">b</code>
    <span class="opacity-70">→ state</span>
    <code class="px-2 py-1 rounded bg-gray-100">S3</code>
    <span class="opacity-70 ml-4">capture</span>
    <code class="px-2 py-1 rounded bg-gray-100">("a", "@", "b")</code>
  </div>

  <div v-click="4" class="flex items-center gap-2 mt-2">
    <span class="opacity-70 w-14">read</span>
    <code class="px-2 py-1 rounded bg-gray-100">.</code>
    <span class="opacity-70">→ state</span>
    <code class="px-2 py-1 rounded bg-gray-100">S4</code>
    <span class="opacity-70 ml-4">capture</span>
    <code class="px-2 py-1 rounded bg-gray-100">("a", "@", "b.")</code>
  </div>

  <div v-click="5" class="flex items-center gap-2 mt-2">
    <span class="opacity-70 w-14">read</span>
    <code class="px-2 py-1 rounded bg-gray-100">c</code>
    <span class="opacity-70">→ state</span>
    <code class="px-2 py-1 rounded bg-gray-100">S5</code>
    <span class="opacity-70 ml-4">capture</span>
    <code class="px-2 py-1 rounded bg-gray-100">("a", "@", "b.c")</code>
  </div>

  <div v-click="6" class="flex items-center gap-2 mt-2">
    <span class="opacity-70 w-14">read</span>
    <code class="px-2 py-1 rounded bg-gray-100">EOF</code>
    <span class="opacity-70 ml-4 text-green-600 font-semibold">✅ accept</span>
  </div>
</div>

<!--
We only need one mental model for Part 3:

Slide goals:
- show the DFA shape (nodes + labeled transitions)
- show how “current state” changes as input is consumed
- show that state can carry extra data (captures/spans)

Key model:
- DFA = deterministic state machine
- read **1 input symbol** → take **1 transition**
- update current state, repeat until EOF

Regex example (simplified):
`[A-Za-z0-9]+@[A-Za-z0-9]+\.[A-Za-z0-9]+`

1) We are always in *one* current state.
2) We read *one* input symbol.
3) We follow *one* outgoing edge labeled by that symbol.
4) That edge lands us in the next state.

This is the “DFA view” of the world:
- at any time, you are in *exactly one* state
- given the next input symbol, you deterministically jump to the next state

Regex engines note (why “regex = DFA” is not always true in programming):
- Many real-world regex engines support “rewind/backtracking” features, so they are *not* a pure DFA at runtime.
- But you can still *compile* (a subset of) regex to a DFA for linear-time matching.

Example (simplified email-ish regex):
`[A-Za-z0-9]+@[A-Za-z0-9]+\.[A-Za-z0-9]+`

Slide note: the `+` is shown as a self-loop: once you enter a state, you can keep consuming more of the same class.

[click] Read `a` → move from S0 to S1, capture `("a")`.

[click] Read `@` → move from S1 to S2, capture `("a", "@")`.

[click] Read `b` → move from S2 to S3, capture `("a", "@", "b")`.

[click] Read `.` → move from S3 to S4, capture `("a", "@", "b.")`.

[click] Read `c` → move from S4 to S5 (accept), capture `("a", "@", "b.c")`.

[click] Read `EOF` → accept if we are in an accept state.

And in mdz, we directly implement this “transition table” style:
- lexer (`mdz/src/lexer.zig`): `switch (codepoint)` decides token kinds
- parser (`mdz/src/dfa/lib.zig`): `switch (token)` updates parsing state
-->

---
layout: two-cols
class: text-left
---

# Lexer and Token

- Bytes → UTF-8 → codepoints
- Classify codepoints into *tokens*
- Each token carries a `span` (where it came from)

::right::

```zig
Token = { item, span }
item ∈ { Sign, Str, AsciiNumber, ... }
```

<!--
Lexer’s job: turn a byte buffer into a stream of tokens.

In mdz (`mdz/src/lexer.zig`):
- input: `[]const u8` buffer
- iteration: `std.unicode.Utf8Iterator` gives us codepoints (so Unicode is “first-class”)
- output: `Token { item: TokenOrError, span: Span }`

Token design (what matters conceptually):
- `TokenItem` is a small union of “surface categories”:
  - whitespace: `Tab`, `Space`, `LineEnd`
  - punctuation: `Sign(u8)` (like '#', '*', '`', '[', '(', ...)
  - text payload: `Str([]const u8)` (a slice of the original buffer)
  - numbers: `AsciiNumber([]const u8)`
  - end marker: `EOF`
- `span` stores (begin, len), so we can recover the exact source slice later.

Why the big `switch` works well:
- the transition is “current codepoint → token kind”
- fast path for ASCII (numbers, punctuation, whitespace)
- a tight loop to coalesce “normal unicode text” into one `Str` token (so the parser sees fewer tokens)

This token stream is the only input the parser needs.
-->

---
layout: two-cols
class: text-left
---

# Parser and AST

- Parser consumes tokens, maintains a state machine
- Output is tree-ish nodes (AST/MIR) with spans
- Ast-Grep matches on this structure (not raw text)

::right::

```txt
tokens → DFA(state, token) → Block/AST
```

<!--
Parser’s job: consume the token stream and build structured nodes.

In mdz, the parsing loop is straightforward (`mdz/src/parser.zig`):
- call `lexer.next()` repeatedly
- for each token, call `DFA.f(&state, token, span)`
- when state reaches `Done`, emit one `mir.Block`

The important part is the state machine (`mdz/src/dfa/state/state.zig`):
- `StateKind` enumerates “where we are” (e.g. `MaybeTitle`, `MaybeFencedCodeContent`, `NormalText`, ...)
- `StateItem` stores any extra data needed to continue (levels, spans, counters, etc.)
- the state is *stateful*, but still follows the same DFA abstraction:
  `next_state = f(cur_state, token)`

AST/MIR output (`mdz/src/mir/lib.zig`):
- nodes like `Title`, `Paragraph`, `Code`, `ThematicBreak`, ...
- nodes store spans back into the source buffer

Why this matters for Ast-Grep:
- once you have a tree, matching is about “syntax shape”
- spans let you report precise locations, rewrite safely, and preserve formatting where needed
-->
