---
layout: post
title: System State and Declarative Languages
tags:
- sql
- devops
status: published
type: post
published: true
listed: true
---

Declartive programming languages are often characterized by how they express computation in terms of a desired outcome. This is in contrast to imperative languages which are often seen, vis-a-vis declarative languages, as expressing computation with series of steps to reach an outcome. The advantages of the declarative approach are clear, the program author states his or her intentions and the rest is left to the runtime [1]. In practice many of the languages that have earned the declarative label require much of the *how* to be defined out of necessity.

In this post we'll look at two examples, SQL and configuration management, where the declarative approach has proven valuable. In particular we'll examine where each requires some intervention on the part of the programmer to assist the runtime in achieving the desired result and how that might be remedied.

## Semantic Dual


### footnotes

1. "It seems like a sufficiently advanced runtime...", famous last words from an anonymous PhD dropout.
