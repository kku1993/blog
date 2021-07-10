---
layout: post
title:  "Classic Web Worker in Vite JS Dev Environment"
date:   2021-07-10
description: How to use classic web worker in Vite JS dev environment.
categories: programming, js
---

By default, when running in dev mode, [Vite](https://vitejs.dev/) loads Web
Worker script in `module` mode [1]. While this is compiled away in production
build, it presents a problem if you're developing with legacy web worker scripts
that must run `classic` mode.

To work around this problem, you can import the web worker script as a string
and run it using a data URL.

```javascript
import workerString from './worker.js?raw';
const workerURL = 'data:text/javascript;base64,' + btoa(workerString);
const worker = new Worker(workerURL, {type: 'classic'});
```

## Notes

- [1] See [this article](https://web.dev/module-workers/) for some background on
`module` vs `classic` web workers.
