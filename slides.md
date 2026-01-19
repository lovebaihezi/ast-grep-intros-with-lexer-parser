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
---

# Ast-Grep in Practice

Structural search & rewrite, from “why” to “how”

<div class="mt-10 opacity-90 leading-7">
  <div class="text-lg">Chai. BoWen (Paraflow)</div>
  <div class="text-sm opacity-80">January 20, 2026</div>
</div>

<div class="mt-8 opacity-80">
  Paraflow · AST · Lexer/Parser
</div>

<!--
Hi everyone, I'm BoWen.

As a code-focused AI agent team, we need to query and transform code by structure. Like how the DOM API lets us traverse and manipulate the HTML DOM tree, Ast-Grep lets us query and rewrite information on the code’s Abstract Syntax Tree (AST).

Today we'll cover practical usage, a few rules of thumb, and how to generate your own AST and match against it.
-->

---
layout: center
class: text-center
---

## Agenda

<div class="max-w-3xl mx-auto text-left">
  <ol class="space-y-3 text-lg">
    <li><b>How we use Ast-Grep</b> (what it is, in Paraflow, our workflow)</li>
    <li><b>Where we need it</b> (where text tools break)</li>
    <li><b>How lexer/parser powers it</b> (text → tree → rewrite)</li>
  </ol>
</div>

<!--
Ast-Grep has been useful for us in Paraflow—and previously in Motiff AI.

Today I'll share Ast-Grep in three parts: what it is, how we use it, and where we need it.

Then we'll briefly look under the hood—from lexer/parser to AST—to understand why the matching works.

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
background: https://source.unsplash.com/1600x900/?question,abstract
---

# Q&A

What would you automate next?

<div class="mt-10 opacity-80 text-sm">
  (rules · migrations · CI · new languages)
</div>
