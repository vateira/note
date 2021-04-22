---
title: "SMTソルバ(z3)で魔方陣を解く"
date: 2021-04-23T07:31:39+09:00
tags: ["python", "formal methods"]
---

minisat で魔方陣を解こうかと思ったけれども、CNF で和の制約を書くのが大変そうだったので SMT ソルバを使って解く。

<!--more-->

```sh
% pip install z3-solver
```

あとはスクリプトを書くだけ。

```python:magic-square.py
from z3 import *

solver = Solver()

# 5 * 5 の魔方陣を扱う
size = 5

# 各行,列,対角線の和
summary = ((size ** 2 + 1) * size) / 2

# 0〜n^2の変数を用意する
cells = [Int("c[%d]" % x) for x in range(size ** 2)]

# 各セルは1〜n^2の数字が入る
for x in range(size ** 2):
    solver.add(1 <= cells[x], cells[x] <= size ** 2)

# それぞれの数字は一度のみ現れる
solver.add(Distinct(cells))

# 各行の和はsummaryになる
for i in range(size):
    rows = [cells[i * size + x] for x in range(size)]
    solver.add(sum(rows) == summary)

# 各列の和はsummaryになる
for i in range(size):
    cols = [cells[x * size + i] for x in range(size)]
    solver.add(sum(cols) == summary)

# 対角線1の和はsummaryになる
dia1 = [cells[x * size + x] for x in range(size)]
solver.add(sum(dia1) == summary)

# 対角線2の和はsummaryになる
dia2 = [cells[(x + 1) * (size - 1)] for x in range(size)]
solver.add(sum(dia2) == summary)

# 予めセルに何か指定しておく
solver.add(cells[0] == 10)
solver.add(cells[12] == 13)


# 解を求める
solver.check()

# 表示
model = solver.model()
for i in range(size):
    for j in range(size):
        cell = cells[i * size + j]
        print("%d\t" % model[cell].as_long(), end="")
    print("")
```

```sh
% python3 ./magic_square.py
10      22      17      1       15
6       16      7       11      25
24      2       13      18      8
4       5       19      23      14
21      20      9       12      3
```

今回は 5 \* 5 の魔方陣用に作成したけれども、size を変更して具体的な値の制約を削除または変更すればそれ以外の大きさの魔方陣にも対応できる。

```diff
- # 5 * 5 の魔方陣を扱う
+ # 3 * 3 の魔方陣を扱う
- size = 5
+ size = 3

- # 予めセルに何か指定しておく
- solver.add(cells[0] == 10)
- solver.add(cells[12] == 13)
```

```sh
% python3 ./magic_square.py
2       7       6
9       5       1
4       3       8
```

SAT ソルバで CNF 形式で行や列ごとの和の制約を表現するのはめちゃくちゃ大変そうだったけれども、SMT ソルバでこういうふうにできるのであれば結構簡単だ。特に python 向けの z3 は通常の算術演算子が使えて、`sum`なども利用できるので直感的に記述できるのが便利。
