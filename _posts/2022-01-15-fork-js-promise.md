---
layout: post
title:  '"Forking" Promise in Javascript'
date:   2022-01-15
categories: programming, javascript, js
---

Avoid duplicate backend fetches or expensive operations by "forking" a promise.

On a recent project, I needed to fetch some large resource from the backend and
use it to render multiple views. Each view transforms the resource
independently, and some views take longer to transform the resource than others.

My goals are:

- Only fetch resource from the backend once.
- Each view should render as soon as data is available, without waiting for
  other views.

The solution, it turns out, is as simple as making copies of `Promise` in
javascript. Each copy of the `Promise` object is an independent view of the data
returned by the original `Promise`.

Here's a demo in node.js:

```javascript
async function f1(p) {
  // f1 can process the data returned by the promise independently of f2,
  // meaning it can do useful work even when f2 is slow.
  const result = await p.then((v) => v + 'from f1');
  console.log("f1 done");
  return result;
}

async function f2(p) {
  // f2 takes a long time to process the data.
  const result = await p.then(async (v) => {
    await new Promise(r => setTimeout(r, 2000));
    return v + 'from f2';
  });
  console.log("f2 done");
  return result;
}

async function main() {
  const p = new Promise((resolve, reject) => {
    console.log("expensive operation only runs once");
    resolve('');
  });

  console.log(await Promise.all([f1(p), f2(p)]));
};

main();
```

Running the script above yields:

```shell
$ node fork_promise.js 
expensive operation only runs once
f1 done
f2 done
[ 'from f1', 'from f2' ]
```
