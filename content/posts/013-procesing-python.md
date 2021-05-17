---
title: "ProcesingのPythonモードでデコレータを使う"
date: 2021-05-17T22:22:29+09:00
tags: ["processing", "python"]
---

久々に Processing でアニメーションとかを考えてみようと思ったところ、python モードがあることを思い出したの使ってみる。

<!--more-->

Python モードを試したのは、学生の頃に授業で触った時は Java で書いていたけれども、ここ数年 Java から遠ざかっていたのが大きい。それと、複数のアニメーションを一定の時間毎に切り替えるのにデコレータが使えるかなと思ったから。

(このあたり Processing で正式な方法があるのかもしれないけど)

```python
kFrameRate = 60

def ms():
  return frameCount * 1000 / kFrameRate

def animation():
    def _animation(f):
        def _wrapper(*args, **keywords):
            if beginAt <= ms() and ms() <= endAt:
                return f(*args, **keywords)
            else:
                pass
        return _wrapper
    return _animation
```

のようなデコレータを用意しておけば、

```python
def draw():
    animation_01()


@animation(1000, 2000)
def animation_01():
    text("hello!", 0, 0)

```

のようにして指定できる。
