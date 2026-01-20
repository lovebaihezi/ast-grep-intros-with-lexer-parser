---
theme: seriph
background: https://cover.sli.dev
title: Ast-Grep in Practice
info: |
  Ast-Grep × Paraflow × Compiler Frontend
layout: cover
class: text-left
drawings:
  persist: false
transition: slide-left
mdc: true
duration: 25min
addons:
  - slidev-addon-typst
  - fancy-arrow
---

# Ast-Grep in Practice

Structural search & rewrite, from “why” to “how”

<div class="mt-10 opacity-90 leading-7">
  <div class="text-lg">Chai. BoWen (Paraflow)</div>
  <div class="text-sm opacity-80">January 20, 2026</div>
</div>

<div class="mt-8 opacity-80">
  Paraflow · Ast-Grep · DFA (Lexer/Parser)
</div>

<!--
Hi everyone, I'm BoWen.

As a code-focused AI agent team, we need to query and transform code by structure. Like how the DOM API lets us traverse and manipulate the HTML DOM tree, Ast-Grep lets us query and rewrite information on the code’s Abstract Syntax Tree (AST).

Today we'll cover practical usage, a few rules of thumb, and a DFA mental model behind lexer/parser.

We won’t go deep into building the AST itself today (time is limited) — just enough to understand why structural matching works.
-->

---
layout: center
class: text-center
---

## Agenda

<div class="max-w-3xl mx-auto text-left">
  <ol class="space-y-3 text-lg">
    <li><b>Part 1</b> — How we use Ast-Grep</li>
    <li><b>Part 2</b> — Where we need it</li>
    <li><b>Part 3</b> — DFA helps build the Lexer + Parser</li>
  </ol>
</div>

<!--
Today I'll share the Ast Grep, which, like editing the HTML using bs4, we edit the code using Ast-Grep.

Consider in the past, we needs bs4 in python for query or fix problems from HTML, like check and update image by search through in the trees.

So, for the Paraflow we needs to work on the TypeScript with React, we will needs tools to work with synctax tree, and, ast-grep can help us.

We'll talks about in three parts: How we use it, how to use it, and the key of impl one of it.

Given the limited time—and to be honest, I don’t have all the formal regex / automata theory at my fingertips—we’ll keep this part to a small subset: just a high-level mental model, up to the DFA abstraction.
-->

---
src: ./pages/01-how-i-use-ast-grep.md
---

---
src: ./pages/02-why-situations-need-ast-grep.md
---

---
src: ./pages/03-lexer-parser-empower-ast-grep.md
---

---
layout: center
class: text-center
---

## Not talked

<div class="max-w-3xl mx-auto text-left">
  <ol class="space-y-3 text-lg">
    <li>Different types of lexer/parser (LL(k), LL(0)/LL(1))</li>
    <li>The DFA we used is one example (not general, nothing special)</li>
    <li>We haven't talked about grammar/syntax here (it's complex). In Q&A, we can only take one question.</li>
    <li>etc.</li>
  </ol>
</div>

<!--
Optional clarifications:

- "LL(k)" means k-token lookahead. There are also LR families, etc.
- The DFA shown is just a teaching example to build the mental model (not a complete theory of lexing/parsing).
-->

---
layout: center
class: text-center
background: https://source.unsplash.com/1600x900/?question,abstract
---

# Q&A

<!--
Prompt ideas:

- What would you automate next?
- rules · migrations · CI · new languages
-->
