---
layout: section
background: https://source.unsplash.com/1600x900/?maze,abstract
---

# Part 2

Where we need it

<!--
Rule of thumb: use Ast-Grep when you need structure.
-->

---
layout: two-cols-header
class: text-left
---

# 1) When semantics matters

Text search is blind to code structure. In Paraflow we often need matching that is:

- scoped (only inside a specific construct)
- immune to strings/comments
- resilient across formatting

::right::

<div v-show="$clicks === 0" class="text-left">

```tsx
// "fetch('/api/user')" in comment
const s = "fetch('/api/user')" // string

useEffect(() => {
  fetch('/api/user')
}, [])

client.fetch('/api/user') // different meaning
```

</div>

<div v-click="1" v-show="$clicks === 1" class="text-left">

```tsx {1-2}
// "fetch('/api/user')" in comment
const s = "fetch('/api/user')" // string

useEffect(() => {
  fetch('/api/user')
}, [])

client.fetch('/api/user') // different meaning
```

```js
/fetch\\(.*\\)/g
```

</div>

<div v-click="2" v-show="$clicks >= 2" class="text-left">

```tsx {5}
// "fetch('/api/user')" in comment
const s = "fetch('/api/user')" // string

useEffect(() => {
  fetch('/api/user')
}, [])

client.fetch('/api/user') // different meaning
```

```yaml
rule:
  inside:
    pattern: useEffect(() => { $BODY }, $DEPS)
  pattern: fetch($URL)
fix: request($URL)
```

</div>

<!--
We reach for Ast-Grep when “text that looks like code” isn’t code, and when *scope* matters.

Here’s a realistic Paraflow-ish example: we want to migrate `fetch(...)` → `request(...)`, but only when it happens inside `useEffect` (so we can centralize retries/logging/tracing in one wrapper).

[click] Regex can find the substring `fetch(`, but it can’t tell:
- comment/string vs real call
- `fetch(...)` vs `client.fetch(...)` (different semantics)
- “only inside useEffect callback” without becoming a fragile mini-parser

[click] With Ast-Grep we encode intent directly:
- `pattern: fetch($URL)` means “a CallExpression with callee `fetch`”
- `inside: useEffect(() => { ... }, ...)` restricts scope
- `fix: request($URL)` makes the change mechanical and reviewable
-->

---
layout: two-cols-header
class: text-left
---

# 2) Safe refactors (transform by syntax)

When we migrate APIs, we want changes that are:

- precise (only the intended call sites)
- consistent (same rewrite everywhere)
- safe (preserve code shape, update imports if needed)

::right::

<div v-click="1" class="text-left">

```ts
// before (old signature)
generateImage("A cat", { ratio: "1:1" })
generateImage(prompt, { ratio })
```

</div>

<div v-click="2" class="text-left">

```ts
// after (new signature)
generateImage({ prompt: "A cat", ratio: "1:1" })
generateImage({ prompt, ratio })
```

</div>

<div v-click="3" class="text-left">

```yaml
rule:
  pattern: generateImage($PROMPT, { ratio: $RATIO })
fix: generateImage({ prompt: $PROMPT, ratio: $RATIO })
```

</div>

<!--
This is the classic “codemod” scenario.

[click] Before: old call shape (but same intent).

[click] After: one consistent new call shape.

Doing this with raw regex is painful:
- it can’t reliably target “the second argument object”
- it can’t safely rebuild the new object literal
- it over-matches when formatting changes

[click] With Ast-Grep, we match the *argument structure* first, then transform.

In practice, this is how we do safe migrations:
- start with a narrow pattern that we’re confident is correct
- add more patterns incrementally (or add constraints) for other call shapes
- keep fixtures for both matches and non-matches so we don’t regress
-->

---
layout: two-cols-header
class: text-left
---

# 3) Project rules (with tests)

Ast-Grep rules scale well because they’re *reviewable code*:

- keep team conventions consistent
- prevent risky patterns from landing
- add fixtures so rules don’t regress

::right::

```yaml
# rules/no-direct-fetch-in-effect.yml (sketch)
id: no-direct-fetch-in-effect
message: Use request() inside effects
rule:
  inside:
    pattern: useEffect(() => { $BODY }, $DEPS)
  pattern: fetch($URL)
fix: request($URL)
```

<!--
Treat rules like any other engineering asset:
- versioned with the repo
- reviewed in PRs
- validated with fixtures (“this should match / this should not match”)

This is a great fit for internal standards and migration guardrails.

Compared to “simple examples”, the real value shows up when:
- you add scope (`inside:`) to prevent over-matching
- you add constraints to capture only intended call sites
- you accumulate fixtures so the rule stays correct over time

This is especially useful for Paraflow because generated code tends to “repeat patterns at scale”:
one rule can prevent thousands of bad outputs, and one migration rule can fix a whole codebase consistently.
-->

---
layout: center
class: text-center
background: https://source.unsplash.com/1600x900/?blueprint,architecture
---

## Deep dive

Advanced patterns from Part 1  
“should match / should not match (and why)”

<!--
Now let’s unpack the 3 advanced patterns from Part 1 as if we were writing real Paraflow rules.

The key idea: we don’t just want “a match”.

We want:
- the *right* match (precise intent)
- clear non-matches (avoid breaking changes)
- a story for reviewers (“why is this safe?”)
-->

---
layout: two-cols-header
class: text-left
---

# Advanced 1) AST + text constraints

Combine structural matching with a small regex guardrail.

```yaml
rule:
  pattern: track($NAME, $DATA)
  constraints:
    NAME:
      regex: '^"pf_'
fix: telemetry.track($NAME, $DATA)
```

::left::

**Should match**

```ts {1-2}
track("pf_signup", { userId })
track("pf_onboarding_done", data)
```

::right::

**Should NOT match**

```ts {1-2}
track("signup", { userId })
telemetry.track("pf_signup", { userId })
```

<!--
[click] Should match: the callee is exactly `track(...)`, and the first arg text starts with `"pf_"`.

Why this combo is powerful:
- AST ensures `$NAME` is the *first argument node* (not some random text)
- the regex constraint is a tiny “business rule” filter for the event namespace

Should NOT match:
- event name doesn’t satisfy the regex constraint (we leave it untouched)
- different callee (`telemetry.track`) so we avoid double-migrating / wrong target
-->

---
layout: two-cols-header
class: text-left
---

# Advanced 2) Named function + specific args shape

Match “the exact call shape” before transforming.

```yaml
rule:
  pattern: generateImage($PROMPT, { ratio: $RATIO })
fix: generateImage({ prompt: $PROMPT, ratio: $RATIO })
```

::left::

**Should match**

```ts {1-2}
generateImage("A cat", { ratio: "1:1" })
generateImage(prompt, { ratio })
```

::right::

**Should NOT match**

```ts {1-2}
generateImage({ prompt: "A cat", ratio: "1:1" })
generateImage("A cat", { size: "1:1" })
```

<!--
[click] Should match: function name + argument structure is exactly what we intend to migrate.

Should NOT match:
- already in the new signature (avoid churn)
- option object doesn’t have `ratio` key, so it’s a different API usage

Note: this pattern is intentionally strict. If we want to support extra keys (e.g. `{ ratio, seed }`),
we’d expand the pattern (or add additional rules) instead of guessing.
-->

---
layout: two-cols-header
class: text-left
---

# Advanced 3) Context match (`inside:`)

Limit a rule to a scope so it doesn’t overreach.

```yaml
rule:
  inside:
    pattern: useEffect(() => { $BODY }, $DEPS)
  pattern: fetch($URL)
fix: request($URL)
```

::left::

**Should match**

```tsx {3}
useEffect(() => {
  // ...
  fetch("/api/user")
}, [])
```

::right::

**Should NOT match**

```tsx {1}
fetch("/api/user")
```

<!--
[click] Should match: `fetch(...)` is inside the `useEffect` callback body.

Should NOT match: fetch outside that scope, because the rule’s intent is “network calls in effects should use request()”.

Note: `inside:` is the “safety belt”. Most bugs in codemods come from matching too broadly.
-->
