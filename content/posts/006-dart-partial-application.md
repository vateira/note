---
title: "Dartでの関数の部分適用"
date: 2021-04-28T09:22:09+09:00
draft: false
tags: ["dart"]
---

Dart は言語として部分適用をサポートしていない。
前回パイプライン演算子的なものを作ったので、それと組み合わせるために部分適用っぽいことができる仕組みを考えてみる。

<!--more-->

```dart
class Partible<T> {
  final int arity;
  T? _value;
  final Function proc;
  Partible(this.arity, this.proc);

  Partible<T> call(List<dynamic> args) {
    if(args.length == arity) {
      _value = Function.apply(proc, args);
      return this;
    } else {
      switch(arity - args.length) {
        case 1:
          return Partible(1, (x) => Function.apply(proc, [...args, x]));
        case 2:
          return Partible(2, (x, y) => Function.apply(proc, [...args, x, y]));
        case 3:
          return Partible(3, (x, y, z) => Function.apply(proc, [...args, x, y, z]));
        default:
          throw 'invalid args';
      }
    }
  }

  T? get value => _value;

}

void main() {
  final times = (x, y, z) => x * y * z;
  final p1 = Partible(3, times);
  final p2 = p1([2, 3]);
  print(p2([4]).value);
}
```

勢いでやってみたので、もっと綺麗にはできると思う。
このやり方の場合、任意の関数のアリティが取得できたり、可変長引数サポートしていたり、メソッドのオーバーロードができたりすればもっとすっきりできそう。
