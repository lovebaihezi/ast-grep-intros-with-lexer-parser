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
Hello everyone, I'm BoWen.

For today's share, I'm goona talk about the Ast Grep, which, we start with three parts, including what is it, how we use it, when we use it, and how we understand its inner works(from lexer and parser to AST).

For the third, we won't talk much, but providing a view of the lex and parse and some compiler principles.

大家好，我是博文。

今天想和大家分享 Ast Grep。我们会从三个部分开始：它是什么、我们怎么用、以及在什么场景下需要用到它；最后也会简单聊一下它的内部原理（从 lexer / parser 到 AST），并补充一些编译器的基本思路。

第三部分不会展开太多，主要是提供一个视角，帮助大家理解 lexer / parser 以及一些编译器原则。
-->

---
layout: center
class: text-center
---

## Agenda

<div class="max-w-3xl mx-auto text-left">
  <ol class="space-y-3 text-lg">
    <li><b>How I use Ast-Grep</b> (what it is, Paraflow, my workflow)</li>
    <li><b>In Which We Needs it</b> (where text tools break)</li>
    <li><b>How Lexer/Parser empower it</b> (text → tree → rewrite)</li>
  </ol>
</div>

<!--
Tip: Presenter Mode shows these notes.
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
