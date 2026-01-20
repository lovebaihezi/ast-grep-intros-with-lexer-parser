---
layout: section
background: https://source.unsplash.com/1600x900/?developer,code
---

# Part 1

How we use Ast-Grep

<!--
We have two simple, repeatable usages (in Paraflow, and previously in Motiff AI): replace “one-off” regular expressions that become hard to read and even harder to maintain as they grow.

Think of it as three verbs:
- Grep: find code by structure (not by text)
- Query: extract the info you care about from matches
- Transform: rewrite code safely (codemods / migrations / fixes)
-->

---
layout: default
class: text-left
---

# Motiff AI: Query + Transform

<div class="grid grid-cols-2 gap-x-10 gap-y-6 mt-6 items-start">
<div v-click="1">
Match imports to verify required packages are present.
</div>

<div v-click="1" class="text-left text-sm">

```ts
import { createRoot } from 'react-dom/client'
```

</div>

<div v-click="2">
Transform <code>export default function App</code> → <code>function App</code> (when the default export is a function).
</div>

<div v-click="2" class="text-left text-sm">

```ts
export default function App() {
  /* ... */
}

// ↓
function App() {
  /* ... */
}
```

</div>

<div v-click="3">
Remove <code>export default App</code> (when it’s a symbol), then append <code>root.render(&lt;App /&gt;)</code> so the snippet renders as expected.
</div>

<div v-click="3" class="text-left text-sm">

```ts
function App() { /* ... */ }

export default App

// ↓
const root = createRoot(document.getElementById('root')!)
root.render(<App />)
```

</div>
</div>

<!--
Motiff AI: Query + Transform

[click] Match imports to verify required packages are present.

[click] Transform `export default function App` → `function App` (when the default export is a function).

[click] Remove `export default App` (when it’s a symbol), then append `root.render(<App />)` so the snippet renders as expected.
-->

---
layout: two-cols-header
class: text-left
layoutClass: paraflow-usage
---

# Paraflow: Query directives in comments

::left::

As Paraflow focuses on code generation, we constantly need to query patterns inside code.

For example, directives embedded in comments:

```tsx
// ...
```

and

```tsx
{/* ... */}
```

With regex, supporting both forms usually turns into complex loops and fragile capture logic.

::right::

<div class="h-full flex items-start justify-center">
<div class="wrap-code w-full max-w-lg">

<div v-show="$clicks === 0" class="text-left">

```tsx
const features = [
    { image: '../a.png' }, // generate-image -p "A" -s 1:1 -o 'public/a.png'
    { image: '../b.png' }, // generate-image -p "B" -s 3:2 -o 'public/b.png'
]

function App() {
    return <>
      <img src='../c.png' alt='C' /> {/* generate-image -p 'C' -s 2:3 -o 'public/c.png' */}
    </>
}
```

</div>

<div v-click="1" v-show="$clicks === 1" class="text-left">

```tsx {2-3,8}
const features = [
    { image: '../a.png' }, // generate-image -p "A" -s 1:1 -o 'public/a.png'
    { image: '../b.png' }, // generate-image -p "B" -s 3:2 -o 'public/b.png'
]

function App() {
    return <>
      <img src='../c.png' alt='C' /> {/* generate-image -p 'C' -s 2:3 -o 'public/c.png' */}
    </>
}
```

```js
/\/\/\s*generate-image\b[^\n]*$|\{\s*\/\*\s*generate-image\b[\s\S]*?\*\/\s*\}/gm
```

</div>

<div v-click="2" v-show="$clicks === 2" class="text-left">

```tsx {2-3,8}
const features = [
    { image: '../a.png' }, // generate-image -p "A" -s 1:1 -o 'public/a.png'
    { image: '../b.png' }, // generate-image -p "B" -s 3:2 -o 'public/b.png'
]

function App() {
    return <>
      <img src='../c.png' alt='C' /> {/* generate-image -p 'C' -s 2:3 -o 'public/c.png' */}
    </>
}
```

```bash
ast-grep --pattern '$COMMENT' --lang javascript
```

</div>

<div v-click="3" v-show="$clicks >= 3" class="text-left text-xs">

```jsonc
[
  {
    "text": "// generate-image -p \"A\" -s 1:1 -o 'public/a.png'",
    // ...
  },
  {
    "text": "// generate-image -p \"B\" -s 3:2 -o 'public/b.png'",
    // ...
  },
  {
    "text": "/* generate-image -p 'C' -s 2:3 -o 'public/c.png' */",
    // ...
  }
]
```

</div>

</div>
</div>

<style>
.paraflow-usage .col-left pre {
  margin-top: 0.9rem;
  margin-bottom: 0.9rem;
}
.paraflow-usage .col-left p {
  margin-top: 0.6rem;
  margin-bottom: 0.6rem;
}
.wrap-code pre {
  white-space: pre-wrap !important;
  word-break: break-word;
}
.wrap-code code {
  white-space: pre-wrap !important;
  word-break: break-word;
}
</style>

<!--
As Paraflow focuses on code generation, we constantly need to query patterns inside code—for example, directives embedded in comments.

That means we need to support both `// ...` and `{/* ... */}`. With regex, this usually turns into complex loops and fragile capture logic.

[click] With regex, you end up writing one “do-everything” pattern (and still needing careful post-processing).

[click] With Ast-Grep, we can match comments structurally (it doesn’t matter whether they’re `//` or `{/* */}`).

[click] The result is structured data (range/text), which is much easier to post-process than regex captures.
-->

---
layout: default
class: text-left
---

# Paraflow: Advanced patterns (sketch)

<div class="grid grid-cols-2 gap-x-10 gap-y-6 mt-6 items-start text-sm">
<div v-click="1">
AST match + text constraint: migrate only `track("pf_*")` calls.
</div>

<div v-click="1" class="text-left">

```yaml
rule:
  pattern: track($NAME, $DATA)
  constraints:
    NAME:
      regex: '^"pf_'
fix: telemetry.track($NAME, $DATA)
```

</div>

<div v-click="2">
Named function + specific args: rewrite API shapes safely.
</div>

<div v-click="2" class="text-left">

```yaml
rule:
  pattern: generateImage($PROMPT, { ratio: $RATIO })
fix: generateImage({ prompt: $PROMPT, ratio: $RATIO })
```

</div>

<div v-click="3">
Context match: enforce rules only in certain scopes (e.g. inside React effects).
</div>

<div v-click="3" class="text-left">

```yaml
rule:
  inside:
    pattern: useEffect(() => { $BODY }, $DEPS)
  pattern: fetch($URL)
fix: request($URL)
```

</div>
</div>

<div v-click="4" class="mt-6 text-lg font-semibold">
And ask AI to generate patterns.
</div>

<!--
These are “recipe” patterns we can grow into real project rules:

[click] Combine structural match with a regex constraint so we only touch the calls we mean to touch.

[click] Match a specific function name + argument shape, then rewrite to a new API signature.

[click] Use `inside:` to limit enforcement/migrations to a precise scope.

[click] And ask AI to generate patterns, then we review + tighten constraints.
-->
