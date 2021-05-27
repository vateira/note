---
title: "denoでトイ言語"
date: 2021-05-27T16:07:22+09:00
tags: ["TypeScript", "deno"]
---

ハイキングに行く予定が雨だったので、勉強に作った。

https://github.com/vateira/sailang

<!--more-->

LR(1)パーサ。

```
-- test program

let x := 10;

let f := \x y ->
    x + y;

let max := \a b ->
    if a > b then a else b;

let min := \a b ->
    if a < b then a else b;

let show := \a s ->
    if a = 0 then
        print 0
    else
        { print s;
          show (a - 1) s
        };


print f 3 (3 + 5);
print (max 1 3);
print min 4 2;

show 3 1
```

実行結果

```
11
3
2
1
1
1
0
```

色々不具合あるだろうけど、とりあえず最低限の機能はできた。
再帰でループしようとしたけど、800 回ほどでスタックオーバーフロー。
