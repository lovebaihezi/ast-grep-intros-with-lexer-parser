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

Text search is blind to code structure. AST matching lets us target *only real syntax*:

- avoid false positives in strings/comments
- respect scope (only inside JSX / only exports / only calls)
- work across formatting styles

::right::

<div v-show="$clicks === 0" class="text-left">

```ts
// console.log("debug") in comment
const s = "console.log('not a call')"
console.log(user.id)
```

</div>

<div v-click="1" v-show="$clicks === 1" class="text-left">

```ts {1-2}
// console.log("debug") in comment
const s = "console.log('not a call')"
console.log(user.id)
```

```js
/console\.log\((.*?)\)/g
```

</div>

<div v-click="2" v-show="$clicks >= 2" class="text-left">

```ts {3}
// console.log("debug") in comment
const s = "console.log('not a call')"
console.log(user.id)
```

```bash
ast-grep --pattern 'console.log($$$ARGS)' --lang typescript
```

</div>

<!--
We reach for Ast-Grep when “text that looks like code” isn’t code.

For example, matching `console.log(...)` calls:

[click] A regex will happily match comments and strings.

[click] AST matching only hits the real CallExpression.
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

<div v-click="1">

```ts
await promiseTask(XX, () => {
  // ...
})

await promiseTask(
  XX,
  () => {
    // ...
  },
  XXX,
)
```

</div>

<div v-click="2">

```ts
globalSteakTasksScheduler.append((ctx, collector) => {
  return promiseTask(ctx, XX, () => {
    // ...
  }).catch(collector)
})
```

</div>

<!--
This is the classic “codemod” scenario.

[click] Before: multiple call shapes (single-line vs multi-line, optional extra args).

[click] After: wrap it into the scheduler closure, pass `ctx`, and attach `.catch(collector)`.

Doing this with raw regex is painful:
- multiline + nested braces make captures brittle
- the optional `XXX` argument changes the shape
- it’s hard to reliably match “the function argument” vs “some text that looks like it”
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
# rules/no-direct-fetch.yml (sketch)
id: no-direct-fetch
message: Use request() wrapper
rule:
  pattern: fetch($URL)
fix: request($URL)
```

<!--
Treat rules like any other engineering asset:
- versioned with the repo
- reviewed in PRs
- validated with fixtures (“this should match / this should not match”)

This is a great fit for internal standards and migration guardrails.
-->
