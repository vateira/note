---
title: "Zigで「はりぼて言語」をつくる "
date: 2021-04-30T10:00:00+09:00
tags: ["zig"]
---

Twitter で[はりぼて言語](http://essen.osask.jp/?a21_txt01)を見かけたので、Zig で実装してみる。

<!--more-->

ほぼそのままだけど、Zig でのコマンドライン引数の扱いをまだ把握していないのでファイル名は固定。

```zig
const std = @import("std");
const print = std.debug.print;
const fs = std.fs;

const LangError = error{
    InvalidSyntax,
};

pub fn main() !void {
    const file = try fs.cwd().openFile("test.hl", .{ .read = true });
    defer file.close();

    var code: [1024]u8 = undefined;
    const size = try file.readAll(code[0..]);

    var vars: [256]u8 = undefined;

    var i: u8 = 0;
    while (i < 10) : (i += 1) {
        vars['0' + i] = i;
    }

    var pc: u8 = 0;
    while (pc < size) : (pc += 1) {
        if (code[pc] == '\n' or code[pc] == '\r' or code[pc] == '\t' or code[pc] == ' ' or code[pc] == ';') continue;
        if (code[pc + 1] == '=' and code[pc + 3] == ';') {
            vars[code[pc]] = vars[code[pc + 2]];
        } else if (code[pc + 1] == '=' and code[pc + 3] == '+' and code[pc + 5] == ';') {
            vars[code[pc]] = vars[code[pc + 2]] + vars[code[pc + 4]];
        } else if (code[pc + 1] == '=' and code[pc + 3] == '-' and code[pc + 5] == ';') {
            vars[code[pc]] = vars[code[pc + 2]] - vars[code[pc + 4]];
        } else if (code[pc] == 'p' and code[pc + 1] == 'r' and code[pc + 5] == ' ' and code[pc + 7] == ';') {
            print("{d}\n", .{vars[code[pc + 6]]});
        } else {
            print("syntax error: {c}\n", .{code[pc]});
            return LangError.InvalidSyntax;
        }
        while (code[pc] != ';') pc += 1;
    }
}
```

こんな感じでスクリプトを用意。

```
a=1;
b=2;
c=a+b;
print c;
```

そして実行

```zsh
% haribote % zig run ./main.zig
3
```
