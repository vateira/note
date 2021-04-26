---
title: "Dartでパイプライン演算子的なのを作る"
date: 2021-04-26T08:00:00+09:00
draft: false
tags: ["dart"]
---

Dart で F#や Elixir みたいなパイプライン演算子ができるかと思って実験。
Dart には部分的用とか無いので、作ってみたものの実用性は無いか。

ちなみに tc52 ではなく tc39 だと stage 1 に [pipeline 演算子が出ているので](https://github.com/tc39/proposal-pipeline-operator)、JavaScript はそのうちできるようになるかも。

<!--more-->

```dart
class Pipeable<T> {
  final T value;
  Pipeable(this.value);

  Pipeable operator >> (Function(T) f)
  {
    return Pipeable(f(value));
  }

}

void main() {
  Pipeable(10)
    >> ((x) => x * x)
    >> ((x) => 'squared value is ${x}')
    >> ((x) => print(x))
    ;
}
```
