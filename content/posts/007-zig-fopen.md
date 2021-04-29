---
title: "zigでファイルを開く方法のメモ"
date: 2021-04-29T10:20:00+09:00
draft: false
tags: ["zig"]
---

zig の std は変更が激しいので、公式以外のネット上の情報があまり参考にならない。
とりあえず、0.7.1 でのファイルを開く方法。

<!--more-->

```zig
const std = @import("std");
const print = std.debug.print;
const fs = std.fs;

pub fn main() !void {
    const file = try fs.cwd().openFile("test.txt", .{ .read = true });
    defer file.close();

    var buffer: [1024]u8 = undefined;
    const size = try file.readAll(buffer[0..]);

    print("{s}\n", .{buffer[0..size]});
}
```

ファイル関係は `std.fs` にまとめられている。
ファイルを開く際は、まず Dir 構造体を生成して、`openFile`関数を呼び出すと良い。

バッファについてはコンパイル時にはサイズが分からないので、1024 で確保している。
