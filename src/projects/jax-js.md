---
title: jax-js
date: 2025-12-18
repo: ekzhang/jax-js
topics: ["TypeScript", "Machine Learning"]
lead: Fast, flexible GPU machine learning framework that runs in-browser.
image: jax-js.png
---

This is an ML framework in the browser; you can think of it as a loose
reimplementation of [JAX](https://jax.dev) and [XLA](https://openxla.org/) in
pure JavaScript. It takes array operations and compiles them into fast WebGPU
and WebAssembly kernels.

I built this as a side project over the course of 2025. The goal was to create a
_truly portable_ and _interactive_ machine learning research library on the web,
as well as enabling numerical computing in general. Currently it supports most
of the JAX and NumPy APIs.

**Links: [Website](https://jax-js.com),
[GitHub](https://github.com/ekzhang/jax-js),
[Announcement](https://ekzhang.substack.com/p/jax-js-an-ml-library-for-the-web)**
