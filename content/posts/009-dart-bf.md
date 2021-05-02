---
title: "DartでBrainf*ckインタプリタ"
date: 2021-05-02T20:56:04+09:00
tags: ["dart"]
---

特にゴルフとか捻りとかもなく愚直に書いた Brainf\*ck 処理系。
コードとメモリは同じクラス(\_Tape)のインスタンスにしている。

<!--more-->

```dart
class Bf {
  final _Tape<String> _code;
  Bf(String code) : _code = _Tape(code.split(''));

  void eval() {
    var memory = _Tape(List.filled(1024, 0));
    var out = '';
    do {
      switch (_code.cur) {
        case '>':
          memory.forward();
          break;
        case '<':
          memory.backward();
          break;
        case '+':
          memory.inc();
          break;
        case '-':
          memory.dec();
          break;
        case '.':
          out += String.fromCharCode(memory.cur);
          break;
        case ']':
          if (memory.cur > 0) {
            var depth = 0;
            do {
              switch (_code.cur) {
                case ']':
                  depth += 1;
                  break;
                case '[':
                  depth -= 1;
                  break;
              }
              _code.backward();
            } while (depth != 0);
          }
      }
      _code.forward();
    } while (_code.forwardable);
    print(out);
  }
}

class _Tape<T> {
  final List<T> data;
  int _index;

  _Tape(this.data) : _index = 0;

  T get cur => data[_index];

  void forward() {
    _index += 1;
  }

  void backward() {
    _index -= 1;
  }

  bool get forwardable => _index < data.length;
}

extension _Memory on _Tape<int> {
  void inc() {
    data[_index]++;
  }

  void dec() {
    data[_index]--;
  }
}
```

呼び出しがわ

```dart

void main(List<String> arguments) {
  final code = '''
    >+++++++++[<++++++++>-]<.>+++++++[<++++>-]
    <+.+++++++..+++.[-]>++++++++[<++++>-]<.>++
    +++++++++[<+++++>-]<.>++++++++[<+++>-]<.++
    +.------.--------.[-]>++++++++[<++++>-]<+.[
    -]++++++++++.
  ''';
  final bf = Bf(code);
  bf.eval();
}
```

実行

```sh
% dart run
Hello World!
```
