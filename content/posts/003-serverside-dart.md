---
title: "サーバサイドDartのアイデア"
date: 2021-04-22T18:18:03+09:00
draft: false
tags: ["設計", "Dart", "サーバサイド"]
---

モバイルアプリ開発において、開発リソースが潤沢でないのであれば Flutter の導入は非常に強力な選択肢となる。じっさい、ここ三年ほどスタートアップにてメインのプロダクトを Flutter で開発していて個人的にはとても気に入っている(Dart への不満はゼロでは無いけれども)。

<!--more-->

そして、Flutter において個人的に状態管理の肝となっているのは freezed パッケージの存在だ。
Kotlin の data class のようなものが言語として提供されていないが、昨今の GUI アプリケーションで immutable なデータ構造無しに状態管理をするのは非常に辛いものがある。当初はこういったもの無しでもなんとかやれていたが、アプリケーションの仕様が複雑になるにつれて、こいつの存在は必須になってきた。

この freezed 自体は Dart のパッケージとして提供されているので、Flutter 以外でも使うことができる。もしモバイルアプリがサーバと通信するのであれば、サーバサイドも Dart で実装して freezed で作成したエンティティなどを共有すれば、非常に効率よくコードを共有できるのでは無いだろうか。また、抽象化されたアプリケーションロジックを共有することができれば、テストなども効率化できそうな気がする。

ということで、Dart の web フレームワークを調べたところ、[aqueduct](https://github.com/stablekernel/aqueduct)が一番メジャーっぽかったのだけれども、これが丁度先月あたりにプロジェクトが凍結されてしまっていた。他の web フレームワークはメンテナンスやユーザー数からしてもプロダクトに採用するには少し心許ない。

などと漁っていたが、どうやら Dart 公式のミドルウェアフレームワークである[shelf](https://github.com/dart-lang/shelf)が非常に使いやすそうである。[shelf_router](https://pub.dev/packages/shelf_router)と組み合わせれば、簡単にアプリケーションサーバーを実装できる。

```dart
import 'package:shelf_router/shelf_router.dart';
import 'package:shelf/shelf.dart';
import 'package:shelf/shelf_io.dart' as io;

var app = Router();

app.get('/hello', (Request request) {
  return Response.ok('hello-world');
});

app.get('/user/<user>', (Request request, String user) {
  return Response.ok('hello $user');
});

var server = await io.serve(app.handler, 'localhost', 8080);
```

試してみたところ、deno の oak や php の slim のような感じだ。モバイルアプリで RESTful api をリクエストするような用途であれば十分だろう。
それとこの Shelf、ミドルウェアを自作するのも容易なので、

```dart
import 'package:shelf/shelf.dart';

Middleware auth() => (innerHandler) {
      return (request) async {
        if(認証失敗) {
            return Response(401, body: 'FORBIDDEN');
        }
        final response = await innerHandler(request);
        return response;
      };
    };

final handler =
      const Pipeline().addMiddleware(auth()).addHandler(router());
```

のように色々追加できる。
あとはこんな感じでプロダクトを分割すれば良さそう。

| package | 概要                                                                                         |
| ------- | -------------------------------------------------------------------------------------------- |
| core    | entity, application logic やリポジトリなどのインタフェース                                   |
| client  | Flutter アプリ。アプリケーションロジックは core を使う。またリポジトリに server を指定する   |
| server  | アプリケーションサーバー。アプリケーションロジックは core を使う。リポジトリは DB などを使う |
