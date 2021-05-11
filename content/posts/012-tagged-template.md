---
title: "google/zxとJavaScriptのTagged Templates"
date: 2021-05-11T20:46:33+09:00
tags: ["JavaScript"]
---

社内の雑談で[google/zx](https://github.com/google/zx)が話題になった際に、`` $`find -type f` ``のような構文について質問があったので。

<!--more-->

特殊な構文でプリプロセスしているとかいうのではなく、これは [Tagged Templages](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#tagged_templates) を使っている。

```javascript
function hoge(strs, ...args) {}
```

のような関数を定義して

```javascript
const va = "[a]";
const vb = "[b]";
hoge`a${va}b${vb}`;
```

とすると、関数`hoge`が呼び出され、引数`strs`に`["a", "b"]`が、`args`に`["[a]", "[b]"]`が渡される。

zx の場合は

```javascript
function $(strs, ...args) {
  return new Promise((resolve, reject) => {
    // execなどにより外部コマンドを実行
  });
}
```

のように関数`$`を定義しているので、冒頭にあったような

```javascript
let branch = await $`git branch --show-current`;
```

のような書き方ができる。

文字列リテラルに対して何らかの関数を適用できると言う点では、[Elixir の sigil](https://elixir-lang.org/getting-started/sigils.html#custom-sigils)に似ていると思う。
