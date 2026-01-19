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
layout: two-cols
class: text-left
---

# Motiff AI: Query + Transform

<div v-click="1">
Match imports to verify required packages are present.
</div>

<div v-click="2" class="mt-3">
Transform `export default function App` → `function App` (when the default export is a function).
</div>

<div v-click="3" class="mt-3">
Remove `export default App` (when it’s a symbol), then append `root.render(&lt;App /&gt;)` so the snippet renders as expected.
</div>

::right::

<div class="h-full flex items-center justify-center">
<div class="w-full max-w-md">

<div v-click="1" class="text-left">

```ts
import { createRoot } from 'react-dom/client'
```

</div>

<div v-click="2" class="text-left">

```ts {1}
export default function App() { /* ... */ }

export default App;
```

</div>

<div v-click="3" class="text-left">

```ts {3}
export default function App() { /* ... */ }

export default App;
```

</div>

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
