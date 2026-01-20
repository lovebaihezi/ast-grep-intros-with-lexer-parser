---
layout: section
background: https://source.unsplash.com/1600x900/?circuit,blueprint
---

# Part 3

DFA helps build the Lexer + Parser

<!--
Motivation: Ast-Grep is only as good as the tree it matches on.
-->

---
layout: default
class: text-left
---

# The DFA(match the `a@b.c`)

<div class="text-sm opacity-90">

**DFA = (Q, Σ, δ, q0, F)**  
For `a@b.c`: `Q={S0,S1,S2,S3,S4,end}`, `Σ={a,@,b,.,c}`, `q0=S0`, `F={end}` (δ = the edges).

</div>

<div class="mt-3">

```typst
#import "@preview/fletcher:0.5.8" as fletcher: diagram, node, edge

#html.frame(diagram(
  spacing: 6.4em,
  node-stroke: 1pt,
  node((0, 0), [S0], shape: circle, radius: 1.25em, extrude: (0, 2)),
  node((1, 0), [S1], shape: circle, radius: 1.25em),
  node((0, 1), [S2], shape: circle, radius: 1.25em),

  node((1, 1), [S3], shape: circle, radius: 1.25em),
  node((2, 1), [S4], shape: circle, radius: 1.25em),
  node((3, 1), [end], shape: circle, radius: 1.25em),

  edge((0, 0), (1, 0), [`a`], "-|>"),

  edge((1, 0), (0, 1), [`@`], "-|>"),

  edge((0, 1), (1, 1), [`b`], "-|>"),

  edge((1, 1), (2, 1), [`.`], "-|>"),

  edge((2, 1), (3, 1), [`c`], "-|>"),
))
```

</div>

<div class="mt-4 text-xs">
  <div class="flex items-center gap-2">
    <span class="opacity-70 w-14">state</span>
    <code data-id="dfa-state-s0" class="px-2 py-1 rounded bg-gray-100">S0</code>
    <span class="opacity-70 ml-4">capture</span>
    <code class="px-2 py-1 rounded bg-gray-100">()</code>
  </div>

  <div v-click="1" class="flex items-center gap-2 mt-2">
    <span class="opacity-70 w-14">read</span>
    <code class="px-2 py-1 rounded bg-gray-100">a</code>
    <span class="opacity-70">→ state</span>
    <code data-id="dfa-state-s1" class="px-2 py-1 rounded bg-gray-100">S1</code>
    <span class="opacity-70 ml-4">capture</span>
    <code class="px-2 py-1 rounded bg-gray-100">("a")</code>
  </div>

  <div v-click="2" class="flex items-center gap-2 mt-2">
    <span class="opacity-70 w-14">read</span>
    <code class="px-2 py-1 rounded bg-gray-100">@</code>
    <span class="opacity-70">→ state</span>
    <code data-id="dfa-state-s2" class="px-2 py-1 rounded bg-gray-100">S2</code>
    <span class="opacity-70 ml-4">capture</span>
    <code class="px-2 py-1 rounded bg-gray-100">("a", "@")</code>
  </div>

  <div v-click="3" class="flex items-center gap-2 mt-2">
    <span class="opacity-70 w-14">read</span>
    <code class="px-2 py-1 rounded bg-gray-100">b</code>
    <span class="opacity-70">→ state</span>
    <code data-id="dfa-state-s3" class="px-2 py-1 rounded bg-gray-100">S3</code>
    <span class="opacity-70 ml-4">capture</span>
    <code class="px-2 py-1 rounded bg-gray-100">("a", "@", "b")</code>
  </div>

  <div v-click="4" class="flex items-center gap-2 mt-2">
    <span class="opacity-70 w-14">read</span>
    <code class="px-2 py-1 rounded bg-gray-100">.</code>
    <span class="opacity-70">→ state</span>
    <code data-id="dfa-state-s4" class="px-2 py-1 rounded bg-gray-100">S4</code>
    <span class="opacity-70 ml-4">capture</span>
    <code class="px-2 py-1 rounded bg-gray-100">("a", "@", "b.")</code>
  </div>

  <div v-click="5" class="flex items-center gap-2 mt-2">
    <span class="opacity-70 w-14">read</span>
    <code class="px-2 py-1 rounded bg-gray-100">c</code>
    <span class="opacity-70">→ state</span>
    <code data-id="dfa-state-end" class="px-2 py-1 rounded bg-gray-100">end</code>
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
We introduce the math definition directly on the example graph:

- DFA = (Q, Σ, δ, q0, F)
- Q: {S0, S1, S2, S3, S4, end}
- Σ: {a, @, b, ., c}
- q0: S0
- F: {end}
- δ: the arrows, e.g. δ(S1, '@') = S2

Acceptance:
for w = a@b.c, δ*(S0, w) = end ∈ F.

This slide only shows the concrete path (no self-loops). General matching just means “add more states / more transitions”.

[click] Read `a` → move from S0 to S1, capture `("a")`.

[click] Read `@` → move from S1 to S2, capture `("a", "@")`.

[click] Read `b` → move from S2 to S3, capture `("a", "@", "b")`.

[click] Read `.` → move from S3 to S4, capture `("a", "@", "b.")`.

[click] Read `c` → move from S4 to end (accept), capture `("a", "@", "b.c")`.

[click] Read `EOF` → accept if we are in an accept state.

Connect back to lexer/parser:
- lexer transition function: `switch (cp)`
- parser transition function: `switch (state, token)`
-->

---
layout: two-cols
class: text-left
---

# Lexer and Token

- Token = `item + span`
- `TokenItem` = `union(enum)` (tagged union)
- Lexer stays “dumb”: emits `Sign('#')`, not “Title”

::right::

```zig
pub const TokenItemTag = enum {
  Tab, Space, LineEnd, Sign, AsciiNumber, Str, EOF,
}

pub const TokenItem = union(TokenItemTag) {
  Tab: void, Space: void, LineEnd: void,
  Sign: u8,                // '#', '*', '`', ...
  AsciiNumber: []const u8,  // slice into buffer
  Str: []const u8,          // slice into buffer
  EOF: void,
}

Token = { item: TokenItem, span: Span }
```

Example:

```txt
Input: "# Hello\n"
→ [ Sign('#'), Space, Str("Hello"), LineEnd, EOF ]
```

<!--
Lexer’s job: turn a byte buffer into a stream of tokens (context-free).

In mdz (`mdz/src/lexer.zig`):
- input: `[]const u8` buffer
- iteration: `std.unicode.Utf8Iterator` gives us codepoints (so Unicode is “first-class”)
- output: `Token { item: TokenOrError, span: Span }`

Design philosophy:
- lexer is intentionally “dumb”: it does not decide semantics
- it just emits surface categories: `Sign('#')`, `Space`, `Str(...)`, `LineEnd`, ...
- this makes it fast and easy to test

Output invariant: every token has an exact `span`.

This token stream is the only input the parser needs.

Token sequence intuition:
- the lexer never says “Title”; it only says `Sign('#')`
- the parser later decides whether `Sign('#') Space ... LineEnd` means Title or just text
-->

---
layout: two-cols
class: text-left
---

# Parser design (LL(k)-style DFA)

- Pull-based: `Parser.next()` pulls tokens from lexer
- Lookahead until one block is done (`Done`)
- State = `union(enum)` + payload + (optional) recover state

::right::

```zig
pub fn next(self: *Self, lexer: *Lexer) !?Block {
  var state = State.empty(self.allocator)
  while (lexer.next()) |value| {
    try DFA.f(&state, value.item.ok, value.span)
    if (state.state == .Done) return state.value
  }
  return null
}
```

<div class="mt-6 text-xs opacity-70">
Read more: <a href="https://notes.lqxclqxc.com/posts/write-full-dfa-in-code/" target="_blank">write-full-dfa-in-code</a>
</div>

<!--
Parser’s job: consume tokens (from the dumb lexer) and produce structured blocks (context-aware).

What makes it “LL(k)-style” in practice:
- the parser controls the loop (pull model)
- it keeps state + payload, and effectively “looks ahead” until a block boundary is confirmed
- when a block is complete, it returns early (so the caller can stream blocks)

Tiny trace for `# Hello\n`:
- Empty + Sign('#') → MaybeTitle(1)
- MaybeTitle(1) + Space → MaybeTitleContent(1)
- MaybeTitleContent(1) + Str("Hello") → (init Title block) → Done on LineEnd

Edge case intuition (`#Invalid`):
- MaybeTitle(1) + Str("Invalid") → not a valid title (missing Space) → recover to NormalText

DFA elements in code:
- Q: `StateKind` (enum of parser states)
- δ: `DFA.f` + `switch (token.item)` dispatch (transition function)
- payload: extra memory to keep it deterministic (levels, spans, buffers, counters)

This is the reason Ast-Grep works well later:
stable structure + precise spans → safe matching and rewriting.
-->

---
layout: center
class: p-0
---

```typst
#import "@preview/fletcher:0.5.8" as fletcher: diagram, node, edge

#let state-node(pos, label, ..args) = node(
  pos,
  label,
  shape: rect,
  corner-radius: 10pt,
  inset: 10pt,
  ..args,
)

#html.frame(diagram(
  spacing: (5.6em, 3.8em),
  node-stroke: 1pt,
  state-node((0, 0), [Empty($q_0$)], extrude: (0, 2)),
  state-node((1, 0), [MaybeTitle(1)]),
  state-node((1, 1), [MaybeTitleContent(1)]),
  state-node((0, 1), [NormalText (Title)]),
  state-node((0, 2), [Done]),

  edge((0, 0), (1, 0), [`Sign('#')`], "-|>"),
  edge((1, 0), (1, 1), [`Space`], "-|>"),
  edge((1, 1), (0, 1), [`Str("Hello")`], "-|>"),
  edge((0, 1), (0, 2), [`LineEnd`], "-|>"),
))
```

<!--
Parser DFA trace for `# Hello\n`:
- the lexer yields surface tokens
- the parser transitions through a small set of states until the block boundary is confirmed

After reaching `Done`, `Parser.next()` returns a block (Title(level=1, text="Hello")) and stops early.
-->
